---
title: 如何在 TypeScript 中实现类的多继承
date: 2024-12-06 11:53:55
tags:
  - typescript
  - javascript
categories: TypeScript
---

首先，在 js 中还没有真正的多继承。但是在实际工作中经常需要抽离通用模块并按需组成新的业务模块，这就对类的多继承有了实际需求。

举个例子，现在我们有个基础类 `Animal` ：

```ts
class Animal {
  constructor(name?: string) {
    if (name) this.myName = name;
  }
  myName: string = "animal";
}
```

另外有两个 `Animal` 的子类，分别是会跑的 `Horse` 和会飞的 `Bird` ：

```ts
class Horse extends Animal {
  constructor(...arg: any[]) {
    super(...arg);
    // ...do something
  }
  run() {
    console.log("I can run");
  }
}

class Bird extends Animal {
  @Decorator() // 带有装饰器
  fly() {
    console.log("I can fly");
  }
  static wings = 2;
}
function Decorator(): MethodDecorator {
  return function (target, propKey, descriptor) {
    console.log(`${target.constructor.name} ${String(propKey)}`);
  };
}
```

现在我们需要一个同时继承 `Horse` 和 `Bird` 特性的既会跑又会飞的 `Unicorn`，重新写新的类既不高效也不优雅。下面我们来讨论如何实现多继承。

为了便于后续说明，先定义几个类型声明：

```ts
type Constructor<T = unknown> = new (...args: any[]) => T;
type ConstructorExtFn<T extends Constructor> = (Cls: T) => T;

type UnionToIntersection<T> = UnionToFunction<T> extends (arg: infer P) => any ? P : never;
type UnionToFunction<T> = T extends any ? (arg: T) => any : never;
```

### 传统 mixin

这种方式原理类似于 `Object.assign`，将多个类的原型方法、属性、静态属性等拷贝到目标类上，以下是简单实现：

```ts
function Mixin<T extends Constructor<{}>[]>(
  ...mixins: T
): Constructor<UnionToIntersection<InstanceType<T[number]>>> & UnionToIntersection<T[number]> {
  class MixinBase {}
  function copyProperties(target: any, source: any) {
    for (const key of Reflect.ownKeys(source)) {
      const skipProps = ["constructor", "prototype", "name"];
      if (!skipProps.includes(String(key))) {
        const desc = Object.getOwnPropertyDescriptor(source, key);
        if (desc) Object.defineProperty(target, key, desc);
      }
    }
  }
  for (const mixin of mixins) {
    copyProperties(MixinBase, mixin);
    copyProperties(MixinBase.prototype, mixin.prototype);
  }
  return MixinBase as any;
}
```

```ts
Animal.prototype.myName = "animal"; // 需在原型链设置默认值

class Unicorn extends Mixin(Animal, Horse, Bird) {
  constructor() {
    super("unicorn");
  }
}

console.log(Unicorn.wings); // 2
const unicorn = new Unicorn();
console.log(unicorn.myName); // animal
unicorn.run(); // I can run
unicorn.fly(); // I can fly
unicorn.speak(); // error
```

优点：

- 返回的类的原型链干净，便于追溯

缺点：

- 需要额外处理构造器、静态属性、同名方法，完备实现较为复杂
- 需要声明返回类型
- 属性默认值需额外定义在原型链上
- 不支持 super
- 无法继承父类中的装饰器

### 使用子类工厂函数

下面介绍的方法将使用子类工厂函数来实现多继承。该方法接受一个基类作为参数，返回继承这个基类的子类，具体逻辑在该子类中实现。下面将重写上述需求：

```ts
function mixinHorse<T extends Constructor<Animal>>(Cls: T) {
  class Horse extends Cls {
    constructor(...arg: any[]) {
      super(...arg);
      // ...do something
    }
    run() {
      console.log("I can run");
    }
  }
  return Horse;
}
const Horse = mixinHorse(Animal);

function mixinBird<T extends Constructor<Animal>>(Cls: T) {
  class Bird extends Cls {
    @Decorator() // 带有装饰器
    fly() {
      console.log("I can fly");
    }
    static wings = 2;
  }
  return Bird;
}
const Bird = mixinBird(Animal);

class Unicorn extends mixinBird(mixinHorse(Animal)) {
  constructor() {
    super("unicorn");
  }
}

console.log(Unicorn.wings); // 2
const unicorn = new Unicorn();
console.log(unicorn.myName); // unicorn
unicorn.run(); // I can run
unicorn.fly(); // I can fly
unicorn.speak(); // error
```

可以看到，只需改变写法而无需额外代码就能完备实现多继承功能。 `super` 和装饰器功能也正常工作

优点：

- 实现简单，执行结果符合预期
- 自带类型推导
- 方便重写同名方法
- super 和装饰器正常工作

缺点：

- 原型链较长，继承过多会导致原型链过于复杂
- 需改写子类工厂函数，不便于直接使用中间类
- 嵌套写法导致继承过多时不便阅读，类似回调地狱

#### 优化写法

我们先来看下继承过多的情况：

```ts
class NewAnimal extends Mixin5(Mixin4(Mixin3(mixinBird(mixinHorse(Animal))))) {
  // ...do something
}
```

这里的多层嵌套像极了 js 中的回调地狱，既不方便阅读，也不方便增减。对于这个问题，我们可以优化写法使使用更加方便。代码如下：

```ts
export function multiExtends(
  extendsBase: Constructor<any>,
  extendsFunctions: ConstructorExtFn<Constructor<any>>[]
) {
  let func: ConstructorExtFn<Constructor<any>>;
  let ans: Constructor<any> = extendsBase;
  while (extendsFunctions.length) {
    func = extendsFunctions.shift()!;
    ans = func(ans);
  }
  return ans;
}
```

上面 `multiExtends` 函数接受一个基类 `extendsBase` 和一个子类工厂函数数组 `extendsFunctions`，返回最终继承结果。但是这样就丢失了 ts 的自动类型推导，需要自己修改返回类型：

```ts
export function multiExtends<
  B extends Constructor<any>,
  E extends ConstructorExtFn<Constructor<any>>
>(
  extendsBase: B,
  extendsFunctions: E[]
): B & UnionToIntersection<ReturnType<E>> {
  let func: ConstructorExtFn<Constructor<any>>;
  let ans: Constructor<any> = extendsBase;
  while (extendsFunctions.length) {
    func = extendsFunctions.shift()!;
    ans = func(ans);
  }
  return ans as any;
}
```

> 返回类型重点是将联合类型`E`转为交叉类型

```ts
class Unicorn extends multiExtends(Animal, [
  mixinHorse,
  mixinBird,
  Mixin3,
  Mixin4,
  Mixin5,
]) {
  constructor() {
    super("unicorn");
  }
  // ...do something
}

console.log(Unicorn.wings); // 2
const unicorn = new Unicorn();
console.log(unicorn.myName); // unicorn
unicorn.run(); // I can run
unicorn.fly(); // I can fly
unicorn.speak(); // error
```

可以看到写法简介优雅了不少，且功能完备。

上述功能笔者已整理并发布了 npm 包，以便使用：

```sh
npm i multi-extends
```

```ts
import { multiExtends } from "multi-extends";

// ...
```

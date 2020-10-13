# TypeScript 学习笔记

写在前头,该文是看半途而废的 ts 官网学习、掘金上的文章以及项目实践总结而来。

## 1. TypeScript 类型

### 常用类型

一些常用的就不赘述了,包括以下

- Boolean 类型
- Number 类型
- String 类型
- Array 类型
- Any 类型
- Void 类型
- Null 和 Undefined 类型
- objct 类型和 Object 类型和{}类型

对象类型又有**objct 类型和 Object 类型和{}类型**,对象类型非常常用,那这三种有什么区别呢?

#### `objct`类型是指非原始类型

包括七大原始类型,这里经过百度搜索,范围锁定为 1 个月内,还看到很多写五个原始类型的...

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4205b815bd20414e8a418dd483a444d6~tplv-k3u1fbpfcp-watermark.webp)

> 在上图中,橙色波浪线就是不符合 TS 检验的语句

#### `Object` 类型是所有 `Object` 类的实例的类型

`Object` 类型有两个接口

1. `Object` 接口定义了 `Object.prototype` 原型对象上的属性
2. `ObjectConstructor` 接口定义了 `Object` 类的属性

那么问题出现了,原型对象上的属性和类上的属性有什么区别呢?

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/180842e4c5e84bbeae633b7b49a7156b~tplv-k3u1fbpfcp-watermark.webp)

这里就不讲了，跑太偏

#### `{}` 类型

按我的理解,这个类型指的就是 `Object` 类的实例,你想表示里面有什么属性就往上面写,由于`{}` 类型是 `Object` 类的实例,所以在使用 `Object.prototype` 原型对象上的属性的时候,ts 也不会提示错误,比如:

```typescript
// people有两个属性,分别是sex和name
let people: { sex: string; name: string };
// 属性正确,ts不会报错
people = { sex: "man", name: "吴亦凡" };
// 属性错误,ts会报错
people = { a: "1" };
// 使用Obect类接口上的属性,ts不会报错
people.toString();
```

### 不常用类型

- Symbol 类型

原因很简单,因为 `Symbol` 也很少用

- Unknown 类型

我们知道 `any` 可以表示任何类型,但是这样会削弱 `ts` 规范类型,避免错误的作用。为了解决 `any` 带来的问题，TypeScript 3.0 引入了 `unknown` 类型来表示不知道是什么类型的意思。`unknown` 类型有以下特性

1. `unknown` 类型可以使用任何类型来赋值（既然变量不知道是什么类型，我就可以为所欲为了）

```typescript
let value: unknown;
value = 1;
value = "a";
value = { a: 1 };
```

2. 任何不是 `any` 或者 `unkown` 类型的值不可以使用 `unkown` 类型来赋值(我是一个有身份的变量，不可以这么随便)

```typescript
let value: unknown;
value = 1;
// ERROR: 不能将类型“unknown”分配给类型“number”。
const count: number = value;
```

个人觉得使用场景在某个变量可能是存在多个类型的，unkown 类型可以帮助我们标示出变量可能有多个类型，当我们要对特定变量处理的时候，可以使用类型守卫(具体在下文解释)让满足类型变量的条件通过。

```typescript
let value: unknown;
value = 1;
if (typeof value === "number") {
  value += 1;
} else if (typeof value === "string") {
  value = parseInt(value, 10) + 1;
}
```

- Enum 类型

枚举类型有各种各样枚举方式，数字/字符串/常量/异构，这些枚举方式我个人觉得不需要怎么记，使用场景就是有时候我们需要一些数字或者字符串常量，但是直接看常量并不知道它代表的意思，这时候我们可以使用枚举来对常量进行类型命名

```typescript
// 枚举顺序从0开始，这里常量0代表待发布
enum Status {
  UNPUBLISHED,
  PUBLISHED,
}
// 相当于type Status = 0 ｜ 1;但使用枚举可以定义每个值的含义
// Status的属性可以表示从0到1的常量
let status: Status = Status.PUBLISHED;
```

- Tuple 类型

这个类型叫做元组，跟数组有关的，数组类型定义的是一组具有相同类型的数组，比如`number[]`代表一组数字组成的数组。那元组跟数组有什么区别呢？

1. 元组是确定数量的数组
2. 数组忠的每个属性都有自己对应的类型

根据以上区别举个例子

```typescript
let tupleType: [string, boolean];
tupleType = ["semlinker", true];
```

- Never 类型

Never 类型代表永远不会存在的值的类型，比如没有返回值的函数。在实际应用中，我发现这个类型也可以用来排查错误。当我们对数据进行一系列操作时，TypeScript 提示某个变量时 never 类型，但这个类型不应该是 never 类型时，可以知道在代码中我们可能进行了错误的操作。

Never 类型还有一种我没有用过的~~高级~~用法，用于检查联合类型。下面查看一下代码。

```typescript
type Foo = string | number;

function controlFlowAnalysisWithNever(foo: Foo) {
  if (typeof foo === "string") {
    // 这里 foo 被收窄为 string 类型
  } else if (typeof foo === "number") {
    // 这里 foo 被收窄为 number 类型
  } else {
    // foo 在这里是 never
    const check: never = foo;
  }
}
```

这里 Foo 是一个联合类型（联合类型就是可以是几种类型中的一种），以上代码编译后不会出错，但是一旦这个 Foo 被他人修改

```typescript
type Foo = string | number | boolean;
```

那么条件判断里最后的 never 类型赋值就会被 TypeScript 报红提示错误。通过 TypeScript 类型对比有助于减少 bug 数量的产出。

## 2. 断言

- 类型断言
  - 尖括号`<number>someValue`
  - `someValue as number`
- 非空断言
- 确定赋值断言

### 类型断言

用于给告诉 TypeScript 某个值你非常确定是你断言的类型，而不是他推测出来的类型。

举个简单的例子，更新按钮只有在 id 存在的时候出现，点击时调用更新方法，这个时候就可以使用类型断言把`id`指定为`number`类型。

```typescript
const Element = () => {
  let id: number | undefined;

  const onUpdate = (updateId: number) => {
    return updateId;
  };

  const onAdd = () => {
    return null;
  };
  return (
    <div>
      {id ? (
        <a onClick={() => onUpdate(id as number)}>编辑</a>
      ) : (
        <a onClick={onAdd}> 新增</a>
      )}
    </div>
  );
};
```

### 非空断言

它可以告诉 TypeScript 某个值不是`null、undefined`，其形式为在变量后添加一个!,我们可以对上面的更新方法进行重写，在调用的时候就不用对`id`进行类型断言可以达到一样的效果

```typescript
  ...
  const onUpdate = (updateId: number | undefined) => {
    return updateId!;
  };
  ...
  <a onClick={() => onUpdate(id)}>编辑</a>
  ...
```

### 确定赋值断言

它用于告诉 TypeScript 某个值已经被赋值，举个例子

```typescript
// Error
let x: number;
initialize();
// Variable 'x' is used before being assigned.(2454)
console.log(2 * x);

function initialize() {
  x = 10;
}

// OK
let y!: number;
initialize();
console.log(2 * x);

function initialize() {
  y = 10;
}
```

所有的断言只能使我们通过 TypeScript 的类型检测，并不会帮我们修正可能错误的代码，比如我们使用了确定赋值断言，但是实际上我们并没有对其进行赋值，编译后的代码依然会是`undefind`

## 3. 类型守卫（~~好帅的名字~~）

印象中我并没有类型守卫这个名词，于是特地在官网上搜索了一下，但是并没有搜索到任何内容，可能是我搜索的关键字不对，类型守卫包括`in/typeof/instanceof`（下称三守卫）以及类型谓词。我的理解是，TypeScript 对通过守卫的变量类型进行了收窄的处理，在使用了类型守卫后，TypeScript 可以从原有类型中推测出符合守卫条件的类型。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1076e9e62a644478af1c1633cd882fa~tplv-k3u1fbpfcp-watermark.webp)

> `string`守卫: 这条路只能由类型为`string`的通过！number 守卫：俺只给`number`类型放行。Cat 类守卫：喵喵喵。喵？

为了方便理解，举个例子

```typescript
const typeGuard = (
  param: { type: string; value: string } | { type: string; name: string }
) => {
  if ("value" in param) {
    // 悬浮在param上，ts确切给出param属于第一种类型
    return param.value;
  }
  if ("name" in param) {
    // 悬浮在param上，ts确切给出param属于第二种类型
    return param.name;
  }
  return undefined;
};

typeGuard({ type: "a", value: "typea" });
```

这里举了三守卫中的`in`,剩余两种的用法也基本一致，其本质都是 TypeScript 根据语法分析进行了类型的收窄，那么在进行类型检查时，就会符合 JavaScript 原有的类型判断效果。

使用三守卫可以覆盖大多数的类型守卫场景，但也有无法覆盖的场景，举个例子

```typescript
interface Cat {
  type: string;
  belong: string;
  name: string;
  say: "mewo";
}

interface Dog {
  type: string;
  belong: string;
  name: string;
}

type Animal = Cat | Dog;
```

如上代码，我们无法通过三守卫在 Animal 类型中区分出 Cat 类型和 Dog 类型，这时候可以使用类型谓词来自定义类型守卫

```typescript
function isTinaCat(animal: Animal): animal is Cat {
  return animal.belong === "tina" && animal.type === "cat";
}

function isTinaDog(animal: Animal): animal is Dog {
  return animal.belong === "tina" && animal.type === "dog";
}
```

上面两个函数有点像平时我们用于判断符合某种条件的通用方法，只是这个函数的返回值类型有点特别，它不是任何类型，它是一个使用了谓语的语句。具体形式为：函数参数 is 类型名，称之为类型谓语。

类型谓语用于标示该函数是一个类型保护的函数，返回 true 表示符合某种类型，反之不符合。

下面使用是类型谓语的具体使用例子

```typescript
function tinaNameTheAnimal(animal: Animal) {
  if (isTinaCat(animal)) {
    // animal为Cat类型，这里悬浮在animal上，可以看到是有say属性的对象，证明是Cat类型
    animal.name = "啊猫";
  }
  if (isTinaDog(animal)) {
    animal.name = "啊狗";
  }
}

const littleCat = { type: "cat", belong: "tina", name: "", say: "mewo" };

// 把littleCat命名为啊猫
tinaNameTheAnimal(littleCat);
```

---

_参考文献_

1. [在 TS 中如何实现类型保护？类型谓词了解一下](https://cloud.tencent.com/developer/article/1600711)
2. [一份不可多得的 TS 学习指南（1.8W 字）](https://juejin.im/post/6872111128135073806#heading-39)
3. [TypeScript 官网](https://www.tslang.cn/docs/home.html)

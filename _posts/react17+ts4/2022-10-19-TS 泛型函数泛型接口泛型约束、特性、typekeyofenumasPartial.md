---
layout:     post
title:     TS 泛型函数/泛型接口/泛型约束、特性、type/keyof/enum/as/Partial
subtitle:  
date:       2022-10-19
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# TS 泛型函数/泛型接口/泛型约束、特性、type/keyof/enum/as/Partial

总结：

- 几个重要的关键词 `type` `keyof` `enum`  `as`：

    - type 类型别名：由 type 关键词创建 `type StringOrNumber = string | number`

    - keyof 获取一个类型的属性名作为联合类型
        - 对象类型：typeof 获取一个值的类型，用于帮助推导，和 keyof 联合使用获取键的联合类型
        
            ```js
            const obj = { a:1, b:2 }
            type Objkeys = keyof typeof obj; // 'a'|'b'
            ```
        
        - 数组类型：keyof 不适合直接获取取值类型
        
            ```js
            const arr = ['a', 'b'];
            type Keys = keyof typeof arr; // '0'|'1'|'length'|'toString'...(无意义)
            // 应该使用 as const + typeof
            const arr = ['a', 'b'] as const;
            type Keys = typeof arr[number]; // 'a' | 'b'
            ```
        
    - enum 用于定义一组命名常量的结构

    - as 类型断言，将一个值强制转换为特定类型 `let length: number = (some as string).length`

- 不同类型

    - 条件类型：根据条件来选择不同的类型 `T extends U? X : Y`-如果T是U类型，则返回X，否则返回Y

        ```js
        type IsString<T> = T extends string? true : false;
        ```

    - 映射类型：根据现有类型创建新的类型，`Partial<T>`  将类型 T 中所有属性变为 **可选属性**

    - 命名空间

    - 装饰器

- 泛型：本质是一个类型函数，输入 **类型参数**，和输出类型建立对应关系

    ```js
    // 1 泛型函数
    function getCode<T>(arr: T[]): T { return arr[0] }
    function getCode<T, U>(a: T, b: U): [T, U] { return [a, b] }
    // 2 泛型接口
    interface Coder<T> { contents: T, compare(v: T): number {} }
    interface Coder { compare<T>(arg: T): T {} }
    // 3 泛型约束 extends - 必须满足 {length: number} 接口的约束
    function getCode<T extends {length: number}>(arg: T): number { return arg.length }
    // 箭头函数
    const func = <T>(a:T):T[] => { return [a] }
    const res = func<string>('Alice');
    ```

- 关键特性

    - TS 是 **面向接口编程**，而不是面向对象编程 - 并不关心变量类型，只需要满足接口
    - TS的代码只涉及类型，所有跟值相关的代码都由JS完成，**TS代码的编译过程**，实际上就是把“类型代码”全部拿掉，只保留“值代码”



### 五、实际使用的补充

#### 1、keyof VS typeof 比较

【`keyof` 操作符】

- 作用：用于获取一个类型的所有已知属性名，返回的结果是一个联合类型 --> 联合类型中的每个成员就是原始类型中属性的名称（作为字符串字面量类型）

```tsx
interface Person {
    name: string;
    age: number;
    address: string;
}
type PersonKeys = keyof Person; // PersonKeys 就是 "name" | "age" | "address"
```

【 `typeof` 操作符】

两种主要用法：

- js 中用于获取一个变量的数据类型
- ts 中用于获取一个值的类型，常用来基于已有值的结构推导类型

```ts
const myObj = {
    prop1: "hello",
    prop2: 10
};
type MyObjType = typeof myObj; // MyObjType 被推导为 { prop1: string; prop2: number; }
```

【组合使用 keyof 、 typeof】

- `keyof typeof Type`：先 typeof 获取类型结构；再 keyof 获取类型结构的所有属性名称
- `as keyof typeof Type` ：as 强制类型转换

```tsx
enum Type {
    READ = '阅读',
    STUDY = '前端'
}
keyof typeof Type; // 先通过 typeof 获取到 Type 的结构类型，再通过 keyof 获取所有属性名称的联合类型
key as keyof typeof Type; // as 进行断言(强制类型转换的安全写法)，key 只能是 Type 所有属性名称组成的联合类型其中之一，即 "READ" | "STUDY"
```

#### 2、Partial

是 TS 提供的内置工具类型

`Partial<T>`：将类型 `T` 中的所有属性变为可选属性



### 一、泛型 

解决问题：主要是为了解决类型声明时，函数 **返回值类型** 和 **参数类型** 是相关的

特点：【**类型参数**】- 函数名后面尖括号的部分`<T>`

##### 本质

泛型本质上是一个 **类型函数**，通过输入类型参数，可以在 **输入类型** 与 **输出类型** 之间，建立一一对应关系

- 注意：可以设置类型参数的默认值，但是 TypeScript 会从实际参数推断 T 的值，**覆盖掉默认值**，不会报错
    - 只是在这种调用时才知道 T 的类型的情况下不报警 - 如果是实例化一个泛型类，设置了默认类型后，再调用时如果 **不符合类型校验，是会报错的**

```js
// 函数的参数类型是 T[]，返回值的类型是 T
function getCode<T = string>(arr: T[]): T {
    return arr[0];
}
// 函数调用时，需要提供类型参数; 也可以省略，让 TS 自己推断
getCode<number>([1, 2, 3]);
getCode([1, 2, 3]); // 不会报错，T的类型被推断为 number，覆盖掉默认值 string
```



有个热知识：ts 是一种鸭子类型的语言，**面向接口编程**，而不是面向对象编程 -> **其实 ts 并不关心变量是什么类型，只要满足接口的都可以**

- 看下面的案例，最常用的 useState 的函数签名如下，函数后面的 `<S> 表示泛型占位符`，字母是什么不重要，但是在后面使用的时候，表示入参是 S 类型，输出也是 S

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20221020091529232.png" alt="image-20221020091529232" style="zoom:50%;" />

---

#### 常见写法

泛型通常使用在 函数、变量、接口和类、别名中，以下是不同场景中的写法：

```js
// 1、泛型函数 - 类型参数放在尖括号中，写在函数名后面
function getCode<T>(arg: T): T { return arg }
```

```js
// 2、变量形式定义的函数 - 两种写法
let myCode1: <T>(arg: T) => T = getCode;
let myCode2: { <T>(arg: T): T } = getCode;
```

3、接口中

- 第一种写法：**类型参数** 定义在整个接口，接口内部的所有属性/方法都可以使用该类型参数

```typescript
interface Code<Type> { contents: Type }
// 泛型接口，调用时需要给出类型参数的值
let code: Code<string>;

// 另一个例子 - 将接口中泛型的写法，结合类使用
interface Comparator<T> { compareTo(value: T): number }
class Rectangle implements Comparator<Rectangle> {
    compareTo(value: Rectangle): number {}
}
```

- 第二种写法：将类型参数定义在接口的某个方法之中，其他属性/方法不能使用该类型参数

```typescript
interface Fn { <T>(arg: T): T }
function getCode<T>(arg: T): T { return arg }
// 具体的类型需要在 getCode 使用时提供
let code: Fn = getCode;
```

4、类中

```typescript
// 1、泛型类的参数写在类名后面
class Rectangle<K, V> { key: K; value: V;}
// 继承时，必须给出泛型类的类型参数
class X extends Rectangle<string, number> {}
// 2、类的本质是一个构造函数，因此泛型类也可以写成构造函数
type MyCode<T> = new (...arg: any[]) => T;
// 或者
interface MyCode<T> { new (...arg: any[]): T;}
```

注意：泛型类描述的是 **类的实例**，不包括静态属性/方法，因为静态属性/方法的定义在类的本身，因此不能引用类型参数。

5、别名中

```typescript
// type 命令定义的类型别名也可以使用泛型
type A<T> = T | undefined | null;
// 树形结构的例子
type Tree<T> = {
    value: T;
    left: Tree<T> | null; // 递归引用了 Tree 自身
    right: Tree<T> | null;
}
```



### 二、类型参数的约束条件

TS提供了一种语法，允许在类型参数上面写明约束条件，不满足约束条件时，编译时报错。

```typescript
// 约束条件：T extends {length: number}
// 表示类型参数 T 必须满足 {length: number}， 否则就会报错
function getCode<T extends {length: number}>(p1: T, p2: T) {
    if(p1.length >= p2.length) {
        return p1;
    }
    return p2;
}
```

- 在 interface 中也可以使用 - <T = Element> 表示默认为 Element 类型

```typescript
interface FormEvent<T = Element> extends SyntheticEvent<T> {}
```

- 同时设置约束条件和默认值（默认值必须满足约束条件）

```typescript
type Fn<A extends string, B extends string = "Alice"> = [A, B];
type Result = Fn<"hello">; // ['hello', 'Alice']
```

.d.ts + js 文件 = ts 文件

tsc（TS官方提供的编译器-一个npm模块）作用：把 .ts 转变成 .js，给出错误，但是仍旧生成编译产物。

### 三、TS 类型（纯基础记录）

所有的类型名称都是小写字母，大写字母是JS中内置对象，而不是类型名称

##### 1、number + bigint

```typescript
// number 包含所有整数、浮点数、非十进制数
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
// bigint 和 number 不兼容（大整数）
let big: bigint = 100n; 
let big1: bigint = 123; // 报错
```

##### 2、string - 普通字符串/模版字符串

```typescript
let name: string = "Alice"; // `${x} Alice`
```

##### 3、array - 所有元素 **类型相同** 的值的集合

:tada:注意：在 js 中 ['Alice', 428] 是一个数组，在 TS 中不算数组，叫做 tuple

```typescript
let list: Array<number> = [1, 2, 3];
// 也可以表示成 list: number[]
```

##### 4、boolean

```typescript
let isFalse: boolean = false;
```

##### 5、函数

两种声明方法：1⃣️ 在 js 函数上直接声明参数和返回值

```typescript
const isFalsy = (value: any) : boolean => {
  return value === 0 ? true : !!value;
} 
```

2⃣️ 声明想要的函数类型

```typescript
const isFalsy: (value: any) => boolean = (value) => {
  return value === 0 ? true : !!value;
} 
```

##### 6、any

不推荐使用，使用 any 相当于关闭了变量的类型检查（甚至会污染其他变量，把错误留到进行时 -> TS引入 unkown类型解决）

##### 7、void

一般只使用在一个地方：表示函数不返回任何值或返回 undefined

##### 8、object

除了 `number、string、boolean、bigint、symbol、null、undefined` 都是 object

在 JS 的设计中，所有对象、数组、函数都是 object 类型

##### 9、tuple

另一种 array - "数量固定，类型可以各异"，在 js 中不区分，在 ts 中区分

```typescript
const [name, setName] = useState<string>('')
```

使用场景 - custom hook 的返回值 - 没经验，截图留念先

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20221019212157924.png" alt="image-20221019212157924" style="zoom:40%;" />

##### 10、enum

```typescript
enum Color {
  Yellow,
  Green,
  Blue,
  Red
}

let c: Color = Color.Green;
```

##### 11、undefined 和 null

既是一个值，也是一个类型

注意⚠️：如果没有声明类型的变量被赋值给 `undefined` 或者 `null`，它们的类型会被推断为 any（在设置编译选项 `strictNullChecks` 可以被推断为正确类型）

```typescript
let u: undefined = undefined;
let n: null = null;
```

##### 12、unkown - 严格版本的 any

和 any 的 **共同点**：表示这个值可以是任何值

和 any 的 **不同点**：

1. 不能直接赋值给其他类型（除了unkown本身和 any）-> 避免了污染

```typescript
let a: unkown = 1;
let b: boolean = a; // 报错，unkown 不能赋值给 boolean
let c: number = a; //报错，unkown不能赋值给 number
```

2. 不能直接调用 unkown 变量的方法和属性

```typescript
let obj: unkown = {a: 1};
obj.a; // 报错，unkown 不能被调用

let fun: unkown = (n = 0) => n+1;
fun(); // 报错，unkown 不能被调用

let str :unkown = 'hello';
str.trim(); // 报错，unkown 不能被调用
```

3. unkown 只能进行比较运算、取反运算、typeof 运算符、instanceof 运算符，其他的运算都会报错

```typescript
let num: unkown = 1;
num + 1; //报错，unkown 不能被运算
num === 1; // 正确
```

##### 13、never 空类型 -> 即不可能有这样的值

```typescript
let x: never; // 不能给变量x赋值，否则都会报错
```

特点：可以赋值给任意其他类型（空集是任何集合的子集）

```typescript
// 函数抛错就是返回的 never 类型
const fun: () => never = () => {
  throw new Error()
}
let num: number = fun(); // 正常
let str: string = fun(); // 正常
```

总结：TS中 `any`和`unkown`是两个顶层类型，但是只有一个底层类型`never`

##### 14、symbol

```typescript
const x: symbol = Symbol();
```





### 四、知识点补充

1、TS规定，变量只有赋值后才能使用，否则就会报错

2、TS的代码只涉及类型，所有跟值相关的代码都由JS完成，TS代码的编译过程，实际上就是把“类型代码”全部拿掉，只保留“值代码”

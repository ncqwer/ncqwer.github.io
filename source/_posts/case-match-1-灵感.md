---
title: case-match(1) 灵感
date: 2021-02-22 23:45:39
tags: ts tricks functianal
---

## 前景提要
在js|ts编码过程中，根据不同类型的数据类型选择不同的代码分支是一个很常见的需求。针对这一情景，程序员门也提供了多种方式处理这一问题。例如：

方案一：
```ts
// 使用 instanceof 操作符
class A{}
class B{}
const simplest = (value:A|B)=>{
  if(value instanceof A) return methodA(value);
  if(value instanceof B) return methodB(value);
}
``` 
方案二：
```ts
// 使用 基类 操作符
abstract class Base{
  abstract method():any;
}

class A extends Base{
  method(){}
} 
class B extends Base{
  method(){}
} 
```
除了以上方案外，程序员门还创造出一些抽象层次更高的解决方案（例如策略模式等），这里就不一一例举了。

> 处于文章结构考虑，我们先“选择性忽视”基于函数重载的方案，关于函数重载的套路我们放在了这儿。

## 可读性
衡量一段代码质量的时候，又一个较为重要的指标-**可读性**，它旨在在阅读代码的过程中提供简明的信息来指明代码的具体含义。基于这一目的，我们可以有一个模糊映像：
  - 代码尽可能偏向自然语言
  - 代码尽可能简短，这提供了尽可能高的信息密度

现在，让我们回过头去，我们会发现之前一些方案的一些问题：

方案一：重复使用`if() return `的形式,但具体编写过程中，判断条件往往由于不同的数据类型而使用不同的判断方式：

  - number： `typeof v === 'number'`
  - string: `typeof v === 'string'`
  - boolean： `typeof v === 'boolean'`
  - array： `Array.is(v)`
  - class A: `v instanceof A`

这种不一致在一定程度上造成了阅读的“障碍”。同时，这种编写方式，总是会允许在`if-return`结构中新增其他代码，而一旦新增了代码，就会破坏该处代码结构上的一致性，进而降低了一致行。（虽然这可以通过代码规范了禁止这种情况的发现，但人类总是会犯错的，不是吗？）

方案二：它使用了ts关于继承和虚函数的相关背景知识，而这些知识处于“隐喻”地位，不直观，需要代码阅读者了解相关知识才能获得信息。同时，它是基于class的方案，对于大部分场景来说，它太重了。

## 函数重载

在之前讨论的中，我们选择性忽视了基于函数重载的方案。现在我们看一下：

>  由于js｜ts语言的动态类型，js并没有函数重载方案，ts的函数重载仅仅作用于类型空间，故使用其他的语言来描述：

C语言：
```c++
int method(const int& v){return v;}
char* method(const char* v){return v;}
```
haskell语言:
```haskell
data Book
  = BookStr String
  | BookInt Int
  | BookComplex String Int

test :: Book -> String
test (BookStr v) = v
test (BookInt _) = "no str"
test (BookComplex v _) = v
```
在函数重载的示例中，不难看出，函数重载的实现方式在某种程度上都好于上述的两种方案：
  - 更符合自然语意：函数接受某种类型->函数执行某段代码逻辑。
  - 更加不容易出错：函数重载的特定性阻止了其他不一致的实现方案。
  
综上，我们可以得出结论，函数重载在一定程度上可以更好的辅助解决常见的`if-then`逻辑。

## js｜ts “函数重载”

之前提到过，受限于动态类型，`js|ts`并没有完全体的函数重载方案。但，这并不影响我们通过编写额外的代码来提供一个仿照版的方案。说干就干，先让我们想想我们希望的客户端代码样子：
```js
const method = m
  .case(TypeA,methodA) // 使用case来定义处理逻辑
  .case(TypeB,methodB)
  .case(TypeC,methodC)
  //...


method(valueA)// run methodA
method(valueB)// run methodB
method(valueC)// run methodC
```
整个实现方案的核心点是
  - 将类型信息与相关的处理函数关联起来，在我们的实现方案中，我们使用`case`函数来完成这一效果。
  - `case`函数返回的是事实上的处理函数，它接受特定类型的值，并从值中获得对应类型（这里我们使用的是值的构造器），并以其存储的值的构造列表为参数来调用通过`case`函数定义的处理函数。

## 总结

为了寻找更好（指可读性）的解决方案（指处理根据不同类型的数据类型选择不同的代码分支这一问题），我们回顾了一些常见的处理方法（`if-return`代码结构，类的继承，函数重载等），并判断出函数重载是一个较为优秀的方式。

但同时，受限于js|ts的函数重载功能的不完善，我们并不能获得开箱即用的函数重载。故我们决定"自力更生",并"暂时"决定了大致的api行为。

接下来，让我们开始编写具体代码实现吧！

> 实际上，如果你并不关心具体实现，仅仅只是想尝试类似api的case。目前npm上已经有了[相关的实现](https://www.npmjs.com/package/@zhujianshi/case-match)（它的实现代码即是是本系列之后文章的主要内容），你可以在本地或[runkit](https://runkit.com/)体验。
---
title: interface & type
date: 2024-11-05 09:27:00
tags:
---

## 一、相同点

### 1、都可以描述对象和函数的类型

但语法不一样，type 使用 = 赋值

``` ts
interface User {
    name: string
    age: number
}

interface SetUser {
    (name: string, age: number): void
}
```

``` ts
type User = {
    name: string
    age: number
}

type SetUser = (name: string, age: number) => void
```

### 2、都允许扩展（extends）

并且 interface 和 type 可以互相扩展

#### （1）interface 扩展 interface

``` ts
interface Name {
    name: string
}

interface User extends Name {
    age: number
}
```

#### （2）type 扩展 type

``` ts
type Name = {
    name: string
}

type User = Name & {
    age: number
}
```

#### （3）interface 扩展 type

``` ts
type Name = {
    name: string
}

interface User extends Name {
    age: number
}
```

#### （4）type 扩展 interface

``` ts
interface Name {
    name: string
}

type User = Name & {
    age: number
}
```

## 二、不同点

### 1、type 可以，但 interface 不行

#### （1）基本类型别名

``` ts
type Name = string
```

#### （2）联合类型

``` ts
interface Dog {
    dogName: string
}
interface Cat {
    catName: string
}
type Pet = Dog | Cat

type StringOrNumber = string | number

type Text = string | { text: string }
```

#### （3）元组

具体定义数组每个位置的类型

``` ts
type PetList = [Dog, List]
```

#### （4）使用 typeof 获取实例的类型 对 type 进行赋值

``` ts
let divEle = document.createElement('div')
type Div = typeof divEle
```

### 2、interface 可以，但 type 不行

#### interface 能够声明合并

``` ts
interface User {
    name: string
}

interface User {
    age: number
}

/**
 * User 接口为 {
 *     name: string
 *     age: number
 * }
 */
```

## 三、总结

优先用 interface，无法实现再用 type，因为 interface 具有更直观和易于理解的特点，并且能够进行声明合并。

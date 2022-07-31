---
title: Javascript core
date: 2021-12-24 18:40:39
categories:
- Javascript
tags:
---

## Javascript basic

 - basic syntax
 - data type
 - operator
 - flow control
 - function
 - object
 
## Built in object and DOM operation

 - Array, Date, String, Math
 - event handling
 - DOM
 - BOM
 
**DOM, BOM:**
DOM describes the methods and interfaces for processing web page content, and BOM describes the methods and interfaces for interacting with the browser.

## Javascript
 - constructor
 - new
 - prototype & \__proto__
 - this
 - call/bind/apply
 - inherit
 - design pattern
 - V8


### prototype & \__proto__
https://www.bilibili.com/video/BV1Q64y1v7fW?p=4

Prototype是类型的原型，\__proto__是实例访问原型的方法, 对于同一类型和其实例，prototype === \__proto__

原型链是一个实例的类型或其父类的prototype形成的链条

### new
the process of new operator:
1. Create a new empty object 创建一个新的空的对象
2. Bind the prototype of the object to the prototype of the constructor 把对象的原型绑定到构造函数的原型
3. Execute constructor and bind this 执行构造函数，绑定this
4. Return the object 返回这个对象

https://www.bilibili.com/video/BV1Na4y1i7U1?from=search&seid=8525508610732534034&spm_id_from=333.337.0.0

### Data types

**Dictionary:**
```
let dic: { [key: string] : string } = {};
dic['abcd'] = 'defg';
```
**Tuple:**
A fixed length array

```
let a: [number, string];
a = [1, 'abc'];
```

**Access modifier**

Private can be written as: #variable, but this is only for fields, not functions

```
class Cat {
    #name: string; // this is a private field
    
    constructor(name: string) {
        this.#name = name;
    }
}

const cat = new Cat('randy);
console.log(cat.#name);  // this will have compile error
```

### Design patterns

**Factory**

The factory pattern is really about adding extra abstraction between object creation and where it is used. This gives you extra options that you can more easily extend in the future.

**Abstract factory**
Use when you want to provide a library of relatively similiar products from multiple different factories.
You want the system to be independent of how the products are created.
在工厂方法的基础上增加一个抽象工厂基类

```Javascript
class AbstractFurnitureFactory {
    static getFurniture(furniture: string): IFurniture | undefined {
        if (['SmallChair', 'BigChair'].includes(furniture)) {
            return ChairFactory.getChair(furniture);
        } else if (['SmallTable', 'BigTable'].includes(furniture)) {
            return TableFactory.getTable(furniture);
        }
        throw new Exception('Factory not found');
    }
}

AbstractFurnitureFactory.getFurniture('BigChair');
```
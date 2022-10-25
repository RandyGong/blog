---
title: The differences between for...in and for...of
date: 2021-09-05 23:37:10
categories:
- Javascript
tags:
- forof
- forin
---

## The most obvious difference is:

- for...in is to get key
- for...of is to get value

Example:

```Javascript
    const arr = [10, 20,30]
    for(let val in arr){
      console.log('for...in', val)
    }
    
    console.log('========')
    
    for(let val of arr){
      console.log('for...of', val)
    }
```
![图片.png-18.2kB][1]

## They are applicable to different data types

- Traversal object: for...in yes, for...of no
- Traverse map set: for...of yes, for...in no
- Traversal generator: for...of no, for...in no

Example

**Object**

```Javascript
const obj = {
  name: 'Randy',
  city: 'beijing'
}

for(let val in obj){
  console.log('for...in', val)
}

for(let val of obj){
  console.log('for...of', val)
}
```

![图片.png-19.2kB][2]

**set**

```Javascript
const set = new Set([10, 20, 30])
for(let n in set) {
  console.log('for...in', n)
}
for(let n of set) {
  console.log('for...of', n)
}
```

![图片.png-14.3kB][3]

**map**

```Javascript
const map = new Map([
  ['x', 10],
  ['x1', 110],
  ['x2', 210],
])
for(let m in map) {
  console.log('for...in', m)
}
for(let m of map) {
  console.log('for...of', m)
}
```

![图片.png-19.1kB][4]

**generator**

```Javascript
function* foo() {
  yield 10
  yield 20
  yield 30
}
for(let g in foo) {
  console.log('for...in', g)
}
for(let g of foo) {
  console.log('for...of', g)
}
```
![图片.png-19.1kB][5]

## Under what circumstances can they be used?

**for...in**
can be used in enumerable data, such as:
- object
- array
- character string

**What is enumerable data?**

For example

```Javascript
const obj = { x: 100 }
```

We use Object.getOwnPropertyDescriptors method gets the property descriptors of all its own properties of the specified object.

![图片.png-23.2kB][6]

At this time, it is found that enumerable in each of its attributes is true, and it is enumerable.

If the enumerable of an attribute is false, the following three operations will not get the attribute.

- for...in loop
- Object.keys method
- JSON.stringify method

So generally, objects, arrays, and strings are enumerable

**for...of**

for...of is used for iterative data, such as:

- Array
- String
- map
- set

**What is iterative data?**

For example

```Javascript
const arr = [10, 20, 30]
```

Array has a **Symbol.iterator** property

1. As long as a data has implemented the iterator interface, the data has an property called **[symbol.iterator]**
2. **[Symbol.iterator]** will return a function
3. An object will be returned after the function returned by **[Symbol.iterator]** is executed
4. Another method called next in the object returned by the **[symbol.iterator]** function
5. The next method will return an object {value: 10, done: false} every time it is executed
6. This object stores the currently fetched data and the mark of whether it has been fetched

Next and next are constantly called:

- A next() => B
- B next() => C
- ...

![图片.png-26.9kB][7]

## To conclude

**for...in** can be used for enumerable data
**for...of** for iterative data


  [1]: http://static.zybuluo.com/RandyGong/by7o5u2skb5vld6vcruknd4r/%E5%9B%BE%E7%89%87.png
  [2]: http://static.zybuluo.com/RandyGong/alvapp62m81qei9u7wdvlm48/%E5%9B%BE%E7%89%87.png
  [3]: http://static.zybuluo.com/RandyGong/4d88r47m6dzzwkqocgyqusms/%E5%9B%BE%E7%89%87.png
  [4]: http://static.zybuluo.com/RandyGong/4igz0ugy3xodxvb78rhuc4k8/%E5%9B%BE%E7%89%87.png
  [5]: http://static.zybuluo.com/RandyGong/pvnrwvd3uvr74t3xffgwbtpa/%E5%9B%BE%E7%89%87.png
  [6]: http://static.zybuluo.com/RandyGong/fh8js6h2j4fv22j4zdii4gnp/%E5%9B%BE%E7%89%87.png
  [7]: http://static.zybuluo.com/RandyGong/q3zs1g538jjedx6enpcom167/%E5%9B%BE%E7%89%87.png
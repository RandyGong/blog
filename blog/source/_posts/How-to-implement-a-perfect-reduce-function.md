---
title: How to implement a perfect reduce function?
date: 2022-04-19 23:17:32
categories:
- Javascript
tags:
---

## Basic Usage

The reduce function is a function provided by JS natively for processing array structures.
Let's take a look at the introduction in MDN:

The reduce() method executes a user provided reducer function for each element in the array in order. Each time the reducer is run, the calculation results of the previous elements will be passed in as parameters, and finally the results will be summarized into a single return value.


### parameter

1. callbackfn is a reducer function, which contains four parameters:
- previousvalue: the return value of the last call to callbackfn. In the first call, if the initial value initialvalue is specified, its value is initialvalue, otherwise it is the element array[0] with array index 0.
- currentvalue: the element being processed in the array. In the first call, if the initial value initialvalue is specified, its value is the element array[0] with array index 0, otherwise it is array[1].
- currentindex: the index of the element being processed in the array. If the initial value initialvalue is specified, the starting index number is 0, otherwise it starts from index 1.
Array: an array used for traversal.

2. initialValue can be selected as the value of the parameter previousvalue when the callback function is called for the first time. If the initialvalue is specified, currentvalue will use the first element of the array; Otherwise, previousValue will use the first element of the array, while currentValue will use the second element of the array.

The reduce function is very powerful and can be used in many scenarios:

For example, use the reduce function to realize **accumulation**:

```Javascript
 let total = [ 0, 1, 2, 3 ].reduce(
    ( previousValue, currentValue ) => previousValue + currentValue,
    0
  )
  // 6
```

**Generate new array**:

```Javascript
let total = [ 0, 1, 2, 3 ].reduce(
    function (pre, cur) {
      pre.push(cur + 1)
      return pre
    },
    []
  )
  // [1, 2, 3, 4]
```
  
**So the question is, how to implement a reduce function by handwriting?**

## Implement basic version

According to the document, the reduce function accepts a running function and an initial default value

```Javascript
/**
    *
    * @param {Array} data 原始数组
    * @param {Function} iteratee 运行函数
    * @param {Any} memo 初始值
    * @returns {boolean} True if value is an FormData, otherwise false
    */
  function myReduce(data, iteratee, memo) {
      // ...
  }
```

Next, realize the basic functions

The key point of the reduce function is to transfer the result to the execution function again for processing

```Javascript
 function myReduce(data, iteratee, memo) {
   for(let i = 0; i < data.length; i++) {
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
  }
```

### Requirement 1: add binding for this

In fact, the reduce function can specify the custom object binding this

Here you can use call to rebind functions

```Javascript
 function myReduce(data, iteratee, memo, context) {
   // 重置iteratee函数的this指向
   iteratee = bind(iteratee, context)
  
   for(let i = 0; i < data.length; i++) {
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
  }
  
  // 绑定函数 使用call进行绑定
  function bind(fn, context) {
    // 返回一个匿名函数，执行时重置this指向
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```

### Requirement 2: add support for the default value of the second parameter

The third parameter of the reduce function is also an optional value. If the third parameter is not passed, it is initialized directly with the first position of the incoming data

```Javascript
 function myReduce(data, iteratee, memo, context) {
   // 重置iteratee函数的this指向
   iteratee = bind(iteratee, context)
   // 判断是否传递了第三个参数
   let initial  = arguments.length >= 3; // 新增
   // 初始的遍历下标
   let index = 0 // 新增
  
   if(!initial) { // 新增
    // 如果用户没有传入默认值，那么就取数据的第一项作为默认值
    memo = data[index] // 新增
    // 所以遍历就要从第二项开始
    index += 1 // 新增
   }
  
   for(let i = index; i < data.length; i++) { // 修改
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
  }
  
  // 绑定函数 使用call进行绑定
  function bind(fn, context) {
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```

### Requirement 3: support objects

JS native reduce function does not support the data structure of object, so how to improve our reduce function?

In fact, you only need to take out all the keys in the object, and then traverse the keys

```Javascript
function myReduce(data, iteratee, memo, context) {
 
   iteratee = bind(iteratee, context)
   // 取出所有的key值
   let _keys = !Array.isArray(data) && Object.keys(data) // 新增
   // 长度赋值
   let len = (_keys || data).length // 新增
 
   let initial  = arguments.length >= 3;
   let index = 0
  
   if(!initial) {
    // 如果没有设置默认值初始值，那么取第一个值的操作也要区分对象/数组
    memo = data[ _keys ? _keys[index] : index] // 修改
    index += 1
   }
  
   for(let i = index; i < len; i++) {
    // 取key值
    let currentKey = _keys ? _keys[i] : i // 新增
    memo = iteratee(memo, data[currentKey], currentKey, data) // 修改
   }
   return memo
  }
  
  function bind(fn, context) {
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```

### Requirement 4: reduceRight

In fact, the above content is already a relatively complete reduce function. The last extension is the reduceRight function. In fact, the function of the reduceRight function is also very simple, that is, to operate in reverse order during traversal, for example:

```Javascript
let total = [ 0, 1, 2, 3 ].reduce(
    function (pre, cur) {
      pre.push(cur + 1)
      return pre
    },
    []
  )
  
  // [1, 2, 3, 4]
```

In fact, it is also very simple to realize this function. You only need to change the value of the initial operation to the position of the last element:

```Javascript
// 加入一个参数dir，用于标识正序/倒序遍历
function myReduce(data, iteratee, memo, context, dir) { //修改

  iteratee = bind(iteratee, context)
  let _keys = !Array.isArray(data) && Object.keys(data)
  let len = (_keys || data).length

  let initial  = arguments.length >= 3;

  // 定义下标
  let index = dir > 0 ? 0 : len - 1 // 修改
 
  if(!initial) {
   memo = data[ _keys ? _keys[index] : index]
   // 定义初始值
   index += dir // 修改
  }
  // 每次修改只需步进指定的值
  for(;index >= 0 && index < len; index += dir) { // 修改
   let currentKey = _keys ? _keys[index] : index
   memo = iteratee(memo, data[currentKey], currentKey, data)
  }
  return memo
 }
 
 function bind(fn, context) {
   if (!context) return fn;
   return function(memo, value, index, collection) {
     return fn.call(context, memo, value, index, collection);
   };
 }
```

When calling, directly pass in the last parameter as 1 / \-1

```Javascript
 myReduce([1, 2, 3, 4], function(pre, cur) {
     console.log(cur)
 }, [], 1)
 
 myReduce([1, 2, 3, 4], function(pre, cur) {
     console.log(cur)
 }, [], -1)
```

Finally, the whole function is reconstructed and separated into a separate function:

```Javascript
function createReduce(dir) {
     function reduce() {
   // ....
     }
 
     return function() {
   return reduce()
     }
 }
```

The final code is as follows:

```Javascript
function createReduce(dir) {
 
     function reduce(data, fn, memo, initial) {
         let _keys = Array.isArray(data) && Object.keys(data),
             len = (_keys || data).length,
             index = dir > 0 ? 0 : len - 1;

         if (!initial) {
             memo = data[_keys ? _keys[index] : index]
             index += dir
         }

         for (; index >= 0 && index < len; index += dir) {
             let currentKey = _keys ? _keys[index] : index
             memo = fn(memo, data[currentKey], currentKey, data)
         }
         return memo
     }


     return function (data, fn, memo, context) {
         let initial = arguments.length >= 3
         return reduce(data, bind(fn, context), memo, initial)
     }
 }

 function bind(fn, context) {
     if (!context) return fn;
     return function (memo, value, index, collection) {
         return fn.call(context, memo, value, index, collection);
     };
 }

 let reduce = createReduce(1)
 let reduceRight = createReduce(-1)
```

This implementation method is also the method of the reduce function implemented by underscore.js.
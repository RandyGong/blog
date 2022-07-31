---
title: Four situations where arrow functions should not be used
date: 2022-03-03 22:55:16
categories:
- Javascript
tags:
- arrow function
---


Arrow functions bring great convenience to our work, but what are their shortcomings? Should we always use arrow functions? In which scenarios should we stop using arrow functions? In today's article, I'll summarize four scenarios in which the arrow function is not recommended. I hope it will be useful to you.

### Some disadvantages of arrow function

- Parameter object is not supported

In arrow functions, we cannot use the arguments object as in ordinary functions.

```Javascript
const fn1 = () => {
  console.log('arguments', arguments)
}

fn1('fatfish', 'medium')

function fn2(){
  console.log('arguments', arguments)
}

fn2('fatfish', 'medium')
```

As you can see, the arrow function of FN1 reports an error, but FN2 can read the arguments object normally.

![image.png-215kB][1]

How can we get all the parameters passed to the function in the arrow function?

You can use spread operator to solve it.

```Javascript
const fn3 = (...values) => {
  console.log('values', values)
}

fn3('fatfish', 'medium')
```

- Cannot change this pointer through apply, call, bind

I believe you can easily know what the following code will output.

```Javascript
const fn1 = () => {
  console.log('this-fn1', this)
}

fn1()

function fn2(){
  console.log('this-fn2', this)
}

fn2()
```

![image.png-175.4kB][2]

We want both FN1 and FN2 to print objects. What should we do?

```Javascript

const thisObj = {
  name: 'fatfish'
}

const fn1 = () => {
  console.log('this-fn1', this)
}

fn1.call(thisObj)

function fn2(){
  console.log('this-fn2', this)
}

fn2.call(thisObj)
```

Because the arrow function determines who its this points to when it is defined, there is no way to change it again with fn1.call (thisobj).

### When can I not use the arrow function?

Arrow functions are not omnipotent. There are at least four situations in which we should not use them.

**Do not use arrow functions in constructors**

```Javascript

function Person (name, age) {
  this.name = name
  this.age = age
}

const Person2 = (name, sex) => {
  this.name = name
  this.sex = sex
}

console.log('Person', new Person('fatfish', 100))
console.log('Person2', new Person2('fatfish', 100))
```

Why does new Person2 throw an error?

Because the constructor generates an object instance through the new keyword. The process of generating an object instance is also the process of binding this to the instance through the constructor.

However, the arrow function does not have its own this, so it cannot be used as a constructor, nor can it be called through the new operator.

**Please do not operate this in the click event**

We often read the element itself through this in the click event.

```Javascript

const $body = document.body

$body.addEventListener('click', function () {
  // this and $body elements are equivalent
  this.innerHTML = 'fatfish'
})
```

But if you use the arrow function to add a callback to DOM elements, it will be equivalent to the global object window.

```Javascript
const $body = document.body

$body.addEventListener('click', () => {
  this.innerHTML = 'fatfish'
})
```

**Do not use arrow functions in methods of objects.**

```Javascript

const obj = {
  name: 'fatfish',
  getName () {
    return this.name
  },
  getName2: () => {
    return this.name
  }
}

console.log('getName', obj.getName())
console.log('getName2', obj.getName2())
```

The getname2 method will not print "fatfish", because at this time this and window are equivalent, not equal to obj.

![image.png-276.8kB][3]

**Please do not use arrow function in prototype chain**

```Javascript
const Person = function (name) {
  this.name = name
}

Person.prototype.showName = function () {
  console.log('showName', this, this.name)
}

Person.prototype.showName2 = () => {
  console.log('showName2', this, this.name)
}

const p1 = new Person('fatfish', 100)

p1.showName()
p1.showName2()
```



  [1]: http://static.zybuluo.com/RandyGong/w3n9x2ir6vadyrelkru6sffi/image.png
  [2]: http://static.zybuluo.com/RandyGong/1q4zl0kmuk3zmoxge9hl935p/image.png
  [3]: http://static.zybuluo.com/RandyGong/qfmjtkiatusc4euhbuwned58/image.png
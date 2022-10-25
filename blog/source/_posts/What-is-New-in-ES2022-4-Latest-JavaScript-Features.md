---
title: What is New in ES2022? 4 Latest JavaScript Features
date: 2022-08-03 23:34:51
categories:
- Javascript
tags:
- ES2022
- ES13
---

This is an overview of the new ES13 specs forwarded from https://betterprogramming.pub/es2022-features-javascript-a9f8f5dcba5a

The new ES13 specs are finally released. Specs? JavaScript is indeed not an open-source language. It’s a language written following the ECMAScript standard specifications. The TC39 committee is in charge of discussing and approving new features. Who are they?

> “ECMA International’s TC39 is a group of JavaScript developers, implementers, academics, and more, collaborating with the community to maintain and evolve the definition of JavaScript.” — TC39.es

Their release process is composed of five stages. Since 2015, they have been doing yearly releases. They normally happen around springtime.

There are two ways to refer to any ECMAScript release:

- By its year: this new release will be ES2022.
- By its iteration number: This new release will be the 13th iteration, so it can be referred to as ES13.
    
So what is new in this release? What features can we get excited about?

## RegExp Match Indices

Currently, only the **start index** of a match is returned when using the JavaScript Regex API in JavaScript. However, that is not enough for some special advanced scenarios.

As part of these specs, a special flag **d** was added. By using it, the regex API will return a two-dimensional array as a key named **indices**. It contains the **start** and **end** index for each match. If there were any named group captured in the regex it would return their start/end indices in the **indices.groups** object. The named group name would be its key.

```Javascript
// ✅ a regex with a 'B' named group capture
const expr = /a+(?<B>b+)+c/d;

const result = expr.exec("aaabbbc")

// ✅ shows start-end matches + named group match
console.log(result.indices);
// prints [Array(2), Array(2), groups: {…}]

// ✅ showing the named 'B' group match
console.log(result.indices.groups['B'])
// prints [3, 6]
```

## Top-level await

Before this proposal, it was not accepted to have **top level awaits**. There were some workarounds to kind of simulate the behavior but all had their drawbacks.

The top-level await feature let us rely on the module to handle these promises. It is an intuitive feature to use. However, note that it might alter the execution order of the modules. If one module depends on another that has a top-level await call, that module’s execution would be halted until the promise is fulfilled.

Let’s see an example:

```Javascript
// users.js
export const users = await fetch('/users/lists');

// usage.js
import { users } from "./users.js";
// ✅ the module will wait for users to be fullfilled prior to executing any code
console.log(users);
```

In the above example, the engine will wait for the users to be done prior to executing the code on the usage.js module.

In summary, it is a nice and intuitive feature that needs to be used with care. We don’t want to be abusing it.

## .at()

For a long time, there has been a request for JavaScript to provide a Python like array negative index accessor. Instead of doing **array[array.length-1]** to do simply **array[-1]**. That is simply not possible because the [] notation is also used on objects in JavaScript.

The accepted proposals take a more practical approach. The Array object will now have a method at that will mimic the above-explained behavior.

```Javascript
const array = [1,2,3,4,5,6]

// ✅ When used with positive index it is equal to [index]
array.at(0) // 1
array[0] // 1

// ✅ When used with negative index it mimicks the Python behaviour
array.at(-1) // 6
array.at(-2) // 5
array.at(-4) // 3
```

Btw, now that we are talking about arrays, do you know you can deconstruct the array position? 👌

```Javascript
const array = [1,2,3,4,5,6];

// ✅ Different ways of accessing the third position
const {3: third} = array; // third = 4
array.at(3) // 4
array[3] // 4
```

## Accessible Object.prototype.hasOwnProperty

The following is just a nice to have simplification. There is already the **hasOwnProperty**. However, it needs to be called within the instance of the lookup we want to do. Therefore it was common for many developers to end up doing this:

```Javascript
const x = { foo: "bar" };

// ✅ grabbing the hasOwnProperty function from prototype
const hasOwnProperty = Object.prototype.hasOwnProperty

// ✅ executing it with the x context
if (hasOwnProperty.call(x, "foo")) {
  ...
}
```

With these new specs, a hasOwn method was added to the Object prototype. Now, we can simply do:

```Javascript
const x = { foo: "bar" };

// ✅ using the new Object method
if (Object.hasOwn(x, "foo")) {
  ...
} 
```

## Error Cause

Errors help us identify and react to unexpected behavior of our application. However, it can become challenging to understand the root cause of the deeply nested errors or even to handle them properly. We lose the stack trace info when catching and rethrowing them.

There was no clear agreement on how to handle that. Given any error handling we have at least 3 choices:

```Javascript
async function fetchUserPreferences() {
  try { 
    const users = await fetch('//user/preferences')
      .catch(err => {
        // What is the best way to wrap the error?
        // 1. throw new Error('Failed to fetch preferences ' + err.message);
        // 2. const wrapErr = new Error('Failed to fetch preferences');
        //    wrapErr.cause = err;
        //    throw wrapErr;
        // 3. class CustomError extends Error {
        //      constructor(msg, cause) {
        //        super(msg);
        //        this.cause = cause;
        //      }
        //    }
        //    throw new CustomError('Failed to fetch preferences', err);
      })
    }
}

fetchUserPreferences();
```

As part of these new specs, we can construct a new error and keep the reference of the caught one. How? Just by passing the object {cause: err} to the Errorconstructor.

It all becomes simpler, standard, and easy to understand deeply nested errors. Let’s see an example:

```Javascript
async function fetcUserPreferences() {
  try { 
    const users = await fetch('//user/preferences')
      .catch(err => {
        throw new Error('Failed to fetch user preferences, {cause: err});
      })
    }
}

fetcUserPreferences();
```

## Class Fields

Before this release, there was no proper way to create private fields. There were some ways around it by using hoisting but it was not a proper private field. Now it is simple. We just need to prepend the # character to our variable declaration.

```Javascript
class Foo {
  #iteration = 0;
  
  increment() {
    this.#iteration++;
  }

  logIteration() {
    console.log(this.#iteration);
  }
}

const x = new Foo();

// ❌ Uncaught SyntaxError: Private field '#iteration' must be declared in an enclosing class
x.#iteration

// ✅ works
x.increment();

// ✅ works
x.logIteration();
```

Having private fields means we have a strong encapsulation boundary. It is not possible to access the class variable from outside it. This showcases that the class keyword is not anymore just sugar syntax.

We can also create private methods:

```Javascript
class Foo {
  #iteration = 0;
  
  #auditIncrement() {
    console.log('auditing');
  }
  
  increment() {
    this.#iteration++;
    this.#auditIncrement();
  }
}

const x = new Foo();

// ❌ Uncaught SyntaxError: Private field '#auditIncrement' must be declared in an enclosing class
x.#auditIncrement

// ✅ works
x.increment();
```

The feature is related to the Class Static Block and Ergonomic checks for Private Classes which we will see below.

## Class Static Block

As part of the new specs, we can now include static blocks inside any Class. They will run just once and are a great way to decorate or perform some per-field initialization of the static side of a class.

We are not limited to using one block, we can have as many as we need.

```Javascript
// ✅ will output 'one two three'
class A {
  static {
      console.log('one');
  }
  static {
      console.log('two');
  }
  static {
      console.log('three');
  }
}
```

They have a nice bonus. They get privileged access to private fields. You can use them to do some interesting patterns.

```Javascript
let getPrivateField;

class A {
  #privateField;
  constructor(x) {
    this.#privateField = x;
  }
  static {
    // ✅ it can access any private field
    getPrivateField = (a) => a.#privateField;
  }
}

const a = new A('foo');
// ✅ Works, foo is printed
console.log(getPrivateField(a));
```

If we tried accessing that private variable from the outside scope of the instance object we would get Cannot read private member #privateField from an object whose class did not declare it.

## Ergonomic brand checks for Private Fields

The new private fields are a great feature. However, it might become handy to check if a field is private or not in some static methods.

Trying to invoke this in outside the class scope will result in the same error we have seen previously.

```Javascript
class Foo {
  #brand;

  static isFoo(obj) {
    return #brand in obj;
  }
}

const x = new Foo();

// ✅ works, it returns true
Foo.isFoo(x);

// ✅ works, it returns false
Foo.isFoo({})

// ❌ Uncaught SyntaxError: Private field '#brand' must be declared in an enclosing class
#brand in x
```

## Final Thoughts

This is an interesting release that provides a lot of small, useful features like at, private fields and error cause. Surely error cause will bring a lot of clarity to our daily bug tracking tasks.

Some advanced features like top-level await need to be well understood prior to using them. They might have unwanted side-effect in your code execution

I hope this article got you as excited as I am about the new ES2022 specs.
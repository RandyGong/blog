---
title: ES2020 New Features
date: 2020-04-21 00:14:07
categories:
- Javascript
tags: 
- ES2020
---

Good news – the new ES2020 features are now finalised! This means we now have a complete idea of the changes happening in ES2020, the new and improved specification of JavaScript. So let's see what those changes are.

## 1: BigInt

BigInt, one of the most anticipated features in JavaScript, is finally here. It actually allows developers to have much greater integer representation in their JS code for data processing for data handling.

At the moment the maximum number you can store as an integer in JavaScript is pow(2, 53) - 1 . But BigInt actually allows you to go even beyond that.  

![BigInt](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.21.47-PM.png)

However, you need to have an n appended at the very end of the number, as you can see above. This n denotes that this is a BigInt and should be treated differently by the JavaScript engine (by the v8 engine or whatever engine it is using).

This improvement is not backwards compatible because the traditional number system is IEEE754 (which just cannot support numbers of this size).

## 2: Dynamic import

Dynamic imports in JavaScript give you the option to import JS files dynamically as modules in your application natively. This is just like how you do it with Webpack and Babel at the moment.

This feature will help you ship on-demand-request code, better known as code splitting, without the overhead of webpack or other module bundlers. You can also conditionally load code in an if-else block if you like.

The good thing is that you actually import a module, and so it never pollutes the global namespace.

![DImport](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.26.27-PM.png)

## 3: Nullish Coalescing

Nullish coalescing adds the ability to truly check nullish values instead of falsey values. What is the difference between nullish and falsey values, you might ask?

In JavaScript, a lot of values are falsey, like empty strings, the number 0, undefined, null, false, NaN, and so on.

However, a lot of times you might want to check if a variable is nullish – that is if it is either undefined or null, like when it's okay for a variable to have an empty string, or even a false value.

In that case, you'll use the new nullish coalescing operator, ??

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.47.03-PM.png)

## 4: Optional Chaining

Optional chaining syntax allows you to access deeply nested object properties without worrying if the property exists or not. If it exists, great! If not, undefined will be returned.

This not only works on object properties, but also on function calls and arrays. Super convenient! Here's an example:

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.51.58-PM.png)

## 5: Promise.allSettled

The Promise.allSettled method accepts an array of Promises and only resolves when all of them are settled – either resolved or rejected.

This was not available natively before, even though some close implementations like race and all were available. This brings "Just run all promises – I don't care about the results" natively to JavaScript.

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.54.58-PM.png)

## 6: String#matchAll

matchAll is a new method added to the String prototype which is related to Regular Expressions. This returns an iterator which returns all matched groups one after another. Let's have a look at a quick example:

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-8.59.14-PM.png)

## 7: globalThis

If you wrote some cross-platform JS code which could run on Node, in the browser environment, and also inside web-workers, you'd have a hard time getting hold of the global object.

This is because it is window for browsers, global for Node, and self for web workers. If there are more runtimes, the global object will be different for them as well.

So you would have had to have your own implementation of detecting runtime and then using the correct global – that is, until now.

ES2020 brings us globalThis which always refers to the global object, no matter where you are executing your code:

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-03-at-9.02.27-PM.png)

## 8: Module Namespace Exports

In JavaScript modules, it was already possible to use the following syntax:

```JavaScript
import * as utils from './utils.mjs'
```

However, no symmetric export syntax existed, until now:

```JavaScript
export * as utils from './utils.mjs'
```

This is equivalent to the following:
```JavaScript
import * as utils from './utils.mjs'
export { utils }
```

## 9: Well defined for-in order

The ECMA specification did not specify in which order for (x in y)  should run. Even though browsers implemented a consistent order on their own before now, this has been officially standardized in ES2020.

## 10: import.meta

The import.meta object was created by the ECMAScript implementation, with a null prototype.

Consider a module, module.js:
```JavaScript
<script type="module" src="module.js"></script>
```

You can access meta information about the module using the `import.meta` object:
```JavaScript
console.log(import.meta); // { url: "file:///home/user/module.js" }
```

It returns an object with a url property indicating the base URL of the module. This will either be the URL from which the script was obtained (for external scripts), or the document base URL of the containing document (for inline scripts).

## Conclusion

I love the consistency and speed with which the JavaScript community has evolved and is evolving. It is amazing and truly wonderful to see how JavaScript came from a language which was booed on, 10 years go, to one of the strongest, most flexible and versatile language of all time today.

Do you wish to learn JavaScript and other programming languages in a completely new way? Head on to a new platform for developers I'm working on to try it out today!
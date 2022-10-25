---
title: +0 and -0 in Javascript
date: 2022-08-29 23:07:17
tags:
- Javascript
---

When I read the isEqual function of the [underscore source code][1], I encountered an interesting problem -- that is, Javascript has two 0: +0 and -0.

You must wonder why there are + 0 and - 0, this is because JavaScript adopts IEEE_754 floating point notation (which is used by almost all modern programming languages). This is a binary representation. According to this standard, the highest bit is the sign bit (0 represents positive, 1 represents negative), and the rest represents the value. For the boundary value of 0, both 1000 (- 0) and 0000 (0) represent 0, which leads to + 0 and - 0.

When will - 0 be generated?

```Javascript
Math.round(-0.1)  // -0
```

JavaScript deliberately wants to smooth out the difference between the two.

```Javascript
// reflection 1
console.log(+0 === -0)  // true

// reflection 2
(-0).toString()   // '0'
(+0).toString()   // '0'

// reflection 3
-0 < +0   // false
+0 < -0   // false
```

Even so, the two are still different.

```Javascript
1 / +0  // Infinity
1 / -0  // -Infinity

1 / +0 === 1 / -0   // false
```

How can we distinguish between + 0 and - 0 when the result of = = = is true, we can do like this:

```Javascript
function isEqual(a, b) {
    if (a === b) return a !== 0 || 1 / a === 1 / b;
    return false;
}

console.log(isEqual(0, 0))  // true
console.log(isEqual(0, -0))  // false
```

This is also how does underscore do:
```Javascript

// first few lines of the isEqual function of underscore

// Internal recursive comparison function for `isEqual`.
  var eq = function(a, b, aStack, bStack) {
    // Identical objects are equal. `0 === -0`, but they aren't identical.
    // See the [Harmony `egal` proposal](http://wiki.ecmascript.org/doku.php?id=harmony:egal).
    if (a === b) return a !== 0 || 1 / a === 1 / b;
```

  [1]: https://github.com/lessfish/underscore-analysis/blob/master/underscore-1.8.3.js/src/underscore-1.8.3.js#L1094-L1190
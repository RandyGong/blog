---
title: Javascript Object
date: 2022-01-24 18:42:10
tags:
---

https://www.bilibili.com/video/BV1zt4y1Y7F5?p=3&spm_id_from=pageDriver

## Object.assign

```Javascript
const obj1 = {
    a: 1,
    b: 2
}

const obj2 = {
    b: 3,
    c: 4
}

const obj3 = {
    c: 5,
    d: 6
}

const obj4 = Object.assign(obj1, obj2, obj3);

console.log(obj1)        // Object { a: 1, b: 3, c: 5, d: 6 }
console.log(obj2)        // Object { b: 3, c: 4 } 
console.log(obj3)        // Object { c: 5, d: 6 }
console.log(obj4)        // Object { a: 1, b: 3, c: 5, d: 6 }

```

```Javascript
Object.myAssign = function(target, ...sources) {
    sources.forEach(source => {
        const descriptors = Object.keys(sources).reduce((descriptor, key) => {
            descriptor[key] = Object.getOwnPropertyDescriptor(source, key)
            return descriptor
        }, {})
        
        Object.defineProperties(target, descriptors)
    })
    
    return target
}

```

## Object.create

```Javascript
let obj = Object.create(null)      // this will create an object with no prototype
console.log(obj)         // Object {  }

obj.a = 1;
obj.b = 2;
console.log(obj)         // Object { a: 1, b: 2 }

obj = {
    a: 1,
    b: 2
}
console.log(obj)        // Object { a: 1, b: 2 }
                            a: 1
                            b: 2
                            <prototype>: Object { … }   // the prototype comes back, as = {} will create obj by new Object()
                            
// so we use Test.Prototype.a = function() {}, but not
// Test.Prototype = { a: function(){} } in order not to change the original prototype

```

```Javascript
// the first argument defines the prototype, but it also has its prototype
let obj5 = Object.create({a: 1})
console.log(obj5)       // Object {  }
                          <prototype>: Object { a: 1 }
                            <prototype>: Object { … }
     
// the second argument defines the object property, but needs to be in the following object format                       
let obj6 = Object.create({a: 1}, {
    b: {
        value: 2,
        configurable: true,
        writable: true,
        enumerable: true
        // get
        // set
    }
})
console.log(obj6)    //  Object { b: 2 }
                        b: 2
                            <prototype>: Object { a: 1 }
                            a: 1
                                <prototype>: Object { … }
```

```Javascript
let obj7 = Object.create({a: 1}, {
    b: {
        value: 2,
        configurable: true
    },
    c: {
        value: 3,
        enumerable: true,
        configurable: false
    }
})

let obj8 = Object.assign({}, obj7)
console.log(obj8)         // Object { c: 3 }   only enumerable properties can be assigned

for (let p in obj7) {
    console.log(p, obj[p])           // c 3, a 1         forin will enumerate all properties including which are in prototype
}

Object.entries(obj7)      // 0: Array [ "c", 3 ]      Object.entries will not enumerate properties in prototype

delete obj8.c
console.log(obj8)         // Object {  }    Although c in obj7 is un-configurable, but when assigning, it property will be just brought to obj8, regardless the descriptor
```

https://medium.com/technofunnel/deep-and-shallow-copy-in-javascript-110f395330c5
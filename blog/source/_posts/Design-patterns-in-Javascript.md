---
title: Design patterns in Javascript
date: 2021-12-26 18:37:14
categories:
- Design patterns
tags:
- Design patterns
---

https://www.bilibili.com/video/BV1PQ4y1i7t7?p=28

## Factory

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

## Builder pattern

The builder pattern is a creational pattern that is used to create more complex objects than you'd expect from a factory

![image.png-161.7kB][1]

```Javascript
class Builder implements IBuiler {
    product: Product;
    
    constructor() {
        this.product = new Product();
    }
    
    buildPartA() {
        this.product.part.push('a');
        return this;
    }
    
    buildPartB() {
        this.product.part.push('b');
        return this;
    }
    
    buildPartC() {
        this.product.part.push('c');
        return this;
    }
    
    getProduct() {
        return this.product;
    }
}

// this class doesn't care about how the product is build
class Director {
    statuc construct(): Product {
        return new Builder().buildPartA().buildPartB().buildPartC().getProduct();
    }
}
```


## Prototype

The prototype design pattern is good for when creating new objects requires more resources than you want to use or have available. You can save resources by just creating a copy of any existing object that is already in the memory.

**Shallow copy:**
will create new copies of objects with new references in memory, but the underlying data, e.g., the actual elements in an array, will point to the same memory location as the orginal array/object being copied. You will now have 2 arrays, but the elements within the arrays will point to the same memory location. So changing any elements of a copied array will affect the original array.

```Javascript
interface IPrototype {
    // interface with clone method
    clone(): this;
    // The clone is deep or shallow, it up to you how you want to implement
}

class MyClass implements IPrototype {
    field: number[];
    
    constructor(field: number[]) {
        this.field = field;
    }
    
    clone() {
        return Object.assign({}, this);  // shallow copy
        return JSON.parse(JSON.stringify(this));  // deep copy
    }
}

const obj1 = new MyClass([1,2,3]);
const obj2 = obj1.clone();

// change the value of element in obj2, depending on clone method, shallow or deep copy
obj2.field[1] = 100;

console.log(obj1);
console.log(obj2);
```

## Decorator
the decorator pattern is a structural pattern, that allows you to attach additional responsibilities to an object at runtime.

```Javascript
interface IComponent {
    method(): string;
}

class Component implements IComponent {
    method() {
        return 'Component method';
    }
}

class Decorator implements IComponent {
    #object: IComponent;
    
    constructor(object: IComponent) {
        this.#object = object;
    }
    
    method() {
        return `Decorator method of ${this.#object.method()}`;   
    }
}
```

## Adapter

```Javascript
interface IA {
    methodA(): void;
}

class A imeplements IA {
    methodA() {
        console.log('methodA');
    }
}

interface IB {
    methodB(): void;
}

class B imeplements IB {
    methodB() {
        console.log('methodB');
    }
}

class AdapterB imeplements IB {
    #classB;

    constructor() {
        this.#classB = new B();
    }
    
    methodA() {
        this.#classB.methodB();
    }
}

// with no adapter, I have to check the object class to know which method to call
const data = [new A(), new B()];
data.forEach(x => {
    if (x instanceof A) {
        x.methodA();
    } else {
        x.methodB();
    }
});

// after creating adapter for B, I can reuse the same method signature as A 
const adapteredData = [new A(), new AdapterB()];
adapteredData.forEach(x => {
    x.methodA();
});
```

## Facade

The Facade pattern essentially is an alternative, reduced or simplified interface to a set of other interfaces, abstractions and implementations within a system that may be full of complexity and / or tightly coupled.

It can also be considered as a higher-level interface that shields the consumer from the unnecessary low-level complications of integrating into many subsystems.

把复杂的接口做简化之用

Abstract Factory and Facade can be considered very similiar. An Abstract Factory is about creating an interface over several creational classes of similiar objects, whereas the Facade is more like an API layer over many creational, structural and/or behavioral patterns.

```Javascript
class SubSystemA() {
    method(): string {
        return 'A';
    }
}

class SubSystemB() {
    method(value: string): string {
        return value;
    }
}

class SubSystemC {
    method(value: {C: number[]}): {C: number[]} {
        return value;
    }
}

// work as a middle man
class Facade {
    subSystemA(): string {
        return new SubSystemA().method();
    }
    
    subSystemB(value: string): string {
        return new SubSystemB(value).method();
    }
    
    subSystemC(value: {C: number[]}): {C: number[]} {
        return new SubSystemC().method(value);
    }
}

// The client
// Calling potentially complited subsystems directly
new SubSystemA().method();
new SubSystemB().method('B');
new SubSystemC().method({C: [1,2,3]});

// or using simplified facade instead
const facade = new Facade();
facade.subSystemA();
facade.subSystemB('B');
facade.subSystemC({C: [1,2,3]});
```

## Bridge

Use when you want to separate a solution where the abstraction and implementation may be tightly coupled and you want to break it up into smaller conceptual parts.

```Javascript
interface IAbstraction {
    method(value: string[]): void;
}

interface IImplementer {
    method(value: string[]): void;
}

class RefinedAbstraction implements IAbstraction {
    #implementer: IImeplementer;
    
    constructor(implementer: IImplementer) {
        this.#implementer = implementer;
    }
    
    method(value: string[]) {
        this.#implementer.method(value);
    }
}

class ConcreateImplementerA implements IImplementer {
    method(value: string[]) {
        console.log(value);
    }
}

class ConcreateImplementerB implements IImplementer {
    method(value: string[]) {
        value.forEach(v => console.log(v););
    }
}

// The client
const values = ['a', 'b', 'c'];

const refinedAbstractionA = new RefinedAbstraction(new ConcreteImplementerA());
refinedAbstractionA.method(values);

const refinedAbstractionB = new RefinedAbstraction(new ConcreteImplementerB());
refinedAbstractionB.method(values);
```

## Composite

```Javascript
interface ICompositeComponent {
    // A component interface describing the common fields and methods of leaves and composites
    name: string;
    referenceToParent?: Composite;
    method(): void; // A method each leaf and composite container should implement
    detach(): void; // called before a leaf is attached to a composite
}

class Lead implements ICompositeComponent {
    // a leaf can be added to a composite, but not a leaf
    referenceToParent? : Composite = undefined;
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class Composite implements ICompositeComponent {
    // a composite can contain leaves and composites
    referenceToParent? : Composite;
    components: ICompositeComponent[];
    name: string;
    constructor(name: string) {
        this.name = name;
        this.components = [];
    }
    
    method(): void {
        const parent = this.referenceToParent ? this.referenceToParent.name : 'none';
        this.components.forEach(component => {
            component.method();
        });
    }
    
    attach(component: ICompositeComponent): void {
        component.detach();
        component.referenceToParent = this;
        this.components.push(component);
    }
    
    detach(): void {
        // detaching this composite from its parent compo 
    }
}

// the client
const leafA = new Leaf('leaf-a');
const leafB = new Leaf('leaf-b');
const composite1 = new Composite('comp-1');
const composite2 = new Composite('comp-2');

composite1.attach(leafA);
composite2.attach(leafA);
composite2.attach(composite1);

leafB.method();  // not in any composites
composite2.method();  // composite2 contains both composite1 and leadA
```

  [1]: http://static.zybuluo.com/RandyGong/mzu6h870jezjjzjn2vl9tjvr/image.png
---
title: Write better conditional statements using JavaScript
date: 2022-05-17 23:03:34
categories:
- Javascript
tags:
---

In any programming language, the code needs to make different decisions and perform corresponding actions in a given input according to different conditions.

For example, in a game, if the player's life point is 0, the game ends. In the weather application, if it is viewed in the morning, a sunrise picture is displayed; if it is at night, stars and the moon are displayed. In this article, we will explore how so-called conditional statements in JavaScript work.

If you work with JavaScript, you will write a lot of code that contains conditional calls. Conditional invocation may be easy to learn at first, but there is more to it than writing pairs of if/else. Here are some useful tips for writing better and clearer conditional code.

- Array method **Array.includes**
- Early exit / early return
- Replace switch statement with object literal or map
- Default parameters and Deconstruction
- Use **Array.every** & **Array.some** matches all / part of the content
- Merge with optional chains and null values

## Array.includes

Use **Array.includes** for multi condition selection

```Javascript
function printAnimals(animal) {
    if (animal === 'dog' || animal === 'cat') {
        console.log(I have a ${animal});
    }
}

console.log(printAnimals('dog')); // I have a dog
```

The code above looks good because we only checked two animals. However, we are not sure about user input. What if we want to check any other animals? If we expand by adding more or statements, the code will become difficult to maintain and unclear.

Solution:

We can use **Array.includes** to override the above condition

```Javascript
function printAnimals(animal) {
   const animals = ['dog', 'cat', 'hamster', 'turtle']; 

   if (animals.includes(animal)) {
     console.log(I have a ${animal});
   }
}

console.log(printAnimals('hamster')); // I have a hamster
```

Here, we create an animal array, so the conditional statement can be abstracted from the rest of the code. Now, if we want to check any other animals, we just need to add a new array item.

We can also use this animal array variable outside the scope of this function to reuse it anywhere else in the code. This is a way to write clearer, easier to understand and maintain code, isn't it?

## Early exit / early return

This is a cool technique for streamlining your code. I remember when I started my professional work, I learned to use early withdrawal to write conditions on the first day.

Let's add more conditions to the previous example. An animal that replaces a simple string with an object that contains certain attributes.

The current demand is:

If there are no animals, throw an exception
Print animal types
Print animal name
Print animal sex

```Javascript
const printAnimalDetails = animal => {
  let result; // declare a variable to store the final value

  // condition 1: check if animal has a value
  if (animal) {

    // condition 2: check if animal has a type property
    if (animal.type) {

      // condition 3: check if animal has a name property
      if (animal.name) {

        // condition 4: check if animal has a gender property
        if (animal.gender) {
          result = ${animal.name} is a ${animal.gender} ${animal.type};;
        } else {
          result = "No animal gender";
        }
      } else {
        result = "No animal name";
      }
    } else {
      result = "No animal type";
    }
  } else {
    result = "No animal";
  }

  return result;
};

console.log(printAnimalDetails()); // 'No animal'

console.log(printAnimalDetails({ type: "dog", gender: "female" })); // 'No animal name'

console.log(printAnimalDetails({ type: "dog", name: "Lucy" })); // 'No animal gender'

console.log(
  printAnimalDetails({ type: "dog", name: "Lucy", gender: "female" })
); // 'Lucy is a female dog'
```

What do you think of the above code?

It works well, but the code is long and difficult to maintain. If you don't use the lint tool, it will waste a lot of time to find out where the closed curly braces are. 😄  Imagine what would happen if the code had more complex logic? A large number of if..else statements.

We can reconstruct the above functions with syntax such as ternary operators and && conditions, but let's write clearer code with multiple return statements.

```Javascript
const printAnimalDetails = ({type, name, gender } = {}) => {
  if(!type) return 'No animal type';
  if(!name) return 'No animal name';
  if(!gender) return 'No animal gender';

// Now in this line of code, we're sure that we have an animal with all //the three properties here.

  return ${name} is a ${gender} ${type};
}

console.log(printAnimalDetails()); // 'No animal type'

console.log(printAnimalDetails({ type: dog })); // 'No animal name'

console.log(printAnimalDetails({ type: dog, gender: female })); // 'No animal name'

console.log(printAnimalDetails({ type: dog, name: 'Lucy', gender: 'female' })); // 'Lucy is a female dog'
```

In this refactored version, deconstruction and default parameters are also included. The default parameter ensures that if we pass undefined as a method parameter, we still have values to deconstruct. Here it is an empty object {}.

Usually, in the professional field, the code is written between the two methods.

Another example:

```Javascript
function printVegetablesWithQuantity(vegetable, quantity) {
  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

  // condition 1: vegetable should be present
   if (vegetable) {
     // condition 2: must be one of the item from the list
     if (vegetables.includes(vegetable)) {
       console.log(I like ${vegetable});

       // condition 3: must be large quantity
       if (quantity >= 10) {
         console.log('I have bought a large quantity');
       }
     }
   } else {
     throw new Error('No vegetable from the list!');
   }
}

printVegetablesWithQuantity(null); //  No vegetable from the list!
printVegetablesWithQuantity('cabbage'); // I like cabbage
printVegetablesWithQuantity('cabbage', 20); 
// 'I like cabbage
// 'I have bought a large quantity'
```

Now, we have:

1 if/else statement filters illegal conditions
Level 3 nested if statements (conditions 1, 2, 3)

A commonly followed rule is to exit early when illegal conditions match.

```Javascript
function printVegetablesWithQuantity(vegetable, quantity) {

  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

   // condition 1: throw error early
   if (!vegetable) throw new Error('No vegetable from the list!');

   // condition 2: must be in the list
   if (vegetables.includes(vegetable)) {
      console.log(I like ${vegetable});

     // condition 3: must be a large quantity
      if (quantity >= 10) {
        console.log('I have bought a large quantity');
      }
   }
}
```

By doing so, we have lost one nesting level. This code style is especially good when you have a long if statement.

We can further reduce nested if statements through conditional inversion and early return. Check condition 2 below to see how we do it

```Javascript
function printVegetablesWithQuantity(vegetable, quantity) {

  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

   if (!vegetable) throw new Error('No vegetable from the list!'); 
   // condition 1: throw error early

   if (!vegetables.includes(vegetable)) return; 
   // condition 2: return from the function is the vegetable is not in 
  //  the list 


  console.log(I like ${vegetable});

  // condition 3: must be a large quantity
  if (quantity >= 10) {
      console.log('I have bought a large quantity');
  }
}
```

By inverting condition 2, the code has no nested statements. This technique is especially useful when we have many conditions and we want to stop further processing when any specific conditions do not match.

Therefore, always focus on less nesting and early return, but don't overuse it.

## Replace switch statement with object literal or map

Let's take a look at the following example. We want to print fruit based on Color:

```Javascript
function printFruits(color) {
  // use switch case to find fruits by color
  switch (color) {
    case 'red':
      return ['apple', 'strawberry'];
    case 'yellow':
      return ['banana', 'pineapple'];
    case 'purple':
      return ['grape', 'plum'];
    default:
      return [];
  }
}

printFruits(null); // []
printFruits('yellow'); // ['banana', 'pineapple']
```

The above code has no errors, but it is still a little verbose. The same function can be realized with object literals in clearer syntax:

```Javascript
// use object literal to find fruits by color
  const fruitColor = {
    red: ['apple', 'strawberry'],
    yellow: ['banana', 'pineapple'],
    purple: ['grape', 'plum']
  };

function printFruits(color) {
  return fruitColor[color] || [];
}
```

In addition, you can also use map to achieve the same function:

```Javascript
// use Map to find fruits by color
  const fruitColor = new Map()
    .set('red', ['apple', 'strawberry'])
    .set('yellow', ['banana', 'pineapple'])
    .set('purple', ['grape', 'plum']);

function printFruits(color) {
  return fruitColor.get(color) || [];
}
```

Map allows to save key value pairs, which is an object type that can be used since es2015.

For the above example, the same function can also use the array method array Filter.

```Javascript
const fruits = [
    { name: 'apple', color: 'red' }, 
    { name: 'strawberry', color: 'red' }, 
    { name: 'banana', color: 'yellow' }, 
    { name: 'pineapple', color: 'yellow' }, 
    { name: 'grape', color: 'purple' }, 
    { name: 'plum', color: 'purple' }
];

function printFruits(color) {
  return fruits.filter(fruit => fruit.color === color);
}
```

## Default parameters and Deconstruction

When working with JavaScript, we always need to check the null/undefined value and assign the default value, otherwise the compilation may fail.

```Javascript
function printVegetablesWithQuantity(vegetable, quantity = 1) { 
// if quantity has no value, assign 1

  if (!vegetable) return;
    console.log(We have ${quantity} ${vegetable}!);
}

//results
printVegetablesWithQuantity('cabbage'); // We have 1 cabbage!
printVegetablesWithQuantity('potato', 2); // We have 2 potato!
```

What if vegetable is an object? Can we assign a default parameter?

```Javascript
function printVegetableName(vegetable) { 
    if (vegetable && vegetable.name) {
     console.log (vegetable.name);
   } else {
    console.log('unknown');
   }
}

printVegetableName(undefined); // unknown
printVegetableName({}); // unknown
printVegetableName({ name: 'cabbage', quantity: 2 }); // cabbage
```

In the above example, if vegetable exists, we want to print vegetable name, otherwise print "unknown".

We can avoid conditional statements if (vegetable && vegetable.name) {} by using default parameters and deconstruction.

```Javascript
// destructing - get name property only
// assign default empty object {}

function printVegetableName({name} = {}) {
   console.log (name || 'unknown');
}


printVegetableName(undefined); // unknown
printVegetableName({ }); // unknown
printVegetableName({ name: 'cabbage', quantity: 2 }); // cabbage
```

Because we only need the name attribute, we can use {name} to deconstruct the parameter, and then we can use name as a variable in the code instead of vegetable name.

We also assign an empty object {} as the default value, because when executing printvegetablename (undefined), we will get an error: the attribute name cannot be deconstructed from undefined or null, because there is no name attribute in undefined.

## Use Array.every && Array.some matches all / part of the content

We can use array method to reduce code lines. Looking at the following code, we want to check whether all fruits are red:

```Javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  let isAllRed = true;

  // condition: all fruits must be red
  for (let f of fruits) {
    if (!isAllRed) break;
    isAllRed = (f.color == 'red');
  }

  console.log(isAllRed); // false
}
```

This code is too long! We can use **Array.every** to reduce the number of lines of code:

```Javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  // condition: short way, all fruits must be red
  const isAllRed = fruits.every(f => f.color == 'red');

  console.log(isAllRed); // false
}
```

Similarly, if we want to test whether there are any red fruits, we can use a row of **Array.some** to implement it.

```Javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
];

function test() {
  // condition: if any fruit is red
  const isAnyRed = fruits.some(f => f.color == 'red');

  console.log(isAnyRed); // true
}
```

## Merge with optional chains and null values

There are two features that will soon become JavaScript enhancements for writing clearer conditional statements. When writing this article, they are not fully supported. You need to compile them with Babel.

Optional chains allow us to deal with tree-like structures without explicitly checking whether intermediate nodes exist. Null value merging and optional chains work well together to ensure a default value is assigned to non-existent values.

Here is an example:

```Javascript
const car = {
    model: 'Fiesta',
    manufacturer: {
    name: 'Ford',
    address: {
        street: 'Some Street Name',
        number: '5555',
        state: 'USA'
      }
    }
} 

// to get the car model
const model = car && car.model || 'default model';

// to get the manufacturer street
const street = car && car.manufacturer && car.manufacturer.address && 
car.manufacturer.address.street || 'default street';

// request an un-existing property
const phoneNumber = car && car.manufacturer && car.manufacturer.address 
&& car.manufacturer.phoneNumber;

console.log(model) // 'Fiesta'
console.log(street) // 'Some Street Name'
console.log(phoneNumber) // undefined
```

So if we want to print whether the vehicle manufacturer is from the United States, the code will look like this:

```Javascript
const isManufacturerFromUSA = () => {
   if(car && car.manufacturer && car.manufacturer.address && 
 car.manufacturer.address.state === 'USA') {
     console.log('true');
   }
}


checkCarManufacturerState() // 'true'
```

You can clearly see how messy this can become when there is a more complex object structure. Some third-party libraries have their own functions, such as lodash or idx. For example, lodash has **_get** method. However, it is very cool that JavaScript language itself is introduced into this feature.

This shows how these new features work:

```Javascript
// to get the car model
const model = car?.model ?? 'default model';

// to get the manufacturer street
const street = car?.manufacturer?.address?.street ?? 'default street';

// to check if the car manufacturer is from the USA
const isManufacturerFromUSA = () => {
   if(car?.manufacturer?.address?.state === 'USA') {
     console.log('true');
   }
}
```
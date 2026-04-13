# Object Literals & Property Access

> **AI boundaries this week:** 70% manual / 30% AI. Habit: *AI as code reviewer -- paste your working code, ask for a critique, decide what to accept.* See [ai.md](../ai.md).

## Learning Objectives
By the end of this lesson, you will be able to:
- Create JavaScript objects using object literal syntax.
- Read, add, update, and remove properties using dot notation and bracket notation.
- Understand the difference between primitive values and reference types.
- Use modern ES6 features like shorthand property names and method shorthand.
- Inspect and iterate over the contents of an object using built-in Object methods.

---

## 1. What Is an Object in JavaScript?

An object is a collection of related data and functionality. It stores data as an unordered list of **key-value pairs**. 

- **Key**: A string (or occasionally a Symbol) that acts as the name for a piece of data. Also called a "property name".
- **Value**: The actual data. This can be any JavaScript data type - a string, a number, a boolean, an array, a function, or even another object.

### The Dictionary Analogy
Think of an object like a dictionary. When you look up a word (the key), you find its definition (the value). In an object, you look up a property name to get the data associated with it.

### Why Use Objects?
Objects are the foundation for modeling complex real-world concepts in code:
- **Grouping related data**: Instead of having separate variables for `userName`, `userAge`, and `userEmail`, you group them into a single `user` object.
- **Namespacing functionality**: Grouping related functions inside a single object to keep the global scope clean.

---

## 2. Object Literal Syntax

The simplest and most common way to create an object is using the **object literal** syntax with curly braces `{}`.

### Basic Syntax

```javascript
const user = {
  name: 'Alice',            // string value
  age: 30,                  // number value
  isStudent: false,         // boolean value
  location: ['New York'],   // array value
  greet: function() {       // function value (called a method)
    console.log(`Hi, I am ${this.name}`);
  }
};

user.greet(); // Output: "Hi, I am Alice"
```

### 2.1 Shorthand Property Names

In modern JavaScript (ES6+), if you are creating an object and the variable names holding your data exactly match the property names you want to create, you can omit the value assignment.

```javascript
// Variables holding our data
const name = 'Bob';
const age = 25;

// Shorthand syntax
const user = { name, age }; 

// This is exactly equivalent to writing:
// const user = { name: name, age: age };

console.log(user); 
// Output: { name: 'Bob', age: 25 }
```

### 2.2 Method Shorthand

When a property's value is a function, it is called a **method**. You can define methods inside an object literal using a more concise syntax that omits the colon and the word `function`.

```javascript
const counter = {
  count: 0,
  
  // Modern method shorthand:
  increment() { 
    this.count++;
  },
  
  // Instead of the older syntax:
  // increment: function() { this.count++; }
};

counter.increment();
console.log(counter.count); // Output: 1
```

---

## 3. Accessing Properties

Once an object is created, you need ways to read and modify its contents. There are two ways to do this: Dot Notation and Bracket Notation.

### 3.1 Dot Notation

Dot notation is the most common and readable way to access object properties. 

**When to use it**: Use dot notation when you know the exact name of the property at the time you write the code, and the property name is a valid JavaScript identifier (e.g., it contains no spaces or hyphens).

```javascript
const car = { make: 'Toyota', model: 'Corolla' };

// Reading a property
console.log(car.make);      // "Toyota"

// Updating an existing property
car.model = 'Camry';

// Adding a completely new property
car.year = 2024;

// Removing a property completely
delete car.make; 

console.log(car); 
// Output: { model: 'Camry', year: 2024 }
```

### 3.2 Bracket Notation

Bracket notation requires you to pass the property name as a string inside square brackets `[]`.

**When to use it**: You must use bracket notation in two specific scenarios:
1. When the property name contains special characters, spaces, or starts with a number.
2. When the property name is **dynamic** (stored inside a variable).

```javascript
const user = { 
  name: 'Alice', 
  'favorite color': 'blue' // Note the string key due to the space
};

// 1. Invalid identifiers (spaces, hyphens)
// console.log(user.favorite color); // SyntaxError!
console.log(user['favorite color']); // "blue"

// 2. Dynamic keys via variables
const propertyToLookUp = 'name';

// This looks for a property named 'propertyToLookUp' (which is undefined)
console.log(user.propertyToLookUp); 

// This evaluates the variable first, finding 'name', then looks up user['name']
console.log(user[propertyToLookUp]); // "Alice"
```

### 3.3 Computed Property Names

You can also use square brackets inside the object literal itself to set dynamic keys at the moment the object is created.

```javascript
const dynamicKey = 'role';

const admin = {
  username: 'superAdmin',
  [dynamicKey]: 'administrator' // Evaluates the variable before assigning
};

console.log(admin.role); // "administrator"
```

---

## 4. Objects Are Reference Types

This is one of the most critical concepts in JavaScript.

Primitive data types (strings, numbers, booleans) are passed by **value**. If you assign them to a new variable, you create an independent copy of the data.

Objects (including arrays and functions) are **reference types**. When you assign an object to a variable, the variable does not store the object itself. Instead, it stores a **reference** (a memory address) pointing to the location where the object lives.

### The House and Address Analogy
Think of the object as a physical house. The variable does not contain the house; it contains a piece of paper with the address of the house.
If you copy the variable, you are just copying the piece of paper. Both pieces of paper still point to the exact same house. If someone paints the house using the first address, the person looking at the second address will see a painted house.

### Demonstration

```javascript
const original = { x: 1 };
const alias = original; // Copies the REFERENCE, not the object

alias.x = 42; // Mutating the object using the alias

console.log(original.x); // Output: 42
// The original variable also sees the change because there is only one object!
```

### Copying Objects

To avoid accidentally modifying shared data, you must explicitly copy the object. The modern way to do this is using the **spread operator** (`...`).

```javascript
const data = { count: 10 };

// Create a completely new object, spreading the old properties into it
const clone = { ...data }; 

clone.count = 99;

console.log(data.count);  // Output: 10
console.log(clone.count); // Output: 99
```

### Shallow vs. Deep Cloning

The spread operator only creates a **shallow clone**. It copies the top-level properties. If a property is a nested object, the clone will still share a reference to that inner object. Handling nested copies requires a **deep clone** (often done using `structuredClone()` in modern JavaScript or libraries like Lodash).

---

## 5. Inspecting & Iterating Properties

JavaScript provides three built-in methods on the global `Object` constructor to help you inspect and loop through an object's contents. Each of these methods takes the object as an argument and returns an array.

### 5.1 `Object.keys(obj)`
Returns an array of all the property names (keys) inside the object.

```javascript
const profile = { name: 'Sam', age: 28, city: 'London' };

const keys = Object.keys(profile);
console.log(keys); 
// Output: ['name', 'age', 'city']
```

### 5.2 `Object.values(obj)`
Returns an array of all the values inside the object.

```javascript
const values = Object.values(profile);
console.log(values); 
// Output: ['Sam', 28, 'London']
```

### 5.3 `Object.entries(obj)`
Returns an array where each item is a two-element array. The first element is the key, and the second is the value: `[key, value]`. This is incredibly useful for looping over both keys and values simultaneously.

```javascript
const entries = Object.entries(profile);
console.log(entries);
// Output: [ ['name', 'Sam'], ['age', 28], ['city', 'London'] ]

// Iterating cleanly using for...of and array destructuring
for (const [key, value] of Object.entries(profile)) {
  console.log(`The ${key} is ${value}`);
}
// Outputs: 
// The name is Sam
// The age is 28
// The city is London
```

---

## 6. In-Class Activity: Product Catalog

Work through these steps to practice object creation, iteration, and manipulation.

### Step 1: Define the Data
Create an array of product objects:

```javascript
const products = [
  { id: 1, name: 'Laptop', price: 1200 },
  { id: 2, name: 'Phone',  price: 800  },
  { id: 3, name: 'Tablet', price: 600  }
];
```

### Step 2: Read Properties
Loop over the array and log each product's name and price using dot notation.

```javascript
products.forEach(p => {
  console.log(`${p.name}: $${p.price}`);
});
```

### Step 3: Dynamic Updates
Write a function that accepts an array of product objects and adds a new `inStock` property set to `true` for every product.

### Step 4: Inspect
Pick the first product in the list and use `Object.keys()` to print out an array of all its current property names.

### Step 5: Experiment with References
Set `products[0] = {}`. Check what happens to the array. 
Reload your code, and instead do `const item = products[0]; item.price = 0;`. Observe how references behave when mutating properties versus reassigning the variable.

---

## 7. Assignment: Most Expensive Finder

### Task overview
Write a function `findMostExpensive(items)` that takes an array of product objects and returns the single object that has the highest price.

### Requirements:
1. The function must expect an array of objects where each object has a numeric `price` property.
2. If the array is empty, the function must return `null`.
3. Do not use the built-in `Array.prototype.sort()` method.
4. Iterate through the items using a `for...of` loop to compare prices and track the winner.

### Standard Implementation

```javascript
function findMostExpensive(items) {
  if (items.length === 0) return null;

  let winner = items[0];
  
  for (const item of items) {
    if (item.price > winner.price) {
      winner = item; // Keep the reference to the more expensive object
    }
  }
  
  return winner;
}

const inventory = [
  { name: 'Mouse', price: 25 },
  { name: 'Monitor', price: 300 },
  { name: 'Keyboard', price: 75 }
];

console.log(findMostExpensive(inventory)); 
// Output: { name: 'Monitor', price: 300 }
```

### Stretch Goals:

1. **Dynamic Property Tracking**: Modify the function signature to accept a second parameter that specifies the key to compare, falling back to 'price' as the default.
   ```javascript
   function findMostExpensive(items, priceKey = 'price') {
     // Implementation here using bracket notation
   }
   ```
2. **Detailed Logging**: Once the function returns the most expensive item, use `Object.entries()` to print a nicely formatted list of all that item's details to the console.

---

By mastering object literal syntax, property access patterns, and object inspection tools, you will be well-prepared to tackle constructor functions, ES6 classes, and the prototype chain.
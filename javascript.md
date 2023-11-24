```table-of-contents
style: nestedList # TOC style (nestedList|inlineFirstLevel)
maxLevel: 0 # Include headings up to the speficied level
includeLinks: true # Make headings clickable
debugInConsole: false # Print debug info in Obsidian console
```

# 1. String interpolation

```javascript
console.log(`${book.author} wrote ${book.title}`);
console.log(` I have ${book.isRead ? "read" : "not read"} this book`);
```

# 2. For-loop

```javascript
for (const something of everything) {
    ...
}
```

# 3. let vs var vs const

## 3.1. Variable hoisting

Hoisting is a JavaScript mechanism where variables and function declarations are moved to the top of their scope before code execution.

This:

```javascript
console.log(greeter);
var greeter = "say hello"
```

is interpreted as this:

```javascript
var greeter;
console.log(greeter); // greeter is undefined
greeter = "say hello"
```

## 3.2. var

```var``` is function-scoped when defined in a function and globally scoped when defined outside.

```javascript
var tester = "hey hi";

function newFunction() {
    var hello = "hello";
}
console.log(hello); // error: hello is not defined
```

However, there is a caveat of bleeding outside of the scoped where the variable was declared:

```javascript
function baz() {
    if (true) {
        var a = 'lol';
        console.log(a);
    }
    console.log(a);
}

baz(); // outputs lollol
```

```var``` variables can be re-declared and updated.

```javascript
var greeter = "hey hi";
var greeter = "say Hello instead";
greeter = "Ciao";
```

## 3.3. let

```let``` is block-scoped.

```javascript
let greeting = "say Hi";
let times = 4;

if (times > 3) {
    let hello = "say Hello instead";
    console.log(hello);// "say Hello instead"
}
console.log(hello) // hello is not defined
```

```let``` variables can be updated but not re-declared.

```javascript
let greeting = "say Hi";
greeting = "say Hello instead";
let greeting = "say Hello instead"; // error: Identifier 'greeting' has already been declared
```

It is okay to define two variables with the same name in different scopes.

```let``` variables are hoisted to the top. They are uninitialized, therefore if you try to use ```let``` variable before its declaration, you will get an ```ReferenceError```.

```javascript
console.log(a); // ReferenceError: Cannot access 'a' before initialization
let a = 5;
```

## 3.4. const

```const``` just like ```let``` but it cannot be updated. However, in the case of an ```object```, you can update its properties.

---

## 3.5. Sources

- [Var, Let, and Const – What's the Difference?](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/)

# 4. Spread operator

Spread syntax can be used when all elements from an object or array need to be included in a new array or object, or should be applied one-by-one in a function call's arguments list. There are three distinct places that accept the spread syntax:

- [Function arguments list](#function-arguments-list)
- [Array literals](#array-literals)
- [Object literals](#object-literals)

## 4.1. Function arguments list

```javascript
function myFunction(x, y, z) {}
const args = [0, 1, 2];
myFunction.apply(null, args);
```

vs.

```javascript
function myFunction(x, y, z) {}
const args = [0, 1, 2];
myFunction(...args);
```

The spread operator `…` may be used as the final parameter to a function. This will take all arguments passed into the function and place them in an array.

```javascript
function foo(bar: string, ...baz: number[]) {}

foo("bar", 1, 2, 3, 4);
```

## 4.2. Array literals

```javascript
const parts = ["shoulders", "knees"];
const lyrics = ["head", ...parts, "and", "toes"];
//  ["head", "shoulders", "knees", "and", "toes"]

let arr1 = [0, 1, 2];
const arr2 = [3, 4, 5];

arr1 = [...arr1, ...arr2];
// arr1 is now [0, 1, 2, 3, 4, 5]
```

You can also conditionally apply the spread operator. Spreading an empty array adds nothing to the final array.

```javascript
const isSummer = false;
const fruits = ["apple", "banana", ...(isSummer ? ["watermelon"] : [])];
// ['apple', 'banana']
```

## 4.3. Object literals

```javascript
const obj1 = { foo: "bar", x: 42 };
const obj2 = { bar: "baz", y: 13 };

const mergedObj = { ...obj1, ...obj2 };
// { foo: "bar", x: 42, bar: "baz", y: 13 }
```

In case of a collision on a property name, the last value is taken. You can also conditionally apply the spread operator here:

```javascript
const isSummer = false;
const fruits = {
  apple: 10,
  banana: 5,
  ...(isSummer ? { watermelon: 30 } : {}),
};
// { apple: 10, banana: 5 }
```

# 5. This

In general, the `this` references the object of which the function is a property. In other words, the `this` references the object that is currently calling the function. For example if a member function of an object is invoked, `this` will refer to that object. 

In the global context, `this` references the global object (`window` on browser, `global` on Node.js). If you assign a property to `this` in this context, JavaScript will add it to the global object.

```javascript
this.color= 'Red';
console.log(window.color); // 'Red'
```

Function context:

- [5.1. Function invocation](#51-function-invocation)
- [5.2. Method invocation](#52-method-invocation)
- [5.3. Constructor invocation](#53-constructor-invocation)
- [5.4. Indirect invocation](#54-indirect-invocation)
- [5.5. Arrow functions](#55-arrow-functions)

## 5.1. Function invocation

`this` is set to the global object.

```javascript
function show() {
   console.log(this === window); // true
}

show(); // equivalent to window.show()
```

In the strict mode, JavaScript sets `this` inside a function to `undefined`.

---

## 5.2. Method invocation

`this` is set to the object that owns the method.

```javascript
let car = {
    brand: 'Honda',
    getBrand: function () {
        return this.brand;
    }
}

console.log(car.getBrand()); // Honda
```

There is a caveat: when you call a method without specifying its object, JavaScript assumes `this` to be either global object or undefined.

```javascript
let brand = car.brand;

console.log(brand()); // undefined


let brand = car.getBrand.bind(car); // bind binds `this` to the instance of car
console.log(brand()); // Honda
```

---

## 5.3. Constructor invocation

`this` refers to the newly-created object. However, the constructor has to be invoked by the `new` keyword. Otherwise we are accessing the global object, once again.

---

## 5.4. Indirect invocation

All functions are instances of the `Function` type which has two methods: `call` and `apply`. Invoking function via either of those is so-called indirect invocation. These two functions allow you to set the `this` inside the function.

```javascript
function getBrand(prefix) {
    console.log(prefix + this.brand);
}

let honda = {
    brand: 'Honda'
};
let audi = {
    brand: 'Audi'
};

getBrand.call(honda, "It's a "); // It's a Honda
getBrand.call(audi, "It's an "); // It's an Audi
```

---

## 5.5. Arrow functions

In arrow functions, the context is lexical. The arrow function inherits `this` from the outer function where the function is called.

---
[Demystifying the JavaScript this Keyword](https://www.javascripttutorial.net/javascript-this/)

# 6. Closures

## 6.1. Lexical scoping

The word lexical refers to the fact that lexical scoping uses the location where a variable is declared within the source code to determine where that variable is available. Nested functions have access to variables declared in their outer scope.

In other words, the children scope has access to the variables in the outer scope.

```javascript
function init() {
  var name = "Mozilla"; // name is a local variable created by init
  function displayName() {
    // displayName() is the inner function, that forms the closure
    console.log(name); // use variable declared in the parent function
  }
  displayName();
}
init();
```

## 6.2. Closure

```javascript
function makeAdder(x) {
  return function add(y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(2)); // 7
console.log(add10(2)); // 12
```

`add` still has access to `x` since JavaScript functions form closures. Once `makeAdder` finishes executing, the `x` variable is still present and available. In the lexical environment of `add5`, `x` is 5 while in the latter, `x` is 10. `add5` and `add10` both form closures.

A closure is the combination of a function and the lexical environment within which that function was declared. This environment consists of any local variables that were in-scope at the time the closure was created. In other words, a closure gives you access to an outer function's scope from an inner function.

Every closure has three scopes:

- local
- enclosing
- global

**Caveat**: if the outer function is itself a nested function, access to the outer function's scope includes the enclosing scope of the outer function -- creating a chain of function scopes. We can say that closures have access to all outer function scopes.

```javascript
// global scope
const e = 10;
function sum(a) {
  return function (b) {
    return function (c) {
      // outer functions scope
      return function (d) {
        // local scope
        return a + b + c + d + e;
      };
    };
  };
}

console.log(sum(1)(2)(3)(4)); // 20
```

---
[Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

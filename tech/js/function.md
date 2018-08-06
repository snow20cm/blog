+++
categories = ["notes"]
date = "2015-09-13T16:20:54-06:00"
tags = ["javascript"]
title = "function"

+++

## Defining functions

### Function declarations

A `function definition` (also called a `function declaration`, or `function statement`) consists of the function keyword, followed by: function name, parameters and body.

- Primitive parameters (such as a number) are passed to functions by value.
- If you pass an object as a parameter and the function changes the __object's properties__, that change is visible outside the function, as shown in the following example:

__Note: Assigning a new object to the parameter will not have any effect outside the function, because this is changing the value of the parameter rather than the value of one of the object's properties__

## Function expressions

```javascript
    var square = function(number) { return number * number };
    var x = square(4) // x gets the value 16
```

In JavaScript, a function can be defined based on a condition. For example, the following function definition defines myFunc only if num equals 0:

```javascript
    var myFunc;
    if (num == 0){
      myFunc = function(theObject) {
        theObject.make = "Toyota"
      }
    }
```

__You can also use the [Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/Function) constructor to create functions from a string at runtime, much like eval()__.

A `method` is a function that is a property of an object.

## Calling functions

Functions must be in scope when they are called, but the function declaration can be hoisted (appear below the call in the code), as in this example:

```javascript
    console.log(square(5));
    /* ... */
    function square(n) { return n*n }
```

The scope of a function is the function in which it is declared, or the entire program if it is declared at the top level.

__Note: This works only when defining the function using the above syntax (i.e. function funcName(){}). The code below will not work.__
```javascript
    console.log(square(5));
    square = function (n) {
      return n * n;
    }
```

__There are other ways to call functions. There are often cases where a function needs to be called dynamically, or the number of arguments to a function vary, or in which the context of the function call needs to be set to a specific object determined at runtime. It turns out that functions are, themselves, objects, and these objects in turn have methods (see the Function object). One of these, the apply() method, can be used to achieve this goal.__

## Function scope
Variables defined inside a function cannot be accessed from anywhere outside the function.

## Scope and the function stack

### Recursion

There are three ways for a function to refer to itself:

- the function's name
- arguments.callee
- an in-scope variable that refers to the function

A function that calls itself is called a recursive function.

### Nested functions and closures

- You can nest a function within a function. The nested (inner) function is private to its containing (outer) function.
- It also forms a closure. A closure is an expression (typically a function) that can have free variables together with an environment that binds those variables (that "closes" the expression).

To summarize:

- The inner function can be accessed only from statements in the outer function.
- The inner function forms a closure: the inner function can use the arguments and variables of the outer function, while the outer function cannot use the arguments and variables of the inner function.

Since the inner function forms a closure, you can call the outer function and specify arguments for both the outer and inner function:

```javascript
    function outside(x) {
      function inside(y) {
        return x + y;
      }
      return inside;
    }
    fn_inside = outside(3); // Think of it like: give me a function that adds 3 to whatever you give it
    result = fn_inside(5); // returns 8

    result1 = outside(3)(5); // returns 8
```

### Preservation of variables

Notice how x is preserved when inside is returned. A closure must preserve the arguments and variables in all scopes it references. Since each call provides potentially different arguments, a new closure is created for each call to outside. The memory can be freed only when the returned inside is no longer accessible.

This is not different from storing references in other objects, but is often less obvious because one does not set the references directly and cannot inspect them.

### Multiply-nested functions

The closures can contain multiple scopes; they recursively contain the scope of the functions containing it. This is called scope chaining.

Consider the following example:

```javascript
    function A(x) {
      function B(y) {
        function C(z) {
          console.log(x + y + z);
        }
        C(3);
      }
      B(2);
    }
    A(1); // logs 6 (1 + 2 + 3)
```

### Name conflicts

More inner scopes take precedence, so the inner-most scope takes the highest precedence, while the outer-most scope takes the lowest. This is the scope chain.

```javascript
function outside() {
  var x = 10;
  function inside(x) {
    return x;
  }
  return inside;
}
result = outside()(20); // returns 20 instead of 10
```

## Closures

An object containing methods for manipulating the inner variables of the outer function can be returned.

```javascript
    var createPet = function(name) {
      var sex;
      
      return {
        setName: function(newName) {
          name = newName;
        },
        
        getName: function() {
          return name;
        },
        
        getSex: function() {
          return sex;
        },
        
        setSex: function(newSex) {
          if(typeof newSex == "string" && (newSex.toLowerCase() == "male" || newSex.toLowerCase() == "female")) {
            sex = newSex;
          }
        }
      }
    }

    var pet = createPet("Vivie");
    pet.getName();                  // Vivie

    pet.setName("Oliver");
    pet.setSex("male");
    pet.getSex();                   // male
    pet.getName();                  // Oliver
```

In the code above, the name variable of the outer function is accessible to the inner functions, __and there is no other way to access the inner variables except through the inner functions__. The inner variables of the inner function act as safe stores for the inner functions. They hold "persistent", yet secure, data for the inner functions to work with. The functions do not even have to be assigned to a variable, or have a name.

```javascript
    var getCode = (function(){
      var secureCode = "0]Eal(eh&2";    // A code we do not want outsiders to be able to modify...
      
      return function () {
        return secureCode;
      };
    })();

    getCode();    // Returns the secureCode
```

If an enclosed function defines a variable with the same name as the name of a variable in the outer scope, there is no way to refer to the variable in the outer scope again.

```javascript
    var createPet = function(name) {  // Outer function defines a variable called "name"
      return {
        setName: function(name) {    // Enclosed function also defines a variable called "name"
          name = name;               // ??? How do we access the "name" defined by the outer function ???
        }
      }
    }
```

The magical `this` variable is very tricky in closures. They have to be used carefully, as what this refers to depends completely on where the function was called, rather than where it was defined.

## Using the arguments object

The arguments of a function are maintained in an array-like object. Within a function, you can address the arguments passed to it as follows:

```javascript
    arguments[i]
```

Using the arguments object, you can call a function with more arguments than it is formally declared to accept.

For example, consider a function that concatenates several strings. The only formal argument for the function is a string that specifies the characters that separate the items to concatenate. The function is defined as follows:

__Note: The arguments variable is "array-like", but not an array. It is array-like in that is has a numbered index and a length property. However, it does not possess all of the array-manipulation methods.__

## Function parameters
Starting with ECMAScript 6, there are two new kinds of parameters: default parameters and rest parameters.

### Default parameters

With default parameters, the check in the function body is no longer necessary. Now, you can simply put 1 as the default value for b in the function head:

```javascript
    function multiply(a, b = 1) {
      // b = typeof b !== 'undefined' ?  b : 1;
      return a*b;
    }

    multiply(5); // 5
```

Fore more details, see default parameters in the reference.

### Rest parameters

The rest parameter syntax allows to represent an indefinite number of arguments as an array.

```javascript
    function multiply(multiplier, ...theArgs) {
      return theArgs.map(x => multiplier * x);
    }

    var arr = multiply(2, 1, 2, 3);
    console.log(arr); // [2, 4, 6]
```

## Arrow functions

Arrow functions are always anonymous.

Two factors influenced the introduction of arrow functions: shorter functions and lexical this.

### Shorter functions

In some functional patterns, shorter functions are welcome. Compare:

```javascript
    var a = [
      "Hydrogen",
      "Helium",
      "Lithium",
      "Beryl­lium"
    ];

    var a2 = a.map(function(s){ return s.length });

    var a3 = a.map( s => s.length );
```

### Lexical this

__Until arrow functions, every new function defined its own this value (a new object in case of a constructor, undefined in strict mode function calls, the context object if the function is called as an "object method", etc.). This proved to be annoying with an object-oriented style of programming.__

```javascript
    function Person() {
      // The Person() constructor defines `this` as itself.
      this.age = 0;

      setInterval(function growUp() {
        // In nonstrict mode, the growUp() function defines `this` 
        // as the global object, which is different from the `this`
        // defined by the Person() constructor.
        this.age++;
      }, 1000);
    }

    var p = new Person();
```

In ECMAScript 3/5, this issue was fixed by assigning the value in this to a variable that could be closed over.

```javascript
    function Person() {
      var self = this; // Some choose `that` instead of `self`. 
                       // Choose one and be consistent.
      self.age = 0;

      setInterval(function growUp() {
        // The callback refers to the `self` variable of which
        // the value is the expected object.
        self.age++;
      }, 1000);
    }
```

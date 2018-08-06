+++
categories = ["notes"]
date = "2015-09-05T21:18:06-06:00"
tags = ["javascript"]
title = "basic"

+++

JavaScript
=====

## Basic

- JavaScript is __case-sensitive__ and uses the __Unicode__ character set.
- instructions are called __statements__

## Comments

The syntax of comments is the same as in C++ and in many other languages

## Declarations

- The names of variables, called `identifiers`.

- `identifier` must start with
    * a letter
    * underscore (\_),
    * dollar sign ($)

### Declaring variables

- Global Variables: current document
    * `var x = 42;`
    * `x = 42;`  __Don’t use this way__
- Local Variables: function
    * `var` in a function
- Block Scope Variables : block
    * `let x = 42;`
- `var` or `let` statement with no initial value specified has the value `undefined`

You can use `undefined` to determine whether a variable has a value.

``` javascript
var input;
if(input === undefined){ // 3 equal signs
    doThis();
} else {
    doThat();
}
```

|             | boolean   | number   | string        |
|-------------|-----------|----------|---------------|
| undefined   | false     | NaN      | “undefined”   |

### hoisting

- __hoisting__ : refer to a variable declared later, without getting an exception.
- Even if you declare and inititalize after you use or refer to this variable, it will still return undefined.
- Because of hoisting, all var statements in a function should be placed as near to the top of the function as possible.

```javascript
    /**
     * Example 1
     */
    console.log(x === undefined); // logs "true"
    var x = 3;

    /**
     * Example 2
     */
    // will return a value of undefined
    var myvar = "my value";
     
    (function() {
      console.log(myvar); // undefined
      var myvar = "local value";
    })();
```

### Global variables

- Global variables are in fact properties of the global object.
- In web pages the global object is window, so you can set and access global variables using the window.variable syntax.
- Consequently, you can access global variables declared in one window or frame from another window or frame by specifying the window or frame name. For example, if a variable called phoneNumber is declared in a document, you can refer to this variable from an iframe as parent.phoneNumber.

### Constants

    const prefix = '212';

- The scope rules for constants are the same as those for let block scope variables.
- You cannot declare a constant with the same name as a function or variable in the same scope.

## Data structures and types

### Data types

The latest ECMAScript standard defines _seven_ data types:

Six data types that are primitives:
- Boolean. true and false.
- null. A special keyword denoting a null value. 
- undefined. A top-level property whose value is undefined.
- Number. 42 or 3.14159.
- String. "Howdy"
- Symbol (new in ECMAScript 6). A data type whose instances are unique and immutable.

and

- Object

### Data type conversion

#### int to string

    “30” + 7; // “307”

__ONLY__ + convert number to string

    “30” - 7; // 23

#### strings to numbers

In the case that a value representing a number is in memory as a string, there are methods for conversion.

- parseInt()
- parseFloat()
- `(+"1.1") + (+"1.1") = 2.2`

Additionally, a best practice for parseInt is to always include the radix parameter.

### Literals

#### Array literals

    var coffees = ["French Roast", "Colombian", "Kona"];

__Note__ : An array literal is a type of object initializer. See Using Object Initializers.

_If an array is created using a literal in a top-level script, JavaScript interprets the array each time it evaluates the expression containing the array literal. In addition, a literal used in a function is created each time the function is called._


##### Extra commas in array literals

    var fish = ["Lion", , "Angel"]; // (fish[0] is "Lion", fish[1] is undefined, and fish[2] is "Angel").
    var myList = ['home', , 'school', ]; // the length of the array is three. There is no myList[3].


#### Boolean literals

The Boolean type has two literal values: true and false.

#### Integers

Integers can be expressed in decimal (base 10), hexadecimal (base 16), octal (base 8) and binary (base 2).

#### Floating-point literals

    [(+|-)][digits][.digits][(E|e)[(+|-)]digits]

For example:

    3.1415926
    -.123456789
    -3.1E+12
    .1e-23

#### Object literals

An object literal is a list of zero or more pairs of property names and associated values of an object, enclosed in curly braces ({}). You should not use an object literal at the beginning of a statement. This will lead to an error or not behave as you expect, because the { will be interpreted as the beginning of a block.

The following is an example of an object literal. The first element of the car object defines a property, myCar, and assigns to it a new string, "Saturn"; the second element, the getCar property, is immediately assigned the result of invoking the function (CarTypes("Honda")); the third element, the special property, uses an existing variable (Sales).

```javascript
    var Sales = "Toyota";

    function CarTypes(name) {
      if (name == "Honda") {
        return name;
      } else {
        return "Sorry, we don't sell " + name + ".";
      }
    }

    var car = { myCar: "Saturn", getCar: CarTypes("Honda"), special: Sales };
```

Object property names can be any string, including the empty string. If the property name would not be a valid JavaScript identifier, it must be enclosed in quotes. Property names that would not be valid identifiers also cannot be accessed as a dot (.) property, but can be accessed and set with the array-like notation("[]").

```javascript
    var unusualPropertyNames = {
      "": "An empty string",
      "!": "Bang!"
    }
    console.log(unusualPropertyNames."");   // SyntaxError: Unexpected string
    console.log(unusualPropertyNames[""]);  // An empty string
    console.log(unusualPropertyNames.!);    // SyntaxError: Unexpected token !
    console.log(unusualPropertyNames["!"]); // Bang!
```

Please note:

```javascript
    var foo = {a: "alpha", 2: "two"};
    console.log(foo.a);    // alpha
    console.log(foo[2]);   // two
    //console.log(foo.2);  // Error: missing ) after argument list
    //console.log(foo[a]); // Error: a is not defined
    console.log(foo["a"]); // alpha
    console.log(foo["2"]); // two
```

#### String literals

A string literal is zero or more characters enclosed in double (") or single (') quotation marks.

You can also escape line breaks by preceding them with backslash. The backslash and line break are both removed from the value of the string.

```javascript
    var str = "this string \
    is broken \
    across multiple\
    lines."
    console.log(str);   // this string is broken across multiplelines.
```

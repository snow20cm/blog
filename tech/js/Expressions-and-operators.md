+++
categories = ["notes"]
date = "2015-09-13T20:27:54-06:00"
tags = ["javascript"]
title = "Expressions and operators"

+++


## Operators

### Assignment operators

#### Destructuring

For more complex assignments, the destructuring assignment syntax is a JavaScript expression that makes it possible to extract data from arrays or objects using a syntax that mirrors the construction of array and object literals.

```javascript
    var foo = ["one", "two", "three"];

    // without destructuring
    var one   = foo[0];
    var two   = foo[1];
    var three = foo[2];

    // with destructuring
    var [one, two, three] = foo;
```

### Comparison operators


|Operator               | Description                                                                                         | Examples returning true|
|-----------------------|-----------------------------------------------------------------------------------------------------|------------------------|
|Strict equal (===)     | Returns true if the operands are equal and of the same type. See also Object.is and sameness in JS. | 3 === var1|
|Strict not equal (!==) | Returns true if the operands are of the same type but not equal, or are of different type.          | var1 !== "3"|
|3 !== '3'|

### Arithmetic operators


    1 / 2; // 0.5
    1 / 2 == 1.0 / 2.0; // this is true


Exponentiation operator (\*\*) 	Calculates the base to the exponent power, that is, baseexponent	2 \*\* 3 returns 8.
10 \*\* -1 returns -1.

### Bitwise operators

A bitwise operator treats their operands as a set of __32 bits__ (zeros and ones). Bitwise operators perform their operations on such binary representations, but they return standard JavaScript numerical values.

    a = 5;
    a >> 1; // 2
    a = -5;
    a >> 1; // -3
    a = 0.3;
    a >> 1; // 0

### Bitwise logical operators

### Logical operators

The && and || operators actually return the value of one of the specified operands, so if these operators are used with non-Boolean values, they may __return a non-Boolean value__.

    null && 1; // return null
    0 && 1; // return 0

NOT(!) always return boolean.


#### Short-circuit evaluation

So any side effects of doing so do not take effect.

### String operators

    console.log("my " + "string"); // console logs the string "my string".

### Conditional (ternary) operator

```javascript
    3 ? console.log('a') : console.log('b');
```

### Comma operator

The comma operator (,) simply evaluates both of its operands and returns the value of the last operand.

### Unary operators

#### delete

The delete operator deletes an object, an object's property, or an element at a specified index in an array. The syntax is:

    delete objectName;
    delete objectName.property;
    delete objectName[index];
    delete property; // legal only within a with statement

The fourth form is legal only within a `with` statement, to delete a property from an object.

You can use the delete operator to delete variables declared __implicitly__ but not those declared with the __var__ statement.

If the `delete` operator succeeds, it sets the property or element to `undefined.` The `delete` operator returns `true` if the operation is possible; it returns `false` if the operation is not possible.

```javascript
    x = 42;
    var y = 43;
    myobj = new Number();
    myobj.h = 4;    // create property h
    delete x;       // returns true (can delete if declared implicitly)
    delete y;       // returns false (cannot delete if declared with var)
    delete Math.PI; // returns false (cannot delete predefined properties)
    delete myobj.h; // returns true (can delete user-defined properties)
    delete myobj;   // returns true (can delete if declared implicitly)
```

##### Deleting array elements

When you `delete` an array element, the array length is not affected. For example, if you delete `a[3]`, `a[4]` is still `a[4]` and `a[3]` is `undefined.`

When the delete operator removes an array element, that element is __no longer__ in the array. In the following example, trees[3] is removed with delete. However, trees[3] is still addressable and returns undefined.

```javascript
    var trees = ["redwood", "bay", "cedar", "oak", "maple"];
    delete trees[3];
    if (3 in trees) {
      // this does not get executed
    }
```

__If you want an array element to exist__ but have an undefined value, use the undefined keyword instead of the delete operator. In the following example, trees[3] is assigned the value undefined, but the array element still exists:

```javascript
    var trees = ["redwood", "bay", "cedar", "oak", "maple"];
    trees[3] = undefined;
    if (3 in trees) {
      // this gets executed
    }
```

### typeof

The typeof operator is used in either of the following ways:

    typeof operand
    typeof (operand)

The `typeof` operator __returns a string__ indicating the type of the unevaluated operand.

Suppose you define the following variables:

```javascript
    var myFun = new Function("5 + 2");
    var shape = "round";
    var size = 1;
    var today = new Date();

    typeof myFun;     // returns "function"
    typeof shape;     // returns "string"
    typeof size;      // returns "number"
    typeof today;     // returns "object"
    typeof dontExist; // returns "undefined"

    typeof true; // returns "boolean"
    typeof null; // returns "object"

    typeof 62;            // returns "number"
    typeof 'Hello world'; // returns "string"

    typeof document.lastModified; // returns "string"
    typeof window.length;         // returns "number"
    typeof Math.LN2;              // returns "number"

    typeof blur;        // returns "function"
    typeof eval;        // returns "function"
    typeof parseInt;    // returns "function"
    typeof shape.split; // returns "function"
```

For predefined objects, the typeof operator returns results as follows:

```javascript
    typeof Date;     // returns "function"
    typeof Function; // returns "function"
    typeof Math;     // returns "object"
    typeof Option;   // returns "function"
    typeof String;   // returns "function"
```

#### void

The void operator is used in either of the following ways:

    void (expression)
    void expression

The void operator specifies an expression to be evaluated without returning a value.

You can use the void operator to specify an expression as a hypertext link. The expression is evaluated but is not loaded in place of the current document.

The following code creates a hypertext link that does nothing when the user clicks it. When the user clicks the link, `void(0)` evaluates to undefined, which has no effect in JavaScript.

    <a href="javascript:void(0)">Click here to do nothing</a>

The following code creates a hypertext link that submits a form when the user clicks it.

    <a href="javascript:void(document.form.submit())">
    Click here to submit</a>

### Relational operators

A relational operator compares its operands and returns a Boolean value based on whether the comparison is true.

#### in

The in operator returns true if the specified property is in the specified object. The syntax is:

    propNameOrNumber in objectName

where propNameOrNumber is a __string__ or __numeric expression__ representing a property name or array index, and objectName is the name of an object.

The following examples show some uses of the in operator.

    // Arrays
    var trees = ["redwood", "bay", "cedar", "oak", "maple"];
    0 in trees;        // returns true
    3 in trees;        // returns true
    6 in trees;        // returns false
    "bay" in trees;    // returns false (you must specify the index number,
                       // not the value at that index)
    "length" in trees; // returns true (length is an Array property)

    // built-in objects
    "PI" in Math;          // returns true
    var myString = new String("coral");
    "length" in myString;  // returns true

    // Custom objects
    var mycar = { make: "Honda", model: "Accord", year: 1998 };
    "make" in mycar;  // returns true
    "model" in mycar; // returns true

#### instanceof

The instanceof operator returns true if the specified object is of the specified object type. The syntax is:

objectName instanceof objectType
where objectName is the name of the object to compare to objectType, and objectType is an object type, such as Date or Array.

Use instanceof when you need to confirm the type of an object at runtime. For example, when catching exceptions, you can branch to different exception-handling code depending on the type of exception thrown.

For example, the following code uses instanceof to determine whether theDay is a Date object. Because theDay is a Date object, the statements in the if statement execute.

var theDay = new Date(1995, 12, 17);
if (theDay instanceof Date) {
  // statements to execute
}
Operator precedence

The precedence of operators determines the order they are applied when evaluating an expression. You can override operator precedence by using parentheses.

The following table describes the precedence of operators, from highest to lowest.

Operator precedence
Operator type	Individual operators
member	. []
call / create instance	() new
negation/increment	! ~ - + ++ -- typeof void delete
multiply/divide	* / %
addition/subtraction	+ -
bitwise shift	<< >> >>>
relational	< <= > >= in instanceof
equality	== != === !==
bitwise-and	&
bitwise-xor	^
bitwise-or	|
logical-and	&&
logical-or	||
conditional	?:
assignment	= += -= *= /= %= <<= >>= >>>= &= ^= |=
comma	,
A more detailed version of this table, complete with links to additional details about each operator, may be found in JavaScript Reference.

Expressions
An expression is any valid unit of code that resolves to a value.

Conceptually, there are two types of expressions: those that assign a value to a variable and those that simply have a value.

The expression x = 7 is an example of the first type. This expression uses the = operator to assign the value seven to the variable x. The expression itself evaluates to seven.

The code 3 + 4 is an example of the second expression type. This expression uses the + operator to add three and four together without assigning the result, seven, to a variable.

JavaScript has the following expression categories:

Arithmetic: evaluates to a number, for example 3.14159. (Generally uses arithmetic operators.)
String: evaluates to a character string, for example, "Fred" or "234". (Generally uses string operators.)
Logical: evaluates to true or false. (Often involves logical operators.)
Primary expressions: Basic keywords and general expressions in JavaScript.
Left-hand-side expressions: Left values are the destination of an assignment.
Primary expressions

Basic keywords and general expressions in JavaScript.

this

Use the this keyword to refer to the current object. In general, this refers to the calling object in a method. Use this either with the dot or the bracket notation:

this["propertyName"]
this.propertyName
Suppose a function called validate validates an object's value property, given the object and the high and low values:

function validate(obj, lowval, hival){
  if ((obj.value < lowval) || (obj.value > hival))
    console.log("Invalid Value!");
}
You could call validate in each form element's onChange event handler, using this to pass it the form element, as in the following example:

<p>Enter a number between 18 and 99:</p>
<input type="text" name="age" size=3 onChange="validate(this, 18, 99);">
Grouping operator

The grouping operator ( ) controls the precedence of evaluation in expressions. For example, you can override multiplication and division first, then addition and subtraction to evaluate addition first.

var a = 1;
var b = 2;
var c = 3;

// default precedence
a + b * c     // 7
// evaluated by default like this
a + (b * c)   // 7

// now overriding precedence 
// addition before multiplication   
(a + b) * c   // 9

// which is equivalent to
a * c + b * c // 9
Comprehensions

Comprehensions are an experimental JavaScript feature, targeted to be included in a future ECMAScript version. There are two versions of comprehensions:

 [for (x of y) x]
Array comprehensions.
 (for (x of y) y)
Generator comprehensions.
Comprehensions exist in many programming languages and allow you to quickly assemble a new array based on an existing one, for example.

[for (i of [ 1, 2, 3 ]) i*i ]; 
// [ 1, 4, 9 ]

var abc = [ "A", "B", "C" ];
[for (letters of abc) letters.toLowerCase()];
// [ "a", "b", "c" ]
Left-hand-side expressions

Left values are the destination of an assignment.

new

You can use the new operator to create an instance of a user-defined object type or of one of the built-in object types. Use new as follows:

var objectName = new objectType([param1, param2, ..., paramN]);
super

The super keyword is used to call functions on an object's parent. It is useful with classes to call the parent constructor, example.

super([arguments]); // calls the parent constructor.
super.functionOnParent([arguments]);
Spread operator

The spread operator allows an expression to be expanded in places where multiple arguments (for function calls) or multiple elements (for array literals) are expected.

Example: Today if you have an array and want to create a new array with the existing one being part of it, the array literal syntax is no longer sufficient and you have to fall back to imperative code, using a combination of push, splice, concat, etc. With spread syntax this becomes much more succinct:

var parts = ['shoulder', 'knees'];
var lyrics = ['head', ...parts, 'and', 'toes'];
Similarly, the spread operator works with function calls:

function f(x, y, z) { }
var args = [0, 1, 2];
f(...args);

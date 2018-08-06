+++
categories = ["notes"]
date = "2015-09-09T20:37:47-06:00"
tags = ["javascript"]
title = "loops and iteration"
+++

## for statement

```javascript
    for ([initialExpression]; [condition]; [incrementExpression])
      statement
```
## do...while statement

```javascript
    do
      statement
    while (condition);
```

## while statement

```javascript
    while (condition)
      statement
```

## label statement

```javascript
    label :
       statement
```

The value of label may be any JavaScript identifier that is not a reserved word. The statement that you identify with a label may be any statement.

Example

In this example, the label markLoop identifies a while loop.

```javascript
    markLoop:
    while (theMark == true) {
       doSomething();
    }
```

## break statement

```javascript
    break;
    break label;
```

__When you use break with a label, it terminates the specified labeled statement.__

Example 2: Breaking to a label

```javascript
    var x = 0;
    var z = 0
    labelCancelLoops: while (true) {
      console.log("Outer loops: " + x);
      x += 1;
      z = 1;
      while (true) {
        console.log("Inner loops: " + z);
        z += 1;
        if (z === 10 && x === 10) {
          break labelCancelLoops;
        } else if (z === 10) {
          break;
        }
      }
    }
```

## continue statement


```javascript
    continue;
    continue label;
```

__When you use continue with a label, it applies to the looping statement identified with that label.__

## for...in statement

```javascript
for (variable in object) {
  statements
}
```

Example


```javascript
    function dump_props(obj, obj_name) {
      var result = "";
      for (var i in obj) {
        result += obj_name + "." + i + " = " + obj[i] + "<br>";
      }
      result += "<hr>";
      return result;
    }
```

### Arrays

Although it may be tempting to use this as a way to iterate over Array elements, the for...in statement will return the name of your user-defined properties in addition to the numeric indexes. Thus it is better to use a traditional for loop with a numeric index when iterating over arrays, because the for...in statement iterates over user-defined properties in addition to the array elements, if you modify the Array object, such as adding custom properties or methods.

## for...of statement

__This is a new technology, part of the ECMAScript 2015 (ES6) standard.__

```javascript
for (variable of object) {
  statement
}
```


```javascript
    let arr = [3, 5, 7];
    arr.foo = "hello";

    for (let i in arr) {
       console.log(i); // logs "0", "1", "2", "foo"
    }

    for (let i of arr) {
       console.log(i); // logs "3", "5", "7"
    }
```

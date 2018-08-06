+++
categories = ["notes"]
date = "2015-09-09T19:37:37-06:00"
tags = ["javascript"]
title = "control flow and error handling"

+++

## Block statement

Important: JavaScript prior to ECMAScript 6 does not have block scope. Starting with ECMAScript 6, the let variable declaration is block scoped.

## Conditional statements

### if...else

```javascript
    if (condition) {
      statement_1;
    } else {
      statement_2;
    }
```

#### Falsy values

The following values will evaluate to false:

- false
- undefined
- null
- 0
- NaN
- the empty string ("")

All other values, including all objects evaluate to true when passed to a conditional statement.

__Do not confuse the primitive boolean values true and false with the true and false values of the Boolean object. For example:__

    var b = new Boolean(false);
    if (b) // this condition evaluates to true

###  switch statement

```javascript
    switch (expression) {
      case label_1:
        statements_1
        [break;]
      case label_2:
        statements_2
        [break;]
        ...
      default:
        statements_def
        [break;]
    }
```

If break is omitted, the program continues execution at the next statement in the switch statement.

## Exception handling statements

- `throw` statement
- `try...catch` statement

### Exception types

Just about any object can be thrown in JavaScript.

- `ECMAScript exceptions`
- `DOMException` and `DOMError`

#### throw statement

```javascript
    throw "Error2";   // String type
    throw 42;         // Number type
    throw true;       // Boolean type
    throw {toString: function() { return "I'm an object!"; } };
```

Note: You can specify an object when you throw an exception. You can then reference the object's properties in the catch block. The following example creates an object myUserException of type UserException and uses it in a throw statement.

```javascript
    // Create an object type UserException
    function UserException(message) {
      this.message = message;
      this.name = "UserException";
    }

    // Make the exception convert to a pretty string when used as a string 
    // (e.g. by the error console)
    UserException.prototype.toString = function() {
      return this.name + ': "' + this.message + '"';
    }

    // Create an instance of the object type and throw it
    throw new UserException("Value too high");
```

#### try...catch statement

```javascript
    function getMonthName(mo) {
      mo = mo - 1; // Adjust month number for array index (1 = Jan, 12 = Dec)
      var months = ["Jan","Feb","Mar","Apr","May","Jun","Jul",
                    "Aug","Sep","Oct","Nov","Dec"];
      if (months[mo] != null) {
        return months[mo];
      } else {
        throw "InvalidMonthNo"; //throw keyword is used here
      }
    }

    try { // statements to try
      monthName = getMonthName(myMonth); // function could throw exception
    }
    catch (e) {
      monthName = "unknown";
      logMyErrors(e); // pass exception object to error handler
    }
```

##### The catch block

You can use a catch block to handle all exceptions that may be generated in the try block.

    catch (catchID) {
      statements
    }
    
The catch block specifies an identifier (catchID in the preceding syntax) that holds the value specified by the throw statement; you can use this identifier to get information about the exception that was thrown. __JavaScript creates this identifier when the catch block is entered; the identifier lasts only for the duration of the catch block; after the catch block finishes executing, the identifier is no longer available.__

##### The finally block

```javascript
    openMyFile();
    try {
      writeMyFile(theData); //This may throw a error
    } catch(e) {  
      handleError(e); // If we got a error we handle it
    } finally {
      closeMyFile(); // always close the resource
    }
```

__If the finally block returns a value, this value becomes the return value of the entire try-catch-finally production, regardless of any return statements in the try and catch blocks:__

```javascript
    function f() {
      try {
        console.log(0);
        throw "bogus";
      } catch(e) {
        console.log(1);
        return true; // this return statement is suspended
                     // until finally block has completed
        console.log(2); // not reachable
      } finally {
        console.log(3);
        return false; // overwrites the previous "return"
        console.log(4); // not reachable
      }
      // "return false" is executed now  
      console.log(5); // not reachable
    }
    f(); // alerts 0, 1, 3; returns false
```

Overwriting of return values by the finally block also applies to exceptions thrown or re-thrown inside of the catch block:

```javascript
    function f() {
      try {
        throw "bogus";
      } catch(e) {
        console.log('caught inner "bogus"');
        throw e; // this throw statement is suspended until 
                 // finally block has completed
      } finally {
        return false; // overwrites the previous "throw"
      }
      // "return false" is executed now
    }

    try {
      f();
    } catch(e) {
      // this is never reached because the throw inside
      // the catch is overwritten
      // by the return in finally
      console.log('caught outer "bogus"');
    }

    // OUTPUT
    // caught inner "bogus"
```

#### Nesting try...catch statements

You can nest one or more try...catch statements. If an inner try...catch statement does not have a catch block, the enclosing try...catch statement's catch block is checked for a match.

### Utilizing Error objects

Depending on the type of error, you may be able to use the 'name' and 'message' properties to get a more refined message. 'name' provides the general class of Error (e.g., 'DOMException' or 'Error'), while 'message' generally provides a more succinct message than one would get by converting the error object to a string.

If you are throwing your own exceptions, in order to take advantage of these properties (such as if your catch block doesn't discriminate between your own exceptions and system ones), you can use the Error constructor. For example:

```javascript
    function doSomethingErrorProne () {
      if (ourCodeMakesAMistake()) {
        throw (new Error('The message'));
      } else {
        doSomethingToGetAJavascriptError();
      }
    }
    ....
    try {
      doSomethingErrorProne();
    }
    catch (e) {
      console.log(e.name); // logs 'Error'
      console.log(e.message); // logs 'The message' or a JavaScript error message)
    }
```

## Promises

__Starting with ECMAScript 6, JavaScript gains support for Promise objects allowing you to control the flow of deferred and asynchronous operations.__

A Promise is in one of these states:

- __pending__: initial state, not fulfilled or rejected.
- __fulfilled__: successful operation
- __rejected__: failed operation.
- __settled__: the Promise is either fulfilled or rejected, but not pending.

![promises.png](https://mdn.mozillademos.org/files/8633/promises.png)


### Loading an image with XHR

A simple example using Promise and XMLHttpRequest to load an image is available at the MDN GitHub promise-test repository. You can also see it in action. Each step is commented and allows you to follow the Promise and XHR architecture closely. Here is the uncommented version, showing the Promise flow so that you can get an idea:

```javascript
    function imgLoad(url) {
      return new Promise(function(resolve, reject) {
        var request = new XMLHttpRequest();
        request.open('GET', url);
        request.responseType = 'blob';
        request.onload = function() {
          if (request.status === 200) {
            resolve(request.response);
          } else {
            reject(Error('Image didn\'t load successfully; error code:' 
                         + request.statusText));
          }
        };
        request.onerror = function() {
          reject(Error('There was a network error.'));
        };
        request.send();
      });
    }
```

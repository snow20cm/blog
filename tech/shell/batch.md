+++
Categories = ["Notes"]
Description = "Batch Script Notes"
Tags = ["shell"]
date = "2015-03-17T16:55:54-06:00"
title = "Batch Script Notes"
+++


[Batch](http://steve-jansen.github.io/guides/windows-batch-scripting/index.html)
=========


Variables
---------

### Variable Declaration

- DOS does not require declaration of variables.
- The value of undeclared/uninitialized variables is an empty string, or `""`.


### Vaiable Assignment

    set foo=bar
    set /A four=2+2  // `\A` <=> arthimetic operation during assignments.

- local variables  --> lowercase
- system variables --> uppercase
- command          --> uppercase

### Reading the Value of a Variable

    echo %foo%

### Listing Existing Variables

    set

### Variable Scope (Global vs Local)
By default, variables are global to your entire command prompt session. Call the SETLOCAL command to make variables local to the scope of your script. After calling SETLOCAL, any variable assignments revert upon calling ENDLOCAL, calling EXIT, or when execution reaches the end of file (EOF) in your script.

### Arguments

    %0 - %9

    %0 means script name

There is no %10, but you can read 10th argument by using `shift`

    shift

Then %9 moves to %8 and 10th argument moves to %9

### Tricks with Command Line Arguments

More details `FOR /?`

* %~1 remove quote
* %~f1 full path to %1
* %~dp1 parent folder of %1
* %~nx1 filename and extension of %1



    ECHO OFF

    SETLOCAL ENABLEEXTENSIONS
    SET me=%~n0
    SET parent=%~dp0

    echo %parent%
    echo %me%


Return Codes
------------


### Checking Return Codes In Your Script Commands

    %ERRORLEVEL%

Built-in DOS commands like `ECHO`, `IF`, and `SET` preserve the existing value of `%ERRORLEVEL%`.

    IF %ERRORLEVEL% NEQ 0 (
      REM do something here to address the error
    )

    IF ERRORLEVEL 1 (
      REM do something here to address the error
    )

`ERRORLEVEL 1`  ==> true if `ERRORLEVEL >= 1.` `ERRORLEVEL` may be negative number, so prefer `NEQ 0`

### Conditional Execution Using the Return Code

The first program/script must conform to the convention of returning 0 on success and non-0 on failure for this to work.

    SomeCommand.exe && ECHO SomeCommand.exe succeeded!
    SomeCommand.exe || ECHO SomeCommand.exe failed with return code %ERRORLEVEL%

### Return from a Script

    EXIT /B 0

or

    GOTO :EOF

### Tips and Tricks for Return Codes

I recommend sticking to zero for success and return codes that are positive values for DOS batch files. The positive values are a good idea because other callers may use the `IF ERRORLEVEL 1` syntax to check your script.

I also recommend documenting your possible return codes with easy to read SET statements at the top of your script file, like this:

    SET /A ERROR_HELP_SCREEN=1
    SET /A ERROR_FILE_NOT_FOUND=2

Note that I break my own convention here and use uppercase variable names `– I` do this to denote that the variable is constant and should not be modified elsewhere. Too bad DOS doesn’t support constant values like Unix/Linux shells.


Stdin, Stdout, Stderr
---------------------


    SomeCommand.exe   > temp.txt
    OtherCommand.exe >> temp.txt
    DIR SomeFile.txt > output.txt 2>&1
    TYPE < SomeFile.txt
    DIR /B | SORT

### A Cool Party Trick

Quickly create a new text file

    TYPE CON > output.txt

`CON` is command prompt’s own stdin.


If/Then Conditionals
--------------------

### Checking that a File or Folder Exists

    IF EXIST "temp.txt" (
        ECHO found
    ) ELSE (
        ECHO not found
    )

    IF NOT EXIST "temp.txt" ECHO not found

NOTE: It’s a good idea to always quote both operands (sides) of any IF check. This avoids nasty bugs when a variable doesn’t exist, which causes the the operand to effectively disappear and cause a syntax error.

### Checking If A Variable Is Not Set

    IF "%var%"=="" (SET var=default value)

Or

    IF NOT DEFINED var (SET var=default value)

### Checking If a Variable Matches a Text String

    SET var=Hello, World!

    IF "%var%"=="Hello, World!" (
        ECHO found
    )
    Or with a case insensitive comparison

    IF /I "%var%"=="hello, world!" (
        ECHO found
    )

### Artimetic Comparisons

    SET /A var=1
    IF /I "%var%" EQU "1" ECHO equality with 1
    IF /I "%var%" NEQ "0" ECHO inequality with 0
    IF /I "%var%" GEQ "1" ECHO greater than or equal to 1
    IF /I "%var%" LEQ "1" ECHO less than or equal to 1

### Checking a Return Code

    IF /I "%ERRORLEVEL%" NEQ "0" (
        ECHO execution failed
    )


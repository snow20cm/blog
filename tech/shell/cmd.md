+++
Categories = ["Notes"]
Description = "Batch Script Notes"
Tags = ["shell"]
date = "2015-03-17T16:55:59-06:00"
title = "Batch Script Notes"

+++


Batch File Names and File Extensions
====================================

Assuming you are using Windows XP or newer, I recommend saving your batch files with the file extension .cmd. 

### Running your Batch File
I usually run the script as `%COMPSPEC% /C /D "C:\Users\User\SomeScriptPath.cmd" Arg1 Arg2 Arg3` This runs your script in a new command prompt child process. The `/C` option instructs the child process to quit when your script quits. The /D disables any auto-run scripts (this is optional, but, I use auto-run scripts). The reason I do this is to keep the command prompt window from automatically closing should my script, or a called script, call the EXIT command. The EXIT command automatically closes the command prompt window unless the EXIT is called from a child command prompt process. This is annoying because you lose any messages printed by your script.

### Comments

    REM This is a comment
    :: This is a comment too(usually, but not in for loop)

### Silencing Display of Commands in Batch Files

    @echo off
    echo on


Variables
=========

### Variable Declaration

- DOS does not require declaration of variables.
- The value of undeclared/uninitialized variables is an empty string, or "". 

### Variable Assignment

The SET command assigns a value to a variable.DOS is case insensitive.

    SET foo=bar
    SET /A four=2+2      /A switch supports arthimetic operations during assigments

- local variables  --> lowercase
- system variables --> uppercase
- command          --> uppercase

### Reading the Value of a Variable

    C:\> SET foo=bar
    C:\> ECHO %foo%
    bar

### Listing Existing Variables

    C:\> SET

### Variable Scope (Global vs Local)

- By default, variables are global to your entire command prompt session.
- Call the `SETLOCAL` command to make variables local to the scope of your script.
- After calling `SETLOCAL`, any variable assignments revert upon calling ENDLOCAL, calling EXIT, or when execution reaches the end of file (EOF) in your script.

### Command Line Arguments to Your Script

%0-9 representing the ordinal position of the argument.

- %0 name of the batch file
- %10 is illegal. Instead using `shift`. Then %9 moves to %8 and 10th argument moves to %9

### Tricks with Command Line Arguments

- `%~1` removes quotes from `%1`
- `%~f1` is the full path to the folder of `%1`
- `%~fs1` is the full path to the folder of `%1`, but in DOS 8.3 short format
- `%~dp1` is the full path to the parent folder of `%1`.
- `%~nx1` is just the file name and file extension of `%1`.

Return Code
===========

- `zero` successful
- `non-zero` failed

### Checking Return Codes In Your Script Commands

    IF %ERRORLEVEL% NEQ 0 (
        REM do something here to address the error
    )

    IF ERRORLEVEL 1 (
        REM do something here to address the error
    )

The `ERRORLEVEL 1` statement is true when the return code is any number equal to or greater than 1. But false for minus number. Built-in DOS commands like ECHO, IF, and SET preserve the existing value of %ERRORLEVEL%.

### Conditional Execution Using the Return Code

To execute a follow-on command after sucess, we use the && operator:

    SomeCommand.exe && ECHO SomeCommand.exe succeeded!

To execute a follow-on command after failure, we use the || operator:

    SomeCommand.exe || EXIT /B 1
    SomeCommand.exe || GOTO :EOF

stdin, stdout, stderr
=====================

    DIR SomeFile.txt > output.txt 2>&1
    TYPE < SomeFile.txt

### Suppressing Program Output

    PING 127.0.0.1 > NUL   :: simulate sleep

### A Cool Party Trick

You can quickly create a new text file, say maybe a batch script, from just the command line by redirecting the command prompt’s own stdin, called `CON`, to a text file. When you are done typing, hit `CTRL+Z`, which sends the end-of-file (EOF) character.
    TYPE CON > output.txt



+++
categories = ["general"]
date = "2015-08-26T10:36:11-06:00"
tags = ["document"]
title = "singals and slots"

+++
## connect()

	connect(sender, SIGNAL(signal), receiver, SLOT(slot));

- `sender` and `receiver` are pointers to QObjects
- `signal` and `slot` are function signatures **without parameter names**.
-  The SIGNAL()  and SLOT()  macros essentially convert their argument to a string.
- A signal can be connected to another signal
- If the parameter types are incompatible, or if the signal or the slot doesn't exist, Qt will issue a warning at run-time if the application is built in debug mode. Similarly, Qt will give a warning if parameter names are included in the signal or slot signatures.


## Qt's Meta-Object System

The mechanism works as follows:

- The Q\_OBJECT macro declares some introspection functions that must be implemented in every QObject subclass: metaObject() , tr() , qt\_metacall() , and a few more.
- Qt's moc tool generates implementations for the functions declared by Q\_OBJECT and for all the signals.
- QObject member functions such as connect()  and disconnect()  use the introspection functions to do their work.
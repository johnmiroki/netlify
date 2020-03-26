title: On Exceptions (WIP)
author: John Miroki
tags: []
categories:
  - Q&A
date: 2018-06-15 10:48:00
---
## Scenario:

Spring MVC application, with three tiers from bottom up: DAO, Service and Controller.

## Things to consider:
* In principle, unchecked exceptions indicate bugs, and checked exceptions indicate problems outside your control[1].
* Checked exception is a unique feature only in Java. It's experimental and hard to get right, to put it anther way, people have controversial opinions on what's *right*. 
* Unchecked exceptions are like exceptions in other languages, such as in JavaScript or Python. 
* One can think of checked exceptions as an alternative type that a method can return. Together with the normal type, exceptions form a union type.


## Patterns

## Anti-patterns
* Catch an exception, log it, and rethrow it (or wrap it in a new exception), cuz "This is cumbersome, error prone (it's easy to lose the stacktrace) and serves no useful purpose"[2].

## Goal/Requirements:

* **required** Keep app from crashing, ie exceptions don't bubble up to users in a raw state (jsp error page with stacktrace, or api not returning intentional and predefined result (resultCode, message))
* **required** Make sure exceptions get logged, preferrably:
	- logged once
    - with previous cause
    - with parameter map
* *optional* Keep influences on business logic minumum, ie clean code
* *optional* Keep concerns separated
* **required** Keep checked exceptions consistent with the tiers throwing them, eg. all classes in DAO throw DAOException, and all classes in service throw ServiceExceptiopn. This also helps with a stable signature of method, otherwise, when the underlining implement changes, the signature might also need to change accordingly.

## Design Decisions
* Validate parameters at the beginning of a method, and throw RuntimeExceptions if violation is found. Or use assertions that throw RuntimeExceptions. It's the responsibility of callers to make sure all parameters are legal.
* **Must** catch exceptions in controllers (or same tier)
	* Use cross-cut exception handlers if possible
    	- how to return error to pages in handlers?
* 

## Relative 
* Handling exceptions in a multithread application

Reading materials:
* [Effective Java Exception](http://www.oracle.com/technetwork/java/effective-exceptions-092345.html)
* [Does Java need Checked Exceptions?](http://www.mindview.net/Etc/Discussions/CheckedExceptions)



[1]:https://github.com/google/guava/wiki/ThrowablesExplained
[2]:Expert One-on-one J2EE Design And Development by Rod Johnson

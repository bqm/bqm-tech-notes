Java Virtual Machine Specification (V8)
=======================================

https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf

Table of contents

???


0. Preface
==========

* To maximize binary compability, JVM 8 now specifies default methods directly in the JVM rather than relying on the compiler magic.
* JSR 235, Lambda Expressions for the Java Programming Language.
* How best to integrate default methods into the constant pool method structures? in the method and interface method resolution algorithms? in the
bytecode instruction set?
* JSR 335 also introduced private and static methods in interfaces at the class file level
* JVM 8: support for method parameter names at run time & storing such names in the class file structure with an API (java.lang.reflect.Parameter)
* First edition of the class specification: 6 attributes of which 3 attributes were deemed critical.
* Now, 23 attributes in the class specification of which 5 are deemed critical. Which means that the other attributes exist primarily for libraries & tools
rather the JVM directly.


1. Introduction
===============

A bit of history
----------------

* Java is general purpose, concurrent, object-oriented language.
* Similar to C & C++ but it omits many of the features that make C & C++ complex, confusing & unsafe.
* Initially started for networked consumer devices.
* Designed to support multiple host architectures & to allow secure delivery of software components.
* To meet these requirements, compiled code had to survive transport across networks, operate on any client, and assure the client that it was safe to run.
* The HotJava browser first showcased the interesting properties of the Java programming language and platform by making it possible to embed programs inside HTML pages.
* The JVM knows nothing of the Java PL, only of a particular binary format, the class file format. A class file contains JVM instructions (or bytecodes) and a symbol table, as well as other ancillary information.
* For the sake of security, the Java Virtual Machine imposes strong syntactic and structural constraints on the code in a class file. However, any language with functionality that can be expressed in terms of a valid class file can be hosted by the Java Virtual Machine.

Organization of the Specification
---------------------------------

* Chapter 2 gives an overview of the JVM.
* Chapter 3 introduces compilation of code written in Java into the instruction set of the JVM.
* Chapter 4 specifies the class file format, the hardware- and operating system independent binary format used to represent compiled classes and interfaces.
* Chapter 5 specifies the start-up of the JVM and the loading, linking, and initialization of classes and interfaces.
* Chapter 6 specifies the instruction set of the JVM, presenting the instructions in alphabetical order of opcode mnemonics.
* Chapter 7 gives a table of the JVM indexed by the opcode value.


2. The structure of the JVM
===========================

It does not describe any particular implementation of the Java Virtual Machine. Implementation is left to the implementors, so
things like memory layout of run-time data areas, garbage collector algorithm (GC) etc. are not specified.

Class file format
-----------------

* Compiled code to be executed by the JVM is represented using a hardware and operating system (OS) independent binary format.
* The name of the file format is the `class` format.

Data types
----------

* Like Java, the JVM has 2 kind of types: primitives types & reference types.
* 2 kind of values can be stored in variables: primitive values & reference values.
* The JVM expects that all type checking is done prior to run time, typically by a compiler.
* Values of primitive types need not be tagged or otherwise be inspectable to determine their types at run time, or to be distinguished from values of reference types.
* Instead, the instruction set of the JVM distinguishes its operand using instructions intended for specific types.
* For instance, iadd, ladd, fadd, and dadd are JVM instructions for {int, long, float, double} respectively.
* An object is either a dynamically allocated class instance or an array.
* A reference to an object is considered to have Java Virtual Machine type reference.
* Values of type reference can be thought of as pointers to objects.
* More than one reference to an object may exist. Objects are always operated on, passed, and tested via values of type reference.


Primitive Types and Values
--------------------------

* Numeric types, boolean type, returnAddress type.
* Numeric types are integral types, floating point types.
* Integral types
** byte (8 bit signed two's complement integers) & whose default value is 0 (from -128 to 127)
** short (16 bit signed two's complement integers) & whose default value is 0 (from -32768 to 32767)
** int (32 bit signed two"s complement integers) & whose default value is 0 (from -2147483648 to 2147483647)
** long (64 bit signed two's complement integers)  & whose default value is 0 (from -9223372036854775808 to 9223372036854775807)
** char (16 bit unsigned integers) representing Unicode code points in the Basic Multilingual Plane, encoded with UTF-16 & whose default value is the null code point (from 0 to 65535)
* Floating point types
** float
** double

returnAddress
-------------

* The values of the returnAddress type are pointers to the opcodes of the JVM instructions.
* Of the primitive types, only the returnAddress type is not directly associated with a Java programming language type.


IEEE 754
--------

* Format is (1 + e + m) bits where value = sign * coefficient *  2^(exponent - delta)
* coefficient is encoded using m bits, exponent is encoded on e bits & delta is equal to 2^(e-1) - 1
* coefficient = (1 + 2^(-23) * number_encoded)
* Special cases:
** -0
** +0
** - infinity
** + infinity
** NaN (not a number)
* float is 32 bits & e = 8 bits & m = 23 bits. delta = 2 ^ 7 -1 = 127

(more to be added here)

boolean type
------------
* JVM provides a very limited set of operations on boolean values.
* Instead, expressions in Java that operate on boolean use the JVM int type.
* JVM supports boolean arrays, `newarray` instruction
* Arrays of type boolean are modified using the `byte` instructions `baload` & `bastore`
* true is encoded as 1 & false is encoded as false


2.4 Reference types & values
----------------------------

* 3 kinds of reference types: `class` types, `array` types and `interface` types.
* which corresponds to class instances, arrays or class instances or arrays that implement interfaces respectively.
* `array` type = component type with a single dimension (whose length not given by the type)
* that component type can be also be an array type.
* The final component type that is not an array type is called the element type which can be {primitive type, class type, interface type}
* A reference can also be a `null` reference.


2.5 Runtime data areas
----------------------

**2.5.1 pc register**

* Each JVM thread has its own pc register (program counter register).
* At any point a JVM thread is executing the code of a single method.
* If that method is not `native`, the pc register contains the address of the JVM instruction being executed
* If the method is `native`, the pc register will be undefined.

**2.5.2 JVM stacks**

* Each JVM thread has its private stack which stores store frames.
* A stack holds local variables & partial results & plays a role in method invocation & return.
* JVM stack is never manipulated except via push & pop so frames must be heap allocated
* A JVM stack size can be fixed at creation of the thread or be dynamically expanded.
* If the computation requires a larger stack than permitted then the JVM will throw an StackOverflowException
* If the expansion failed, the JVM will throw a OutOfMemory

**2.5.3 Heap**

* Heap is shared across all JVM threads
* Heap is created on the VM start up
* Heap storage for objects is reclaimed by an automatic storage management system (garbage collector)
* Objects are never explicitly deallocated





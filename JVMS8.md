Java Virtual Machine Specification (V8)
=======================================

https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf

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

2.1 Class file format
---------------------

* Compiled code to be executed by the JVM is represented using a hardware and operating system (OS) independent binary format.
* The name of the file format is the `class` format.

2.2 Data types
--------------

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


2.3 Primitive Types and Values
------------------------------

* Numeric types, boolean type, returnAddress type.
* Numeric types are integral types, floating point types.
* Integral types
  * byte (8 bit signed two's complement integers) & whose default value is 0 (from -128 to 127)
  * short (16 bit signed two's complement integers) & whose default value is 0 (from -32768 to 32767)
  * int (32 bit signed two"s complement integers) & whose default value is 0 (from -2147483648 to 2147483647)
  * long (64 bit signed two's complement integers)  & whose default value is 0 (from -9223372036854775808 to 9223372036854775807)
  * char (16 bit unsigned integers) representing Unicode code points in the Basic Multilingual Plane, encoded with UTF-16 & whose default value is the null code point (from 0 to 65535)
* Floating point types
  * float
  * double

**returnAddress**


* The values of the returnAddress type are pointers to the opcodes of the JVM instructions.
* Of the primitive types, only the returnAddress type is not directly associated with a Java programming language type.


**IEEE 754**

* Format is (1 + e + m) bits where value = sign * coefficient *  2^(exponent - delta)
* coefficient is encoded using m bits, exponent is encoded on e bits & delta is equal to 2^(e-1) - 1
* coefficient = (1 + 2^(-23) * number_encoded)
* Special cases:
  * -0
  * +0
  * \- infinity
  * \+ infinity
  * NaN (not a number)
* float is 32 bits & e = 8 bits & m = 23 bits. delta = 2 ^ 7 -1 = 127

(more to be added here)

**2.3.4 boolean type**

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

* Each JVM thread has its private stack which stores frames.
* A stack holds local variables & partial results & plays a role in method invocation & return.
* JVM stack is never manipulated except via push & pop so frames must be heap allocated
* A JVM stack size can be fixed at creation of the thread or be dynamically expanded.
* If the computation requires a larger stack than permitted then the JVM will throw an StackOverflowException
* If the expansion failed, the JVM will throw a OutOfMemoryError exception

**2.5.3 Heap**

* Heap is shared across all JVM threads
* Heap is created on the VM start up
* Heap storage for objects is reclaimed by an automatic storage management system (garbage collector)
* Objects are never explicitly deallocated
* Heap may be of fixed size or may be expanded / contracted as required
* If the computation requires more heap than can be made available by the automatic storage management system, the JVM will throw an OutOfMemoryError exception


**2.5.4 Method area**

* JVM has a method area that is common across all threads
* It stores per class structures such as the run time constant pool, field & method data, code for methods & constructors, including the special methods (\$2.9)
* Method area is created on VM startup
* Method is logically part of the heap but implementations may choose to GC or not.
* JVM implementation may let the programmer choose a fixed vs variable size for the method area
* If memory can not be made in the method area, JVM will throw an OutOfMemoryError

**2.5.5 Run-time Constant pool**

* A run-time constant pool is a per class or per interface representation of the `constant_pool` table in a `class` file.
* It contains several constants, from numeric literal known at compile time to method & field references that must be resolved at run time.
* It serves a similar purpose than a symbol table for a conventional programming language although it has a wider variety than a typical symbol table.
* Run-time constant pool is allocated from the method area
* It's constructed when the class or interface is created by the JVM
* If it can't be created, the JVM will throw a OutOfMemoryError exception

**2.5.6 Native Method stacks**

* JVM implementation may use conventional stacks to support `native` methods, a.k.a "C stacks"
* If supplied, native method stacks are typically allocated per thread when each thread is created.
* Possible exceptions: StackOverflowError, OutOfMemoryError

2.6 Frames
----------

* A frame is used to store data & partial results | perform dynamic linking | return values for methods | dispatch exceptions.
* A frame is created every time a method is invoked.
* A frame is destroyed when its method invocation completes, whether that completion is abrupt or normal.
* Frames are allocated from the stack of the JVM thread creating it.
* Each frame has its own variables, its own operand stack, a reference to the run-time constant pool of the class of the current method.
* Size of the local variable array & operand stack are determined at compile time & are supplied along with the code for the method associated with the frame.
* Size of the frame depends only on the data implementation of the JVM & the memory for these data structures can be allocated simultaneously on method invocation.
* Only one frame is active at any time in a given thread.
* This frame is called current frame & associated method is current method. Class of the associated method is called current class.
* Operations on local variables & the operand stack are typically with reference to the current frame.
* A frame ceases to be current if its method invokes another method or if its current method completes.
* When a method is invoked, a new frame is created & becomes current when control transfers to the new method.
* On method return, the current frame passes back the result of its method invocation, if any, to the previous frame. Current frame is then discarded.
* Note: A frame created by a thread is local to that thread & can't be referenced in another thread.

**2.6.1 Local variables**

* Each frame contains an array of variables called local variables.
* Length of a local variable is determined at compile time & supplied in the binary representation of a class or interface along with the code of the method associated with the frame.
* A local variable can either be `boolean`, `byte`, `char`, `short`, `int`, `float`, `reference`, `returnAddress`.
* A pair of local variables can either be a `long` or `double`
* Local variables are addressed by indexing. Index of first local variable is 0. 
* A `long` or `double` are using two consecutive local variables.
* JVM uses local variables to pass parameters on method invocation.
* On class method invocation, any parameters are passed in consecutive local variables starting from local variable 0.
* On instance method invocation, local variable 0 is always used to pass a reference to the object on which instance method is being invoked (`this` in Java).

**2.6.2 Operand stacks**

* Each frame contains a LIFO stack known as operand stack.
* Maximum depth is determined at compile time & is supplied along with the code in the frame.
* Operand stack is empty when frame is created.
* JVM supplies instructions to load constants or values from local variables or fields onto the operand stack.
* Other JVM instructions take operands from the operand stack, operate on them & push back the result.
* Example: `iadd` adds 2 integers. It requires that the 2 integers should be on top of the stack, pushed by previous instructions. They are added & then result is pushed on the top.

**2.6.3 Dynamic linking**

* Each frame contains a reference to the run-time constant pool (\$2.5.5) to support dynamic linking.
* `class` file code refers to method to be invoked & variables to be accessed via symbolic references.
* Dynamic linking translate these symbolic method references into concrete method references.

**2.6.4 Normal method invocation completion**

* A method invocation completes normally if that invocation does not cause an exception to be thrown.

**2.6.5 Abrupt method invocation completion**

* Method invocation completes abruptly if JVM instruction throws an exception & method is not handling it.
* For instance, `athrow` causes an exception to be explicitly thrown.

2.7 Representation of Objects
-----------------------------

* JVM does not mandate any particular internal structure.
* In some Oracle's implementation, a reference to a class instance is a pointer to a handle that is itself a pair of pointers.
* One to a table containing the methods of the object, one to the `Class` object that represents the type & other to the memory allocated from the heap for the obj data.

2.8 Floating-Point Arithmetic
-----------------------------

* JVM implements a subset of the floating-point arithmetic specific in IEEE 754.

**2.8.1 Differences**

* JVM do not throw exceptions when dividing by 0, overflow, underflow or inexact.
* No NaN, no signaling floating-point comparisons.
* JVM doesn't allow a way to choose the rounding mode.

**2.8.2 Floating-point modes**

* Every method can be either floating point strict or not (set via `ACC_STRICT` flag of the `access_flags` item of the `method_info`).
* If a float-extended-exponent value set is supported (\$2.3.2), values of type `float` on an operand stack that is not FP-strict may range over that value set except where prohibited by value set conversion (\$2.8.3). Same for `double`.
* In all other contexts & regardless of the floating point mode, `float` & `double` may range over the float value set & double value set respectively.

**2.8.3 Value set conversion**

* Won't be detailed here: some rules around when to use a conversion between the extended & standard value sets.

2.9 Special methods
-------------------

* Every constructor written in Java appears as an instance initialization method that has a special name `<init>`.
* `<init>` is not a valid identifier & can't be used directly in Java (same for `clinit`).
* Instance initialization may be invoked only within the JVM by `invokespecial` instruction & may be invoked on uninstantiated class instances.
* An instance initialization takes on the access permissions of the constructor from which it was derived.
* A class or interface has at most one class or interface initialization method & is initialized (\$5.5) by invoking that method.
* The static initializer of a class is called `<clinit>`, takes not argument and is void (\$4.3.3).
* Other methods called `<clinit>` are of no consequence.
* For version number > 51.0, method must have its `ACC_STATIC` flag set in order to be considered the init method.
* A method is signature polymorphic if all the following are true:
  * Declared in `java.lang.invoke.MethodHandle` class.
  * Single formal parameter of type Object[]
  * Returns an Object
  * It has the `ACC_VARARGS` and `ACC_NATIVE` flags set
* JVM gives special treatement to signature polymorphic methods in the `invokevirtual` instruction.
* See `java.lang.invoke` for more info

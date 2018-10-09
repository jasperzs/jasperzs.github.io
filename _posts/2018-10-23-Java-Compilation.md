---
layout:     post
title:      How Java Compilation Works
subtitle:   Understanding Compilation
date:       2018-10-23
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Compilation
---

## 1. Overview
In Java, programs are not compiled into executable files; they are compiled into bytecode (as discussed earlier), which the JVM (Java Virtual Machine) then executes at runtime. Java source code is compiled into bytecode when we use the `javac` compiler. The bytecode gets saved on the disk with the file extension `.class`. When the program is to be run, the bytecode is converted, using the `just-in-time (JIT)` compiler. The result is machine code which is then fed to the memory and is executed.

Java code needs to be compiled twice in order to be executed:
1. Java programs need to be compiled to bytecode through `javac`.
2. When the bytecode is run, it needs to be converted to machine code via `JIT`.

The Java classes/bytecode are compiled to machine code and loaded into memory by the JVM when needed the first time. This is different from other languages like C/C++ where programs are to be compiled to machine code and linked to create an executable file before it can be executed.

## 2. Quick Compilation Procedure
1. Write a `HelloWorld.java` application
```java
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World!");
  }
}
```
2. Run the command `javac HelloWorld.java`
3. Run the application by running the command `java HelloWorld`

## 3. Automatic Compilation of Dependent Classes
In Java, if you have used any reference to any other java object, then the class for that object will be automatically compiled, if that was not compiled already. These automatic compilations are nested, and this continues until all classes are compiled that are needed to run the program. So it is usually enough to compile only the high level class, since all the dependent classes will be automatically compiled.
> __javac ... MainClass.java__

However, you can't rely on this feature if your program is using reflection to create objects, or you are compiling for servlets or for a "jar", package. In these cases you should list these classes for explicit compilation.
> __javac ... MainClass.java ServletOne.java ...__

## 4. Packages, Subdirectories, and Resources
Each Java top level class belongs to a package. This may be declared in a `package` statement at the beginning of the file; if that is missing, the class belongs to the unnamed package.

For compilation, the file must be in the right directory structure. A file containing a class in the unnamed package must be in the current/root directory; if the class belongs to a package, it must be in a directory with the same name as the package.

The convention is that package names and directory names corresponding to the package consist of only lower case letters.

### 4.1 Top level package
A class with this package declaration
```java
package example;
```
has to be in a directory named `example`.

### 4.2 Subpackages
A class with this package declaration
```java
package org.wikibooks.en;
```
has to be in a directory named `en` which has to be a sub-directory of `wikibooks` which in turn has to be a sub-directory of `org` resulting in `org/wikibooks/en` on Linux or `org\wikibooks\en` on Windows.

Java programs often contain non-code files such as images and properties files. These are referred to generally as resources and stored in directories local to the classes in which they're used. For example, if the class `com.example.ExampleApp` uses the `icon.png` file, this file could be stored as `/com/example/resources/icon.png`. These resources present a problem when a program is compiled, because `javac` does not copy them to wherever the `.class` files are being compiled to (see above); it is up to the programmer to move the resource files and directories.

## 5. Filename Case
The Java source file name must be the same as the public class name that the file contains. There can be only one public class defined per file. The Java class name is case sensitive, as is the source file name.

The naming convention for the class name is for it to start with a capital letter.

## 6. Compiler Options
### 6.1 Debugging and Symbolic Information
To debug into Java system classes such as String and ArrayList, you need a special version of the JRE which is compiled with "debug information". The JRE included inside the JDK provides this info, but the regular JRE does not. Regular JRE does not include this info to ensure better performance.

> Modern compilers do a pretty good job converting your high-level code, with its nicely indented and nested control structures and arbitrarily typed variables into a big pile of bits called machine code (or bytecode in case of Java), the sole purpose of which is to run as fast as possible on the target CPU (virtual CPU of your JVM). Java code gets converted into several machine code instructions. Variables are shoved all over the place – into the stack, into registers, or completely optimized away. Structures and objects don’t even exist in the resulting code – they’re merely an abstraction that gets translated to hard-coded offsets into memory buffers.
> So how does a debugger know where to stop when you ask it to break at the entry to some function? How does it manage to find what to show you when you ask it for the value of a variable? The answer is – debugging information.
> Debugging information is generated by the compiler together with the machine code. It is a representation of the relationship between the executable program and the original source code. This information is encoded into a pre-defined format and stored alongside the machine code. Many such formats were invented over the years for different platforms and executable files.

Symbolic Information : Symbolic resolution is done at class loading time at linking resolution step. It is the process of replacing symbolic references from the type with direct references. It is done by searching into method area to locate the referenced entity

## 7. The JIT compiler
The Just-In-Time (JIT) compiler is the compiler that converts the byte-code to machine code. It compiles byte-code once and the compiled machine code is re-used again and again, to speed up execution. Early Java compilers compiled the byte-code to machine code each time it was used, but more modern compilers cache this machine code for reuse on the machine. Even then, java's JIT compiling was still faster than an "interpreter-language", where code is compiled from high level language, instead of from byte-code each time it was used.

The standard JIT compiler runs on demand. When a method is called repeatedly, the JIT compiler analyzes the bytecode and produces highly efficient machine code, which runs very fast. The JIT compiler is smart enough to recognize when the code has already been compiled, so as the application runs, compilation happens only as needed. As Java applications run, they tend to become faster and faster, because the JIT can perform runtime profiling and optimization to the code to meet the execution environment. Methods or code blocks which do not run often receive less optimization; those which run often (so called hotspots) receive more profiling and optimization.

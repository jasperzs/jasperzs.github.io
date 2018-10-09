---
layout:     post
title:      Guide to Java Native Interface
subtitle:   Walk through JNI
date:       2019-01-21
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Introduction
As we know, one of the main strengths of Java is its portability – meaning that once we write and compile code, the result of this process is platform-independent bytecode.

Simply put, this can run on any machine or device capable of running a Java Virtual Machine, and it will work as seamlessly as we could expect.

However, sometimes __we do actually need to use code that’s natively-compiled for a specific architecture__.

There could be some reasons for needing to use native code:
- The need to handle some hardware
- Performance improvement for a very demanding process
- An existing library that we want to reuse instead of rewriting it in Java.

__To achieve this, the JDK introduces a bridge between the bytecode running in our JVM and the native code__ (usually written in C or C++).

The tool is called Java Native Interface. In this article, we’ll see how it is to write some code with it.

## 2. How It Works

### 2.1 Native Methods: The JVM Meets Compiled Code
Java provides the _native_ keyword that’s used to indicate that the method implementation will be provided by a native code.

Normally, when making a native executable program, we can choose to use static or shared libs:
- Static libs – all library binaries will be included as part of our executable during the linking process. Thus, we won’t need the libs anymore, but it’ll increase the size of our executable file.
- Shared libs – the final executable only has references to the libs, not the code itself. It requires that the environment in which we run our executable has access to all the files of the libs used by our program.

The latter is what makes sense for JNI as we can’t mix bytecode and natively compiled code into the same binary file.

Therefore, our shared lib will keep the native code separately within its _.so/.dll/.dylib_ file (depending on which Operating System we’re using) instead of being part of our classes.

__The native keyword transforms our method into a sort of abstract method:__
```java
private native void aNativeMethod();
```

With the main difference that __instead of being implemented by another Java class, it will be implemented in a separated native shared library.__

A table with pointers in memory to the implementation of all of our native methods will be constructed so they can be called from our Java code.

### 2.2. Components Needed
Here’s a brief description of the key components that we need to take into account. We’ll explain them further later in this article
- Java Code – our classes. They will include at least one native method.
- Native Code – the actual logic of our native methods, usually coded in C or C++.
- JNI header file – this header file for C/C++ (include/jni.h into the JDK directory) includes all definitions of JNI elements that we may use into our native programs.
- C/C++ Compiler – we can choose between GCC, Clang, Visual Studio, or any other we like as far as it’s able to generate a native shared library for our platform.

### 2.3. JNI Elements In Code (Java And C/C++)
Java elements:
- “native” keyword – as we’ve already covered, any method marked as native must be implemented in a native, shared lib.
- System.loadLibrary(String libname) – a static method that loads a shared library from the file system into memory and makes its exported functions available for our Java code.

C/C++ elements (many of them defined within jni.h)
- JNIEXPORT- marks the function into the shared lib as exportable so it will be included in the function table, and thus JNI can find it
- JNICALL – combined with JNIEXPORT, it ensures that our methods are available for the JNI framework
- JNIEnv – a structure containing methods that we can use our native code to access Java elements
- JavaVM – a structure that lets us manipulate a running JVM (or even start a new one) adding threads to it, destroying it, etc…

### 3. Hello World JNI
Next, __let’s look at how JNI works in practice.__

In this tutorial, we’ll use C++ as the native language and G++ as compiler and linker.

We can use any other compiler of our preference, but here’s how to install G++ on Ubuntu, Windows, and MacOS:
- Ubuntu Linux – run command _“sudo apt-get install build-essential”_ in a terminal
- Windows – Install [MinGW](http://www.mingw.org/)
- MacOS – run command _“g++”_ in a terminal and if it’s not yet present, it will install it.

### 3.1. Creating the Java Class
Let’s start creating our first JNI program by implementing a classic “Hello World”.

To begin, we create the following Java class that includes the native method that will perform the work:
```java
package com.baeldung.jni;

public class HelloWorldJNI {

    static {
        System.loadLibrary("native");
    }

    public static void main(String[] args) {
        new HelloWorldJNI().sayHello();
    }

    // Declare a native method sayHello() that receives no arguments and returns void
    private native void sayHello();
}
```

As we can see, _we load the shared library in a static block_. This ensures that it will be ready when we need it and from wherever we need it.

Alternatively, in this trivial program, we could instead load the library just before calling our native method because we’re not using the native library anywhere else.

### 3.2 Implementing a Method in C++
Now, we need to create the implementation of our native method in C++.

Within C++ the definition and the implementation are usually stored in _.h_ and _.cpp_ files respectively.

First, __to create the definition of the method, we have to use the -h flag of the Java compiler:__
```java
javac -h . HelloWorldJNI.java
```

This will generate a _com_baeldung_jni_HelloWorldJNI.h_ file with all the native methods included in the class passed as a parameter, in this case, only one:
```c++
JNIEXPORT void JNICALL Java_com_baeldung_jni_HelloWorldJNI_sayHello
  (JNIEnv *, jobject);
```

__As we can see, the function name is automatically generated using the fully qualified package, class and method name.__

Also, something interesting that we can notice is that we’re getting two parameters passed to our function; a pointer to the current _JNIEnv_; and also the Java object that the method is attached to, the instance of our _HelloWorldJNI_ class.

Now, we have to create a new _.cpp_ file for the implementation of the _sayHello_ function. __This is where we’ll perform actions that print “Hello World” to console.__

We’ll name our _.cpp_ file with the same name as the .h one containing the header and add this code to implement the native function:
```c++
JNIEXPORT void JNICALL Java_com_baeldung_jni_HelloWorldJNI_sayHello
  (JNIEnv* env, jobject thisObject) {
    std::cout << "Hello from C++ !!" << std::endl;
}
```

### 3.3. Compiling And Linking
__At this point, we have all parts we need in place and have a connection between them.__

We need to build our shared library from the C++ code and run it!

To do so, we have to use G++ compiler, __not forgetting to include the JNI headers from our Java JDK installation.__

Ubuntu version:
```
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/linux com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

Windows version:
```
g++ -c -I%JAVA_HOME%\include -I%JAVA_HOME%\include\win32 com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

MacOS version:
```
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/darwin com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

Once we have the code compiled for our platform into the file _com_baeldung_jni_HelloWorldJNI.o_, we have to include it in a new shared library. __Whatever we decide to name it is the argument passed into the method *System.loadLibrary*__.

We named ours “native”, and we’ll load it when running our Java code.

The G++ linker then links the C++ object files into our bridged library.

Ubuntu version:
```
g++ -shared -fPIC -o libnative.so com_baeldung_jni_HelloWorldJNI.o -lc
```

Windows version:
```
g++ -shared -o native.dll com_baeldung_jni_HelloWorldJNI.o -Wl,--add-stdcall-alias
```

MacOS version:
```
g++ -dynamiclib -o libnative.dylib com_baeldung_jni_HelloWorldJNI.o -lc
```

And that’s it!

We can now run our program from the command line.

However, __we need to add the full path to the directory containing the library we’ve just generated.__ This way Java will know where to look for our native libs:
```
java -cp . -Djava.library.path=/NATIVE_SHARED_LIB_FOLDER com.baeldung.jni.HelloWorldJNI
```

Console output:
```
Hello from C++ !!
```

## 4. Using Advanced JNI Features
Saying hello is nice but not very useful. __Usually, we would like to exchange data between Java and C++ code and manage this data in our program.__

### 4.1. Adding Parameters To Our Native Methods
We’ll add some parameters to our native methods. Let’s create a new class called _ExampleParametersJNI_ with two native methods using parameters and returns of different types:
```
private native long sumIntegers(int first, int second);

private native String sayHelloToMe(String name, boolean isFemale);
```

And then, repeat the procedure to create a new .h file with “javac -h” as we did before.

Now create the corresponding .cpp file with the implementation of the new C++ method:
```c++
...
JNIEXPORT jlong JNICALL Java_com_baeldung_jni_ExampleParametersJNI_sumIntegers
  (JNIEnv* env, jobject thisObject, jint first, jint second) {
    std::cout << "C++: The numbers received are : " << first << " and " << second << std::endl;
    return (long)first + (long)second;
}
JNIEXPORT jstring JNICALL Java_com_baeldung_jni_ExampleParametersJNI_sayHelloToMe
  (JNIEnv* env, jobject thisObject, jstring name, jboolean isFemale) {
    const char* nameCharPointer = env->GetStringUTFChars(name, NULL);
    std::string title;
    if(isFemale) {
        title = "Ms. ";
    }
    else {
        title = "Mr. ";
    }

    std::string fullName = title + nameCharPointer;
    return env->NewStringUTF(fullName.c_str());
}
...
```

__We’ve used the pointer _*env_ of type _JNIEnv_ to access the methods provided by the JNI environment instance.__

_JNIEnv_ allows us, in this case, to pass Java _Strings_ into our C++ code and back out without worrying about the implementation.

__We can check the equivalence of Java types and C JNI types into [Oracle official documentation](https://docs.oracle.com/javase/9/docs/specs/jni/types.html).__

To test our code, we’ve to repeat all the compilation steps of the previous _HelloWorld_ example.

### 4.2. Using Objects And Calling Java Methods From Native Code
In this last example, we’re going to see how we can manipulate Java objects into our native C++ code.

We’ll start creating a new class _UserData_ that we’ll use to store some user info:
```java
package com.baeldung.jni;

public class UserData {

    public String name;
    public double balance;

    public String getUserInfo() {
        return "[name]=" + name + ", [balance]=" + balance;
    }
}
```

Then, we’ll create another Java class called _ExampleObjectsJNI_ with some native methods with which we’ll manage objects of type _UserData_:
```java
...
public native UserData createUser(String name, double balance);

public native String printUserData(UserData user);
```

One more time, let’s create the _.h_ header and then the C++ implementation of our native methods on a new _.cpp_ file:
```c++
JNIEXPORT jobject JNICALL Java_com_baeldung_jni_ExampleObjectsJNI_createUser
  (JNIEnv *env, jobject thisObject, jstring name, jdouble balance) {

    // Create the object of the class UserData
    jclass userDataClass = env->FindClass("com/baeldung/jni/UserData");
    jobject newUserData = env->AllocObject(userDataClass);

    // Get the UserData fields to be set
    jfieldID nameField = env->GetFieldID(userDataClass , "name", "Ljava/lang/String;");
    jfieldID balanceField = env->GetFieldID(userDataClass , "balance", "D");

    env->SetObjectField(newUserData, nameField, name);
    env->SetDoubleField(newUserData, balanceField, balance);

    return newUserData;
}

JNIEXPORT jstring JNICALL Java_com_baeldung_jni_ExampleObjectsJNI_printUserData
  (JNIEnv *env, jobject thisObject, jobject userData) {

    // Find the id of the Java method to be called
    jclass userDataClass=env->GetObjectClass(userData);
    jmethodID methodId=env->GetMethodID(userDataClass, "getUserInfo", "()Ljava/lang/String;");

    jstring result = (jstring)env->CallObjectMethod(userData, methodId);
    return result;
}
```

Again, we’re using the _JNIEnv *env_ pointer to access the needed classes, objects, fields and methods from the running JVM.

Normally, we just need to provide the full class name to access a Java class, or the correct method name and signature to access an object method.

We’re even creating an instance of the class _com.baeldung.jni.UserData_ in our native code. __Once we have the instance, we can manipulate all its properties and methods in a way similar to Java reflection.__

We can check all other methods of JNIEnv into the [Oracle official documentation](https://docs.oracle.com/javase/9/docs/specs/jni/functions.html).

## 5. Disadvantages Of Using JNI
JNI bridging does have its pitfalls.

The main downside being the dependency on the underlying platform; __we essentially lose the “write once, run anywhere”__ feature of Java. This means that we’ll have to build a new lib for each new combination of platform and architecture we want to support. Imagine the impact that this could have on the build process if we supported Windows, Linux, Android, MacOS…

JNI not only adds a layer of complexity to our program. __It also adds a costly layer of communication__ between the code running into the JVM and our native code: we need to convert the data exchanged in both ways between Java and C++ in a marshaling/unmarshaling process.

__Sometimes there isn’t even a direct conversion between types so we’ll have to write our equivalent.__

## 6. Conclusion
Compiling the code for a specific platform (usually) makes it faster than running bytecode.

This makes it useful when we need to speed up a demanding process. Also, when we don’t have other alternatives such as when we need to use a library that manages a device.

However, this comes at a price as we’ll have to maintain additional code for each different platform we support.

That’s why it’s usually a good idea to __only use JNI in the cases where there’s no Java alternative.__

As always the code for this article is available over on [GitHub](https://github.com/eugenp/tutorials/tree/master/jni).

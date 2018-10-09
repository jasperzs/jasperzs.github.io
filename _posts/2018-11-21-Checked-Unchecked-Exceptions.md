---
layout:     post
title:      Checked vs Unchecked Exceptions in Java
subtitle:   Understanding what checked and unchecked exceptions are
date:       2018-11-21
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

In Java, there are two types of exceptions:

## 1. Checked Exceptions
__Checked__: are the exceptions that are checked at compile time. If some code within a method throws a checked exception, then the method must either handle the exception or it must specify the exception using throws keyword.

For example, consider the following Java program that opens file at location `“C:\test\a.txt”` and prints the first three lines of it. The program doesn’t compile, because the function `main()` uses `FileReader()` and `FileReader()` throws a checked exception `FileNotFoundException`. It also uses `readLine()` and `close()` methods, and these methods also throw checked exception `IOException`
```java
import java.io.*;

class Main {
    public static void main(String[] args) {
        FileReader file = new FileReader("C:\\test\\a.txt");
        BufferedReader fileInput = new BufferedReader(file);

        // Print first 3 lines of file "C:\test\a.txt"
        for (int counter = 0; counter < 3; counter++)
            System.out.println(fileInput.readLine());

        fileInput.close();
    }
}
```
Output:
> Exception in thread "main" java.lang.RuntimeException: Uncompilable source code -
  unreported exception java.io.FileNotFoundException; must be caught or declared to be
  thrown
	at Main.main(Main.java:5)

To fix the above program, we either need to specify list of exceptions using `throws`, or we need to use `try-catch` block. We have used throws in the below program. Since `FileNotFoundException` is a subclass of IOException, we can just specify `IOException` in the throws list and make the above program compiler-error-free.

Output: First three lines of file “C:\test\a.txt”

## 2. Unchecked Exceptions
__Unchecked__: are the exceptions that are not checked at compiled time. <ins>In Java exceptions under Error and RuntimeException classes are unchecked exceptions, everything else under throwable is checked.</ins>
Consider the following Java program. It compiles fine, but it throws `ArithmeticException` when run. The compiler allows it to compile, because `ArithmeticException` is an unchecked exception.
```java
class Main {
   public static void main(String args[]) {
      int x = 0;
      int y = 10;
      int z = y/x;
  }
}
```
Output:
> Exception in thread "main" java.lang.ArithmeticException: / by zero
	at Main.main(Main.java:5)
  Java Result: 1

## 3. Should we make our exceptions checked or unchecked?
_If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception_

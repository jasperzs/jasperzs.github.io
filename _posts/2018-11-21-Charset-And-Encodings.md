---
layout:     post
title:      Charset and Encodings
subtitle:   Understanding Charset and Encodings
date:       2018-11-21
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Charset
__Charset__ is a combination of a coded character set and a character-encoding scheme. A __character set__ is a defined list of characters recognized by the computer hardware and software. Each character is represented by a number. The ASCII character set, for example, uses the numbers 0 through 127. A __character encoding__ system consists of a code that pairs each character from a given set with something else—such as a bit pattern or sequence of natural numbers. A charset can be considered as a bunch of characters, the numerical codes that represent them, and a way to translate back and forth between a sequence of character codes and a sequence of bytes.

## 2. Charset vs Character Encoding
Here ‘set’ refers to the set of characters and their numbers (code points), and ‘encoding’ refers to the representation of these code points. For example, Unicode is a character set, and UTF-8 and UTF-16 are different character encodings of Unicode. For the letter A, the code point in the Unicode character set is 65 (U+0041, in hexadecimal notation). The UTF-8 encoding of that code point is a byte with value 41. In practice however, the two terms charset and character encoding are used interchangeably.

In character encoding terminology, a code point or code position is any of the numerical values that make up the code space (or code page). For example, ASCII comprises 128 code points in the range 0 hex to 7F hex.

In certain character encoding, a single character might be represented using more than one byte. If a code point is above 128 in UTF-8, it is encoded as two bytes. These bits can be decoded back.

One example of character encoding is ISO-8859-1, more commonly known as Latin-1 [ISO-8859-1]. Another example is Windows-1252 or CP-1252 used by default in the legacy components of Microsoft Windows and is a superset of ISO 8859-1, but differs from it by using displayable characters rather than control characters in the 80 to 9F (hex) range.

It is very common to mislabel Windows-1252 text with the charset label ISO-8859-1. A common result was that all the quotes and apostrophes were replaced with question marks or boxes on non-Windows operating systems. Most modern web browsers and e-mail clients treat the MIME charset ISO-8859-1 as Windows-1252 in order to accommodate such mislabeling. In the draft HTML 5 specification, it is required that documents advertised as ISO-8859-1 actually be parsed with the Windows-1252 encoding.

## 3. Charset and Java
A J2SE Runtime Environment's default charset depends on the underlying operating system and locale.

If you want to know your JRE's default charset, you can find out by invoking java.nio.charset.Charset.defaultCharset(). In my case it is windows-1252.

The String(byte[]) constructor without charset constructs a new String by decoding the specified byte array using the platform's default charset. The length of the new String is a function of the charset, and hence may not be equal to the length of the byte array. In certain character encoding, a single character might be represented using more than one byte as we have already seen. The behavior of this constructor when the given bytes are not valid in the default charset is unspecified. __Therefore, when translating between char sequences and byte sequences, you can and usually should specify a charset explicitly__. A String constructor that takes a charset name in addition to a byte array is provided for this purpose.

An Example
Consider an example. What will the following program print?
```java
public static void main(String args[]) {

  byte bytes[] = new byte[256];

   for(int i = 0; i < 256; i++)
     bytes[i] = (byte)i;

   String str = new String(bytes);

    for(int i = 0, n = str.length(); i < n; i++)
      System.out.print((int)str.charAt(i) + " ");

}
```

This program is neither guaranteed to terminate normally nor to print any particular sequence. Its behavior is completely unspecified and is dependent on the charset. One charset that will make the program print the integers from 0 to 255 in order is ISO-8859-1. In my case it windows-1252, which is a superset of ISO-8859-1 and hence I got same result.

## 4. Common Issues Related to Encoding
Here we will discuss common issues related to encoding we have faced in day to day work. Please add those to comments and I will add them here, if required, with more explanation or soulutions.

### 4.1 ï»¿ Problem
I wanted to multiply an inner element block within an xml. So I saved the xml in three parts - inner element block page, a head block page and a tail block page. However the file was not behaving as expected in the application due to some syntax error. The generated xml file was opening in browser, but there was some extra space before all inner element blocks. In notepad++, I found some special character before all page blocks, except head block. Later when I printed the file in eclipse I found the character to be ï»¿.

### 4.2 What is ï»¿ and How to Handle It
ï»¿ mark (BOM) is used at the beginning of a text file for which the format isn’t specified. BOM mark tells that a file is in UTF-8 format. BOM mark(ï»¿) in UTF-8 files is interpreted it as U+FEFF which is a noncharacter and hence invisible. For other encodings, its visible as ï»¿ sign. The problem arises when an application or text editor (Notepad, NotePad++ or anything else) interpret the file with BOM mark in ISO-8859-1 or CP1252 not in UTF-8 encoding. Many text editors like Microsoft’s default text editor i.e Notepad adds this ï»¿ BOM mark at the beginning of every text file. There is no specific use of BOM mark in UTF-8 files, but they often create problems difficult to debug.

How to get rid of this? In a text editor, say Notepad++, open the file, click ‘Encoding’ tab at the top and change encoding to ‘Encode in UTF-8 without BOM’. Save your file.

Why I got it only for non head files? My head file was already having the utf-8 encoding declaration.

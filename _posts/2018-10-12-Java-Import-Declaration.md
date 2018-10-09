---
layout:     post
title:      Java Import Declaration
subtitle:   Understanding Java Import
date:       2018-10-12
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. What is Import Declaration?
`import` declarations (not statements) are essentially short-hand enabler at the source code level: it allows you to refer to a type or a `static` member using a single identifier (e.g. `List`, `min`) as opposed to the fully qualified name (e.g. `java.util.List`, `Math.min`).

`import` declaration section is a compile-time element of the source codes, and has no presence at run-time. In JVM bytecodes, type names are always fully qualified, and unless you're using a poorly written compiler, the binary should only contain names for types that are actually being used.

> From Oracle Java SE:
> + An import declaration allows a static member or a named type to be referred to by a simple name that consists of a single identifier. Without the use of an appropriate import declaration, the only way to refer to a type declared in another package, or a static member of another type, is to use a fully qualified name.
> + A single-type-import declaration imports a single named type, by mentioning its canonical name.
> + A type-import-on-demand declaration imports all the accessible types of a named type or package as needed. It is a compile time error to import a type from the unnamed package.
> + A single static import declaration imports all accessible static members with a given name from a type, by giving its canonical name.
> + A static-import-on-demand declaration imports all accessible static members of a named type as needed.

## 2. Import package.* (Import on Demand) vs import package.SpecificType (Import Single Type)

### 2.1 Clear Vision of Dependencies
Never use `import xxx.*` to have a clear vision of dependencies.
You can know quicker that you are using a specific class of another package because it is listed right at the beginning of the source file.

### 2.2 Naming Conflict
For example:
```java
java.lang.reflect.Array
java.sql.Array
```
If you import `java.lang.reflect.*` and `java.sql.*`, you'll have a collision on the Array type, and have to fully qualify them in your code.

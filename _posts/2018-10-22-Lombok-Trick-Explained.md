---
layout:     post
title:      Lombok Trick Explained
subtitle:   Understanding Lombok
date:       2018-10-22
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Lombok
    - Java
    - Compilation
---

## Java Compilation
To understand how Project Lombok works, one must first understand how Java compilation works. OpenJDK provides an excellent overview of the compilation process. To paraphrase, compilation has 3 stages:
1. Parse and Enter
2. Annotation Processing
3. Analyse and Generate

![Java Compilation Phases]({{ site.url }}/img/java/lombok/compilationPhases.png)

In the Parse and Enter phase, the compiler parses source files into an Abstract Syntax Tree (AST). Think of the AST as the DOM-equivalent for Java code. Parsing will only throw errors if the syntax is invalid. Compilation errors such as invalid class or method usage are checked in phase 3.

In the Annotation Processing phase, custom annotation processors are invoked. This is considered a pre-compilation phase. Annotation processors can do things like validate classes or generate new resources, including source files. Annotation processors can generate errors that will cause the compilation process to fail. If new source files are generated as a result of annotation processing, then compilation loops back to the Parse and Enter phase and the process is repeated until no new source files are generated.

In the last phase, Analyse and Generate, the compiler generates class files (byte code) from the Abstract Syntax Trees generated in phase 1. As part of this process, the AST is analyzed for broken references (e.g. class not found, method not found), valid flow is checked (e.g. no unreachable statements), type erasure is performed, syntactic sugar is desugared (e.g. enhanced for loops become iterator loops) and finally, if everything is successful, class files are written out.

## Project Lombok and Compilation
Project Lombok hooks itself into the compilation process as an annotation processor. But Lombok is not your normal annotation processor. Normally, annotation processors only generate new source files whereas Lombok modifies existing classes.

The trick is that Lombok modifies the AST. It turns out that changes made to the AST in the Annotation Processing phase will be visible to the Analyse and Generate phase. Thus, changing the AST will change the generated class file. For example, if a method node is added to the AST, then the class file will contain that new method. By modifying the AST, Lombok can do things like generate new methods (getter, setter, equals, etc) or inject code into an existing method (e.g. cleaning up resources).

## Trick or Hack?
Some people call Lombok's trick a hack, and I'd agree. But don't pass judgement yet. Like any hack, you should examine the risk/reward and alternatives before determining if you are comfortable with it.

The "hack" in Lombok is that, strictly speaking, the annotation processing spec doesn't allow you to modify existing classes. The annotation processing API doesn't provide a mechanism for changing the AST of a class. The clever people at Project Lombok got around this through some unpublished APIs of javac. Since Eclipse uses an internal compiler, Lombok also needs access to internal APIs of the Eclipse compiler.

If Java officially supported compile-time AST transformations then Lombok wouldn't need to rely on backdoor APIs. This makes Project Lombok vulnerable to future changes in the JDK. There is no guarantee the private APIs won't change in a later JDK and break Project Lombok. If that happens, then you're left hoping that the guys at Lombok will be responsive about patching their library to work with the new JDK. Same thing goes for the new Eclipse compilers. Given how often we get a new version of Java, this may not be that big of an issue.

## Alternatives in Java
There are other alternatives for modifying the behavior of classes. One approach is to use byte-code manipulation at runtime via a library like CGLib or ASM. This is how Hibernate is able to do things like lazily initialize a persistent Collection the first time it is accessed. In general, this can be used to enhance the behavior of existing methods. This trick could possibly be used to implement the @Cleanup behavior in Lombok, so that a resource is closed when it goes out of scope. Runtime byte-code manipulation is no help for generating getters and setters which you intend to reference in source code.

Another approach is to use byte-code manipulation on the class files. For example, Kohsuke Kawaguchi of Hudson fame created a library called Bridge Method Injector, that helps perserve binary compatibility when changing a method's return type in a way that is source compatible but not binary compatible. Kohsuke implements this by using ASM to modify the byte-code in a class file after compilation. This trick could be used to mimic the behavior of the Getter/Setter/ToString/EqualsHashCode annotations of Lombok with one caveat: generated methods would only be visible to classes external to your library but not to classes within your library. In other words, projects that depended on classes in your library as a jar would see your getters and setters, but classes within your library would not see these getters and setters at compile time.

The trick that makes Lombok special is that the code it generates is weaved in before Analyze and Generate phase of compilation. This allows classes within the same compilation unit to have visibility to the generated methods. It appears another library called Juast may be using a similar trick (modifying the AST) to do things like operator overloading. For some developers, the immediate benefits of Lombok's approach may outweigh the potential risks.

## Alternatives outside Java
If you're willing to switch to Scala, Lombok becomes a moot point. Scala has Case classes that eliminate the getter/setter/toString/hashCode/equals boiler-plate. Scala also has Automatic Resource Management that covers Lombok's @Cleanup behavior.

Another option is Groovy if you don't care about static typing. Groovy has similar support for Scala-like Case classes. Groovy also officially supports compile-time, AST transformations.

## Final thoughts
Project Lombok can do tricks that are impossible via other dynamic code generation methods in Java but you should be aware the it uses some back-door APIs to accomplish it.

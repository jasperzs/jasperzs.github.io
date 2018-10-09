---
layout:     post
title:      Java Annotation Processing and Creating a Builder
subtitle:   Examples of Java Annotation Processor
date:       2019-03-16
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
      - Java
---

## 1. Introduction
This article is __an intro to Java source-level annotation processing__ and provides examples of using this technique for generating additional source files during compilation.

## 2. Applications of Annotation Processing
The source-level annotation processing first appeared in Java 5. It is a handy technique for generating additional source files during the compilation stage.

The source files don’t have to be Java files — you can generate any kind of description, metadata, documentation, resources, or any other type of files, based on annotations in your source code.

Annotation processing is actively used in many ubiquitous Java libraries, for instance, to generate metaclasses in QueryDSL and JPA, to augment classes with boilerplate code in Lombok library.

An important thing to note is __the limitation of the annotation processing API — it can only be used to generate new files, not to change existing ones.__

The notable exception is the [Lombok](https://projectlombok.org/) library which uses annotation processing as a bootstrapping mechanism to include itself into the compilation process and modify the AST via some internal compiler APIs. This hacky technique has nothing to do with the intended purpose of annotation processing and therefore is not discussed in this article.

## 3. Annotation Processing API
The annotation processing is done in multiple rounds. Each round starts with the compiler searching for the annotations in the source files and choosing the annotation processors suited for these annotations.

Each annotation processor, in turn, is called on the corresponding sources. If any files are generated during this process, another round is started with the generated files as its input. This process continues until no new files are generated during the processing stage.

The annotation processing API is located in the _javax.annotation.processing_ package. The main interface that you’ll have to implement is the _Processor_ interface, which has a partial implementation in the form of _AbstractProcessor_ class. This class is the one we’re going to extend to create our own annotation processor.

## 4. Setting Up the Project
To demonstrate the possibilities of annotation processing, we will develop a simple processor for generating fluent object builders for annotated classes.

We’re going to split our project into two Maven modules. One of them, _annotation-processor_ module, will contain the processor itself together with the annotation, and another, the _annotation-user_ module, will contain the annotated class. This is a typical use case of annotation processing.

The settings for the _annotation-processor_ module are as follows. We’re going to use the Google’s [auto-service](https://github.com/google/auto/tree/master/service) library to generate processor metadata file which will be discussed later, and the _maven-compiler-plugin_ tuned for the Java 8 source code. The versions of these dependencies are extracted to the properties section.
```xml
<properties>
    <auto-service.version>1.0-rc2</auto-service.version>
    <maven-compiler-plugin.version>
      3.5.1
    </maven-compiler-plugin.version>
</properties>

<dependencies>

    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>${auto-service.version}</version>
        <scope>provided</scope>
    </dependency>

</dependencies>

<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven-compiler-plugin.version}</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>

    </plugins>
</build>
```

The _annotation-user_ Maven module with the annotated sources does not need any special tuning, except adding a dependency on the annotation-processor module in the dependencies section:
```xml
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>annotation-processing</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## 5. Defining an Annotation
Suppose we have a simple POJO class in our annotation-user module with several fields:
```java
public class Person {

    private int age;

    private String name;

    // getters and setters …

}
```

We want to create a builder helper class to instantiate the _Person_ class more fluently:
```java
Person person = new PersonBuilder()
  .setAge(25)
  .setName("John")
  .build();
```

This _PersonBuilder_ class is an obvious choice for a generation, as its structure is completely defined by the _Person_ setter methods.

Let’s create a _@BuilderProperty_ annotation in the _annotation-processor_ module for the setter methods. It will allow us to generate the _Builder_ class for each class that has its setter methods annotated:
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface BuilderProperty {
}
```

The _@Target_ annotation with the _ElementType.METHOD_ parameter ensures that this annotation can be only put on a method.

The _SOURCE_ retention policy means that this annotation is only available during source processing and is not available at runtime.

The _Person_ class with properties annotated with the _@BuilderProperty_ annotation will look as follows:
```java
public class Person {

    private int age;

    private String name;

    @BuilderProperty
    public void setAge(int age) {
        this.age = age;
    }

    @BuilderProperty
    public void setName(String name) {
        this.name = name;
    }

    // getters …

}
```

## 6. Implementing a Processor

### 6.1. Creating an AbstractProcessor Subclass
We’ll start with extending the _AbstractProcessor_ class inside the _annotation-processor_ Maven module.

First, we should specify annotations that this processor is capable of processing, and also the supported source code version. This can be done either by implementing the methods _getSupportedAnnotationTypes_ and _getSupportedSourceVersion_ of the _Processor_ interface or by annotating your class with _@SupportedAnnotationTypes_ and _@SupportedSourceVersion_ annotations.

The _@AutoService_ annotation is a part of the _auto-service_ library and allows to generate the processor metadata which will be explained in the following sections.
```java
@SupportedAnnotationTypes(
  "com.baeldung.annotation.processor.BuilderProperty")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@AutoService(Processor.class)
public class BuilderProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations,
      RoundEnvironment roundEnv) {
        return false;
    }
}
```

You can specify not only the concrete annotation class names but also wildcards, like _“com.baeldung.annotation.*”_ to process annotations inside the _com.baeldung.annotation_ package and all its sub packages, or even _“*”_ to process all annotations.

The single method that we’ll have to implement is the process method that does the processing itself. It is called by the compiler for every source file containing the matching annotations.

Annotations are passed as the first _Set<? extends TypeElement> annotations_ argument, and the information about the current processing round is passed as the _RoundEnviroment_ roundEnv argument.

The return _boolean_ value should be _true_ if your annotation processor has processed all the passed annotations, and you don’t want them to be passed to other annotation processors down the list.

### 6.2. Gathering Data
Our processor does not really do anything useful yet, so let’s fill it with code.

First, we’ll need to iterate through all annotation types that are found in the class — in our case, the _annotations_ set will have a single element corresponding to the _@BuilderProperty_ annotation, even if this annotation occurs multiple times in the source file.

Still, it’s better to implement the _process_ method as an iteration cycle, for completeness sake:
```java
@Override
public boolean process(Set<? extends TypeElement> annotations,
  RoundEnvironment roundEnv) {

    for (TypeElement annotation : annotations) {
        Set<? extends Element> annotatedElements
          = roundEnv.getElementsAnnotatedWith(annotation);

        // …
    }

    return true;
}
```

In this code, we use the _RoundEnvironment_ instance to receive all elements annotated with the _@BuilderProperty_ annotation. In the case of the _Person_ class, these elements correspond to the _setName_ and _setAge_ methods.

_@BuilderProperty_ annotation’s user could erroneously annotate methods that are not actually setters. The setter method name should start with _set_, and the method should receive a single argument. So let’s separate the wheat from the chaff.

In the following code, we use the _Collectors.partitioningBy()_ collector to split annotated methods into two collections: correctly annotated setters and other erroneously annotated methods:
```java
Map<Boolean, List<Element>> annotatedMethods = annotatedElements.stream().collect(
  Collectors.partitioningBy(element ->
    ((ExecutableType) element.asType()).getParameterTypes().size() == 1
    && element.getSimpleName().toString().startsWith("set")));

List<Element> setters = annotatedMethods.get(true);
List<Element> otherMethods = annotatedMethods.get(false);
```

Here we use the _Element.asType()_ method to receive an instance of the _TypeMirror_ class which gives us some ability to introspect types even though we are only at the source processing stage.

We should warn the user about incorrectly annotated methods, so let’s use the _Messager_ instance accessible from the _AbstractProcessor.processingEnv_ protected field. The following lines will output an error for each erroneously annotated element during the source processing stage:
```java
otherMethods.forEach(element ->
  processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR,
    "@BuilderProperty must be applied to a setXxx method "
      + "with a single argument", element));
```

Of course, if the correct setters collection is empty, there is no point of continuing the current type element set iteration:
```java
if (setters.isEmpty()) {
    continue;
}
```

If the setters collection has at least one element, we’re going to use it to get the fully qualified class name from the enclosing element, which in case of the setter method appears to be the source class itself:
```java
String className = ((TypeElement) setters.get(0)
  .getEnclosingElement()).getQualifiedName().toString();
```

The last bit of information we need to generate a builder class is a map between the names of the setters and the names of their argument types:
```java
Map<String, String> setterMap = setters.stream().collect(Collectors.toMap(
    setter -> setter.getSimpleName().toString(),
    setter -> ((ExecutableType) setter.asType())
      .getParameterTypes().get(0).toString()
));
```

### 6.3. Generating the Output File
Now we have all the information we need to generate a builder class: the name of the source class, all its setter names, and their argument types.

To generate the output file, we’ll use the _Filer_ instance provided again by the object in the _AbstractProcessor.processingEnv_ protected property:
```java
JavaFileObject builderFile = processingEnv.getFiler()
  .createSourceFile(builderClassName);
try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {
    // writing generated file to out …
}
```

The complete code of the _writeBuilderFile_ method is provided below. We only need to calculate the package name, fully qualified builder class name, and simple class names for the source class and the builder class. The rest of the code is pretty straightforward.
```java
private void writeBuilderFile(
  String className, Map<String, String> setterMap)
  throws IOException {

    String packageName = null;
    int lastDot = className.lastIndexOf('.');
    if (lastDot > 0) {
        packageName = className.substring(0, lastDot);
    }

    String simpleClassName = className.substring(lastDot + 1);
    String builderClassName = className + "Builder";
    String builderSimpleClassName = builderClassName
      .substring(lastDot + 1);

    JavaFileObject builderFile = processingEnv.getFiler()
      .createSourceFile(builderClassName);

    try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {

        if (packageName != null) {
            out.print("package ");
            out.print(packageName);
            out.println(";");
            out.println();
        }

        out.print("public class ");
        out.print(builderSimpleClassName);
        out.println(" {");
        out.println();

        out.print("    private ");
        out.print(simpleClassName);
        out.print(" object = new ");
        out.print(simpleClassName);
        out.println("();");
        out.println();

        out.print("    public ");
        out.print(simpleClassName);
        out.println(" build() {");
        out.println("        return object;");
        out.println("    }");
        out.println();

        setterMap.entrySet().forEach(setter -> {
            String methodName = setter.getKey();
            String argumentType = setter.getValue();

            out.print("    public ");
            out.print(builderSimpleClassName);
            out.print(" ");
            out.print(methodName);

            out.print("(");

            out.print(argumentType);
            out.println(" value) {");
            out.print("        object.");
            out.print(methodName);
            out.println("(value);");
            out.println("        return this;");
            out.println("    }");
            out.println();
        });

        out.println("}");
    }
}
```

## 7. Running the Example
To see the code generation in action, you should either compile both modules from the common parent root or first compile the _annotation-processor_ module and then the _annotation-user_ module.

The generated PersonBuilder class can be found inside the _annotation-user/target/generated-sources/annotations/com/baeldung/annotation/PersonBuilder.java_ file and should look like this:
```java
package com.baeldung.annotation;

public class PersonBuilder {

    private Person object = new Person();

    public Person build() {
        return object;
    }

    public PersonBuilder setName(java.lang.String value) {
        object.setName(value);
        return this;
    }

    public PersonBuilder setAge(int value) {
        object.setAge(value);
        return this;
    }
}
```

## 8. Alternative Ways of Registering a Processor
To use your annotation processor during the compilation stage, you have several other options, depending on your use case and the tools you use.

### 8.1. Using the Annotation Processor Tool
The apt tool was a special command line utility for processing source files. It was a part of Java 5, but since Java 7 it was deprecated in favour of other options and removed completely in Java 8. It will not be discussed in this article.

### 8.2. Using the Compiler Key
The _-processor_ compiler key is a standard JDK facility to augment the source processing stage of the compiler with your own annotation processor.

Note that the processor itself and the annotation have to be already compiled as classes in a separate compilation and present on the classpath, so the first thing you should do is:
```java
javac com/baeldung/annotation/processor/BuilderProcessor
javac com/baeldung/annotation/processor/BuilderProperty
```

Then you do the actual compilation of your sources with the _-processor_ key specifying the annotation processor class you’ve just compiled:
```java
javac -processor com.baeldung.annotation.processor.MyProcessor Person.java
```

To specify several annotation processors in one go, you can separate their class names with commas, like this:
```java
javac -processor package1.Processor1,package2.Processor2 SourceFile.java
```

### 8.3. Using Maven
The _maven-compiler-plugin_ allows specifying annotation processors as part of its configuration.

Here’s an example of adding annotation processor for the compiler plugin. You could also specify the directory to put generated sources into, using the _generatedSourcesDirectory_ configuration parameter.

Note that the _BuilderProcessor_ class should already be compiled, for instance, imported from another jar in the build dependencies:
```java
<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <generatedSourcesDirectory>${project.build.directory}
                  /generated-sources/</generatedSourcesDirectory>
                <annotationProcessors>
                    <annotationProcessor>
                        com.baeldung.annotation.processor.BuilderProcessor
                    </annotationProcessor>
                </annotationProcessors>
            </configuration>
        </plugin>

    </plugins>
</build>
```

### 8.4. Adding a Processor Jar to the Classpath
Instead of specifying the annotation processor in the compiler options, you may simply add a specially structured jar with the processor class to the classpath of the compiler.

To pick it up automatically, the compiler has to know the name of the processor class. So you have to specify it in the _META-INF/services/javax.annotation.processing.Processor_ file as a fully qualified class name of the processor:
```
com.baeldung.annotation.processor.BuilderProcessor
```

You can also specify several processors from this jar to pick up automatically by separating them with a new line:
```
package1.Processor1
package2.Processor2
package3.Processor3
```

If you use Maven to build this jar and try to put this file directly into the _src/main/resources/META-INF/services_ directory, you’ll encounter the following error:
```
[ERROR] Bad service configuration file, or exception thrown while
constructing Processor object: javax.annotation.processing.Processor:
Provider com.baeldung.annotation.processor.BuilderProcessor not found
```

This is because the compiler tries to use this file during the _source-processing_ stage of the module itself when the _BuilderProcessor_ file is not yet compiled. The file has to be either put inside another resource directory and copied to the _META-INF/services_ directory during the resource copying stage of the Maven build, or (even better) generated during the build.

The Google _auto-service_ library, discussed in the following section, allows generating this file using a simple annotation.

### 8.5. Using the Google auto-service Library
To generate the registration file automatically, you can use the _@AutoService_ annotation from the Google’s _auto-service_ library, like this:
```java
@AutoService(Processor.class)
public BuilderProcessor extends AbstractProcessor {
    // …
}
```

This annotation is itself processed by the annotation processor from the auto-service library. This processor generates the _META-INF/services/javax.annotation.processing.Processor_ file containing the _BuilderProcessor_ class name.

## 9. Conclusion
In this article, we’ve demonstrated source-level annotation processing using an example of generating a Builder class for a POJO. We have also provided several alternative ways of registering annotation processors in your project.

The source code for the article is available on [GitHub](https://github.com/eugenp/tutorials/tree/master/annotations).

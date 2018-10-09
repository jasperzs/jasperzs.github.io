---
layout:     post
title:      Resource vs. Inject vs. Autowired
subtitle:   Understanding annotation wiring in Spring
date:       2019-01-14
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
---

## 1. Overview
Annotations used to perform dependency injection:
- `javax.annotation.Resource`
- `javax.inject.Inject`
- `org.springframework.beans.factory.annotation.Autowired`

## 2. The @Resource Annotation
This annotation has the following execution paths, listed by precedence:
- Match by Name
- Match by Type
- Match by Qualifier

These execution paths are applicable to both setter and field injection.

### 2.1. Field Injection
Annotate an instance variable with the @Resource annotation.

#### 2.1.1 Match by Name
Test to observe this behavior:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class FieldResourceInjectionTest {

    @Resource(name="namedFile")
    private File defaultFile;

    @Test
    public void givenResourceAnnotation_WhenOnField_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

The application context which contains the bean __namedFile__.
```java
@Configuration
public class ApplicationContextTestResourceNameType {

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

Note that the bean id and the corresponding reference attribute value must match, otherwise a `org.springframework.beans.factory.NoSuchBeanDefinitionException` will be thrown.

#### 2.1.2 Match by Type
Remove the __name__ attribute value:
```java
@Resource
private File defaultFile;
```

The above test will still pass because if the `@Resource` annotation does not receive a bean name as an attribute value, the Spring Framework will proceed with the next level of precedence, match-by-type, in order to try resolve the dependency.

#### 2.1.3. Match by Qualifier
Application context:
```java
@Configuration
public class ApplicationContextTestResourceQualifier {

    @Bean(name="defaultFile")
    public File defaultFile() {
        File defaultFile = new File("defaultFile.txt");
        return defaultFile;
    }

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

Test to observe this behavior:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class QualifierResourceInjectionTest {

    @Resource
    private File dependency1;

    @Resource
    private File dependency2;

    @Test
    public void givenResourceAnnotation_WhenField_ThenDependency1Valid(){
        assertNotNull(dependency1);
        assertEquals("defaultFile.txt", dependency1.getName());
    }

    @Test
    public void givenResourceQualifier_WhenField_ThenDependency2Valid(){
        assertNotNull(dependency2);
        assertEquals("namedFile.txt", dependency2.getName());
    }
}
```

An `org.springframework.beans.factory.NoUniqueBeanDefinitionException` is thrown after running the test.
This is because the application context has found two bean definitions of type File, and it is confused as to which bean should resolve the dependency.

To fix this, add `@Qualifier` annotations:
```java
@Resource
@Qualifier("defaultFile")
private File dependency1;

@Resource
@Qualifier("namedFile")
private File dependency2;
```

### 2.2. Setter Injection
The execution paths taken when injecting dependencies on a field are applicable to setter-based injection.

#### 2.2.1 Match by Name
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodResourceInjectionTest {

    private File defaultFile;

    @Resource(name="namedFile")
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

#### 2.2.2. Match by Type
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodByTypeResourceTest {

    private File defaultFile;

    @Resource
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenValidDependency(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

#### 2.2.3. Match by Qualifier
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class MethodByQualifierResourceTest {

    private File arbDependency;
    private File anotherArbDependency;

    @Test
    public void givenResourceQualifier_WhenSetter_ThenValidDependencies(){
      assertNotNull(arbDependency);
        assertEquals("namedFile.txt", arbDependency.getName());
        assertNotNull(anotherArbDependency);
        assertEquals("defaultFile.txt", anotherArbDependency.getName());
    }

    @Resource
    @Qualifier("namedFile")
    public void setArbDependency(File arbDependency) {
        this.arbDependency = arbDependency;
    }

    @Resource
    @Qualifier("defaultFile")
    public void setAnotherArbDependency(File anotherArbDependency) {
        this.anotherArbDependency = anotherArbDependency;
    }
}
```

## 3. The @Inject Annotation
This annotation has the following execution paths, listed by precedence:
- Match by Type
- Match by Qualifier
- Match by Name

These execution paths are applicable to both setter and field injection.

### 3.1. Field Injection

#### 3.1.1. Match by Type
```java
@Component
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectType.class)
public class FieldInjectTest {

    @Inject
    private ArbitraryDependency fieldInjectDependency;

    @Test
    public void givenInjectAnnotation_WhenOnField_ThenValidDependency(){
        assertNotNull(fieldInjectDependency);
        assertEquals("Arbitrary Dependency",
          fieldInjectDependency.toString());
    }
}
```

#### 3.1.2. Match by Qualifier
```java
public class AnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectQualifier.class)
public class FieldQualifierInjectTest {

    @Inject
    @Qualifier("defaultFile")
    private ArbitraryDependency defaultDependency;

    @Inject
    @Qualifier("namedFile")
    private ArbitraryDependency namedDependency;

    @Test
    public void givenInjectQualifier_WhenOnField_ThenDefaultFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Arbitrary Dependency",
          defaultDependency.toString());
    }

    @Test
    public void givenInjectQualifier_WhenOnField_ThenNamedFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Another Arbitrary Dependency",
          namedDependency.toString());
    }
}
```

#### 3.1.3. Match by Name
```java
public class YetAnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Yet Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}

@Configuration
public class ApplicationContextTestInjectName {

    @Bean
    public ArbitraryDependency yetAnotherFieldInjectDependency() {
        ArbitraryDependency yetAnotherFieldInjectDependency =
          new YetAnotherArbitraryDependency();
        return yetAnotherFieldInjectDependency;
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectName.class)
public class FieldByNameInjectTest {

    @Inject
    @Named("yetAnotherFieldInjectDependency")
    private ArbitraryDependency yetAnotherFieldInjectDependency;

    @Test
    public void givenInjectQualifier_WhenSetOnField_ThenDependencyValid(){
        assertNotNull(yetAnotherFieldInjectDependency);
        assertEquals("Yet Another Arbitrary Dependency",
          yetAnotherFieldInjectDependency.toString());
    }
}
```

### 3.2 Setter Injection
Setter-based injection for the @Inject annotation is similar to the approach used for @Resource setter-based injection. Instead of annotating the reference variable, the corresponding setter method is annotated. The execution paths followed by field-based dependency injection also apply to setter based injection.

## 4. The @Autowired Annotation
The behaviour of @Autowired annotation is similar to the @Inject annotation. The only difference is that the @Autowired annotation is part of the Spring framework. This annotation has the same execution paths as the @Inject annotation, listed in order of precedence:
- Match by Type
- Match by Qualifier
- Match by Name

These execution paths are applicable to both setter and field injection.

### 4.1. Field Injection

#### 4.1.1. Match by Type
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredType.class)
public class FieldAutowiredTest {

    @Autowired
    private ArbitraryDependency fieldDependency;

    @Test
    public void givenAutowired_WhenSetOnField_ThenDependencyResolved() {
        assertNotNull(fieldDependency);
        assertEquals("Arbitrary Dependency", fieldDependency.toString());
    }
}

@Configuration
public class ApplicationContextTestAutowiredType {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }
}
```

#### 4.1.2. Match by Qualifier
```java
@Configuration
public class ApplicationContextTestAutowiredQualifier {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }

    @Bean
    public ArbitraryDependency anotherAutowiredFieldDependency() {
        ArbitraryDependency anotherAutowiredFieldDependency =
          new AnotherArbitraryDependency();
        return anotherAutowiredFieldDependency;
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredQualifier.class)
public class FieldQualifierAutowiredTest {

    @Autowired
    @Qualifier("autowiredFieldDependency")
    private ArbitraryDependency fieldDependency1;

    @Autowired
    @Qualifier("anotherAutowiredFieldDependency")
    private ArbitraryDependency fieldDependency2;

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep1Valid(){
        assertNotNull(fieldDependency1);
        assertEquals("Arbitrary Dependency", fieldDependency1.toString());
    }

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep2Valid(){
        assertNotNull(fieldDependency2);
        assertEquals("Another Arbitrary Dependency",
          fieldDependency2.toString());
    }
}
```

#### 4.1.3. Match by Name
When autowiring dependencies by name, the @ComponentScan annotation must be used with the application context,
```Java
@Configuration
@ComponentScan(basePackages={"com.baeldung.dependency"})
    public class ApplicationContextTestAutowiredName {
}
```

The @ComponentScan annotation will search packages for Java classes that have been annotated with the @Component annotation. For example, in the application context, the com.baeldung.dependency package will be scanned for classes that have been annotated with the @Component annotation. In this scenario, the Spring Framework must detect the ArbitraryDependency class, which has the @Component annotation:
```java
@Component(value="autowiredFieldDependency")
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

The attribute value, autowiredFieldDependency, passed into the @Component annotation, tells the Spring Framework that the ArbitraryDependency class is a component named autowiredFieldDependency. In order for the @Autowired annotation to resolve dependencies by name, the component name must correspond with the field name.
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredName.class)
public class FieldAutowiredNameTest {

    @Autowired
    private ArbitraryDependency autowiredFieldDependency;

    @Test
    public void givenAutowiredAnnotation_WhenOnField_ThenDepValid(){
        assertNotNull(autowiredFieldDependency);
        assertEquals("Arbitrary Dependency",
          autowiredFieldDependency.toString());
    }
}
```

### 4.2. Setter Injection
Setter-based injection for the @Autowired annotation is similar the approach demonstrated for @Resource setter-based injection. Instead of annotating the reference variable with the @Autowired annotation, the corresponding setter is annotated. The execution paths followed by field-based dependency injection also apply to setter-based injection.

## 5. Applying These Annotations
This raises the question, which annotation should be used and under what circumstances? The answer to these questions depends on the design scenario faced by the application in question, and how the developer wishes to leverage polymorphism based on the default execution paths of each annotation.

### 5.1. Application-Wide use of Singletons Through Polymorphism
If the design is such that application behaviors are based on implementations of an interface or an abstract class, and these behaviors are used throughout the application, then use either the @Inject or @Autowired annotation.

The benefit of this approach is that when the application is upgraded, or a patch needs to be applied in order to fix a bug; then classes can be swapped out with minimal negative impact to the overall application behavior. In this scenario, the primary default execution path is match-by-type.

### 5.2. Fine-Grained Application Behavior Configuration Through Polymorphism
If the design is such that the application has complex behavior, each behavior is based on different interfaces/abstract classes, and usage of each of these implementations varies across the application, then use the @Resource annotation. In this scenario, the primary default execution path is match-by-name.

### 5.3. Dependency Injection Should be Handled Solely by the Java EE Platform
If there is a design mandate for all dependencies to be injected by the Java EE Platform and not Spring, then the choice is between the @Resource annotation and the @Inject annotation. You should narrow down the final decision between the two annotations, based on which default execution path is required.

### 5.4. Dependency Injection Should be Handled Solely by the Spring Framework
If the mandate is for all dependencies to be handled by the Spring Framework, the only choice is the @Autowired annotation.

### 5.5. Discussion Summary

| Scenario | @Resource | @Inject | @Autowired |
| -------- | :-------: | :-----: | :--------: |
| Application-wide use of singletons through polymorphism | ✗ | ✔ | ✔ |
| Fine-grained application behavior configuration through polymorphism | ✔ | ✗ | ✗ |
| Dependency injection should be handled solely by the Java EE platform | ✔ | ✔ | ✗ |
| Dependency injection should be handled solely by the Spring Framework | ✗ | ✗ | ✔ |

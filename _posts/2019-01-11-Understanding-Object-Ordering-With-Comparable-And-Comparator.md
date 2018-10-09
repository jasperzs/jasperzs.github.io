---
layout:     post
title:      Understanding Object Ordering in Java with Comparable and Comparator
subtitle:   What is Comparable and Comparator
date:       2019-01-11
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Overview
To understand how sorting actually works with elements in collections and arrays, let’s see some examples where we use the utility class Collections to sort elements of a collection (or Arrays class to sort elements in an array):
- __Collections.sort(list)__: sorts a List collection.
- __Arrays.sort(array)__: sorts an array.

### Example #1: Sorting a list of String objects
```java
List<String> names = Arrays.asList(
            "Tom", "Peter", "Alice", "Bob", "Sam",
            "Mary", "Jane", "Bill", "Tim", "Kevin");
System.out.println("Before sorting: " + names);
Collections.sort(names);
System.out.println("After sorting: " + names);
```

Output:
```java
Before sorting: [Tom, Peter, Alice, Bob, Sam, Mary, Jane, Bill, Tim, Kevin]
After sorting: [Alice, Bill, Bob, Jane, Kevin, Mary, Peter, Sam, Tim, Tom]
```

In this example, the list `names` is sorted by alphabetic order of String.

### Example #2: Sorting a list of Integer objects
```java
List<Integer> numbers = Arrays.asList(8, 2, 5, 1, 3, 4, 9, 6, 7, 10);
System.out.println("Before sorting: " + numbers);
Collections.sort(numbers);
System.out.println("After sorting: " + numbers);
```

Output:
```java
Before sorting: [8, 2, 5, 1, 3, 4, 9, 6, 7, 10]
After sorting: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

Here, the integer numbers in the list `numbers` are sorted by alphanumeric order.

### Example #3: Sorting a list of Date objects
```java
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
List<Date> birthdays = new ArrayList<>();
birthdays.add(dateFormat.parse("2016-01-20"));
birthdays.add(dateFormat.parse("1998-12-03"));
birthdays.add(dateFormat.parse("2009-07-15"));
birthdays.add(dateFormat.parse("2012-04-30"));
System.out.println("Before sorting: ");
for (Date date : birthdays) {
    System.out.println(dateFormat.format(date));
}
Collections.sort(birthdays);
System.out.println("After sorting: ");
for (Date date : birthdays) {
    System.out.println(dateFormat.format(date));
}
```

Output:
```java
Before sorting:
2016-01-20
1998-12-03
2009-07-15
2012-04-30
After sorting:
1998-12-03
2009-07-15
2012-04-30
2016-01-20
```

Here, the list `birthdays` is sorted by chronological order of its elements - objects of type `Date`.
From the 3 examples above, the collections are sorted by natural ordering of its elements:
- The natural ordering of String objects is alphabetic order.
- The natural ordering of Integer objects is alphanumeric order.
- The natural ordering of Date objects is chronological order.
Let’s dive into the concept of natural ordering.

## 2. Understanding Natural Ordering
__*Natural ordering*__ is the default ordering of objects of a specific type when they are sorted in an array or a collection. The Java language provides the __Comparable__ interface that allows us define the natural ordering of a class. This interface is declared as follows:
```java
public interface Comparable<T> {
    public int compareTo(T object);
}
```

As you can see, this interface is parameterized (generics) and it has a single method __compareTo()__ that allows two objects of a same type to be compared with each other. The important point here is the value returned by this method: an integer number indicates the comparison result of two objects. Remember these rules:
- compare value = 0: two objects are equal.
- compare value > 0: the first object (the current object) is greater than the second one.
- compare value &lt; 0: the first object is less than the second one.

Imagine that, when the objects are being sorted, their __compareTo()__ methods are invoked to compare with other objects. And based on the compare value returned, the objects are sorted by natural ordering.

Classes whose objects used in collections or arrays should implement the __Comparable__ interface for providing the natural ordering of its objects when being sorted. Otherwise we will get an error at runtime.

A class that implements the __Comparable__ interface is said to have __*class natural ordering*__. And the __compareTo()__ method is called the __*natural comparison method*__.

In the above examples, we don’t have to write code to implement the __Comparable__ interface because the `String`, `Integer` and `Date` classes already implemented this interface. Hence we can sort a collection containing objects of these types.

Other wrapper types in Java are also comparable: `Long`, `Double`, `Float`, etc.

When we create our own type, we have to implement the __Comparable__ interface in order to have objects of our type eligible to be sorted in collections or arrays. Let’s see an example to understand how the __Comparable__ interface is used.

Let’s say we have the `Employee` class which is defined as shown below:
```java
/**
 * Employee.java
 * @author www.codejava.net
 */
public class Employee {
    String firstName;
    String lastName;
    Date joinDate;
    public Employee(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    public String toString() {
        return firstName + " " + lastName;
    }
    // getters and setters
}
```

Add some employees to a list collection like this:
```java
List<Employee> listEmployees = new ArrayList<>();
Employee employee1 = new Employee("Tom", "Eagar");
Employee employee2 = new Employee("Tom", "Smith");
Employee employee3 = new Employee("Bill", "Joy");
Employee employee4 = new Employee("Bill", "Gates");
Employee employee5 = new Employee("Alice", "Wooden");
listEmployees.add(employee1);
listEmployees.add(employee2);
listEmployees.add(employee3);
listEmployees.add(employee4);
listEmployees.add(employee5);
```

Try to sort this list:
```java
Collections.sort(listEmployees);
```

We will get an error at runtime: __no suitable method found for sort(List&lt;Employee&gt;)...__

WHY?

It’s because the `Employee` class doesn’t implement the `Comparable` interface so the `sort()` method cannot compare the objects.

Now, let’s have the `Employee` class implements the `Comparable` interface, and we define the natural ordering is first name - last name, meaning the employees are sorted by first name first, then by last name. Here’s the updated version of the `Employee` class:
```java
/**
 * Employee.java
 * Implementing Comparable
 * @author www.codejava.net
 */
public class Employee implements Comparable<Employee> {
    // fields...
    // constructors...
    // getters...
    // setters...
    // implement the natural comparison method:

    public int compareTo(Employee another) {
        int compareValue = this.firstName.compareTo(another.firstName);
        if (compareValue == 0) {
            return this.lastName.compareTo(another.lastName);
        }
        return compareValue;
    }
}
```

Look at how the `compareTo()` method is implemented here:
- First, we compare the first name by using the String’s `compareTo()` method. We can safely use this method of the built-in types in Java: `String`, `Date`, `Integer`, `Long`, etc.
- If two employees have same first name (compare value = 0), then we compare their last name. Finally the compare value is returned as per the contract of the `Comparable` interface.

Now, run this test code and observe the result:
```java
System.out.println("Before sorting: " + listEmployees);
Collections.sort(listEmployees);
System.out.println("After sorting: " + listEmployees);
```

Output:
```java
Before sorting: [Tom Eagar, Tom Smith, Bill Joy, Bill Gates, Alice Wooden]
After sorting: [Alice Wooden, Bill Gates, Bill Joy, Tom Eagar, Tom Smith]
```

Awesome! It works perfectly as we expected: the employees are sorted by their first name, and then last name.

### Note #1:
We cannot compare objects of different types, e.g. a `String` object cannot be compared with an `Integer` object. As the `compareTo()` method enforces this rule, we can only compare objects of the same type. If we add objects of different types to a collection and sort it, we will get `ClassCastException`.

### Note #2:
If we want to reverse the natural ordering, simply swap the objects being compared in the `compareTo()` method. For example, the following implementation sorts employees by their first name into descending order:
```java
public int compareTo(Employee another) {
    return another.firstName.compareTo(this.firstName);
}
```

In case we use a sorted collection i.e. `TreeSet`, we don’t have to use the `Collections.sort()` utility method, as a `TreeSet` sorts its elements by their natural ordering. The following example demonstrates how to use a `TreeSet` to sort Strings:
```java
Set<String> setNames = new TreeSet<>();
setNames.addAll(Arrays.asList("Tom", "Peter", "Alice", "Bob", "Sam",
                  "Mary", "Jane", "Bill", "Tim", "Kevin"));
System.out.println(setNames);
```

Output:
```java
[Alice, Bill, Bob, Jane, Kevin, Mary, Peter, Sam, Tim, Tom]
```

Similarly, we can sort the `Employee` objects using a `TreeSet` like this:
```java
Set<Employee> setEmployees = new TreeSet<>();
Employee employee1 = new Employee("Tom", "Eagar");
Employee employee2 = new Employee("Tom", "Smith");
Employee employee3 = new Employee("Bill", "Joy");
Employee employee4 = new Employee("Bill", "Gates");
Employee employee5 = new Employee("Alice", "Wooden");
setEmployees.add(employee1);
setEmployees.add(employee2);
setEmployees.add(employee3);
setEmployees.add(employee4);
setEmployees.add(employee5);
System.out.println(setEmployees);
```

Output:
```java
[Alice Wooden, Bill Gates, Bill Joy, Tom Eagar, Tom Smith]
```

So far we have got understanding about the natural ordering of objects and how the `Comparable` interface defines the ordering.

What if we want to sort objects in an order which differs from the natural ordering? For example, sort the employees list above by seniority (based on their join dates)?

## 3. Understanding Comparator
The `Collections` utility class provides a method for sorting a list using an external comparator:
```java
Collections.sort(list, comparator)
```

This overloaded version takes two parameters: a list collection and a comparator, which is any object that implements the __Comparator__ interface. This interface declares this method:
```java
public interface Comparator<T> {
    public int compare(T obj1, T obj2);
}
```

Like the `Comparable` interface, this interface is also parameterized for any specific type. The `compare()` method is similar except it takes both the objects to be compared as arguments. The return value is also evaluated similarly .

For example, the following class compares two `Employee` objects using the `Comparator` interface:
```java
/**
 * EmployeeComparator.java
 * Implementing Comparator
 * @author www.codejava.net
 */
public class EmployeeComparator implements Comparator<Employee> {
    public int compare(Employee emp1, Employee emp2) {
        return emp1.getJoinDate().compareTo(emp2.getJoinDate());
    }
}
```

In this comparator, we compare two `Employee` objects by their join dates. And update the `Employee` class like this (add an overloaded constructor and update the `toString()` method):
```java
/**
 * Employee.java
 * Implementing Comparable
 * @author www.codejava.net
 */
public class Employee implements Comparable<Employee> {
    // fields...
    // getters & setters....
    // constructor

    public Employee(String firstName, String lastName, Date joinDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.joinDate = joinDate;
    }
    public String toString() {
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        return firstName + " " + lastName + " " + dateFormat.format(joinDate);
    }
}
```

And here’s the test code:
```java
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
List<Employee> listEmployees = new ArrayList<>();
Employee employee1 = new Employee("Tom", "Eagar", dateFormat.parse("2007-12-03"));
Employee employee2 = new Employee("Tom", "Smith", dateFormat.parse("2005-06-20"));
Employee employee3 = new Employee("Bill", "Joy", dateFormat.parse("2009-01-31"));
Employee employee4 = new Employee("Bill", "Gates", dateFormat.parse("2005-05-12"));
Employee employee5 = new Employee("Alice", "Wooden", dateFormat.parse("2009-01-22"));
listEmployees.add(employee1);
listEmployees.add(employee2);
listEmployees.add(employee3);
listEmployees.add(employee4);
listEmployees.add(employee5);
System.out.println("Before sorting: ");
System.out.println(listEmployees);
Collections.sort(listEmployees, new EmployeeComparator());
System.out.println("After sorting: ");
System.out.println(listEmployees);
Collections.sort(listEmployees, (emp1, emp2) -> emp1.getJoinDate().compareTo(emp2.getJoinDate()));
```

Output:
```java
Before sorting:
[Tom Eagar 2007-12-03, Tom Smith 2005-06-20, Bill Joy 2009-01-31, Bill Gates 2005-05-12, Alice Wooden 2009-01-22]
After sorting:
[Bill Gates 2005-05-12, Tom Smith 2005-06-20, Tom Eagar 2007-12-03, Alice Wooden 2009-01-22, Bill Joy 2009-01-31]
```

### Note #3:
Since Java 8, we can use Lambda expressions to create a comparator more easily like this:
```java
Collections.sort(listEmployees,
            (emp1, emp2) -> emp1.getJoinDate().compareTo(emp2.getJoinDate()));
```

We can also pass a comparator when creating a new instance of a `TreeSet` like this:
```java
Set<Employee> setEmployees = new TreeSet<>(new EmployeeComparator());
```

Then the `TreeSet` will sort its elements according to the order defined by the specified comparator.

Using a comparator is useful in the following scenarios:
- The class doesn’t have natural ordering (or we don’t have source code to update it).
- We want to sort objects in orders other than the natural ordering.
- We want to provide multiple ways for sorting the objects, e.g. one comparator for each sorting criteria.

## 4. The contract between natural ordering and equals:
You know, the documentation of both `Comparable` and `Comparator` states that the natural ordering and the ordering specified by a comparator should be consistent with the `equals()` method of the class. Let’s say we have two objects `obj1` and `obj2` of class `A`, then:
> If obj1.compareTo(obj2) = 0 then obj1.equals(obj2) = true

If this contract is violated, we will get strange behavior when using sorted collections such as `TreeSet` and `TreeMap`.

Let’s examine an example to understand why this constraint really matters. Come back to the example of sorting a list of `Employee` objects.

We haven’t overridden the `equals()` method yet. Now, let’s override it for the `Employee` class:
```java
/**
 * Employee.java
 * Implementing Comparable and override equals()
 * @author www.codejava.net
 */
public class Employee implements Comparable<Employee> {
    // fields, constructors, getters and setters and toString()...

    public int compareTo(Employee another) {
        int compareValue = this.firstName.compareTo(another.firstName);
        if (compareValue == 0) {
            return this.lastName.compareTo(another.lastName);
        }
        return compareValue;
    }
    public boolean equals(Object obj) {
        if (obj instanceof Employee) {
            Employee another = (Employee) obj;
            if (this.firstName.equals(another.firstName)
                && this.lastName.equals(another.lastName)) {
                    return true;
            }
        }
        return false;
    }
}
```

Currently, it is compatible with the `compareTo()` method which also compares first name and then last name.

What if we need to change the `compareTo()` method for comparing two `Employee` objects by their seniority (join date) like this:
```java
public int compareTo(Employee another) {
    return this.joinDate.compareTo(another.joinDate);
}
```

Let’s execute some test code to see the outcome:
```java
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
Set<Employee2> setEmployees = new TreeSet<>(new EmployeeComparator2());
Employee2 employee1 = new Employee2("Tom", "Eagar", dateFormat.parse("2007-12-03"));
Employee2 employee2 = new Employee2("Tom", "Smith", dateFormat.parse("2005-06-20"));
Employee2 employee3 = new Employee2("Bill", "Joy", dateFormat.parse("2009-01-31"));
Employee2 employee4 = new Employee2("Bill", "Gates", dateFormat.parse("2009-01-31"));
Employee2 employee5 = new Employee2("Alice", "Wooden", dateFormat.parse("2007-12-03"));
setEmployees.add(employee1);
setEmployees.add(employee2);
setEmployees.add(employee3);
setEmployees.add(employee4);
setEmployees.add(employee5);
System.out.println(setEmployees);
```

Note that the `employee1` and `employe5` have same join date, so do the `employee3` and `employee4`. Add all of these 5 objects to the set:
```java
setEmployees.add(employee1);
setEmployees.add(employee2);
setEmployees.add(employee3);
setEmployees.add(employee4);
setEmployees.add(employee5);
```

And print the set:
```java
System.out.println(setEmployees);
```

Can you guess the output? Here is it:
```java
[Tom Smith 2005-06-20, Tom Eagar 2007-12-03, Bill Joy 2009-01-31]
```

Ouch! Why are there only 3 employees in the set?

It’s because the set compares the objects using the `compareTo()` method which considers two employees are equal if they have same join date, whereas the set does not allow duplicate elements, hence the `employee4` and `employee5` objects are not added to the set.

Now, you understand the consequence if natural ordering and equals are not consistent, right?

### So is there any solution or workaround?
Yes, there is.

Suppose that we still want to keep the natural ordering based on join date, while keep compatible with the `equals()` method, here’s how we update the `compareTo()` method:
```java
public int compareTo(Employee another) {
    int compareValue = this.joinDate.compareTo(another.joinDate);
    if (compareValue == 0) {
        compareValue = this.firstName.compareTo(another.firstName);
        if (compareValue == 0) {
            return this.lastName.compareTo(another.lastName);
        }
        return compareValue;
    }
    return compareValue;
}
```

That’s it! In this solution, we compare the `Employee` objects by their join dates first. If equal, continue comparing by their first names. And if equal, continue comparing their last names. This way we can keep the `compareTo()` method compatible with the `equals()` method.

Run the test code again and observe the output:
```java
[Tom Smith 2005-06-20, Alice Wooden 2007-12-03, Tom Eagar 2007-12-03, Bill Gates 2009-01-31, Bill Joy 2009-01-31]
```

Perfect!
The same problem and solution applies for a comparator.

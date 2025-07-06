# Simplifying Immutability with Java Records
#tag EN/Java

## What are Java Records?

**Records** are a special type of **class** introduced in Java to help **decrease boilerplate code**. They serve as "**data carriers**" for **immutable objects.** A Record automatically generates a class with its **canonical constructor**, **accessor methods** (called getters), `hashCode()`, `equals()`, and `toString()` implementations. The true essence of a Record is to **encapsulate a fixed conjunction of attributes** that represent the **state of an object** in a **concise and transparent** way.

---

## Advantages of Using Records in Java

- **Decreases boilerplate code** in **Plain Old Java Objects (POJOs)** and **Data Transfer Objects (DTOs)**.
- **Increases maintainability** due to less code and clearer intent.
- **Promotes immutability by default**, leading to safer and more predictable code.

---
## Declaring a Record Class 

```java
access_modifier record ClassName(list of components) {}

// A simple record to represent a person
public record Person(String firstName, String lastName) {}

// A record class to represent 2D dimensions and coordinates
record Point(double x, double y) {} [5, 8]

// A record for the dimensions of a rectangle
record Rectangle(double length, double width) {} [4]
```

Records can also implement **generics** to create data structures, for example:
```java
record Triangle<C extends Coordinate> (C top, C left, C right) {}
```


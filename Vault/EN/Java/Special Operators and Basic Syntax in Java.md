#tag: EN/Java
# Special Operator
The Java language includes six constructs that are sometimes considered **operators** and sometimes simply fundamental elements of its **syntax**. Let's explore each one.

---
## 1. Member Access (`.`)

The **dot operator (`.`)** is used to access the **members** (data fields and methods) of an object or a class.

- **Syntax:** `object.member` or `ClassName.staticMember`
- **Example:**
    - If `o` is an expression that evaluates to an object reference (or a class name), and `f` is the name of a field, `o.f` evaluates to the value contained in that field.
    - `myAccount.balance` (accesses the `balance` field of the `myAccount` object)
    - `System.out.println` (accesses the `println` method of the `out` object within the `System` class)

---

## 2. Array Element Access (`[]`)

Arrays are numbered lists of values of the same type. The **square brackets operator (`[]`)** allows you to refer to individual elements of an array using their **index** (position).

- **Syntax:** `array[index]`
- **Example:**
    - If `a` is an array, and `i` is an expression that evaluates to an integer, `a[i]` refers to one of the elements of `a`.
    - `myArray[0]` (accesses the first element of `myArray`)
    - `grades[2] = 95;` (assigns the value 95 to the third element of the `grades` array)

---

## 3. Method Invocation (`()`)

A **method** is a named collection of Java code that can be "run" or "invoked" by following the method's name with zero or more comma-separated expressions enclosed in **parentheses (`()`)**. The values of these expressions are the **arguments** to the method.

- **Syntax:** `object.methodName(arguments)`
- **Details:**
    - If a method doesn't expect any values, you can simply invoke it with `o.m()`.
    - If the method expects arguments, you need to pass these values when you invoke it, for example: `o.m(x, y, z)`.
    - The object `o` is referred to as the **receiver** of the method.
- **Example:**
    - `myList.add("Item");`
    - `calculator.sum(10, 5);`

---

## 4. Lambda Expression (`->`)

A **lambda expression** is an anonymous collection of executable Java code, essentially a method body without a name. It consists of a method argument list (zero or more comma-separated expressions contained within parentheses) followed by the **arrow operator (`->`)**.

- **Syntax:** `(parameters) -> { expression_body }`
- **Example:**
    - `list.forEach(item -> System.out.println(item));` (for each `item` in the `list`, print the `item`)
    - `(a, b) -> a + b;` (a lambda that takes two numbers and returns their sum)

---

## 5. Object Creation (`new`)

In Java, **objects** are created using the **`new` operator**. It's followed by the type of the object to be created and a parenthesized list of arguments to be passed to the object's **constructor**. A constructor is a special block of code that initializes a newly created object.

- **Syntax:** `new ObjectType(arguments)`
- **Example:**
    - `new ArrayList<String>();` (creates a new list of Strings)
    - `new Point(1, 2);` (creates a new `Point` object with x=1 and y=2 coordinates)

---

## 6. Array Creation (`new`)

Arrays are a special case of objects and are also created with the `new` operator, but with a slightly different syntax. The `new` keyword is followed by the type of the array to be created and the **size** of the array enclosed in square brackets.

- **Syntax:** `new ArrayType[size]`
- **Example:**
    - `new int[5]` (creates an array capable of storing 5 integers)
- **Array Literal Syntax:** In some circumstances, arrays can also be created using array literal syntax, where you directly provide the initial values.
    - `int[] numbers = {1, 2, 3};`

---

## 7. Type Conversion or Casting (`()`)

Type conversion, or **casting**, uses **parentheses (`()`)** as an operator to perform **narrowing type conversions**.

- **Syntax:** `(DesiredType) valueToConvert`
- **Details:** The first operand of this operator is the type to convert to (placed between the parentheses). The second operand is the value to be converted, which follows the parentheses.
- **Example:**
    - `(byte) 28` // An integer literal `28` cast to a `byte` type
    - `(int) (x + 3.14f)` // A floating-point sum value cast to an integer (truncating decimal places)
    - `(String)h.get(k)` // A generic object (returned from `h.get(k)`) cast to a `String`

---
# Statements in Java

A **statement** is a basic unit of execution in the Java language—it expresses a single piece of intent by the programmer. Unlike expressions (which evaluate to a value), Java statements do not have a value. Statements typically contain expressions and operators (especially assignment operators) and are frequently executed for the **side effects** they cause (i.e., what they change in the program).

Many Java statements are **flow control statements**, such as conditionals and loops, which can alter the default, linear order of execution in well-defined ways.

Here's a summary of common Java statements:

---

## Table: Java Statements

|Statement|Purpose|Syntax|Example|
|:--|:--|:--|:--|
|**Expression**|Produces side effects|`variable = expr;` &lt;br/> `expr++;` &lt;br/> `method();` &lt;br/> `new Type();`|`counter = 10;` &lt;br/> `i++;` &lt;br/> `printMessage();` &lt;br/> `new Car();`|
|**Compound**|Groups statements|`{ statements }`|`{ int x = 1; System.out.println(x); }`|
|**Empty**|Does nothing|`;`|`;` (rarely used intentionally)|
|**Labeled**|Names a statement|`label : statement`|`mainLoop: for (...) { ... break mainLoop; }`|
|**Variable**|Declares a variable|`[final] type name [= value ] [, name [= value ]] …;`|`int age = 30;` &lt;br/> `final String NAME = "Java";`|
|**`if`**|Conditional execution|`if (expr) statement [ else statement ]`|`if (age > 18) { ... } else { ... }`|
|**`switch`**|Conditional with multiple cases|`switch (expr) { [ case expr : statements ] … [ default: statements ] }`|`switch (day) { case 1: ... default: ... }`|
|**`while`**|Loop (tests condition before each iteration)|`while (expr) statement`|`while (counter < 5) { counter++; }`|
|**`do`**|Loop (executes at least once)|`do statement while (expr);`|`do { readInput(); } while (invalidInput);`|
|**`for`**|Simplified loop|`for (init ; test ; increment ) statement`|`for (int i = 0; i < 10; i++) { ... }`|
|**`foreach`**|Collection iteration|`for ( variable : iterable ) statement`|`for (String item : list) { System.out.println(item); }`|
|**`break`**|Exits a block (loop, switch)|`break [ label ] ;`|`break;` &lt;br/> `break myLoop;`|
|**`continue`**|Restarts a loop (skips to next iteration)|`continue [ label ] ;`|`continue;` &lt;br/> `continue myLoop;`|
|**`return`**|Ends a method (optionally returns a value)|`return [ expr ] ;`|`return 0;` &lt;br/> `return;`|
|**`synchronized`**|Critical section (for threads)|`synchronized ( expr ) { statements }`|`synchronized (this) { ... }`|
|**`throw`**|Throws an exception|`throw expr ;`|`throw new IllegalArgumentException("Error!");`|
|**`try`**|Handles exceptions|`try { statements } [ catch ( type name ) { statements } ] … [ finally { statements } ]`|`try { ... } catch (Exception e) { ... } finally { ... }`|
|**`assert`**|Verifies invariant (for debugging)|`assert invariant [ error ]`|`assert (age > 0) : "Age cannot be zero or negative!";`|

---
## 1. Expression Statements

As we've seen, some Java **expressions** don't just calculate a value; they also have **side effects**. This means they change the program's state in some way. You can turn any such expression into a statement simply by ending it with a **semicolon (`;`)**.

Common types of expression statements include:

* **Assignments:** Giving a value to a variable.
    * `a = 1;`
    * `x *= 2;` (This is equivalent to `x = x * 2;`)
* **Increments and Decrements:** Changing a variable's value by one.
    * `i++;` (Post-increment: uses the current value of `i`, then adds 1)
    * `c--;` (Pre-decrement: subtracts 1 from `c`, then uses the new value)
* **Method Calls (Invocations):** Executing a piece of named code.
    * `System.out.println("statement");`
* **Object Creation:** Instantiating (creating) a new object.
    * `new MyObject();`

---

## 2. Compound Statements

A **compound statement** lets you group multiple statements together as if they were a single unit. It's essentially any number of statements enclosed within **curly braces (`{}`)**. You can use a compound statement anywhere Java syntax expects a single statement.

* **Example:** A loop body is often a compound statement.
    ```java
    for (int i = 0; i < 10; i++) { // -> { start the block for statements
        a[i]++;   // Statement 1: increments the array element
        b[i]--;   // Statement 2: decrements the array element
    } // This entire block within curly braces is a compound statement.
    ```

---

## 3. The Empty Statement

An **empty statement** in Java is simply a single **semicolon (`;`)**. It doesn't perform any action, but it can be syntactically useful in specific scenarios, particularly in loops where all the work is done within the loop's header.

* **Syntax:** `;`
* **Example:** An empty loop body in a `for` loop.

```java 
for (int i = 0; i < 10; a[i++]++); // Increments array elements; the loop body is empty. 
```

> [!NOTE]
> * Adding a comment like `/* empty */` can improve clarity when using empty statements, as they can sometimes be hard to spot or might appear like an accidental typo.

---

## 4. Labeled Statements

A **labeled statement** is a statement that you've given a name to. You define a label by placing an **identifier** (the name) followed by a **colon (`:`)** directly before the statement you want to label.

* **Syntax:** `myLabel: statement;`
* **Purpose:** Labels are primarily used with the `break` and `continue` keywords to control the flow of **nested loops**. They allow you to specify which outer loop you want to `break` out of completely or `continue` to the next iteration of.

***Example:**  
```java
rowLoop: // This is a label
for (int r = 0; r < rows.length; r++) {
    colLoop: // Another label
    for (int c = 0; c < columns.length; c++) {
        // ... some code ...
        if (someCondition) {
            break rowLoop; // This exits the 'rowLoop' entirely, not just 'colLoop'.
        }
    }
}
```

> [!NOTE]
> ***Important :** While `labeled statements` are part of Java, their use is generally **discouraged** in modern Java programming. They can make code harder to read and debug, as they allow for non-linear jumps in logic. Often, refactoring code into separate methods or using boolean flags offers clearer and more maintainable solutions.

---
# 5. Local Variable Declaration Statements

A local variable, often simply called a variable, is a symbolic name for a location to store a value that is defined within a method or compound statement. All variables must be declared before they can be used; this is done with a variable declaration statement. Because Java is a statically typed language, a variable declaration specifies the type of the variable, and only values of that type can be stored in the variable.

## 5.1 Basic Variable Declaration

In its simplest form, a variable declaration specifies a variable's type and name:

```java
int counter;
String s;
```

--- 
## 5.2 Variable Declaration with Initializers

A variable declaration can also include an initializer: an expression that specifies an initial value for the variable. For example:

```java
int i = 0;
String s = readLine();
int[] data = {x+1, x+2, x+3}; // Array initializers are discussed later
```

The Java compiler does not allow you to use a local variable that has not been initialized, so it is usually convenient to combine variable declaration and initialization into a single statement. The initializer expression need not be a literal value or a constant expression that can be evaluated by the compiler; it can be an arbitrarily complex expression whose value is computed when the program is run.

---
## 5.3 Type Inference with `var`

If a variable has an initializer then the programmer can use a special syntax to ask the compiler to automatically work out the type, if it is possible to do so:

```java
var i = 0; // type of i inferred as int
var s = readLine(); // type of s inferred as String
```

This can be a useful syntax, but when learning the Java language it is probably better to avoid it at first while you become familiar with the Java type system.

---
## 5.4 Multiple Variable Declaration

A single variable declaration statement can declare and initialize more than one variable, but all variables must be of the same explicitly declared type. Variable names and optional initializers are separated from each other with commas:

```java
int i, j, k;
float x = 1.0f, y = 1.0f;
String question = "Really Quit?", response;
```

---
## 5.5 Final Variables

Variable declaration statements can begin with the `final` keyword. This modifier specifies that once an initial value is defined for the variable, that value is never allowed to change:

```java
final String greeting = getLocalLanguageGreeting();
```

We will have more to say about the `final` keyword later on, especially when talking about the immutable style of programming.

---
## 5.6 Variable Scope

Java variable declaration statements can appear anywhere in Java code; they are not restricted to the beginning of a method or block of code. Local variable declarations can also be integrated with the initialize portion of a `for` loop, as we'll discuss shortly.

Local variables can be used only within the method or block of code in which they are defined. This is called their **scope** or **lexical scope**:

```java
void method() { // A method definition
    int i = 0; // Declare variable i
    while (i < 10) { // i is in scope here
        int j = 0; // Declare j; the scope of j begins here
        i++; // i is in scope here; increment it
    } // j is no longer in scope;
    System.out.println(i); // i is still in scope here
} // The scope of i ends here
```

---

## Understand other concepts about Java statement

- [[Conditional Statements If Else and Else If]]
- [[While Loop]]
- [[Foreach Loop]]
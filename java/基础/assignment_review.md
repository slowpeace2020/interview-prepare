### 1. What is Java variables and How to declare Java variables?

**Java Variables:**
A variable in Java is a container that holds data that can be changed during the execution of a program. Variables have a data type, a name, and a scope. They are fundamental to programming because they allow developers to store and manipulate data. In Java, variables must be declared with a specific type, which determines the kind of data they can hold, such as integers, floating-point numbers, characters, and more.

**Types of Java Variables:**
**Instance Variables**: These are non-static fields and are declared inside a class but outside any method. They are unique to each instance of a class.  
**Class Variables (Static Variables)**: These are declared with the static keyword and are shared among all instances of a class.  
**Local Variables**: These are declared inside a method, constructor, or block and are only accessible within that method, constructor, or block.  
**Parameters**: These are variables that are passed to methods as arguments.

**How to Declare Java Variables:**
Declaring a variable in Java involves specifying the variable's type and name. The syntax is:

In Java, variables are crucial for storing and manipulating data. They must be declared with a type and a name, and can be categorized into instance variables, class variables, local variables, and parameters, each serving a different purpose and scope within a program. Understanding how to declare and use these variables effectively is essential for writing robust and efficient Java programs.


### 2. What are Java data types (primitive & non-primitive)?

In Java, data types are divided into two main categories: primitive and non-primitive.

#### 1. Primitive Data Types

Primitive data types are the most basic data types built into the Java language. They are not objects and hold their values directly in memory. There are eight primitive data types:

1. **byte**:
    - Size: 8-bit
    - Range: -128 to 127
    - Example: `byte b = 100;`

2. **short**:
    - Size: 16-bit
    - Range: -32,768 to 32,767
    - Example: `short s = 1000;`

3. **int**:
    - Size: 32-bit
    - Range: -2^31 to 2^31-1
    - Example: `int i = 100000;`

4. **long**:
    - Size: 64-bit
    - Range: -2^63 to 2^63-1
    - Example: `long l = 100000L;`

5. **float**:
    - Size: 32-bit floating-point
    - Example: `float f = 10.5f;`

6. **double**:
    - Size: 64-bit floating-point
    - Example: `double d = 10.5;`

7. **char**:
    - Size: 16-bit Unicode character
    - Range: '\u0000' (or 0) to '\uffff' (or 65,535 inclusive)
    - Example: `char c = 'A';`

8. **boolean**:
    - Size: 1-bit
    - Values: true or false
    - Example: `boolean b = true;`

#### 2. Non-Primitive Data Types

Non-primitive data types, also known as reference types, are more complex data types that refer to objects. Unlike primitive types, non-primitive types are defined by the user. They can hold multiple values and have methods to perform operations on these values. Examples include:

#### Key Differences

- **Size**: Primitive types have a fixed size, whereas non-primitive types can vary in size.
- **Default Values**: Primitive types have default values (e.g., 0 for numeric types, false for boolean), while non-primitive types default to `null`.
- **Methods**: Non-primitive types have methods to perform various operations, while primitive types do not.
- **Storage**: Primitive types store actual values, while non-primitive types store references to objects.

### 3. What is wrapper class in Java and why we need wrapper class?

Wrapper classes in Java are classes that provide a way to use primitive data types (such as `int`, `boolean`, etc.) as objects. Each primitive data type has a corresponding wrapper class in the `java.lang` package:

- `byte` -> `Byte`
- `short` -> `Short`
- `int` -> `Integer`
- `long` -> `Long`
- `float` -> `Float`
- `double` -> `Double`
- `char` -> `Character`
- `boolean` -> `Boolean`

#### Why We Need Wrapper Classes

1. **Object Requirements**:
   Some Java APIs require objects instead of primitive types. For example, collections in the `java.util` package (like `ArrayList`, `HashMap`) work with objects, not primitives. Using wrapper classes allows you to store primitive data types in these collections.

   ```java
   List<Integer> intList = new ArrayList<>();
   intList.add(5); // Adding an int value to an ArrayList
   ```

2. **Utilities and Methods**:
   Wrapper classes provide numerous utility methods to handle primitive data types. For example, the `Integer` class provides methods for parsing strings to integers (`Integer.parseInt()`), comparing integers (`Integer.compare()`), and more.

   ```java
   String str = "123";
   int num = Integer.parseInt(str); // Converts string to int
   ```

3. **Immutability**:
   Wrapper classes are immutable, meaning their values cannot be changed once they are created. This is useful for ensuring that the data they hold remains constant throughout the program.

   ```java
   Integer num = 10;
   num = 20; // This creates a new Integer object rather than modifying the existing one
   ```

4. **Autoboxing and Unboxing**:
   Java provides a feature called autoboxing, which automatically converts primitive types into their corresponding wrapper class objects, and unboxing, which converts wrapper class objects back to primitive types. This makes it easier to work with primitives and objects interchangeably.

   ```java
   Integer num = 5; // Autoboxing: int to Integer
   int n = num; // Unboxing: Integer to int
   ```

5. **Null Values**:
   Wrapper classes can be used to represent the absence of a value with `null`, whereas primitive types cannot. This is useful in situations where a variable needs to be able to represent a missing or undefined value.

   ```java
   Integer num = null; // Possible with wrapper class
   // int num = null; // Not possible with primitive type
   ```

6. **Type Conversion**:
   Wrapper classes provide methods to convert between different data types. For example, `Integer` has methods to convert an integer to a string, and vice versa.

   ```java
   int num = 100;
   String str = Integer.toString(num); // Convert int to String
   int convertedNum = Integer.valueOf(str); // Convert String to int
   ```

#### Example of Wrapper Classes in Use

Here is an example that demonstrates the use of wrapper classes in a Java program:

```java
import java.util.ArrayList;
import java.util.List;

public class WrapperClassExample {
    public static void main(String[] args) {
        // Autoboxing: Automatically converting int to Integer
        List<Integer> numbers = new ArrayList<>();
        numbers.add(5);
        numbers.add(10);

        // Unboxing: Automatically converting Integer to int
        int total = 0;
        for (Integer num : numbers) {
            total += num; // Unboxing occurs here
        }

        System.out.println("Total: " + total);

        // Using utility methods of wrapper classes
        String str = "123";
        int parsedValue = Integer.parseInt(str);
        System.out.println("Parsed Value: " + parsedValue);
    }
}
```


Wrapper classes are essential in Java for providing object representations of primitive data types, which allows for the use of primitives in collections, provides utility methods, supports autoboxing and unboxing, and represents null values. This makes them a vital part of Java programming, especially in modern applications where primitives and objects are used interchangeably.


### 4. What is the differences between passing by value and passing be reference? (please do some research)
#### Pass by Value
- **Definition**: In pass by value, a copy of the actual parameter's value is made in memory and passed to the function. Changes made to the parameter inside the function do not affect the original value.
- **Memory Requirement**: This method generally requires more memory because a new copy of the data is created.
- **Time Requirement**: Pass by value can be slower due to the overhead of copying values.
- **Usage**: Commonly used in languages like C and Java, where primitives are passed by value. Even in Java, objects are passed by value; the value is the reference to the object, not the object itself.

#### Pass by Reference
- **Definition**: In pass by reference, a reference (or address) to the actual parameter is passed to the function. This allows the function to modify the original parameter's value.
- **Memory Requirement**: Typically requires less memory since no copy of the actual data is made.
- **Time Requirement**: Pass by reference can be faster because it avoids the overhead of copying data.
- **Usage**: Utilized in languages like C++ and Python. This method is particularly useful when you want a function to modify the original variable or when passing large data structures that would be inefficient to copy.

### Key Differences
1. **Effect on Original Data**:
    - **Pass by Value**: Changes within the function do not affect the original data.
    - **Pass by Reference**: Changes within the function directly affect the original data.

2. **Memory and Performance**:
    - **Pass by Value**: Involves copying data, leading to higher memory usage and potential performance overhead.
    - **Pass by Reference**: Involves passing a reference, thus lower memory usage and typically faster performance.

3. **Usage Scenarios**:
    - **Pass by Value**: Preferred when you do not want the function to modify the original data.
    - **Pass by Reference**: Preferred when you want the function to modify the original data or when dealing with large data structures.

### 5. What is an Immutable class in Java?

An immutable class in Java is a class whose objects cannot be modified once they are created. Any attempt to change the state of an immutable object results in the creation of a new object with the modified state. Immutable classes are particularly useful in concurrent programming because they are inherently thread-safe.

#### Key Characteristics of Immutable Classes

1. **Final Class**: The class is declared as final so that it cannot be subclassed.
2. **Private Final Fields**: All fields of the class are private and final, ensuring that they cannot be modified after initialization.
3. **No Setters**: The class does not provide any setter methods that can alter the fields.
4. **Initialization through Constructor**: All fields are initialized via a constructor, ensuring that the object is fully constructed before it can be accessed.
5. **Deep Copy for Mutable Fields**: If the class has fields that are mutable objects, a deep copy of these objects is made during construction and returned by any getter methods to prevent external modification.

#### Example of an Immutable Class

Here is an example of an immutable class in Java:

```java
public final class ImmutableClass {
    private final String name;
    private final int age;
    private final List<String> hobbies;

    public ImmutableClass(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Creating a deep copy of the mutable field
        this.hobbies = new ArrayList<>(hobbies);
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public List<String> getHobbies() {
        // Returning a copy of the mutable field to prevent modification
        return new ArrayList<>(hobbies);
    }
}
```

#### Benefits of Immutable Classes

1. **Thread Safety**: Immutable objects are inherently thread-safe because their state cannot be modified after creation. This makes them ideal for use in concurrent applications without needing additional synchronization.
2. **Simplicity**: The design of immutable classes is simpler and less error-prone because there are no concerns about the object's state changing over time.
3. **Caching and Reuse**: Immutable objects can be cached and reused freely without concern for their state being altered, which can lead to performance improvements.

#### Creating Immutable Objects in Java

When creating immutable objects, it is crucial to ensure that all fields are properly initialized and that no references to mutable objects are leaked. Here is a checklist to follow:

1. **Declare the class as final**: This prevents subclassing.
   ```java
   public final class ImmutableClass {
       // Class implementation
   }
   ```

2. **Make all fields private and final**: This ensures fields are not accessible or modifiable directly.
   ```java
   private final String name;
   private final int age;
   ```

3. **Provide a constructor for initializing fields**: Ensure that all fields are initialized through the constructor.
   ```java
   public ImmutableClass(String name, int age) {
       this.name = name;
       this.age = age;
   }
   ```

4. **No setters**: Do not provide setter methods for fields.
   ```java
   // No setter methods
   ```

5. **Return deep copies for mutable fields**: If a field is a mutable object, return a deep copy to prevent external modifications.
   ```java
   public List<String> getHobbies() {
       return new ArrayList<>(hobbies);
   }
   ```

### 6. How to create a custom immutable class in Java?
Creating a custom immutable class in Java involves ensuring that once an object is created, its state cannot be modified. Here is a step-by-step guide to creating a custom immutable class in Java:

#### Steps to Create an Immutable Class

1. **Declare the Class as Final**: This prevents other classes from inheriting it.
2. **Make All Fields Private and Final**: This ensures that the fields are not accessible directly and cannot be modified after initialization.
3. **Initialize All Fields via a Constructor**: Ensure that all fields are initialized when the object is created.
4. **No Setter Methods**: Do not provide any methods that modify fields.
5. **Deep Copy for Mutable Fields**: If the class contains fields that reference mutable objects, create deep copies for these fields.

```java
import java.util.ArrayList;
import java.util.List;

public final class CustomImmutableClass {
    private final String name;
    private final int age;
    private final List<String> hobbies;

    // Constructor to initialize all fields
    public CustomImmutableClass(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Create a deep copy of the mutable list to prevent external modifications
        this.hobbies = new ArrayList<>(hobbies);
    }

    // Getter methods to access the fields
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // Return a copy of the mutable list to ensure immutability
    public List<String> getHobbies() {
        return new ArrayList<>(hobbies);
    }

    // Override equals and hashCode for correct behavior in collections
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        CustomImmutableClass that = (CustomImmutableClass) o;

        if (age != that.age) return false;
        if (!name.equals(that.name)) return false;
        return hobbies.equals(that.hobbies);
    }

    @Override
    public int hashCode() {
        int result = name.hashCode();
        result = 31 * result + age;
        result = 31 * result + hobbies.hashCode();
        return result;
    }

    // Optional: Override toString for better debugging
    @Override
    public String toString() {
        return "CustomImmutableClass{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", hobbies=" + hobbies +
                '}';
    }
}
```


1. **Final Class**: `public final class CustomImmutableClass` ensures that the class cannot be subclassed.
2. **Private Final Fields**: All fields are private and final, meaning they can only be set once.
3. **Constructor Initialization**: The constructor initializes all fields. For mutable fields like lists, a deep copy is made.
4. **No Setters**: No setter methods are provided to modify the fields.
5. **Deep Copy**: The `hobbies` list is copied in the constructor and in the getter method to prevent modifications to the original list from outside the class.
6. **Override `equals` and `hashCode`**: These methods are overridden to ensure correct behavior when instances are used in collections like `HashMap` or `HashSet`.
7. **Override `toString`**: This is optional but useful for debugging purposes.

### 7. What is String pool in Java and why we need String pool?

The **String Pool** (also known as the intern pool or string literal pool) is a special storage area in the Java heap memory. It is part of the Java runtime and stores string literals. When a string literal is created, the JVM first checks if the literal is already present in the pool. If it is, the reference to the existing literal is returned. If it is not, the literal is added to the pool.

#### Why We Need String Pool

1. **Memory Efficiency**:
    - **Reduction of Duplicate Strings**: The string pool helps save memory by avoiding the storage of multiple instances of identical strings. When a new string literal is created, the JVM checks the pool. If an identical string exists, the same reference is used, eliminating the need to create a new object.
    - **Example**:
      ```java
      String str1 = "Hello";
      String str2 = "Hello";
      // Both str1 and str2 point to the same memory location in the string pool
      ```

2. **Performance Improvement**:
    - **Faster Comparisons**: Comparing string literals is faster since it involves comparing references instead of actual string values. This is particularly beneficial in large-scale applications where string comparisons are frequent.

3. **Immutable Strings**:
    - **Thread Safety**: Since strings in the pool are immutable, they can be safely shared across multiple threads without synchronization. This reduces the overhead and complexity of concurrent programming.

#### How String Pool Works

When you create a string literal, the JVM checks if the string already exists in the pool:


**String Object Creation Using `new`**:
- When using the `new` keyword, a new string object is created in the heap, not in the string pool. Even if the same string exists in the pool, a new object is created.


**Interning Strings**:
- You can explicitly add a string to the pool using the `intern()` method. This method checks if the string is in the pool, and if not, it adds the string to the pool.

#### Example Code Demonstration

Here is a code example that demonstrates the concept of the string pool:

```java
public class StringPoolExample {
    public static void main(String[] args) {
        // String literals
        String str1 = "Hello";
        String str2 = "Hello";
        
        // Reference comparison, returns true
        System.out.println(str1 == str2);

        // String objects using new
        String str3 = new String("Hello");
        String str4 = new String("Hello");
        
        // Reference comparison, returns false
        System.out.println(str3 == str4);
        
        // Using intern() method
        String str5 = str3.intern();
        
        // Reference comparison, returns true
        System.out.println(str1 == str5);
    }
}
```

In this example:
- `str1` and `str2` are string literals, so they refer to the same object in the string pool.
- `str3` and `str4` are created using `new`, so they are different objects in the heap.
- `str5` is the interned version of `str3`, so it refers to the same object as `str1` in the string pool.

The string pool is a crucial part of Java's memory management, providing both memory efficiency and performance improvements. By storing string literals in a centralized pool, Java minimizes memory usage and allows for faster string comparisons, which is particularly beneficial in applications with heavy use of strings.

### 8. What are the results of following expressions? Please include the calculation process.
5 & 6
5 | 6
5 ^ 6
To understand the results of the expressions `5 & 6`, `5 | 6`, and `5 ^ 6`, we need to first convert the numbers to their binary forms and then apply the bitwise operators.

#### Binary Representations
- \(5_{10} = 0101_2\)
- \(6_{10} = 0110_2\)

#### Bitwise AND (`&`)
The bitwise AND operator compares each bit of its first operand to the corresponding bit of its second operand. If both bits are 1, the corresponding result bit is set to 1. Otherwise, the corresponding result bit is set to 0.

```
   0101 (binary for 5)
&  0110 (binary for 6)
----------
   0100 (binary result)
```

Converting `0100` back to decimal:
- \( 0100_2 = 4_{10} \)

So, \( 5 & 6 = 4 \).

#### Bitwise OR (`|`)
The bitwise OR operator compares each bit of its first operand to the corresponding bit of its second operand. If either bit is 1, the corresponding result bit is set to 1. If both bits are 0, the corresponding result bit is set to 0.

```
   0101 (binary for 5)
|  0110 (binary for 6)
----------
   0111 (binary result)
```

Converting `0111` back to decimal:
- \( 0111_2 = 7_{10} \)

So, \( 5 | 6 = 7 \).

#### Bitwise XOR (`^`)
The bitwise XOR operator compares each bit of its first operand to the corresponding bit of its second operand. If the bits are different, the corresponding result bit is set to 1. If the bits are the same, the corresponding result bit is set to 0.

```
   0101 (binary for 5)
^  0110 (binary for 6)
----------
   0011 (binary result)
```

Converting `0011` back to decimal:
- \( 0011_2 = 3_{10} \)

So, \( 5 ^ 6 = 3 \).

#### Summary
- \( 5 & 6 = 4 \)
- \( 5 | 6 = 7 \)
- \( 5 ^ 6 = 3 \)

These results are derived from applying the bitwise operations on the binary representations of the numbers 5 and 6.


### 9. Why we need to use break statement in switch-case statement?

The `break` statement is essential in a `switch-case` statement to ensure that the program exits the `switch` block after executing the code for a matched `case`. Without a `break` statement, the program will continue executing the subsequent `case` blocks regardless of their conditions, a behavior known as "fall-through."

#### Key Reasons for Using `break` in `switch-case` Statements:

1. **Preventing Fall-Through**:
    - By default, in a `switch` statement, once a matching `case` is found and its associated code is executed, the execution does not stop. Instead, it continues to the next `case` statement. This is often undesirable and can lead to unexpected behavior. The `break` statement ensures that once a `case` is executed, the program exits the `switch` block.

2. **Code Clarity and Maintainability**:
    - Including `break` statements makes the code easier to understand and maintain. It clearly indicates the end of a `case` block, reducing the likelihood of unintentional fall-through bugs.

3. **Performance Efficiency**:
    - Using `break` prevents the unnecessary execution of subsequent `case` blocks. This can be particularly important in scenarios with complex or resource-intensive operations within each `case`.

#### What Happens Without `break` Statements:

If `break` statements are omitted, the program will exhibit fall-through behavior, executing all subsequent `case` blocks until a `break` is encountered or the `switch` block ends. This can lead to unexpected results and bugs.

The `break` statement in a `switch-case` construct is crucial for controlling the flow of execution, preventing fall-through, improving code clarity, and ensuring performance efficiency. Proper use of `break` statements helps maintain the expected behavior of the program and avoids potential bugs.


### 10. What are access modifiers and their corresponding scopes in Java?

Access modifiers in Java determine the visibility and accessibility of classes, methods, and variables. They define the scope in which these entities can be accessed. There are four primary access modifiers in Java: `public`, `protected`, `default` (package-private), and `private`.

#### 1. `public`

- **Scope**: Visible to all classes everywhere, whether they are in the same package or have been imported.
- **Usage**:
    - **Classes**: Can be accessed from any other class.
    - **Methods/Variables**: Can be accessed from any other class.

#### 2. `protected`

- **Scope**: Visible within the same package and subclasses (including those in different packages).
- **Usage**:
    - **Methods/Variables**: Accessible within its own package and by subclasses.
    - **Classes**: `protected` is not applicable to top-level classes.

#### 3. `default` (Package-Private)

- **Scope**: Visible only within its own package; not accessible from classes in other packages.
- **Usage**:
    - **Classes**: Can be accessed only within the same package.
    - **Methods/Variables**: Can be accessed only within the same package.

#### 4. `private`

- **Scope**: Visible only within the class it is declared; not accessible from any other class.
- **Usage**:
    - **Methods/Variables**: Can be accessed only within the same class.
    - **Classes**: `private` is not applicable to top-level classes.

### Access Modifiers Summary

| Modifier    | Class | Package | Subclass (same pkg) | Subclass (diff pkg) | Global |
|-------------|-------|---------|---------------------|----------------------|--------|
| `public`    | Yes   | Yes     | Yes                 | Yes                  | Yes    |
| `protected` | No    | Yes     | Yes                 | Yes                  | No     |
| `default`   | No    | Yes     | Yes                 | No                   | No     |
| `private`   | No    | No      | No                  | No                   | No     |


Access modifiers in Java are essential for encapsulating and protecting data. They allow developers to control the visibility and accessibility of classes, methods, and variables, ensuring that sensitive data is protected and that the internal implementation details are hidden from external classes.

### 11. What is static field and static method?

#### Static Field

A static field (also known as a class variable) is a field that belongs to the class itself rather than to any particular instance of the class. This means that there is only one copy of a static field that is shared among all instances of the class.

**Key Characteristics:**
1. **Shared Across All Instances**: Static fields are shared among all instances of a class. If one instance modifies a static field, the change is reflected across all instances.
2. **Class-Level Scope**: Static fields are associated with the class, not the instances. They can be accessed directly using the class name.
3. **Memory Management**: Static fields are created when the class is loaded and destroyed when the class is unloaded. They exist for the lifetime of the application if the class remains loaded.

**Example:**
```java
public class MyClass {
    public static int staticField = 0;

    public void incrementStaticField() {
        staticField++;
    }
}

public class Main {
    public static void main(String[] args) {
        MyClass obj1 = new MyClass();
        MyClass obj2 = new MyClass();

        obj1.incrementStaticField();
        obj2.incrementStaticField();

        System.out.println(MyClass.staticField); // Output: 2
    }
}
```

#### Static Method

A static method (also known as a class method) is a method that belongs to the class itself rather than to any particular instance of the class. Static methods can be called without creating an instance of the class.

**Key Characteristics:**
1. **Class-Level Scope**: Static methods can be called using the class name. They do not require an instance of the class to be invoked.
2. **No Access to Instance Variables**: Static methods cannot access instance variables or instance methods directly. They can only access static fields and other static methods.
3. **Utility Methods**: Static methods are often used for utility or helper methods that do not require any object state.

**Example:**
```java
public class MathUtils {
    public static int add(int a, int b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        int sum = MathUtils.add(5, 10);
        System.out.println(sum); // Output: 15
    }
}
```

#### Differences Between Static and Instance Members

1. **Association**:
    - **Static**: Associated with the class itself.
    - **Instance**: Associated with an instance of the class.

2. **Access**:
    - **Static Field**: Accessed using the class name.
      ```java
      MyClass.staticField;
      ```
    - **Instance Field**: Accessed using an instance of the class.
      ```java
      obj.instanceField;
      ```

3. **Lifetime**:
    - **Static**: Exists for the duration of the program as long as the class is loaded.
    - **Instance**: Exists as long as the instance of the class exists.

4. **Memory**:
    - **Static**: Stored in the static memory area.
    - **Instance**: Stored in the heap memory area.

### Use Cases

- **Static Fields**:
    - Used for constants (e.g., `public static final int MAX_VALUE = 100;`).
    - Used to keep track of global states or counters shared across all instances.

- **Static Methods**:
    - Used for utility functions (e.g., `Math.max()`, `Collections.sort()`).
    - Used when a method does not depend on the state of an object.

### Summary

Static fields and methods in Java are associated with the class rather than any particular instance of the class. Static fields are useful for shared data across all instances, while static methods are useful for utility operations that do not depend on instance data. They are accessed using the class name and provide a way to define class-level behavior and data.

### 12. Explain the main method in Java.
### The `main` Method in Java

The `main` method in Java is the entry point of any standalone Java application. It is the method that the Java Virtual Machine (JVM) calls to start the execution of a program. Understanding the `main` method's structure, purpose, and common usage is essential for any Java developer.

#### Structure of the `main` Method

The `main` method has a specific signature that must be followed exactly for the JVM to recognize it as the entry point:

```java
public static void main(String[] args)
```

1. **public**:
    - The method is public, meaning it can be accessed from outside the class. This is necessary because the JVM needs to access this method from outside the class to start the application.

2. **static**:
    - The method is static, meaning it belongs to the class itself rather than an instance of the class. This allows the JVM to call the `main` method without creating an instance of the class.

3. **void**:
    - The method returns no value. The `main` method is meant to run the application and does not need to return any data to the JVM.

4. **main**:
    - This is the name of the method. The JVM looks for this specific method name to start the execution of the program.

5. **String[] args**:
    - The parameter is an array of `String` objects. This array stores any command-line arguments passed to the application when it is started. The `args` array allows users to pass arguments to the program, enabling the customization of program behavior based on input.

#### Example of a `main` Method

Here is a simple example of a `main` method in a Java program:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

When this program is executed, the JVM calls the `main` method, which prints "Hello, World!" to the console.

#### Command-Line Arguments

The `args` array allows you to pass arguments to your program from the command line. For example:


#### Why `main` Method is Important

1. **Starting Point**:
    - The `main` method is where the JVM begins program execution. Without a `main` method, the JVM wouldn't know where to start running the code.

2. **Program Initialization**:
    - It's often used to initialize the program, set up any necessary resources, and launch the primary logic of the application.

3. **Command-Line Interface**:
    - Allows the application to accept input from the command line, which can be used to customize the application's behavior without modifying the code.

#### Alternative Signatures

The `main` method can also accept `varargs` (variable-length argument list) instead of an array:

```java
public static void main(String... args)
```

This is functionally equivalent to `String[] args` and provides a more flexible way to handle arguments.


The `main` method in Java is crucial as the entry point for any standalone application. Its specific signature allows the JVM to start the execution of the program, and the `args` array provides a mechanism for passing command-line arguments to the application. 


## Day 3
1. **What is Object in Java and why do we need objects?**

   In Java, an object is an instance of a class. It is a fundamental entity that combines state (fields or attributes) and behavior (methods). Objects are essential because they enable the implementation of object-oriented programming principles, such as encapsulation, inheritance, and polymorphism. They allow us to model real-world entities and manage complex systems more efficiently.

2. **What is Inheritance and how many types of inheritance are supported by Java?**

   Inheritance is a mechanism in Java that allows one class (subclass or derived class) to inherit fields and methods from another class (superclass or base class). This promotes code reuse and establishes a relationship between classes.

   Java supports the following types of inheritance:
    - Single Inheritance: A class inherits from one superclass.
    - Multilevel Inheritance: A class inherits from a superclass, which in turn inherits from another superclass.
    - Hierarchical Inheritance: Multiple classes inherit from a single superclass.

   Java does not support multiple inheritance (a class inheriting from multiple superclasses) directly due to the diamond problem. However, it can be achieved using interfaces.

3. **What is the diamond problem in Java? And how can we resolve the problem?**

   The diamond problem occurs when a class inherits from two classes that both inherit from a common superclass, leading to ambiguity. Java avoids the diamond problem by not allowing multiple inheritance with classes. Instead, Java uses interfaces to achieve multiple inheritance. If two interfaces provide default methods with the same signature, the implementing class must override the method to resolve the ambiguity.

4. **What is an Interface and what is an abstract class? What are the differences between them?**

   An interface in Java is a reference type that can contain abstract methods, default methods, static methods, and constants. Interfaces are used to define a contract that implementing classes must adhere to.

   An abstract class is a class that cannot be instantiated on its own and may contain abstract methods (methods without a body) as well as concrete methods (methods with a body).

   Differences:
    - Interfaces can only contain abstract methods (until Java 8 introduced default and static methods), whereas abstract classes can contain both abstract and concrete methods.
    - A class can implement multiple interfaces but can only inherit from one abstract class.
    - Interfaces are used to define a contract for functionality, while abstract classes provide a common base with shared implementation.

5. **What is Polymorphism? And how does Java implement it?**

   Polymorphism is the ability of an object to take on many forms. It allows one interface to be used for a general class of actions. The specific action is determined by the exact nature of the situation.

   Java implements polymorphism in two ways:
    - Compile-time polymorphism (method overloading): Multiple methods have the same name but different parameter lists within the same class.
    - Runtime polymorphism (method overriding): A subclass provides a specific implementation of a method that is already defined in its superclass.

6. **What are the differences between overriding and overloading?**

    - Overriding: It occurs when a subclass provides a specific implementation for a method that is already defined in its superclass. The method signature (name and parameters) must be the same, and it happens at runtime.
    - Overloading: It occurs when multiple methods in the same class have the same name but different parameter lists. It happens at compile-time and allows different ways to perform similar operations.

7. **What is Encapsulation? How does Java implement it? And why do we need encapsulation?**

   Encapsulation is the practice of bundling data (fields) and methods that operate on that data into a single unit or class. It restricts direct access to some of an object's components and can prevent the accidental modification of data.

   Java implements encapsulation by:
    - Using private access modifiers for fields to restrict access.
    - Providing public getter and setter methods to access and update the private fields.

   Encapsulation is needed to:
    - Protect the internal state of an object.
    - Enhance code maintainability and flexibility.
    - Ensure controlled access to the data.

8. **What is the difference between abstraction and encapsulation?**

    - Abstraction: It is the concept of hiding the complex implementation details and showing only the essential features of an object. It is achieved through abstract classes and interfaces.
    - Encapsulation: It is the practice of bundling data and methods into a single unit and restricting access to the inner workings of that unit. It is implemented using access modifiers and getter/setter methods.

   While abstraction focuses on exposing the relevant information, encapsulation focuses on restricting access and protecting the data.

9. **What is toString() and why do we need it?**

   The `toString()` method in Java is defined in the `Object` class and is used to return a string representation of an object. It is often overridden in classes to provide a meaningful string representation of the object's state. It is useful for debugging and logging purposes, allowing developers to easily inspect object values.

10. **Can we use the this keyword in a constructor? How about super?**

- `this`: The `this` keyword can be used in a constructor to refer to the current object instance. It is often used to invoke another constructor in the same class (constructor chaining).
- `super`: The `super` keyword can be used in a constructor to call the constructor of the superclass. It must be the first statement in the constructor if used.

## Day 4
### 1. Describe the Collections Type Hierarchy. What Are the Main Interfaces, and What Are the Differences Between Them?

The Java Collections Framework is a unified architecture for representing and manipulating collections. The main interfaces in the collection hierarchy are:

- **Collection**: The root of the collection hierarchy.
    - **List**: An ordered collection (also known as a sequence). Duplicates are allowed.
    - **Set**: A collection that does not allow duplicates.
    - **Queue**: A collection used to hold multiple elements prior to processing. Typically ordered in a FIFO (First-In-First-Out) manner.
    - **Deque**: A double-ended queue that allows insertion and removal from both ends.
    - **Map**: An object that maps keys to values. Not a true collection, but part of the framework.

### 2. What Are List Interface Implementations and What Are the Differences Between Them and When to Use What?

**List** interface implementations include:

- **ArrayList**:
    - **Characteristics**: Resizable-array implementation.
    - **Usage**: Best for fast random access and frequent iteration.

- **LinkedList**:
    - **Characteristics**: Doubly-linked list implementation.
    - **Usage**: Best for frequent insertions and deletions.

- **Vector**:
    - **Characteristics**: Synchronized resizable-array.
    - **Usage**: Legacy class; use ArrayList for non-synchronized needs.

- **CopyOnWriteArrayList**:
    - **Characteristics**: Thread-safe variant of ArrayList.
    - **Usage**: Best for concurrent read-mostly scenarios.

### 3. What Are Queue Interface Implementations and What Are the Differences and When to Use What?

**Queue** interface implementations include:

- **LinkedList**:
    - **Characteristics**: Implements both List and Queue.
    - **Usage**: Suitable for general-purpose queues and double-ended queues (Deque).

- **PriorityQueue**:
    - **Characteristics**: Priority heap-based implementation.
    - **Usage**: Best for priority-based element processing.

- **ArrayDeque**:
    - **Characteristics**: Resizable array implementation.
    - **Usage**: Suitable for implementing stacks and queues.

### 4. What Are Set Interface Implementations and What Are the Differences and When to Use What?

**Set** interface implementations include:

- **HashSet**:
    - **Characteristics**: Backed by a hash table.
    - **Usage**: Best for fast lookups and unique elements without order.

- **LinkedHashSet**:
    - **Characteristics**: Hash table and linked list implementation.
    - **Usage**: Maintains insertion order; use when order is important.

- **TreeSet**:
    - **Characteristics**: Navigable set implementation backed by a tree.
    - **Usage**: Automatically sorted; use for sorted order elements.

### 5. Explain the Structure of the Deque Implementation of LinkedList and Its Usage.

- **LinkedList as Deque**:
    - **Characteristics**: Implements the Deque interface with a doubly-linked list.
    - **Usage**: Can be used as both a stack and a queue. Provides efficient insertions and deletions from both ends.

### 6. What Are the Differences Between HashMap, LinkedHashMap and TreeMap?

- **HashMap**:
    - **Characteristics**: Unordered; allows one null key and multiple null values.
    - **Usage**: Best for fast access and insertion without regard to order.

- **LinkedHashMap**:
    - **Characteristics**: Maintains insertion order.
    - **Usage**: Use when iteration order is important.

- **TreeMap**:
    - **Characteristics**: Sorted according to the natural ordering of its keys or by a comparator.
    - **Usage**: Use for naturally sorted order or custom order using a comparator.

### 7. What Is the hashCode() and equals() Function?

- **hashCode()**:
    - **Purpose**: Returns an integer hash code value for the object.
    - **Usage**: Used in hashing-based collections like HashMap, HashSet.

- **equals()**:
    - **Purpose**: Compares the current object with another object for equality.
    - **Usage**: Used to determine logical equality of objects.

### 8. How Is HashMap Implemented in Java? How Does Its Implementation Use hashCode and equals Methods of Objects? What Is the Time Complexity of Putting and Getting an Element from Such Structure?

- **Implementation**:
    - Uses an array of linked lists (or buckets). Each bucket corresponds to a hash code.

- **Use of hashCode and equals**:
    - **hashCode()**: Determines the bucket where the key-value pair should be stored.
    - **equals()**: Ensures correct key-value pair is accessed or modified.

- **Time Complexity**:
    - **Average**: O(1) for put() and get().
    - **Worst-case**: O(n) when many keys hash to the same bucket.

### 9. What Is Comparable and Comparator Interface? What Are Differences Between Them and How to Use Them?

- **Comparable**:
    - **Purpose**: Used to define the natural ordering of objects.
    - **Method**: `compareTo(Object o)`
    - **Usage**: Implemented by a class to define its natural ordering.

- **Comparator**:
    - **Purpose**: Used to define an external comparison logic.
    - **Method**: `compare(Object o1, Object o2)`
    - **Usage**: Implemented by a separate class or as a lambda expression to define custom ordering.

### 10. What Is Functional Interface? How Do You Create Your Own Functional Interface?

- **Functional Interface**:
    - **Purpose**: An interface with a single abstract method (SAM).
    - **Example**: `@FunctionalInterface interface MyFunctionalInterface { void execute(); }`
    - **Usage**: Can be used as the target for lambda expressions and method references.

### 11. What Is Lambda Expression? Why Does Lambda Expression Work So Closely with Functional Interface?

- **Lambda Expression**:
    - **Purpose**: Provides a clear and concise way to represent one method interface using an expression.
    - **Syntax**: `(parameters) -> expression`
    - **Usage**: Primarily used to define the behavior of functional interfaces.

- **Relationship with Functional Interface**:
    - **Reason**: Lambda expressions implement the single abstract method defined by a functional interface, making them a natural fit for providing inline implementation of these interfaces.


## Day 5

#### 1. What is the Stream API in Java 8? What are the differences between intermediate and terminal operations for Stream?

The **Stream API** in Java 8 provides a modern and functional approach to process collections of objects. It allows operations such as filtering, mapping, and reducing data in a fluent style.

**Intermediate Operations**:
- Transform a stream into another stream.
- Are lazy, meaning they are not executed until a terminal operation is invoked.
- Examples: `filter()`, `map()`, `flatMap()`, `sorted()`, `distinct()`

**Terminal Operations**:
- Produce a result or a side-effect.
- Trigger the execution of the pipeline of stream operations.
- Examples: `collect()`, `forEach()`, `reduce()`, `count()`, `findAny()`

#### 2. What is the Optional class in Java and why do we need it?

The **Optional** class in Java is a container object which may or may not contain a non-null value. It helps to avoid `NullPointerException` by providing a more expressive way to handle nullable values.

**Usage**:
- To represent optional values explicitly instead of using null.
- To provide methods that facilitate the handling of present or absent values.
- Example:
  ```java
  Optional<String> optional = Optional.ofNullable(value);
  optional.ifPresent(System.out::println);
  String defaultValue = optional.orElse("default");
  ```

#### 3. What is an exception and what is an error in Java? What are the differences between them?

**Exception**:
- An event that disrupts the normal flow of the program.
- Can be caught and handled by the application.
- Example: `IOException`, `SQLException`

**Error**:
- Serious problems that a reasonable application should not try to catch.
- Indicates issues such as system crashes, memory leaks, etc.
- Example: `OutOfMemoryError`, `StackOverflowError`

**Differences**:
- **Exceptions** are conditions that applications might want to catch.
- **Errors** are conditions that are generally outside the control of the application.

#### 4. What are the types of exceptions in Java? What are the differences?

**Checked Exceptions**:
- Checked at compile time.
- Must be declared in the method signature using `throws` or handled with a try-catch block.
- Example: `IOException`, `SQLException`

**Unchecked Exceptions**:
- Checked at runtime.
- Subclass of `RuntimeException`.
- No requirement to declare them or handle explicitly.
- Example: `NullPointerException`, `ArrayIndexOutOfBoundsException`

#### 5. How can we handle an exception in Java?

Exceptions in Java can be handled using `try`, `catch`, `finally`, and `throw`.

- **try-catch** block:
  ```java
  try {
      // code that may throw an exception
  } catch (ExceptionType e) {
      // code to handle the exception
  }
  ```

- **finally** block:
  ```java
  try {
      // code that may throw an exception
  } catch (Exception e) {
      // handling exception
  } finally {
      // code that will always execute
  }
  ```

- **throw** keyword:
  ```java
  public void method() throws ExceptionType {
      // code that may throw an exception
      throw new ExceptionType("Error message");
  }
  ```

#### 6. What is a custom exception and why do we need to use it?

**Custom Exception**:
- A user-defined exception class that extends `Exception` or `RuntimeException`.

**Purpose**:
- To create meaningful and domain-specific exceptions that provide more information about an error condition.
- Example:
  ```java
  public class CustomException extends Exception {
      public CustomException(String message) {
          super(message);
      }
  }
  ```

#### 7. What is the difference between the final and finally keywords?

**final**:
- Used to declare constants, prevent inheritance, and prevent method overriding.
- Example:
  ```java
  final int CONSTANT = 10; // constant variable
  final class FinalClass {} // final class
  final void method() {} // final method
  ```

**finally**:
- Used in exception handling to execute important code such as closing resources, irrespective of whether an exception is thrown or not.
- Example:
  ```java
  try {
      // code that may throw an exception
  } catch (Exception e) {
      // handling exception
  } finally {
      // code that will always execute
  }
  ```

#### 8. Is the following code legal? Why?

```java
try {
    // some code
} finally {
    // some code
}
```

Yes, the code is legal. A `try` block can be followed by a `finally` block without a `catch` block. The `finally` block will always execute regardless of whether an exception is thrown or not.

#### 9. Is there anything wrong with the following exception handler as written? Will this code compile?

```java
try {
    // some code
} catch (Exception e) {
    // error handling 1
} catch (ArithmeticException ae) {
    // error handling 2
}
```

Yes, there is an issue with this code. The second catch block is unreachable because `ArithmeticException` is a subclass of `Exception`, and the first catch block will catch all exceptions of type `Exception` and its subclasses. This code will not compile.

#### 10. Match each situation in the first list with an item in the second list.

| Scenarios                                                           | Exceptions/Errors       |
|---------------------------------------------------------------------|-------------------------|
| 1. `int num = "four";`                                              | iii. Compile error      |
| 2. `public void recursion() { recursion(); }`                       | i. Error                |
| 3. A program is reading a stream and reaches the end of the stream marker. | iv. No exception        |
| 4. You write code to read from a file but misspelled the filename.  | ii. Checked exception   |



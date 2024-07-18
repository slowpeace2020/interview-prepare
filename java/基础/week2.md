# Thread

## 1. What is Thread and What is Process? What are differences between them? 
### What is a Thread?

A thread is a lightweight sub-process, the smallest unit of processing that can be performed in an operating system. A thread is a single sequence stream within a process. Because threads have some of the properties of processes, they are sometimes called lightweight processes. A thread has its own program counter (PC), a register set, and a stack space.

### What is a Process?

A process is a self-contained execution environment that has its own private set of basic run-time resources, particularly its own memory space. It is an instance of a program that is being executed. Multiple processes can run concurrently on an operating system, each one having its own memory space and resources.

### Differences between Thread and Process:

1. **Memory Space**:
    - **Process**: Has its own memory space. One process cannot directly access the memory of another process.
    - **Thread**: Shares the memory space of the process to which it belongs. All threads within a process can directly access each other's data.

2. **Resource Allocation**:
    - **Process**: Is allocated separate resources by the operating system. Each process operates independently and has its own CPU, memory, and I/O resources.
    - **Thread**: Shares resources allocated to the process. Threads share the CPU, memory, and I/O resources of their parent process.

3. **Inter-Process Communication (IPC)**:
    - **Process**: Requires more complex communication mechanisms (like pipes, sockets, shared memory, or message queues) to interact with other processes.
    - **Thread**: Can communicate directly with other threads of the same process without needing IPC mechanisms.

4. **Overhead**:
    - **Process**: Creating and managing processes is more resource-intensive due to the overhead of allocating and managing separate memory space and resources.
    - **Thread**: Creating and managing threads is less resource-intensive as threads share the resources of the parent process.

5. **Isolation**:
    - **Process**: Processes are isolated from each other, providing better security and stability. A failure in one process does not affect others.
    - **Thread**: Threads within the same process are not isolated. A failure in one thread can potentially corrupt the memory space of the process and affect other threads.

6. **Context Switching**:
    - **Process**: Context switching between processes is more expensive as it involves switching the memory space and resources.
    - **Thread**: Context switching between threads is less expensive as it only involves switching the CPU context (like program counter, registers) without changing the memory space.

### Example Scenarios:

- **Processes**: Used for running independent applications. For example, a web browser and a word processor running simultaneously are separate processes.
- **Threads**: Used for performing multiple tasks within the same application. For example, a web browser may have multiple threads for rendering pages, downloading files, and running JavaScript.

### Summary:

Threads are lightweight and share the same memory space within a process, making them suitable for tasks that require shared resources and quick context switches. Processes are more isolated, providing a higher level of security and stability, suitable for running independent applications.

## 2. How to create threads in Java?
### How to Create Threads in Java

In Java, you can create threads in two primary ways: by extending the `Thread` class or by implementing the `Runnable` interface. Below are detailed explanations and examples of both methods.

### 1. Extending the `Thread` Class

When you extend the `Thread` class, you create a new class that inherits from `Thread` and override its `run` method. Here’s an example:

#### Example:

```java
// Creating a new thread by extending the Thread class
class MyThread extends Thread {
    // Override the run method to define the thread's task
    public void run() {
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        // Create an instance of the thread
        MyThread thread = new MyThread();
        // Start the thread
        thread.start();
    }
}
```

### 2. Implementing the `Runnable` Interface

When you implement the `Runnable` interface, you define the `run` method and then create a `Thread` object, passing your `Runnable` instance to its constructor. This method is preferred as it allows your class to extend another class if needed.

#### Example:

```java
// Creating a new thread by implementing the Runnable interface
class MyRunnable implements Runnable {
    // Override the run method to define the thread's task
    public void run() {
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        // Create an instance of the Runnable
        MyRunnable myRunnable = new MyRunnable();
        // Create a thread with the Runnable instance
        Thread thread = new Thread(myRunnable);
        // Start the thread
        thread.start();
    }
}
```

### Differences between Extending `Thread` and Implementing `Runnable`

- **Single Inheritance Limitation**: Java does not support multiple inheritance for classes. If you extend the `Thread` class, you cannot extend any other class, which limits your design options. Implementing `Runnable` allows you to extend another class if needed.
- **Flexibility**: Implementing `Runnable` is more flexible because it decouples the task to be performed from the thread management, allowing better separation of concerns.

### Using Lambda Expressions (Java 8 and above)

Since Java 8, you can also use lambda expressions to create threads more concisely if your task is simple and fits in a single method.

#### Example:

```java
public class LambdaThread {
    public static void main(String[] args) {
        // Creating a thread using a lambda expression
        Thread thread = new Thread(() -> {
            System.out.println("Thread is running");
        });
        // Start the thread
        thread.start();
    }
}
```

### Summary

- **Extending `Thread`**: Suitable for simple cases but limits inheritance.
- **Implementing `Runnable`**: More flexible and preferred in most cases.
- **Using Lambda Expressions**: Provides a concise way to create threads for simple tasks in Java 8 and above.

By using these methods, you can create and manage threads in Java to perform concurrent tasks efficiently.


## 3. Runnable or Thread, which do you prefer to using? Why?
## Runnable or Thread: Which Do You Prefer to Use and Why?

### Preferred Approach: Implementing `Runnable`

### Reasons for Preferring `Runnable`:

1. **Flexibility**:
    - **Class Inheritance**: Java does not support multiple inheritance for classes. By implementing the `Runnable` interface, you retain the ability to extend another class. This allows for greater flexibility in your class design. For instance, if your class already extends another class, you cannot extend `Thread`, but you can still implement `Runnable`.

      ```java
      class MyClass extends SomeOtherClass implements Runnable {
          @Override
          public void run() {
              System.out.println("Thread is running");
          }
      }
      ```

2. **Separation of Concerns**:
    - **Task and Thread Management**: Implementing `Runnable` separates the task to be performed from the thread management. This separation enhances the clarity and maintainability of your code. The `Runnable` implementation focuses solely on the task, while the `Thread` class handles the execution environment.

      ```java
      class MyRunnable implements Runnable {
          @Override
          public void run() {
              System.out.println("Thread is running");
          }
      }
 
      public class Main {
          public static void main(String[] args) {
              MyRunnable myRunnable = new MyRunnable();
              Thread thread = new Thread(myRunnable);
              thread.start();
          }
      }
      ```

3. **Code Reusability**:
    - **Reusable Task**: By implementing `Runnable`, you can easily reuse the task definition across multiple threads. This is especially useful when you need to run the same task concurrently in different threads.

      ```java
      class MyRunnable implements Runnable {
          @Override
          public void run() {
              System.out.println("Thread is running");
          }
      }
 
      public class Main {
          public static void main(String[] args) {
              MyRunnable task = new MyRunnable();
              Thread thread1 = new Thread(task);
              Thread thread2 = new Thread(task);
              thread1.start();
              thread2.start();
          }
      }
      ```

4. **Thread Pool Compatibility**:
    - **Executor Framework**: The `Runnable` interface is compatible with Java's Executor framework, which provides a higher-level replacement for working directly with threads. The Executor framework simplifies concurrent programming by providing thread pools and task scheduling.

      ```java
      import java.util.concurrent.ExecutorService;
      import java.util.concurrent.Executors;
 
      class MyRunnable implements Runnable {
          @Override
          public void run() {
              System.out.println("Thread is running");
          }
      }
 
      public class Main {
          public static void main(String[] args) {
              ExecutorService executor = Executors.newFixedThreadPool(2);
              MyRunnable task = new MyRunnable();
              executor.submit(task);
              executor.submit(task);
              executor.shutdown();
          }
      }
      ```

### When to Use `Thread`

While implementing `Runnable` is generally preferred, there are scenarios where extending the `Thread` class might be more straightforward or appropriate:

1. **Simple, Standalone Threads**:
    - For quick and simple threads where no other class inheritance is needed, extending `Thread` might be more straightforward.

      ```java
      class MyThread extends Thread {
          @Override
          public void run() {
              System.out.println("Thread is running");
          }
 
          public static void main(String[] args) {
              MyThread thread = new MyThread();
              thread.start();
          }
      }
      ```

2. **When Customizing Thread Behavior**:
    - If you need to customize the thread itself, such as overriding other `Thread` methods or adding additional state or behavior to the thread, extending `Thread` could be beneficial.

### Summary

In most cases, implementing `Runnable` is the preferred approach due to its flexibility, separation of concerns, reusability, and compatibility with the Executor framework. Extending the `Thread` class can still be useful in simple or specific scenarios where you need to customize the thread's behavior.

## 4. What are differences between start() and run()?
## Differences between `start()` and `run()` in Java

### `start()` Method

#### Purpose
- **Creates a New Thread**: The `start()` method is used to create a new thread and make it runnable. This method is called to begin the execution of a thread.
- **Calls `run()` Method**: Internally, the `start()` method invokes the `run()` method of the `Thread` class or the `Runnable` interface.

#### Behavior
- **New Thread**: Calling `start()` results in a new thread being created and the `run()` method being executed in that new thread.
- **Thread Lifecycle**: This method changes the state of the thread from "new" to "runnable". The actual execution of the thread's `run()` method is managed by the thread scheduler.

#### Example

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start(); // This creates a new thread and runs the run() method in that new thread
    }
}
```

### `run()` Method

#### Purpose
- **Contains Task Code**: The `run()` method contains the code that constitutes the new thread's task. This method is meant to be overridden in a subclass or provided via a `Runnable` implementation.
- **Direct Invocation**: If you directly call the `run()` method, it does not create a new thread. Instead, it executes the code within the current thread, just like a normal method call.

#### Behavior
- **Same Thread Execution**: Calling `run()` directly does not create a new thread. It simply runs the `run()` method's code in the current thread.
- **No New Thread Lifecycle**: This method does not change the thread's state or invoke the thread scheduler.

#### Example

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.run(); // This runs the run() method in the main thread, no new thread is created
    }
}
```

### Key Differences

| Aspect                 | `start()`                                             | `run()`                                            |
|------------------------|-------------------------------------------------------|---------------------------------------------------|
| **Thread Creation**    | Creates a new thread                                  | Does not create a new thread                      |
| **Execution**          | Invokes the `run()` method in a new thread            | Executes the `run()` method in the current thread |
| **Thread Lifecycle**   | Changes thread state to "runnable"                    | No impact on thread state                         |
| **Invocation**         | Calls the `run()` method indirectly via thread scheduler | Directly calls the `run()` method                  |
| **Usage**              | Used to start a new thread                            | Used to define the thread's task code              |

### Practical Example Demonstrating Differences

```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Thread is running: " + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        
        // Using start() - new thread is created
        Thread thread1 = new Thread(myRunnable);
        thread1.start(); // Output will show a different thread name
        
        // Using run() - no new thread is created
        myRunnable.run(); // Output will show the main thread name
    }
}
```

### Summary

- **`start()` Method**: Used to initiate a new thread, invoking the `run()` method in a separate thread of execution. It is essential for starting concurrent execution.
- **`run()` Method**: Contains the code to be executed by the thread. Directly calling this method does not create a new thread; it simply executes the method's code in the calling thread.

## 5. What is Thread Life Cycle in Java? Explain how to get to each stage.
### Thread Life Cycle Stages
- **New**: Created but not yet started.
  ```java
  Thread thread = new Thread();
  ```
- **Runnable**: Ready to run but waiting for CPU time.
  ```java
  thread.start();
  ```
- **Running**: Actively executing.
- **Blocked**: Waiting for a monitor lock.
  ```java
  synchronized(object) {
      // thread becomes blocked
  }
  ```
- **Waiting**: Waiting indefinitely for another thread to perform a particular action.
  ```java
  thread.wait();
  ```
- **Timed Waiting**: Waiting for another thread to perform an action up to a specified waiting time.
  ```java
  thread.sleep(time);
  ```
- **Terminated**: Has completed execution.

## 6. Explain join() method in Java Thread.
- **join() Method**:
    - Allows one thread to wait for the completion of another.
    - Ensures that a thread has finished before the next thread starts.
  ```java
  Thread thread = new Thread(new MyRunnable());
  thread.start();
  thread.join();
  ```

## 7. What are differences sleep() and wait()?
- **sleep() Method**:
    - From `Thread` class.
    - Temporarily halts the execution of the current thread for a specified period.
    - Does not release locks.
  ```java
  Thread.sleep(time);
  ```
- **wait() Method**:
    - From `Object` class.
    - Causes the current thread to wait until another thread invokes `notify()` or `notifyAll()`.
    - Releases the lock on the object.
  ```java
  synchronized(object) {
      object.wait();
  }
  ```

## 8. What is daemon thread in Java? Why do we need it?
- **Daemon Thread**:
    - A low-priority thread that runs in the background to perform tasks such as garbage collection.
    - Terminates when all user threads terminate.
  ```java
  Thread daemonThread = new Thread(new MyRunnable());
  daemonThread.setDaemon(true);
  daemonThread.start();
  ```

## 9. What is thread interference? Give an example.
- **Thread Interference**:
    - Occurs when two or more threads access shared data and try to change it simultaneously.
    - Leads to inconsistent data.
  ```java
  class Counter {
      private int count = 0;
      public void increment() {
          count++;
      }
      public int getCount() {
          return count;
      }
  }
  ```

## 10. What are 2 ways to perform synchronization in Java?
- **Synchronized Method**:
  ```java
  synchronized void synchronizedMethod() {
      // method body
  }
  ```
- **Synchronized Block**:
  ```java
  void synchronizedBlock() {
      synchronized(this) {
          // block body
      }
  }
  ```

## 11. What is deadlock? Why it would happen, and how to resolve it?
- **Deadlock**:
    - A situation where two or more threads are blocked forever, waiting for each other.
- **Cause**: Circular wait, mutual exclusion, hold and wait, no preemption.
- **Resolution**:
    - Avoid nested locks.
    - Use a timeout for locks.
    - Deadlock detection algorithms.

## 12. What are the differences among Hashtable, SynchronizedMap and ConcurrentHashMap?
- **Hashtable**:
    - Synchronized, thread-safe, but performance overhead.
- **SynchronizedMap**:
    - Wrapper class, provides synchronized access.
- **ConcurrentHashMap**:
    - Higher concurrency, better performance, and thread safety.

## 13. What is a Singleton class? How can it be initialized? How to ensure its thread-safety?
- **Singleton Class**:
    - Ensures only one instance exists.
- **Initialization**:
    - Eager initialization.
    - Lazy initialization.
  ```java
  class Singleton {
      private static Singleton instance;
      private Singleton() {}
      public static synchronized Singleton getInstance() {
          if (instance == null) {
              instance = new Singleton();
          }
          return instance;
      }
  }
  ```
- **Thread-Safety**:
    - Use synchronized keyword.
    - Double-checked locking.
    - Bill Pugh Singleton Design.

# MySQL Intro

## 1. What is data modeling? Why do you need it? When do you need it?
- **Data Modeling**:
    - Process of creating a visual representation of a whole information system or parts of it to communicate connections between data points and structures.
- **Need**:
    - For designing databases efficiently.
- **When**:
    - During initial stages of a project.

## 2. Explain these concepts: entity, attribute, tuple(record, row), domain
- **Entity**: An object or thing of interest.
- **Attribute**: A property or characteristic of an entity.
- **Tuple (Record, Row)**: A single entry in a table representing a set of related data.
- **Domain**: Set of permissible values for a given attribute.

## 3. What is a superkey? Give an example of a superkey that is not a candidate key and explain why.
- **Superkey**: A set of one or more columns that can uniquely identify a row.
- **Example**: `{StudentID, Email}` is a superkey; `{StudentID}` is a candidate key because it's minimal.

## 4. What is the primary key? How is it different from a unique key (composite key)?
- **Primary Key**: Unique identifier for table rows, cannot be null.
- **Unique Key**: Ensures uniqueness but can have one null value.
- **Composite Key**: Primary key consisting of more than one column.

## 5. What are the relationships in data modeling? (categorized by cardinality, e.g., one-to-one)
- **One-to-One**: Each entity in one table is linked to one entity in another table.
- **One-to-Many**: One entity in one table is linked to multiple entities in another table.
- **Many-to-Many**: Entities in one table are linked to multiple entities in another table via a junction table.

## 6. What are composite attributes, multi-valued attributes, and derived attributes? Give at least one example for each type of attribute.
- **Composite Attributes**: Attributes that can be divided into smaller sub-parts. Example: Address (Street, City, State).
- **Multi-Valued Attributes**: Attributes that can have multiple values. Example: PhoneNumbers.
- **Derived Attributes**: Attributes whose values are calculated from other attributes. Example: Age (calculated from Date of Birth).

## 7. How do you represent a many-to-many relationship in a database? Please describe in detail using an example.


- **Many-to-Many Relationship**:
    - Use a junction table to represent the relationship.
    - Example: Students and Courses.
        - `Students`: StudentID, StudentName
        - `Courses`: CourseID, CourseName
        - `Enrollments`: StudentID, CourseID

## 8. What is normalization? What are the pros and cons?
- **Normalization**:
    - Process of organizing data to minimize redundancy.
- **Pros**:
    - Reduces redundancy, improves data integrity.
- **Cons**:
    - Complexity, can impact performance.

## 9. What are the different types of dependencies? How are they different? Give an example of each type.
- **Full Dependency**: Attribute depends on the entire primary key.
- **Partial Dependency**: Attribute depends on part of the primary key.
- **Transitive Dependency**: Attribute depends on another non-key attribute.

## 10. What are normal forms (NFs)? Explain each NF briefly. You can use an example to demonstrate.
- **First Normal Form (1NF)**: Each table cell contains a single value.
- **Second Normal Form (2NF)**: 1NF + all non-key attributes fully functionally dependent on the primary key.
- **Third Normal Form (3NF)**: 2NF + all non-key attributes non-transitively dependent on the primary key.

## 11. What is database integrity? Why do you need it? Provide an example of user-defined integrity.
- **Database Integrity**:
    - Ensures data accuracy and consistency.
- **Need**:
    - For reliable data management.
- **Example**:
    - Constraint ensuring employee salary does not exceed the maximum salary for their job title.

## 12. What is a relational database? What is a non-relational database? Difference? When would you choose a relational or non-relational database?
- **Relational Database**:
    - Stores data in tables with rows and columns.
    - Uses SQL.
- **Non-Relational Database**:
    - Stores data in various formats (key-value, document, graph).
    - Flexible schema.
- **Difference**:
    - Structured vs. flexible schema.
- **Choice**:
    - Use relational for structured data, complex queries.
    - Use non-relational for flexibility, scalability.

## 13. How do you insert values into a table if you don’t know the order of the columns?
- Specify the column names in the INSERT statement.
  ```sql
  INSERT INTO TableName (Col2, Col1) VALUES ('Value2', 'Value1');
  ```

## 14. How is TRUNCATE different from DELETE?
- **TRUNCATE**:
    - Removes all rows, cannot be rolled back, faster.
- **DELETE**:
    - Removes specific rows, can be rolled back.

## 15. Is syntax in SQL case-sensitive?
- SQL keywords are case-insensitive, but identifiers can be case-sensitive depending on the database system settings.

## 16. What are constraints? Why do we need them? Are they mandatory to have?
- **Constraints**:
    - Rules enforced on data columns.
- **Need**:
    - Ensure data integrity and reliability.
- **Mandatory**:
    - Not mandatory but highly recommended.

# MySQL Part 2

## 1. What is the difference between WHERE vs HAVING?
- **WHERE**:
    - Filters records before any groupings are made.
- **HAVING**:
    - Filters records after groupings are made.

## 2. How do you make sure there are no duplicate values in a column?
- Use the `UNIQUE` constraint on the column.

## 3. What is an aggregate function? What are some examples of aggregate functions?
- **Aggregate Function**:
    - Performs a calculation on a set of values.
- **Examples**:
    - COUNT(), AVG(), SUM(), MAX(), MIN().

## 4. What are joins? What are the types of joins?
- **Joins**:
    - Combine rows from two or more tables based on a related column.
- **Types**:
    - Inner Join, Left Join, Right Join, Full Outer Join, Cross Join, Self Join.

## 5. What is a view and what is it used for?
- **View**:
    - A virtual table created by a query.
- **Use**:
    - Simplifies complex queries, enhances security by restricting data access.

## 6. What are some differences between relational (SQL) and non-relational (NoSQL) databases?
- **Relational (SQL)**:
    - Structured schema, table-based.
- **Non-Relational (NoSQL)**:
    - Flexible schema, various data models (key-value, document, graph).

## 7. What are indexes? Why are they needed?
- **Indexes**:
    - Data structures that improve the speed of data retrieval.
- **Need**:
    - Enhance query performance.

## 8. Is index always useful? Is it a good idea to always create as many indexes as possible?
- **Not Always Useful**:
    - Indexes can slow down write operations.
- **Not Always Good**:
    - Too many indexes can increase maintenance overhead.

## 9. How is data stored if there is no clustered index? How about when you create a clustered index?
- **Without Clustered Index**:
    - Data is stored in a heap, unordered.
- **With Clustered Index**:
    - Data is stored in a B-tree structure, ordered by the clustered index key.

## 10. What are the differences between clustered and non-clustered indexes?
- **Clustered Index**:
    - Determines physical order of data.
    - Only one per table.
- **Non-Clustered Index**:
    - Separate from data storage.
    - Multiple allowed per table.

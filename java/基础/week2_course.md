
# Day 2 Java Backend Development Program - Thread

## Outline
- Thread & Process
- Thread in Java
- Thread
- Runnable
- Daemon Threads
- Synchronization
- Thread Problems

## Recap: JVM, JRE, JDK
- **Java Virtual Machine (JVM)**: Abstract computing machine that runs bytecode files, platform-independent.
- **Java Runtime Environment (JRE)**: Implementation of JVM, includes libraries and other files.
- **Java Development Kit (JDK)**: Toolkit for Java developers, includes JRE and development tools like compiler, debugger, etc.

## Process
- A process has a self-contained execution environment with a complete, private set of basic run-time resources, including its own memory space.
- Processes are often seen as synonymous with programs or applications but may involve a set of cooperating processes.
- Operating systems support Inter Process Communication (IPC) resources like pipes and sockets.

## Thread
- A thread is a lightweight sub-process, the smallest unit of processing, with a separate path of execution.
- Threads are independent; an exception in one thread does not affect other threads.

## Thread vs Process

## Why Thread?
- Enhance parallel processing
- Reduce response time for users
- Utilize idling CPUs
- Prioritize work depending on priority
- Examples: Server monitoring infrastructure, HTTP/J2EE servers, computer games

## Thread in Java
- Two approaches to create a new thread:
  - Extend the `Thread` class
  - Implement the `Runnable` interface

### Thread Class
- In Java, `Thread` class belongs to `java.lang` package:
  ```java
  public class Thread extends Object implements Runnable {
      public void run() {
          // statement for implementing thread
      }
  }
  ```

### Runnable Interface
- `Runnable` interface is implemented by the `Thread` class in `java.lang` package:
  ```java
  public interface Runnable {
      void run();
  }
  ```
- Runnable is a functional interface; it can be implemented using lambda expressions from Java 8 onwards.

## Runnable vs Thread
- Prefer `Runnable` over `Thread` for better flexibility and to avoid single inheritance constraints.

## Thread Life Cycle
- Managed by Thread Scheduler of JVM:
    - **new**: Just created but not started
    - **runnable**: Created, started, and able to run
    - **running**: Currently running
    - **blocked**: Waiting for some event to occur
    - **dead**: Finished or stopped

## Start vs Run
- `start()` method creates a new thread and executes `run()` method.
- Directly calling `run()` method executes it as a normal method call on the current thread.

## Main Thread
- Every Java program has a default main thread that starts when the JVM calls the program's `main()` method.

## Blocked State
- **sleep()**: Suspends execution for a specified period.
- **join()**: Allows one thread to wait for the completion of another.
- **wait()**: Forces the current thread to wait until another thread invokes `notify()` or `notifyAll()`.

## Notify and NotifyAll
- **notify()**: Wakes up one waiting thread.
- **notifyAll()**: Wakes up all waiting threads.

## Daemon Thread
- Provides services to user threads for background supporting tasks.
- Low priority thread that terminates when user threads die.

## Thread Safety
- Threads communicate by sharing access to the same resources, leading to thread interference and memory consistency errors.

## Thread Interference
- Occurs when two threads act on the same data, causing race conditions.

## Memory Consistency Errors
- Occur when different threads have inconsistent views of the same data.

## Volatile
- A keyword in Java ensuring visibility of variable changes to all threads.

## Synchronization
- Prevents thread interference and memory consistency errors using the `synchronized` keyword in methods or blocks.

## Synchronized Keyword
- Blocks other threads from accessing synchronized methods or blocks until the current thread completes execution.

## HashMap Safety
- HashMap is not thread-safe. Use `SynchronizedMap` or `ConcurrentHashMap`.

## Java.util.concurrent
- Provides thread-safe classes like `ConcurrentHashMap`, `ReentrantLock`, and `AtomicInteger`.

## Deadlock
- A situation where two or more threads are blocked forever, waiting for each other to release the lock.

## Singleton Design Pattern
- Ensures one instance of a singleton class exists at a time.
    - **Eager initialization**: Instance initialized at creation.
    - **Lazy initialization**: Instance initialized when `getInstance()` method is called.


# Java Backend Development Program - Day 3: Data Modeling and MySQL Introduction

## Outline
- Data Modeling
- ER Diagram
- Normalization
- Database
- SQL
- Constraints
- Basic SQL Query

## 01. Data Modeling

### Data Modeling
- Data Modeling is the process of analyzing business requirements, designing, and creating a physical instance (a plan or a blueprint) for the database or data warehouse.
- It is a stage in SDLC (Software Development Life Cycle).
- It also impacts the user interface.

### Relational Database Terminology
- **Entity**: An item that can exist independently or uniquely identified.
- **Attribute**: Column label (name).
- **Domain**: Set of valid values for an attribute.
- **Relationship**: How entities relate.
- **Degree**: Number of entities in a relationship.
- **Cardinality**: Measure of participation.

### Keys Used in Data Modeling
- **Surrogate Key**: An artificial key assigned to a record.
- **Natural Key**: A key that is formed of attributes that already exist in the real world.

### Relationship, Cardinality, and Degree
- **Cardinality**:
  - **One-to-One**: One element in Entity A may link to one element in Entity B and vice versa.
  - **One-to-Many**: One element in Entity A may link to many elements in Entity B but one element in Entity B may only link to one element in Entity A.
  - **Many-to-One**: Reverse A and B in one-to-many.
  - **Many-to-Many**: One element in Entity A may link to any number of elements in Entity B and vice versa.
- **Degree**:
  - Number of entities involved in the relationship, usually 2 (binary relationship), but unary and higher degree relationships can exist.

## 02. ERD (Entity Relationship Diagram)

### ERD
- Used to create or design a blueprint of the database or data warehouse.
- Design entities, attributes, and show relationships.

### Crow’s Foot Notation
- Connection symbols display relationships.
- Entity and attributes in a table-like format.

### Convert ERD to Tables
- Each entity type becomes a table.
- Each single-valued attribute becomes a column.
- Derived attributes are ignored/computed column.
- Multi-valued attributes are represented by a separate table.
- Use conjunction table to break up the many-to-many relationship.
- The key attribute of the entity type becomes the primary key or unique key of the table.

## 03. Normalization

### Normalization
- **Redundancy**: Values repeated unnecessarily in multiple records or fields within one or more tables.

### Anomalies
- **Update Anomaly**: The same information can be expressed on multiple rows; therefore, updates to the relation may result in logical inconsistencies.
- **Insertion Anomaly**: Certain facts cannot be recorded at all.
- **Deletion Anomaly**: Deletion of data representing certain facts necessitates deletion of data representing completely different facts.

### Goals of Normalization
- Reduce redundancy, avoid anomaly, and create a well-structured series of tables without error or inconsistencies.
- Minimize redesign when extending the database structure.
- Ensure data dependencies are properly enforced by data integrity constraints (Entity Integrity, Referential Integrity, Domain Integrity).

### Types of Dependencies
- **Full Dependencies**: Depend on all prime attributes fully.
- **Partial Dependencies**: Depend on some prime attributes.
- **Transitive Dependencies**: Depend on an attribute that depends on a prime attribute.

### Normal Forms
- **First Normal Form (1NF)**: Each table cell should contain a single value, and each record needs to be unique.
- **Second Normal Form (2NF)**: Meets all of 1NF and ensures all non-prime attributes are fully dependent on a prime attribute.
- **Third Normal Form (3NF)**: Meets 1NF and 2NF, and ensures every non-prime attribute is non-transitively dependent on the prime attributes.

### When to Normalize
- Usually normalize in databases with lots of writes.
- Data integrity is crucial, especially when there is a lot of user input.

### Cons of Normalization
- Normalization is an expensive process.
- Designing can be difficult – good for final design, not testing.
- More tables lead to more time; joins are costly and can cause slowing.

## 04. Database Management System

### DBMS or RDBMS

### Relational vs. Non-Relational Database
- **Relational Database (SQL)**:
  - Traditional way of storing data.
  - Data is stored in tables with rows and columns.
  - Rigid schema with well-defined relationships.
  - Difficult to scale.
  - Examples: MySQL, MS SQL Server, Oracle Database.
- **Non-Relational Database (NoSQL)**:
  - Developed more recently.
  - Data can be stored in a variety of formats (JSON documents, key-value pairs, wide-column, graphs).
  - Flexible schema with loose relationships.
  - Easily scalable.
  - Examples: MongoDB, Redis.

## 05. SQL

### SQL (Structured Query Language)
- **DDL (Data Definition Language)**
- **DML (Data Manipulation Language)**
- **DCL (Data Control Language)**
- **DQL (Data Query Language)**

### DDL – Data Definition Language
- **Create**
- **Alter**
- **Drop**
- **Truncate**

#### CREATE Statement
- Used to create database objects like databases, tables, and indexes.

#### ALTER TABLE Statement
- Modifies an existing database table to add, drop, or change columns and their data types.

#### DROP TABLE Statement
- Deletes objects such as a table, index, or view. Cannot be rolled back.

#### TRUNCATE TABLE Statement
- Quickly removes all records from a table while preserving its structure for reuse.

### DML – Data Manipulation Language
- **Insert**
- **Update**
- **Delete**

#### Insert Statements
- Used to input data into tables.

#### Update Statements
- Used to change or modify data inside a table.

#### Delete Statements
- Used to remove specific rows inside a table.

### DCL – Data Control Language
- Controls what permission a user can have.

#### Grant
- Used to "grant" or give permissions to a user or role.

#### Revoke
- Used to take away previously granted permissions.

### DQL – Data Query Language
- **Select From Where**:
  - SELECT: Pick up which column of data to fetch.
  - FROM: Select which table or dataset to fetch.
  - WHERE: Specify criteria to sort data using operators (Filter).
  - Operators: IN, OR, AND

- **Group by, Having, Order by**:
  - GROUP BY: Combine similar values in a column.
  - HAVING: Filter conditions for aggregate only.
  - ORDER BY: Display the data ordered by a specific column.

## 06. Constraints

### Constraints
- Associated with a table, created with a CREATE CONSTRAINT SQL statement.

### Key Constraints
- **Primary Key**: 1 per table, unique clustered index, not null.
- **Unique Key**: 999 per table, unique non-clustered index, 1 null allowed.
- **Foreign Key**: Cannot exist before PK, must be deleted before PK.

### Other Constraints
- **Null, Not Null**: Specify if nulls are allowed.
- **Check**: Data must meet rule.
- **Default**: Specify default value if none provided.
- **Data Types**: Define the type of data allowed in a column (e.g., Char(2), Varchar(10), Money).

### Adding Constraints
- Constraints specify rules for the data in a table.
- Can be added:
  - After table creation using ALTER.
  - While specifying the column in the CREATE statement.
  - After specifying the table in the CREATE statement.

# Day 4 MySQL Queries

## Recap: Concepts

### Data Modeling for MySQL
- Planning the structure of data to be optimally stored and accessed in a MySQL database.
- Involves normalization, building ERDs, creating tables, defining relationships, setting primary and foreign keys, and planning indexes for efficient data retrieval.

### SQL
- SQL (Structured Query Language) is a programming language for storing and processing information in a relational database.
- Has an international standard (ISO/IEC 9075).

### RDBMS
- RDBMS is software used to create, update, and manage relational databases.
- Variants include T-SQL (used by SQL Server), PostgreSQL, MySQL.
- Client-Server Architecture: MySQL Server (Server), MySQL Workbench (GUI client).

### SQL Statements
- **Data Definition Language (DDL)**: CREATE, DROP, ALTER, TRUNCATE.
- **Data Manipulation Language (DML)**: INSERT, UPDATE, DELETE.
- **Data Control Language (DCL)**: GRANT, REVOKE.
- **Data Query Language (DQL)**: SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY.

## Outline
- Data Query Language (DQL)
- Joins
- Set Operations
- Subquery and View
- Aggregate Functions
- Stored Procedures & Triggers
- Index

## Basic SQL Query (DQL)

### Query Execution
1. **Parser**: Checks query syntax and breaks the query into tokens (intermediate files).
2. **Query Optimizer**: Creates the best possible execution plan based on current resource utilization.
3. **DB Engine**: Runs the query.

### SELECT/FROM/WHERE Clause
- **SELECT**: Used to select data from a database.
- **WHERE**: Used to filter records.
- **AS**: Renames a column or table with an alias, optional and exists only for the duration of the query.
- **Quotation Marks**:
    - Single quote (`'`): Enclose string literals.
    - Double quotes (`"`): Used in PostgreSQL, Oracle, and other SQL standard-compliant databases to enclose identifiers.
    - Backticks (`` ` ``): Used in MySQL to enclose identifiers.

## Joins

### Types of Joins
- **Cross Join**: Displays every possible combination of all values in the designated columns.
- **Inner Join**: Connects on only matching data and displays values that match on both sides of the table.
- **Left Join (Left Outer Join)**: Displays matching data from inner join and all unique values from the left table.
- **Right Join (Right Outer Join)**: Displays matching data from inner join and all unique values from the right table.
- **Full Outer Join**: Displays all values, matching and non-matching (not supported in MySQL, can be achieved using left join and right join).
- **Self Join**: Joins a table to itself.

### Syntax Examples
- **CROSS JOIN**: `T1 CROSS JOIN T2`
- **INNER JOIN**: `T1 INNER JOIN T2 ON <condition>`
- **LEFT JOIN**: `T1 LEFT JOIN T2 ON <condition>`
- **RIGHT JOIN**: `T1 RIGHT JOIN T2 ON <condition>`

## Set Operations

### Types of Set Operations
- **Union**: Eliminates duplicates.
- **Union All**: Does not eliminate duplicates.
- **Intersection**: Not directly supported in MySQL, can be implemented using INNER JOIN.
- **Difference**: Implemented via EXCEPT, displaying values in the first select statement minus any values found in the second select statement.

## Subquery and View

### Subquery
- A query nested inside another query.
- Must be written in parentheses and include the SELECT and FROM clauses.
- **Example with IN (non-correlated subquery)**: `Who likes ‘Code Brew’ and ‘Espresso’?`
- **Example with EXISTS (correlated subquery)**: `Who goes to a cafe that serves ‘Code Brew’?`

### View
- A virtual table created by a query, providing a dynamic window into the stored data without storing the data itself.
- Good for security as it can prevent showing extra data.
- DML operations on a view modify the source data.

## Aggregate Functions

### Common Aggregate Functions
- **COUNT()**: Returns the number of rows that matches a specified criterion.
- **AVG()**: Returns the average value of a numeric column.
- **SUM()**: Returns the total sum.
- **MAX()**: Returns the largest value of the selected columns.
- **MIN()**: Returns the smallest value of the selected columns.

### Handling NULL Values
- **COUNT(*)** or **COUNT(1)**: Includes NULL values.
- **COUNT(column_name)**: Excludes NULL values.
- **AVG, MIN, MAX**: Ignore NULL values.
- **GROUP BY**: Includes a row for NULL values.

### Clauses
- **GROUP BY**: Forms groups of records, often used with aggregate functions.
- **HAVING**: Filters grouping records, similar to WHERE but can work with aggregate functions.
- **ORDER BY**: Sorts the result in ascending or descending order, used with LIMIT to restrict the number of records returned.

## Stored Procedure & Trigger

### Stored Procedure
- A set of SQL statements pre-compiled and stored in the database.
- **CREATE PROCEDURE**: Used to create a stored procedure.
- **CALL**: Executes the stored procedure.
- **Benefits**: Performance improvement, code reusability.

### Trigger
- A named database object associated with a table, activated by events (INSERT, UPDATE, DELETE).
- **Components**:
    - **Event**: The type of action that activates the trigger.
    - **Timing**: Specifies when the trigger will be executed (BEFORE or AFTER the event).
    - **Triggered Action**: The set of statements to execute when the trigger activates.
- **Uses**: Ensure data integrity, automatically format or transform data.

## Index

### Index
- Used to speed up the retrieval of data by providing quick access paths.
- Implemented using a B+ Tree or B-Tree.
- **Clustered Index**: Associated with the primary key, determines the physical order of data.
- **Non-clustered Index**: Associated with unique keys, stores data pointers to the data in the table.

### Query Optimization
- **Execution Plan**: Analyze the performance of your query.
- **Index**: Ensure tables are properly indexed.
- **Optimize JOINs**: Avoid using JOIN statements on large tables if possible.
- **Database Design**: Ensure data is normalized.

## Summary
- **Table**: The physical and fundamental storage structure in a database to store data permanently.
- **View**: A virtual table created by a query, providing a dynamic window into the stored data without storing the data itself.
- **Result Set**: The temporary collection of data returned from a query, used for display and manipulation during a database session.





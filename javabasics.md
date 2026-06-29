
# JAVA NAMING CONVENTIONS 

### 1. camelCase
*   **Explanation:** The first word starts with a lowercase letter, and every subsequent word starts with a capital letter. It is used for **variables, methods, and objects** in Java.
*   **Example:** 
    ```java
    int userAge = 25;
    public void displayDetails() {}
    ```

### 2. PascalCase (Upper Camel Case)
*   **Explanation:** Every word in the name starts with a capital letter. It is used for **classes, interfaces, enums, and records**.
*   **Example:** 
    ```java
    public class BankAccount {}
    public interface Payable {}
    ```

### 3. snake_case
*   **Explanation:** Words are written in all lowercase and separated by underscores. This style is **discouraged in Java** but widely used in languages like Python and SQL.
*   **Example:** 
    ```java
    // Discouraged in Java:
    int total_marks = 90; 
    ```

### 4. UPPER_SNAKE_CASE
*   **Explanation:** Words are written in all uppercase and separated by underscores. This is used strictly for **constants** (`static final` variables) whose values cannot change.
*   **Example:** 
    ```java
    public static final double PI_VALUE = 3.14159;
    ```

### 5. kebab-case
*   **Explanation:** Words are separated by hyphens. This is **illegal in Java** because the compiler reads the hyphen as a subtraction symbol. It is commonly used in URLs, HTML, and CSS.
*   **Example:** 
    ```java
    // Illegal in Java (Causes compilation error):
    int student-age = 20; 
    ```

### 6. lowercase
*   **Explanation:** Every letter is kept lowercase with no capitals, underscores, or hyphens. This is used strictly for **Java package names** to ensure compatibility across different operating systems.
*   **Example:** 
    ```java
    package com.oracle.database;
    ```

# JAVA BASICS 


### Topic 1: Class & Object (Blueprint vs. Instance)

*   **The Core Concept:** A **Class** is a logical template (no physical memory is allocated when declared). An **Object** is a physical entity created from that template (memory is allocated on the heap).
*   **Deep-Dive Mechanics:**
    *   When you write: `Student s;`
        *   No object is created. Only a **reference variable** (`s`) is created on the **Stack**. It currently points to `null`.
    *   When you write: `s = new Student();`
        *   The `new` keyword dynamically allocates memory on the **Heap** for the object.
        *   The constructor initializes the object.
        *   The address of this heap memory is assigned to the reference variable `s`.

#### Placement Interview Insight
*   **Question:** What is the difference between a reference variable and an object?
*   **Answer:** An object is the actual memory block on the heap containing the data. A reference variable is a stack variable that holds the memory address (pointer) of that object.

---

### Topic 2: Methods & Parameter Passing

*   **The Core Concept:** Methods define the behavior of an object.
*   **Deep-Dive Mechanics:**
    *   **Method Signature:** Consists of the **Method Name** + **Parameter List** (types and order).
    *   *Note:* Return type and access modifiers are **not** part of the method signature.
    *   **Java is strictly Pass-by-Value:** Java never passes references by reference; it passes the *value* of the reference (the memory address is copied).

```java
void change(int x) { x = 20; } // Original variable remains unchanged (primitive value copied)

void modify(Student s) { s.name = "Alex"; } // Original object changes because the copied address points to the same heap object.
```

---

### Topic 3: Constructors

*   **The Core Concept:** A special block of code used to initialize a newly created object.
*   **Deep-Dive Mechanics:**
    *   Constructors have **no return type** (not even `void`).
    *   **The Default Constructor:** If you write a class with *no* constructors, the compiler automatically inserts a default no-argument constructor. If you write *any* constructor yourself, the compiler does **not** provide the default one.
*   **Constructor Overloading:** Defining multiple constructors with different parameter lists.

#### Placement Interview Insight
*   **Question:** Can a constructor be private?
*   **Answer:** Yes. Private constructors prevent direct instantiation from outside the class. This is commonly used in Utility classes (e.g., `java.lang.Math`) or the **Singleton Design Pattern** (restricting class creation to a single instance).

---

### Topic 4: The `this` Keyword

*   **The Core Concept:** `this` is a reference variable that refers to the **current object** instance.
*   **Deep-Dive Mechanics:**
    1.  **Shadowing Resolution:** Resolves ambiguity when instance variables and parameter variables have the same name.
    2.  **Constructor Chaining:** Using `this()` to call one constructor from another within the same class. **Rule:** `this()` must be the very first statement in the constructor.

```java
public class Student {
    private String name;

    public Student() {
        this("Unknown"); // Calls the parameterized constructor below
    }

    public Student(String name) {
        this.name = name; // Resolves variable shadowing
    }
}
```

---

### Topic 5: Access Modifiers (Visibility Control)

Access modifiers define the scope of visibility for classes, constructors, variables, and methods.

| Modifier | Within Class | Within Package | Outside Package (Subclass only) | World |
| :--- | :--- | :--- | :--- | :--- |
| **`private`** | Yes | No | No | No |
| **`default` (no modifier)** | Yes | Yes | No | No |
| **`protected`** | Yes | Yes | Yes (via inheritance) | No |
| **`public`** | Yes | Yes | Yes | Yes |

---

### Topic 6: Static vs. Non-Static

This is one of the most frequently tested concepts in technical rounds.

| Feature | `static` (Class Level) | Non-static (Instance Level) |
| :--- | :--- | :--- |
| **Memory Allocation** | Once, when the class is loaded. | Every time a new object is created. |
| **Storage Location** | Metaspace (Method Area). | Heap Memory (inside the object). |
| **Access** | Via Class name (e.g., `Math.sqrt()`). | Via Object reference (e.g., `s.getName()`). |
| **Context** | Cannot access non-static members directly. | Can access both static and non-static members. |

#### Placement Interview Insight
*   **Question:** Why is the `main` method always `static` (`public static void main`)?
*   **Answer:** To allow the JVM to call the `main` method directly using the class name without needing to instantiate an object of that class first. This saves memory and bootstrap time.

---

### Topic 7: Memory Management (Stack vs. Heap)

Understanding how the JVM manages memory is essential for debugging and answering questions on memory leaks.

```
       STACK (LIFO)                        HEAP (Dynamic)
┌─────────────────────────┐         ┌─────────────────────────┐
│ [main() Frame]          │         │  Student Object         │
│ s ──────────────────────┼────────>│  - name: "John"         │
│ x: 10                   │         │  - age: 20              │
└─────────────────────────┘         └─────────────────────────┘
```

#### Stack Memory
*   **What it stores:** Local variables, reference variables, and active method call frames.
*   **Lifecycle:** Automatically allocated and deallocated when a method starts and ends (LIFO - Last In, First Out).
*   **Speed:** Fast access.

#### Heap Memory
*   **What it stores:** All objects, arrays, and instance variables.
*   **Lifecycle:** Managed dynamically. Objects remain until they are no longer reachable and are claimed by the **Garbage Collector (GC)**.
*   **Speed:** Slower access than Stack.

---

### Topic 8: The Four OOP Principles

#### 1. Encapsulation (Data Hiding)
*   **Mechanic:** Grouping variables and methods into a single unit (class) and restricting direct access using `private` fields. Access is granted only through public `getter` and `setter` methods.
*   **Why:** Provides control over data validation (e.g., rejecting negative values in `setAge()`).

#### 2. Inheritance (Code Reusability)
*   **Mechanic:** A child class (subclass) inherits state and behavior from a parent class (superclass) using the `extends` keyword.
*   **Key Rule:** Java does **not** support multiple inheritance with classes (e.g., `class C extends A, B` is illegal due to the **Diamond Problem**—ambiguity over which parent method to inherit).

#### 3. Polymorphism (Many Forms)
*   **Compile-time (Static) Polymorphism:** Achieved via **Method Overloading** (same method name, different parameter lists). Resolved at compile time.
*   **Runtime (Dynamic) Polymorphism:** Achieved via **Method Overriding** (child class redefines parent method with identical signature). Resolved at runtime using **Dynamic Method Dispatch**.

```java
Parent obj = new Child(); // Upcasting
obj.show(); // Calls Child's show() method at runtime
```

#### 4. Abstraction (Hiding Complexity)
*   **Mechanic:** Showing essential features and hiding background implementation details. Achieved via **Abstract Classes** (0 to 100% abstraction) and **Interfaces** (100% abstraction prior to Java 8).
*   **Interface vs. Abstract Class:**
    *   **Abstract Class:** Can have instance variables, constructors, and private/protected methods. Use when classes share a common identity ("is-a" relationship).
    *   **Interface:** Cannot have instance variables (variables are implicitly `public static final`) or constructors. Use when classes share a common behavior ("can-do" relationship).


 # CONSTRUCTORS

 ### Why Constructors Are Required

To understand why constructors are necessary, consider what happens when an object is created without one.

#### 1. Preventing Invalid States
If Java did not have constructors, objects would be created with default JVM values (numerical fields set to `0`, object references to `null`, booleans to `false`). 

For example, if you have a `User` class that requires an `email` and an `id` to function:
*   Without a constructor, a `new User()` would exist in memory with `id = 0` and `email = null`. 
*   If another part of your program tries to use this object immediately, it may throw a `NullPointerException` because the `email` field was never initialized.
*   **A constructor ensures that an object is fully initialized and in a valid state the moment it is created.**

#### 2. Enforcing Mandatory Data
Constructors allow you to demand specific information before an object can even be created. If a class requires certain parameters to exist, you can make those parameters mandatory inside the constructor.

#### 3. Convenience and Readability
Without constructors, you would have to write multiple lines of code using setter methods every time you want to create and set up an object:

```java
// Without a constructor (verbose and error-prone if you forget a setter):
Student s = new Student();
s.setId(101);
s.setName("Alice");
s.setCourse("CS");

// With a constructor (clean, single-line atomic operation):
Student s = new Student(101, "Alice", "CS");
```

---

### How Constructors Are Used (With Examples)

Here are three distinct ways constructors are utilized in Java development.

#### Example A: The Default (No-Argument) Constructor
If you do not write any constructor, Java inserts an invisible default constructor. However, you can write your own explicit no-argument constructor to assign custom default values.

```java
public class Smartphone {
    String operatingSystem;

    // Explicit No-Argument Constructor
    public Smartphone() {
        // Sets a default value so the OS is never 'null'
        this.operatingSystem = "Android"; 
    }
}

// Usage:
Smartphone phone = new Smartphone(); 
System.out.println(phone.operatingSystem); // Output: Android
```

#### Example B: The Parameterized Constructor (Enforcing Requirements)
This constructor requires data to be passed at the exact moment of object creation.

```java
public class BankAccount {
    String accountHolder;
    double balance;

    // Parameterized Constructor
    public BankAccount(String name, double initialDeposit) {
        this.accountHolder = name;
        this.balance = initialDeposit;
    }
}

// Usage:
// This line compiles and successfully initializes the object:
BankAccount account = new BankAccount("John Doe", 500.0);

// This line will throw a COMPILATION ERROR because the compiler 
// is protecting the class from being created in an empty state:
// BankAccount emptyAccount = new BankAccount(); 
```
**Question:** Does a constructor return a value?
**Answer:** No, a constructor does not return a value, and it cannot have a return type (not even `void`). Although it looks like the `new` keyword returns the newly created object, the constructor's sole responsibility is to initialize that object's memory.


In Java, the **`this`** keyword is a reference variable that points directly to the **current object** instance executing the code. 

There are four primary use cases for the `this` keyword that frequently appear in placement tests and technical interviews.

---

### 1. Resolving Instance Variable Shadowing (Most Common)
When a local variable (or method parameter) has the exact same name as an instance variable, the local variable "shadows" the instance variable. You use `this` to tell the compiler you want to access the instance variable, not the local one.

```java
public class User {
    private String name; // Instance variable

    public User(String name) { // 'name' here is a local parameter
        // name = name;     // Error: assigns the parameter to itself (shadowing)
        this.name = name;   // Correct: assigns parameter 'name' to instance variable 'name'
    }
}
```

---

### Key Interview Question (The Static Context Constraint)

**Q: Can you use the `this` keyword inside a static method (like the `main` method)?**

**A:** No. You will get a compilation error: *"non-static variable this cannot be referenced from a static context"*. 

*   **Why:** Static methods belong to the class itself and are loaded into memory before any objects are created. The `this` keyword represents a specific *object instance*. Because a static method can be run without any instances existing, `this` has nothing to point to.

---
# DATA TYPES 
---


### Part 1: The Absolute Basics (Bits & Bytes)

#### 1. What is a Data Type?
A data type is an instruction to the Java compiler that specifies:
1.  **What** kind of value is being stored (integer, decimal, character, etc.).
2.  **How much** memory to allocate for it.
3.  **Which** operations are allowed on it.

#### 2. What is a Bit?
*   The smallest unit of computer data.
*   Can only hold one of two values: **`0` (OFF)** or **`1` (ON)**.

#### 3. What is a Byte?
*   **$1 \text{ Byte} = 8 \text{ Bits}$** (e.g., `10101010`).
*   Each bit has 2 possibilities, so 1 byte yields $2^8 = 256$ unique binary combinations.
*   Because Java uses **signed integers** (which represent both positive and negative numbers), this range of 256 combinations is split:
    *   **Negative side:** $-128$ values.
    *   **Positive side (including 0):** $127$ values.
    *   **Total Range for 1 Byte:** **$-128$ to $127$**.

---

### Part 2: The Integer Family

Java has 4 integer data types. By default, any plain number (like `100` or `-50`) is treated as an **`int`**.

#### 1. `byte` (1 Byte / 8 Bits)
*   **Range:** $-128$ to $127$
*   **Critical Edge Case (Overflow):**
    ```java
    byte x = 127;
    x++; // Adding 1 causes an overflow.
    System.out.println(x); // Output: -128
    ```
    *   *Why?* In binary, `127` is `01111111`. Adding `1` changes it to `10000000`. In Two's Complement representation, this leftmost `1` makes the number negative, rendering it as `-128`.

#### 2. `short` (2 Bytes / 16 Bits)
*   **Range:** $-32,768$ to $32,767$
*   **Usage:** Rarely used today; reserved for highly memory-constrained environments.

#### 3. `int` (4 Bytes / 32 Bits)
*   **Range:** $\approx \pm 2.14$ Billion
*   **Usage:** The **default** integer type. Modern processors are optimized to handle 32-bit blocks of data highly efficiently.

#### 4. `long` (8 Bytes / 64 Bits)
*   **Range:** $\approx \pm 9.22 \times 10^{18}$
*   **Rule:** If an integer literal exceeds the range of an `int`, you must append the **`L`** suffix to tell Java to allocate 8 bytes.
    ```java
    // long pop = 8000000000;  // COMPILATION ERROR
    long pop = 8000000000L;    // Correct
    ```
    *   *Note:* Always use uppercase **`L`** because lowercase **`l`** is easily mistaken for the number **`1`**.

---

### Part 3: The Decimal Family (Floating Point)

Decimals are treated as **`double`** by default in Java.

#### 1. `float` (4 Bytes / 32 Bits)
*   **Rule:** You must append the **`f`** or **`F`** suffix because decimal literals default to `double`.
    ```java
    float price = 19.99f;
    ```

#### 2. `double` (8 Bytes / 64 Bits)
*   **Usage:** The default decimal type. Offers double the precision (~15–16 decimal places) of a `float` (~7 decimal places).

#### 3. Critical Floating-Point Precision Flaw (IEEE 754)
Computers store decimal fractions in binary format. Many decimal fractions (like `0.1` or `0.2`) cannot be represented perfectly in binary, leading to minute rounding differences.
```java
System.out.println(0.1 + 0.2); // Output: 0.30000000000000004
```
*   *Placement Solution:* For financial software or applications requiring absolute mathematical accuracy, use the **`BigDecimal`** class rather than `double` or `float`.

---

### Part 4: Character and Boolean

#### 1. `char` (2 Bytes / 16 Bits)
*   Stores a single Unicode character. Must be enclosed in **single quotes** (`'A'`), whereas double quotes (`"A"`) denote a `String`.
*   **Under the Hood:** Characters map to numeric Unicode values.
    ```java
    char c = 65;
    System.out.println(c); // Output: A (since 65 is the Unicode value for 'A')
    ```

#### 2. `boolean` (JVM Dependent Size)
*   Only accepts **`true`** or **`false`**.
*   It does not have a fixed size defined by the Java Virtual Machine specification, as it depends on implementation optimizations.

---

### Part 5: Data Type Cheat Sheet

| Type | Size | Range | Best Used For |
| :--- | :--- | :--- | :--- |
| **`byte`** | 1 Byte | $-128$ to $127$ | Low-level file/stream manipulation |
| **`short`** | 2 Bytes | $-32,768$ to $32,767$ | Memory-critical systems (rarely used) |
| **`int`** | 4 Bytes | $\approx \pm 2.1$ Billion | Standard counting, loops, indices |
| **`long`** | 8 Bytes | $\approx \pm 9.22 \times 10^{18}$ | Database IDs, population counts, time in ms |
| **`float`** | 4 Bytes | ~7 decimal digits of accuracy | Graphics rendering, low-precision decimals |
| **`double`**| 8 Bytes | ~15–16 decimal digits of accuracy | General math, scientific calculations |
| **`char`** | 2 Bytes | $0$ to $65,535$ | Single characters, text processing |
| **`boolean`**| VM Dependent| `true` / `false` | Conditional flags, logic checks |

---

### 3. Critical Edge Cases (Highly Tested in MCQs/Interviews)

#### Edge Case 1: Integer Overflow and Underflow (The Wrap-Around)
If you exceed the maximum limit of an integer type, it silently wraps around to the minimum negative value without throwing any error.

```java
byte b = 127; // Max byte value
b++; 
System.out.println(b); // Output: -128 (Overflow)

int max = Integer.MAX_VALUE; // 2147483647
System.out.println(max + 1); // Output: -2147483648
```

#### Edge Case 2: Automatic Type Promotion in Expressions
When performing arithmetic operations on `byte`, `short`, or `char`, Java automatically promotes them to `int` before evaluating. This frequently leads to unexpected compilation errors.

```java
byte x = 10;
byte y = 20;
// byte z = x + y; // COMPILATION ERROR: x + y is promoted to 'int'

// Correct Way: Explicit casting
byte z = (byte)(x + y); 
```

#### Edge Case 3: Floating-Point Precision Loss
Because computers represent fractional numbers in binary, certain decimal values (like `0.1` or `0.2`) cannot be represented with perfect accuracy. 

```java
double a = 0.1;
double b = 0.2;
System.out.println(a + b); // Output: 0.30000000000000004
System.out.println((a + b) == 0.3); // Output: false
```
*   **Interview Follow-up:** How do you handle precise monetary values?
*   **Answer:** Do not use `float` or `double`. Use the **`BigDecimal`** class (from `java.math`).

#### Edge Case 4: Division by Zero (Integer vs. Floating-Point)
The JVM handles dividing by zero differently depending on whether the variables are integers or decimals.

```java
// Case A: Integer Division
try {
    int result = 10 / 0; 
} catch (ArithmeticException e) {
    System.out.println("Throws ArithmeticException"); // This executes
}

// Case B: Floating-Point Division
double resultDouble = 10.0 / 0.0;
System.out.println(resultDouble); // Output: Infinity

double nanResult = 0.0 / 0.0;
System.out.println(nanResult);    // Output: NaN (Not a Number)
```
*   **Key Rule:** Integer division by zero throws an `ArithmeticException`. Floating-point division by zero yields `Infinity` or `NaN` without crashing.

#### Edge Case 5: Literal Type Defaults
*   Any integer literal (e.g., `100`, `5000`) is treated as an `int` by default. To make it a `long`, you must append `L` or `l`.
*   Any decimal literal (e.g., `5.5`, `99.9`) is treated as a `double` by default. To store it in a `float`, you must append `F` or `f`.

```java
// long val = 10000000000; // COMPILATION ERROR: Out of range for default 'int'
long val = 10000000000L;    // Correct

// float f = 5.5;          // COMPILATION ERROR: Cannot convert 'double' to 'float'
float f = 5.5f;             // Correct
```


---
# Java Development Kit ( JDK )
---



### Part 1: The Java Execution Pipeline

To understand how Java works, you must distinguish between what happens at **compile-time** and what happens at **runtime**.

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/5e35e21a-8eff-4d08-ada7-8f031869cb86" />


```


[Source Code] (.java) 
       │
       ▼  (javac - Compiler)
[Bytecode] (.class)  <--- Platform Independent
       │
       ▼  (java - Launcher/JVM)
┌──────────────────────────────────────────────┐
│ JVM (Platform Dependent)                     │
│  ├─► Class Loader                            │
│  └─► Execution Engine                        │
│       ├─► Interpreter (Line-by-line execution)│
│       └─► JIT Compiler (Compiles hot code)   │
└──────────────────────────────────────────────┘
       │
       ▼
[Native Machine Code] (Platform Specific)
```

#### Step-by-Step Execution Mechanics
1.  **Compilation (Compile-Time):**
    *   You write human-readable source code in `.java` files.
    *   The Java Compiler (`javac`) translates this source code into intermediate **Bytecode** stored in `.class` files.
    *   **Key Concept:** Bytecode is a highly optimized, platform-independent set of instructions designed for the JVM, not for any specific physical CPU.
2.  **Loading & Verification (Runtime):**
    *   The JVM's **Class Loader** loads the `.class` files into memory.
    *   The **Bytecode Verifier** checks the code to ensure it does not violate Java's security constraints (e.g., illegal memory access or stack overflows).
3.  **Translation & Execution (Runtime):**
    *   The JVM's **Execution Engine** converts bytecode into native machine code (1s and 0s) that the host CPU can execute. This is done using two components:
        *   **Interpreter:** Reads and executes bytecode instructions line-by-line. This starts quickly but executes slowly overall.
        *   **Just-In-Time (JIT) Compiler:** Identifies frequently executed sections of bytecode ("hot spots") and compiles them into native machine code once. Subsequent calls to this code run at native speed, significantly boosting performance.

---

### Part 2: The Core Trinity (JDK vs. JRE vs. JVM)

Understanding the nesting relationship of these three components is a standard requirement for technical interviews.

$$\text{JDK} = \text{JRE} + \text{Development Tools}$$
$$\text{JRE} = \text{JVM} + \text{Standard Libraries}$$

<img width="600" height="450" alt="image" src="https://github.com/user-attachments/assets/d15fcfdf-aa79-44fb-a7f2-1da1a93373d9" />


#### 1. JVM (Java Virtual Machine)
*   **Definition:** The abstract computing machine that executes compiled Java bytecode.
*   **Key Property:** **Platform Dependent**. There are distinct JVM installations optimized for Windows, macOS, and Linux. The JVM is what makes Java bytecode platform-*independent* (Write Once, Run Anywhere).
*   **Responsibilities:** Memory allocation, garbage collection, thread management, and security enforcement.

#### 2. JRE (Java Runtime Environment)
*   **Definition:** The minimal software package needed to **run** an existing Java application.
*   **Components:** Contains the JVM plus the Java Standard Libraries (pre-built code like `java.lang.String`, `java.util.List`, and input/output systems).
*   **Target User:** End-users running Java desktop apps or servers that do not require any code modifications.

#### 3. JDK (Java Development Kit)
*   **Definition:** The complete software development environment needed to **create, compile, and run** Java applications.
*   **Components:** Contains the JRE plus the Development Tools (compiler, debugger, profilers).
*   **Target User:** Software developers.

---

### Part 3: High-Yield Development Tools

The JDK includes a variety of specialized command-line tools. The following are the most frequently discussed in technical assessments:

#### A. Core Compilation & Execution Tools
*   **`javac`**: The Java compiler. Translates `.java` source files into `.class` bytecode files.
*   **`java`**: Launches a Java application by starting the JVM and running the specified `.class` file.
*   **`javap`**: The Java Disassembler. It reads `.class` files and prints their bytecode structure, method signatures, and fields.
*   **`jshell`**: An interactive Read-Evaluate-Print Loop (REPL) introduced in Java 9. Used to test short code snippets instantly without compiling a full class.

#### B. Diagnostics, Monitoring, & Troubleshooting Tools
*   **`jps`** (JVM Process Status tool): Lists all active JVM processes running on your system along with their Process IDs (PIDs).
*   **`jstack`**: Prints the thread stack trace for a given Java process. Crucial for diagnosing **deadlocks** or performance bottlenecks.
*   **`jmap`**: Generates a memory map or heap dump of a running Java process. Crucial for finding **memory leaks**.
*   **`jcmd`**: A unified diagnostic command tool used to send diagnostic requests directly to a running JVM.

---

### Part 4: 2025 JDK Vendor Landscape

While Java specifications are standardized, multiple organizations distribute their own versions of the JDK based on the open-source **OpenJDK** project.

| Vendor / Distribution | Target Use Case & Characteristics |
| :--- | :--- |
| **Oracle JDK** | The official commercial distribution. Highly optimized with long-term support (LTS), governed by specific commercial licensing terms. |
| **Eclipse Temurin (Adoptium)** | Community-supported, open-source, and free. Highly popular for general cloud and on-premises deployments. |
| **Amazon Corretto** | Free, production-ready, and long-term supported distribution. Highly optimized for AWS-based systems. |
| **Microsoft Build of OpenJDK** | Specifically optimized and supported for running Java workloads on Azure Cloud. |
| **Azul Zulu** | Known for enterprise-grade performance, high compliance testing, and robust support for Docker/containers. |
| **IBM Semeru** | Tailored for IBM Cloud and mainframes; optimized to work with IBM enterprise systems. |

---

### Part 5: Placement Interview Q&A

**Q1: Why is Java considered both a compiled and an interpreted language?**
*   **Answer:** Because it uses a two-step translation process. First, the source code is **compiled** into bytecode by `javac`. Second, during runtime, the JVM's execution engine **interprets** that bytecode line-by-line while selectively using the JIT compiler to compile performance-critical code blocks directly into native machine code.

**Q2: If Java bytecode is platform-independent, how does it run on different Operating Systems?**
*   **Answer:** While the generated bytecode (`.class` file) is identical across all systems, the JVM itself is platform-dependent. A Windows machine uses a Windows-compatible JVM, while a Linux machine uses a Linux-compatible JVM. Each specific JVM reads the same universal bytecode and translates it into the appropriate native machine code for its host operating system.

**Q3: How do you identify a thread deadlock in a production environment?**
*   **Answer:** 
    1.  Run the **`jps`** command to find the Process ID (PID) of the running Java application.
    2.  Run **`jstack <PID>`** to generate a complete thread dump.
    3.  Analyze the output; the JVM automatically appends a "Found one Java-level deadlock" section at the end of a stack trace when a deadlock is detected, identifying the specific threads and locked monitors involved.



---

# JVM MEMORY ARCHITECTURE 

---


<img width="600" height="450" alt="image" src="https://github.com/user-attachments/assets/83d37343-06da-4e27-9d68-fbef3d87cc49" />


### 1. Stack Memory
The execution area. It manages method calls and tracks what your program is actively doing.

*   **What it stores:**
    *   Local variables.
    *   Primitive values (like `int`, `double`, `boolean`).
    *   Reference variables (the pointers/addresses of objects).
*   **How it works (LIFO):**
    *   Last In, First Out. Like a stack of plates.
    *   Each method call pushes a new "frame" onto the stack.
*   **Key Rules:**
    *   **Thread-specific:** Every thread has its own private Stack.
    *   **Automatic Cleanup:** When a method finishes, its frame is instantly popped off and deleted.
    *   **No Access sharing:** A method cannot see or use variables in another method’s frame.

 <img width="500" height="350" alt="image" src="https://github.com/user-attachments/assets/0d4b5b1a-99d3-4cb5-bfae-49b15310c16a" />


#### Quick Stack Trace Example:
```java
void main() {
    int i = 1;
    foo(i);
}
void foo(int param) {
    int k = 3;
}
```
**Inside Stack Memory (during `foo` execution):**
```
[ foo() Frame:  param = 1, k = 3 ]  <-- Active (Top)
[ main() Frame: i = 1            ]  <-- Waiting
```
When `foo()` finishes, its entire block disappears.

---

### 2. Heap Memory
The dynamic storage area. It holds data that needs to stay alive across different method calls.

*   **What it stores:**
    *   All **Objects** (created using the `new` keyword).
    *   All Arrays.
*   **Key Rules:**
    *   **Shared:** One single heap is shared among all running threads.
    *   **Garbage Collected:** You do not delete objects manually. The Garbage Collector (GC) deletes them when they are no longer in use.
    *   **Crash Condition:** If you create too many objects and the Heap runs out of space, Java crashes with an `OutOfMemoryError`.

---

<img width="500" height="350" alt="image" src="https://github.com/user-attachments/assets/eb99bed7-85fa-43df-b7e9-724643281a54" />


### 3. Stack-to-Heap Relationship
```
STACK (Stores address)             HEAP (Stores actual object)
┌────────────────┐                 ┌──────────────────────┐
│  reff ─────────┼────────────────►│  Heap_Test Object    │
└────────────────┘                 └──────────────────────┘
```
The **reference variable** (`reff`) lives on the Stack. It holds the memory address of the **actual object**, which lives on the Heap.

---

### 4. String Pool (SCP)
A dedicated, protected storage space **inside the Heap** used to save memory.

*   **The Goal:** Avoid creating duplicate String objects. If a string already exists in the pool, Java reuses it.

#### The Golden Rule of Comparison:
*   **`==`** compares the **Memory Address** (Are they pointing to the same spot?).
*   **`.equals()`** compares the **Actual Text** (Are the characters identical?).

---

### 5. String Pool Allocation Mechanics (Memorize This)

<img width="500" height="350" alt="image" src="https://github.com/user-attachments/assets/131aa4a4-3ac8-476e-960c-a8a29a713a62" />


#### Case A: Using String Literals (String Pool)
```java
String s1 = "Hello";
String s2 = "Hello";
```
*   `s1` creates `"Hello"` in the String Pool.
*   `s2` sees `"Hello"` already exists in the pool, so it points to the exact same memory location.
*   **`s1 == s2` is `true`** (same memory location).

#### Case B: Using the `new` Keyword (Heap)
```java
String s3 = new String("Hi");
String s4 = "Hi";
```
*   `new String("Hi")` forces Java to create a **brand-new object** in general Heap memory.
*   `s4 = "Hi"` creates or reuses `"Hi"` directly inside the String Pool.
*   **`s3 == s4` is `false`** (different memory locations).

#### Case C: Using `.intern()` (Forcing the Pool)
```java
s3 = s3.intern();
```
*   The `.intern()` method looks at `s3` ("Hi"), searches the String Pool, and returns the memory address of the pool version.
*   Now, `s3` points to the String Pool version of `"Hi"`.
*   **`s3 == s4` is now `true`** (both point to the pool version).


---

# JAVA PROJECT STRUCTURE 

---


### 1. The Big Picture (Standard Directory Structure)
In professional development, almost all Java projects use a tool called **Maven** to organize files. Here is what a standard project looks like:

```
my-app/
│
├── pom.xml                 <-- The Project Configuration (POM file)
├── .gitignore              <-- A "Dot File"
│
└── src/                    <-- Source Code Directory
    ├── main/               <-- Main Code (Production)
    │   └── java/
    │       └── com/company/app/Main.java  <-- Main File & Package
    │
    └── test/               <-- Test Code (Development Verification)
        └── java/
            └── com/company/app/MainTest.java
```

---

### 2. The POM File (`pom.xml`)
*   **What it is:** **P**roject **O**bject **M**odel.
*   **Simple Definition:** The "recipe book" or "shopping list" of your project. Written in XML.
*   **What it does:**
    *   It tells Maven what external libraries (dependencies) your project needs (e.g., Spring Boot, Database drivers).
    *   Maven reads this file, automatically downloads those libraries from the internet, and adds them to your project.
    *   Saves you from manually downloading `.jar` files.

---

### 3. Main Code vs. Test Code
Java separates production code from development tests.

#### Main Code (`src/main/java`)
*   The actual application code.
*   This is the code that will eventually be packaged and delivered to the client or deployed to a server.

#### Test Code (`src/test/java`)
*   Code used to test the Main Code (e.g., Unit tests using JUnit).
*   **Crucial Rule:** Test code is only run during development. When packaging the app for release, Maven automatically excludes the test code so the final app is smaller and cleaner.

---

### 4. The Main File
*   **What it is:** The Java file that contains the entry point of the program:
    ```java
    public static void main(String[] args) { ... }
    ```
*   **What it does:** It is the starting line. When you run a Java application, the JVM looks for this specific file and method to begin execution.

---

### 5. Packages & Organization Naming
*   **What a Package is:** A package is simply a folder structure used to group related classes together and avoid naming conflicts.

#### How Companies Name Packages:
*   Companies use the **Reverse Domain Name** convention.
*   **Formula:** `domain_extension.company_name.project_name.module`
*   **Examples:**
    *   If Google makes a project named "maps", the package will be: `com.google.maps`
    *   If Amazon makes a payment module, the package will be: `com.amazon.payment`
*   **Why?** Because domain names are globally unique. This guarantees that your package names will never clash with another company's package names when libraries are combined.

---

### 6. The Maven Compiler
*   **What it is:** A built-in plugin inside Maven.
*   **What it does:**
    *   It automates the process of compiling your entire project.
    *   Instead of you manually running `javac File1.java File2.java`, the Maven compiler compiles all your `.java` files into `.class` files with a single command (`mvn compile`).
    *   It also lets you specify which Java version (e.g., Java 17 or Java 21) the project should compile with inside the `pom.xml`.

---

### 7. Dot Files (e.g., `.gitignore`)
*   **What they are:** Files that start with a dot (`.`). Operating systems (Windows, Linux, macOS) treat these as **hidden files** by default.
*   **Purpose:** They are used to store configurations for external developer tools.
*   **The most common example:** `.gitignore`
    *   This file tells Git (version control) which local files to ignore.
    *   For example, you do not want to upload compiled `.class` files, temporary folders (like `target/`), or local IDE configurations to GitHub. You list those folder names inside `.gitignore`.



---


# TYPE CONVERSIONS 

---

Here is how Java handles Type Conversion at the **byte and bit level**. 

In Java, there are two types of conversion: **Widening (Implicit)** and **Narrowing (Explicit)**.

---

### 1. Widening Conversion (Implicit / Automatic)
*   **What it is:** Converting a smaller data type to a larger data type (e.g., `byte` to `int`).
*   **Safety:** 100% safe. No data loss can occur.
*   **Byte-level mechanic:** **Sign Extension**. The JVM pads the extra bytes on the left with `0`s (for positive numbers) or `1`s (for negative numbers) to preserve the sign.

#### Byte-Level Visualization (Positive Number)
```java
byte b = 5;
int i = b; // Automatic widening
```
*   **`b` (1 Byte / 8 bits):**
    ```
    00000101
    ```
*   **`i` (4 Bytes / 32 bits):**
    The JVM pads the left 24 bits with `0`s:
    ```
    00000000 00000000 00000000 00000101
    ```

#### Byte-Level Visualization (Negative Number)
```java
byte b = -5;
int i = b; 
```
*   **`b` (1 Byte / 8 bits in Two's Complement):**
    ```
    11111011
    ```
*   **`i` (4 Bytes / 32 bits):**
    The JVM performs sign extension, padding the left 24 bits with `1`s to keep the number negative:
    ```
    11111111 11111111 11111111 11111011
    ```

---

### 2. Narrowing Conversion (Explicit Casting)
*   **What it is:** Converting a larger data type to a smaller data type (e.g., `int` to `byte`).
*   **Safety:** Unsafe. Can cause data corruption or unexpected values.
*   **Byte-level mechanic:** **Truncation (Bit-Lopping)**. The JVM throws away the higher-order bytes and keeps only the lower-order bytes that fit into the target type.

#### Scenario A: Safe Narrowing (Value fits in target range)
```java
int i = 50;
byte b = (byte) i; // Explicit cast required
```
*   **`i` (32 bits):**
    ```
    00000000 00000000 00000000 00110010
    ```
*   **`b` (8 bits):**
    The JVM throws away the first three bytes (24 bits) and keeps only the last byte:
    ```
    [thrown away]             00110010  --> Value is still 50.
    ```

#### Scenario B: Unsafe Narrowing (Value overflows target range)
This is a classic placement test question.
```java
int i = 130;
byte b = (byte) i; 
System.out.println(b); // What is the output?
```
*   **`i` (32 bits) representation of 130:**
    ```
    00000000 00000000 00000000 10000010
    ```
*   **`b` (8 bits):**
    The JVM truncates the first 24 bits, leaving only the last byte:
    ```
    [thrown away]             10000010
    ```
*   **The Result:** 
    *   In an 8-bit signed byte, the leftmost bit is the sign bit. 
    *   Since the leftmost bit is `1`, the JVM reads `10000010` as a negative number in Two's Complement.
    *   `10000010` converts to **`-126`**.
    *   **Output:** `-126` (Data has been corrupted/wrapped around).

---

### Summary Cheat Sheet for Interviews

| Conversion Type | Direction | JVM Action | Risk |
| :--- | :--- | :--- | :--- |
| **Widening** | Small $\to$ Large | **Sign Extension** (Pads left bits with `0` or `1`) | None (Safe) |
| **Narrowing** | Large $\to$ Small | **Truncation** (Chops off the leftmost bytes) | High (Data loss/Overflow) |


### 1. Other Common Primitive Conversions

#### A. `long` (64-bit) to `int` (32-bit) — Narrowing
*   **What happens:** The JVM chops off (truncates) the leftmost 32 bits and keeps the rightmost 32 bits.

```java
long l = 4294967297L; // Binary: 00000000000000000000000000000001 00000000000000000000000000000001
int i = (int) l;
System.out.println(i); // Output: 1
```
*   *Why?* The upper 32 bits containing the first `1` are completely discarded. Only the lower 32 bits (which represent `1`) remain.

#### B. `double` (64-bit) to `int` (32-bit) — Floating-Point to Integer
*   **What happens:** This is a **truncation of the decimal part**, not rounding. The JVM simply discards everything after the decimal point.

```java
double d = 9.99;
int i = (int) d;
System.out.println(i); // Output: 9 (Not 10!)
```

---

### 2. ASCII (American Standard Code for Information Interchange)

*   **What it is:** An early 7-bit character encoding system that maps 128 characters (English letters, numbers, and basic symbols) to numbers from `0` to `127`.
*   **Key Values to Memorize for Placements:**
    *   **`'A'` to `'Z'`** $\to$ $65$ to $90$
    *   **`'a'` to `'z'`** $\to$ $97$ to $122$
    *   **`'0'` to `'9'`** $\to$ $48$ to $57$
    *   **Space (`' '`)** $\to$ $32$

```java
char ch = 'A';
int asciiValue = ch; // Implicit widening (char to int)
System.out.println(asciiValue); // Output: 65

char nextChar = (char)(asciiValue + 1); 
System.out.println(nextChar);   // Output: B
```

---

### 3. Unicode and UTF (Unicode Transformation Format)

*   **The Problem with ASCII:** 128 slots are not enough to represent non-English characters (like Chinese, Hindi, or emojis).
*   **The Solution (Unicode):** A global standard that assigns a unique number (a code point) to every character in every language.

#### How Unicode relates to UTF and Java:
*   **UTF-8:** A variable-width encoding (uses 1 to 4 bytes per character). It is the standard for web pages because it is backward-compatible with ASCII.
*   **UTF-16 (Java's Choice):** Uses a fixed 2 bytes (16 bits) for most common characters. 
*   **Java's `char` Memory Structure:**
    *   A Java `char` uses **2 bytes (16 bits)** and is unsigned (range: $0$ to $65,535$).
    *   Because Java uses UTF-16 internally, you can natively store international characters:
    ```java
    char hindiChar = 'अ'; // Valid in Java
    ```

---

### 4. Booleans in Java (The Interview "Trap")

In languages like C or C++, an integer value of `0` is considered `false`, and any non-zero value is considered `true`. 

**Java does not allow this.**

*   **Rule:** In Java, the `boolean` type (`true`/`false`) is completely incompatible with all other primitive types.
*   You **cannot** cast an integer to a boolean, and you **cannot** cast a boolean to an integer.

```java
// int x = 1;
// boolean flag = (boolean) x; // COMPILATION ERROR: Incompatible types

// Correct Java logic:
int x = 1;
boolean flag = (x == 1); // Evaluates to true
```


---
BIT MANIPULATION

---

Here is the complete guide to Bit Manipulation, condensed into simple, high-yield revision notes for placement rounds.

---

### Part 1: The Bitwise Operators (The Rules)

| Operator | Name | Rule / Behavior | Key Property |
| :--- | :--- | :--- | :--- |
| **`&`** | **Bitwise AND** | Yields `1` only if both bits are `1`. | Clears bits. `X & 0 = 0` |
| **`\|`** | **Bitwise OR** | Yields `1` if at least one bit is `1`. | Sets bits. `X \| 1 = 1` |
| **`^`** | **Bitwise XOR** | Yields `1` if the bits are **different**. | Flipped behavior. `X ^ X = 0` and `X ^ 0 = X` |
| **`~`** | **Bitwise NOT** | Inverts all bits (`0` becomes `1`, `1` becomes `0`). | Unary operator (takes one operand). |
| **`<<`** | **Left Shift** | Shifts bits left, fills empty slots with `0`. | **`a << b`** is equivalent to $a \times 2^b$ |
| **`>>`** | **Right Shift** | Shifts bits right, preserves the sign bit. | **`a >> b`** is equivalent to $a / 2^b$ |
| **`>>>`**| **Unsigned Right Shift**| Shifts bits right, fills left slots with `0` regardless of sign. | Only used in Java, not C++. |

---

### Part 2: The "Big Four" Bit Operations

For all operations below, we assume 0-based indexing from **right to left** (Least Significant Bit is index `0`).

```
Bits:    1  0  1  1  0  1  0
Index:   6  5  4  3  2  1  0
```

#### 1. Get the $i$-th Bit (Find if it is 0 or 1)
Shift the number right by `i` positions, and inspect the last bit using `& 1`.
```java
int bitValue = (n >> i) & 1;
```

#### 2. Set the $i$-th Bit (Make it 1, leave others alone)
Use a bitwise OR (`|`) with a mask where only the $i$-th bit is 1.
```java
int result = n | (1 << i);
```

#### 3. Clear the $i$-th Bit (Make it 0, leave others alone)
Create a mask where only the $i$-th bit is 0 (e.g., `1110111`), and perform a bitwise AND (`&`).
```java
int result = n & ~(1 << i);
```

#### 4. Toggle the $i$-th Bit (Flip 0 $\to$ 1 or 1 $\to$ 0)
Use a bitwise XOR (`^`) with a mask where only the $i$-th bit is 1.
```java
int result = n ^ (1 << i);
```

---

### Part 3: High-Yield Placement Tricks

These are highly popular patterns in multiple-choice questions (MCQs) and coding tests.

#### Trick 1: Check if a number is Even or Odd
Instead of using modulo `n % 2 == 0`, look at the last bit. Any odd number must end in `1`.
```java
if ((n & 1) == 0) {
    // Number is Even
} else {
    // Number is Odd
}
```

#### Trick 2: Check if a number is a Power of 2
Powers of 2 in binary contain exactly one `1` bit (e.g., $8 = 1000_2$, $16 = 10000_2$). 
If you subtract 1, all bits flip (e.g., $7 = 0111_2$).
```java
boolean isPowerOfTwo = (n > 0) && ((n & (n - 1)) == 0);
```

#### Trick 3: Clear the lowest (rightmost) set bit
Executing `n & (n - 1)` instantly turns off the lowest `1` bit in a binary number.
*   *Use Case:* Count the number of set bits (Kernighan's Algorithm).
```java
int count = 0;
while (n > 0) {
    n = n & (n - 1); // Clears the lowest 1 bit
    count++;
}
```

#### Trick 4: Swap two variables without a third variable
Uses the XOR self-canceling property (`X ^ X = 0`).
```java
int a = 5;
int b = 9;

a = a ^ b;
b = a ^ b; // b becomes original a
a = a ^ b; // a becomes original b
```

#### Trick 5: Find the unique (non-duplicate) number
*   **Problem:** You are given an array where every number appears twice except one. Find the single number.
*   **Logic:** XOR everything in the array. Duplicates will cancel each other out to `0`.
```java
int[] arr = {4, 1, 2, 1, 2};
int unique = 0;
for (int num : arr) {
    unique ^= num; // XOR accumulation
}
System.out.println(unique); // Output: 4
```


---

# TODO 

---

1. **For a standalone boolean variable:** 
   * **Size:** It takes **4 bytes (32 bits)** of memory, behaving just like an `int`.
   * **The Reason:** **CPU Optimization**. Modern processors read and write memory in 32-bit or 64-bit blocks (words). Treating a single boolean as a 32-bit integer allows the processor to read and execute it in a single clock cycle without executing extra bit-shifting instructions. Additionally, physical RAM is byte-addressable, not bit-addressable.

2. **For boolean arrays (`boolean[]`):**
   * **Size:** Each element takes **1 byte (8 bits)**.
   * **The Reason:** **Memory Conservation**. If a boolean array with millions of elements padded every boolean to 32 bits, it would waste massive heap space. To prevent this, the JVM compiles and executes boolean arrays as `byte` arrays under the hood."

---

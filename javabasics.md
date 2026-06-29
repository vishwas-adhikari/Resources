
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



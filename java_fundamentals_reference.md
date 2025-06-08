# Java Fundamentals Reference Manual

## Table of Contents
1. [Keywords](#keywords)
2. [Variables](#variables)
3. [Loops](#loops)
4. [Methods](#methods)
5. [Access Modifiers](#access-modifiers)
6. [Javadoc](#javadoc)
7. [Arrays](#arrays)

---

## Keywords

Java keywords are reserved words that have special meaning in the language. They cannot be used as identifiers (variable names, method names, class names, etc.).

### Common Keywords by Category

**Access Modifiers:**
- `public`, `private`, `protected`

**Class/Interface/Method Modifiers:**
- `class`, `interface`, `abstract`, `final`, `static`

**Data Types:**
- `int`, `double`, `float`, `boolean`, `char`, `byte`, `short`, `long`

**Control Flow:**
- `if`, `else`, `switch`, `case`, `default`, `for`, `while`, `do`, `break`, `continue`

**Exception Handling:**
- `try`, `catch`, `finally`, `throw`, `throws`

**Object-Oriented:**
- `new`, `this`, `super`, `extends`, `implements`

**Other Important:**
- `return`, `void`, `null`, `true`, `false`

### Example Usage
```java
public class KeywordExample {
    private static final int MAX_SIZE = 100;
    
    public boolean isValid(String input) {
        if (input == null) {
            return false;
        }
        return true;
    }
}
```

---

## Variables

Variables in Java are containers that store data values. Java is statically typed, meaning you must declare the type of a variable before using it.

### Variable Declaration and Initialization

**Syntax:**
```java
dataType variableName = value;
```

### Primitive Data Types

| Type | Size | Range | Default | Example |
|------|------|-------|---------|---------|
| `byte` | 8 bits | -128 to 127 | 0 | `byte age = 25;` |
| `short` | 16 bits | -32,768 to 32,767 | 0 | `short year = 2024;` |
| `int` | 32 bits | -2³¹ to 2³¹-1 | 0 | `int count = 1000;` |
| `long` | 64 bits | -2⁶³ to 2⁶³-1 | 0L | `long population = 8000000000L;` |
| `float` | 32 bits | IEEE 754 | 0.0f | `float price = 19.99f;` |
| `double` | 64 bits | IEEE 754 | 0.0d | `double pi = 3.14159;` |
| `boolean` | 1 bit | true/false | false | `boolean isActive = true;` |
| `char` | 16 bits | Unicode | '\u0000' | `char grade = 'A';` |

### Reference Types
```java
String name = "John Doe";
Integer number = 42;
List<String> items = new ArrayList<>();
```

### Variable Examples
```java
public class VariableExamples {
    // Instance variables
    private String name;
    private int age;
    
    // Class variable (static)
    private static int totalStudents = 0;
    
    // Constants
    private static final double PI = 3.14159;
    
    public void demonstrateVariables() {
        // Local variables
        int localNumber = 100;
        String message = "Hello, World!";
        boolean isReady = false;
        
        // Type inference (Java 10+)
        var automaticType = "This is a string";
        var numberList = new ArrayList<Integer>();
    }
}
```

### Key Differences from Python
- **Static Typing:** Must declare types explicitly
- **Compilation:** Variables are type-checked at compile time
- **Primitives vs Objects:** Java has primitive types and reference types
- **Final Variables:** Use `final` keyword for constants (like Python's convention with UPPERCASE)

---

## Loops

Loops allow you to execute code repeatedly until a condition is met.

### For Loop

**Traditional For Loop:**
```java
for (initialization; condition; increment) {
    // code to execute
}

// Example
for (int i = 0; i < 10; i++) {
    System.out.println("Count: " + i);
}
```

**Enhanced For Loop (For-Each):**
```java
int[] numbers = {1, 2, 3, 4, 5};
for (int number : numbers) {
    System.out.println(number);
}

// With collections
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
for (String name : names) {
    System.out.println(name);
}
```

### While Loop
```java
int count = 0;
while (count < 5) {
    System.out.println("Count: " + count);
    count++;
}
```

### Do-While Loop
```java
int number;
do {
    number = scanner.nextInt();
    System.out.println("You entered: " + number);
} while (number != 0);
```

### Loop Control Statements
```java
for (int i = 0; i < 10; i++) {
    if (i == 3) {
        continue; // Skip iteration
    }
    if (i == 7) {
        break; // Exit loop
    }
    System.out.println(i);
}
```

### Practical Examples
```java
public class LoopExamples {
    
    // Find sum of array elements
    public int sumArray(int[] numbers) {
        int sum = 0;
        for (int number : numbers) {
            sum += number;
        }
        return sum;
    }
    
    // Process user input until valid
    public int getValidInput(Scanner scanner) {
        int input;
        do {
            System.out.print("Enter a positive number: ");
            input = scanner.nextInt();
        } while (input <= 0);
        return input;
    }
    
    // Nested loops for 2D processing
    public void print2DArray(int[][] matrix) {
        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix[i].length; j++) {
                System.out.print(matrix[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

---

## Methods

Methods are reusable blocks of code that perform specific tasks. They help organize code and avoid repetition.

### Method Syntax
```java
accessModifier returnType methodName(parameters) {
    // method body
    return value; // if not void
}
```

### Method Examples
```java
public class MethodExamples {
    
    // Simple method with no parameters
    public void sayHello() {
        System.out.println("Hello, World!");
    }
    
    // Method with parameters and return value
    public int add(int a, int b) {
        return a + b;
    }
    
    // Method with multiple parameters
    public String createFullName(String firstName, String lastName) {
        return firstName + " " + lastName;
    }
    
    // Method that returns boolean
    public boolean isEven(int number) {
        return number % 2 == 0;
    }
    
    // Method with array parameter
    public double calculateAverage(double[] scores) {
        if (scores.length == 0) {
            return 0;
        }
        
        double sum = 0;
        for (double score : scores) {
            sum += score;
        }
        return sum / scores.length;
    }
    
    // Overloaded methods (same name, different parameters)
    public int multiply(int a, int b) {
        return a * b;
    }
    
    public double multiply(double a, double b) {
        return a * b;
    }
    
    public int multiply(int a, int b, int c) {
        return a * b * c;
    }
}
```

### Static Methods
```java
public class MathUtils {
    
    // Static method - belongs to class, not instance
    public static int factorial(int n) {
        if (n <= 1) {
            return 1;
        }
        return n * factorial(n - 1);
    }
    
    // Usage: MathUtils.factorial(5)
}
```

### Method Best Practices
- **Single Responsibility:** Each method should do one thing well
- **Descriptive Names:** Use verbs that describe what the method does
- **Parameter Validation:** Check inputs when necessary
- **Return Early:** Use guard clauses to reduce nesting

```java
public boolean isValidEmail(String email) {
    // Guard clauses
    if (email == null || email.trim().isEmpty()) {
        return false;
    }
    
    if (!email.contains("@")) {
        return false;
    }
    
    // Main logic
    return email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$");
}
```

---

## Access Modifiers

Access modifiers control the visibility and accessibility of classes, methods, and variables.

### Access Modifier Types

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| (default) | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

### Detailed Explanations

**Public:** Accessible from anywhere
```java
public class PublicExample {
    public String name; // Anyone can access
    
    public void publicMethod() {
        // Anyone can call this method
    }
}
```

**Private:** Only accessible within the same class
```java
public class PrivateExample {
    private String secretData; // Only this class can access
    private int privateCounter;
    
    private void internalMethod() {
        // Only methods in this class can call this
    }
    
    // Getter/Setter pattern for controlled access
    public String getSecretData() {
        return secretData;
    }
    
    public void setSecretData(String secretData) {
        if (secretData != null && !secretData.trim().isEmpty()) {
            this.secretData = secretData;
        }
    }
}
```

**Protected:** Accessible within package and by subclasses
```java
public class ProtectedExample {
    protected String familySecret; // Subclasses and package can access
    
    protected void familyMethod() {
        // Subclasses and package classes can call this
    }
}

class ChildClass extends ProtectedExample {
    public void demonstrateAccess() {
        familySecret = "I can access this"; // ✓ Allowed
        familyMethod(); // ✓ Allowed
    }
}
```

**Package-Private (Default):** Accessible within the same package
```java
class PackageExample { // No modifier = package-private
    String packageData; // Package-private field
    
    void packageMethod() { // Package-private method
        // Only classes in the same package can access
    }
}
```

### Practical Example
```java
public class BankAccount {
    private double balance; // Private - controlled access
    private String accountNumber;
    protected String accountType; // Protected - for inheritance
    
    public BankAccount(String accountNumber, String accountType) {
        this.accountNumber = accountNumber;
        this.accountType = accountType;
        this.balance = 0.0;
    }
    
    // Public interface
    public void deposit(double amount) {
        if (isValidAmount(amount)) {
            balance += amount;
        }
    }
    
    public boolean withdraw(double amount) {
        if (isValidAmount(amount) && hasSufficientFunds(amount)) {
            balance -= amount;
            return true;
        }
        return false;
    }
    
    public double getBalance() {
        return balance;
    }
    
    // Private helper methods
    private boolean isValidAmount(double amount) {
        return amount > 0;
    }
    
    private boolean hasSufficientFunds(double amount) {
        return balance >= amount;
    }
}
```

---

## Javadoc

Javadoc is a documentation tool that generates HTML documentation from specially formatted comments in Java source code.

### Javadoc Comment Syntax
```java
/**
 * This is a Javadoc comment
 * @param parameterName description of parameter
 * @return description of return value
 * @throws ExceptionType description of when exception is thrown
 * @author author name
 * @since version number
 * @see related class or method
 */
```

### Common Javadoc Tags

| Tag | Description | Usage |
|-----|-------------|-------|
| `@param` | Parameter description | `@param name the user's name` |
| `@return` | Return value description | `@return true if valid, false otherwise` |
| `@throws` | Exception description | `@throws IllegalArgumentException if input is null` |
| `@author` | Author information | `@author John Smith` |
| `@since` | Version introduced | `@since 1.0` |
| `@see` | Reference to related items | `@see String#length()` |
| `@deprecated` | Mark as deprecated | `@deprecated Use newMethod() instead` |

### Comprehensive Example
```java
/**
 * Represents a student in the university system.
 * This class provides functionality to manage student information
 * including grades, enrollment, and academic status.
 * 
 * @author Tech Lead Team
 * @since 1.0
 * @version 2.1
 */
public class Student {
    
    /**
     * The student's unique identification number.
     * This value cannot be changed after initialization.
     */
    private final String studentId;
    
    /**
     * The student's full name.
     */
    private String fullName;
    
    /**
     * List of courses the student is enrolled in.
     */
    private List<Course> enrolledCourses;
    
    /**
     * Creates a new Student with the specified ID and name.
     * 
     * @param studentId unique identifier for the student, cannot be null or empty
     * @param fullName the student's full name, cannot be null or empty
     * @throws IllegalArgumentException if studentId or fullName is null or empty
     * @since 1.0
     */
    public Student(String studentId, String fullName) {
        if (studentId == null || studentId.trim().isEmpty()) {
            throw new IllegalArgumentException("Student ID cannot be null or empty");
        }
        if (fullName == null || fullName.trim().isEmpty()) {
            throw new IllegalArgumentException("Full name cannot be null or empty");
        }
        
        this.studentId = studentId;
        this.fullName = fullName;
        this.enrolledCourses = new ArrayList<>();
    }
    
    /**
     * Enrolls the student in a new course.
     * If the student is already enrolled in the course, no action is taken.
     * 
     * @param course the course to enroll in, cannot be null
     * @return true if enrollment was successful, false if already enrolled
     * @throws IllegalArgumentException if course is null
     * @throws EnrollmentException if enrollment capacity is exceeded
     * @see Course#hasCapacity()
     * @since 1.0
     */
    public boolean enrollInCourse(Course course) {
        if (course == null) {
            throw new IllegalArgumentException("Course cannot be null");
        }
        
        if (enrolledCourses.contains(course)) {
            return false;
        }
        
        if (!course.hasCapacity()) {
            throw new EnrollmentException("Course capacity exceeded");
        }
        
        enrolledCourses.add(course);
        return true;
    }
    
    /**
     * Calculates the student's current GPA based on completed courses.
     * Only courses with assigned grades are included in the calculation.
     * 
     * @return the GPA as a double value between 0.0 and 4.0,
     *         or 0.0 if no courses have been completed
     * @since 2.0
     */
    public double calculateGPA() {
        // Implementation details...
        return 0.0;
    }
    
    /**
     * Gets the student's unique ID.
     * 
     * @return the student ID as a String
     * @since 1.0
     */
    public String getStudentId() {
        return studentId;
    }
    
    /**
     * Sets the student's full name.
     * 
     * @param fullName the new full name, cannot be null or empty
     * @throws IllegalArgumentException if fullName is null or empty
     * @deprecated Use {@link #updatePersonalInfo(String, String)} instead
     * @since 1.0
     */
    @Deprecated
    public void setFullName(String fullName) {
        if (fullName == null || fullName.trim().isEmpty()) {
            throw new IllegalArgumentException("Full name cannot be null or empty");
        }
        this.fullName = fullName;
    }
}
```

### Generating Javadoc
```bash
# Command line
javadoc -d docs *.java

# With specific packages
javadoc -d docs -sourcepath src com.company.package
```

---

## Arrays

Arrays are data structures that store multiple values of the same type in a contiguous memory location.

### Array Declaration and Initialization

**Declaration:**
```java
dataType[] arrayName;
dataType arrayName[]; // Alternative syntax (less preferred)
```

**Initialization:**
```java
// Method 1: Declare and initialize separately
int[] numbers;
numbers = new int[5]; // Creates array of 5 integers (all 0)

// Method 2: Declare and initialize together
int[] scores = new int[10];

// Method 3: Initialize with values
int[] primes = {2, 3, 5, 7, 11};
String[] names = {"Alice", "Bob", "Charlie"};

// Method 4: Using new with values
int[] evenNumbers = new int[]{2, 4, 6, 8, 10};
```

### Array Operations
```java
public class ArrayExamples {
    
    public void basicArrayOperations() {
        // Creating and accessing arrays
        int[] numbers = {10, 20, 30, 40, 50};
        
        // Access elements (0-indexed)
        int first = numbers[0]; // 10
        int last = numbers[numbers.length - 1]; // 50
        
        // Modify elements
        numbers[2] = 35; // Changes 30 to 35
        
        // Array length
        int size = numbers.length; // 5
        
        // Iterate through array
        for (int i = 0; i < numbers.length; i++) {
            System.out.println("Index " + i + ": " + numbers[i]);
        }
        
        // Enhanced for loop (for-each)
        for (int number : numbers) {
            System.out.println(number);
        }
    }
    
    /**
     * Finds the maximum value in an integer array.
     * 
     * @param array the array to search, cannot be null or empty
     * @return the maximum value found in the array
     * @throws IllegalArgumentException if array is null or empty
     */
    public int findMaximum(int[] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Array cannot be null or empty");
        }
        
        int max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i] > max) {
                max = array[i];
            }
        }
        return max;
    }
    
    /**
     * Searches for a specific value in an array.
     * 
     * @param array the array to search
     * @param target the value to find
     * @return the index of the target value, or -1 if not found
     */
    public int linearSearch(int[] array, int target) {
        for (int i = 0; i < array.length; i++) {
            if (array[i] == target) {
                return i;
            }
        }
        return -1; // Not found
    }
    
    /**
     * Reverses the elements of an array in place.
     * 
     * @param array the array to reverse
     */
    public void reverseArray(int[] array) {
        int left = 0;
        int right = array.length - 1;
        
        while (left < right) {
            // Swap elements
            int temp = array[left];
            array[left] = array[right];
            array[right] = temp;
            
            left++;
            right--;
        }
    }
}
```

### Multi-dimensional Arrays
```java
public class MultiDimensionalArrays {
    
    public void demonstrate2DArrays() {
        // 2D array declaration and initialization
        int[][] matrix = new int[3][4]; // 3 rows, 4 columns
        
        // Initialize with values
        int[][] numbers = {
            {1, 2, 3, 4},
            {5, 6, 7, 8},
            {9, 10, 11, 12}
        };
        
        // Access elements
        int element = numbers[1][2]; // Gets 7 (row 1, column 2)
        
        // Iterate through 2D array
        for (int i = 0; i < numbers.length; i++) {
            for (int j = 0; j < numbers[i].length; j++) {
                System.out.print(numbers[i][j] + " ");
            }
            System.out.println();
        }
        
        // Enhanced for loop with 2D arrays
        for (int[] row : numbers) {
            for (int value : row) {
                System.out.print(value + " ");
            }
            System.out.println();
        }
    }
    
    /**
     * Calculates the sum of all elements in a 2D array.
     * 
     * @param matrix the 2D array to sum
     * @return the sum of all elements
     */
    public int sum2DArray(int[][] matrix) {
        int sum = 0;
        for (int[] row : matrix) {
            for (int value : row) {
                sum += value;
            }
        }
        return sum;
    }
}
```

### Array Utility Methods
```java
import java.util.Arrays;

public class ArrayUtilities {
    
    public void demonstrateArrayMethods() {
        int[] numbers = {64, 34, 25, 12, 22, 11, 90};
        
        // Convert array to string
        System.out.println(Arrays.toString(numbers));
        
        // Sort array
        Arrays.sort(numbers);
        System.out.println("Sorted: " + Arrays.toString(numbers));
        
        // Binary search (array must be sorted)
        int index = Arrays.binarySearch(numbers, 25);
        System.out.println("Index of 25: " + index);
        
        // Fill array with specific value
        int[] zeros = new int[5];
        Arrays.fill(zeros, 0);
        
        // Copy array
        int[] copy = Arrays.copyOf(numbers, numbers.length);
        
        // Copy range
        int[] partial = Arrays.copyOfRange(numbers, 1, 4);
        
        // Compare arrays
        boolean areEqual = Arrays.equals(numbers, copy);
    }
    
    /**
     * Creates a copy of an array with only unique elements.
     * 
     * @param array the source array
     * @return a new array containing only unique elements
     */
    public int[] removeDuplicates(int[] array) {
        if (array == null || array.length <= 1) {
            return array;
        }
        
        Arrays.sort(array); // Sort first to group duplicates
        
        int[] temp = new int[array.length];
        int j = 0;
        
        for (int i = 0; i < array.length - 1; i++) {
            if (array[i] != array[i + 1]) {
                temp[j++] = array[i];
            }
        }
        temp[j++] = array[array.length - 1]; // Add last element
        
        return Arrays.copyOf(temp, j);
    }
}
```

### Common Array Patterns
```java
public class ArrayPatterns {
    
    // Initialize array with calculated values
    public int[] createSequence(int start, int count, int step) {
        int[] sequence = new int[count];
        for (int i = 0; i < count; i++) {
            sequence[i] = start + (i * step);
        }
        return sequence;
    }
    
    // Filter array based on condition
    public int[] filterEvenNumbers(int[] array) {
        // First pass: count even numbers
        int count = 0;
        for (int num : array) {
            if (num % 2 == 0) {
                count++;
            }
        }
        
        // Second pass: create result array
        int[] result = new int[count];
        int index = 0;
        for (int num : array) {
            if (num % 2 == 0) {
                result[index++] = num;
            }
        }
        
        return result;
    }
    
    // Merge two sorted arrays
    public int[] mergeSortedArrays(int[] arr1, int[] arr2) {
        int[] merged = new int[arr1.length + arr2.length];
        int i = 0, j = 0, k = 0;
        
        while (i < arr1.length && j < arr2.length) {
            if (arr1[i] <= arr2[j]) {
                merged[k++] = arr1[i++];
            } else {
                merged[k++] = arr2[j++];
            }
        }
        
        // Copy remaining elements
        while (i < arr1.length) {
            merged[k++] = arr1[i++];
        }
        while (j < arr2.length) {
            merged[k++] = arr2[j++];
        }
        
        return merged;
    }
}
```

---

## Summary

This reference manual covers the fundamental building blocks of Java programming. As you transition back to Java from Python, remember these key differences:

- **Static Typing:** Java requires explicit type declarations
- **Compilation:** Code is compiled to bytecode before execution
- **Memory Management:** Java handles garbage collection automatically
- **Object-Oriented:** Everything is organized around classes and objects
- **Verbosity:** Java tends to be more verbose than Python but offers strong type safety

Keep this reference handy as you work with your development team. These fundamentals will serve as the foundation for more advanced Java and Spring Boot concepts you'll encounter in your tech lead role.
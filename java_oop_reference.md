# Java Object-Oriented Programming Reference Manual

## Table of Contents
1. [Understanding Objects](#understanding-objects)
2. [Classes vs Objects](#classes-vs-objects)
3. [Defining Classes](#defining-classes)
4. [Working with Objects](#working-with-objects)
5. [Object State and Behavior](#object-state-and-behavior)
6. [Garbage Collection](#garbage-collection)
7. [Advanced OOP Concepts](#advanced-oop-concepts)

---

## Understanding Objects

An **object** is a runtime instance of a class that encapsulates both **state** (data) and **behavior** (methods). Objects are the fundamental building blocks of object-oriented programming.

### Key Concepts

**State (Data):** The attributes or properties that describe the object's current condition
**Behavior (Methods):** The actions or operations that the object can perform

### Real-World Analogy
Think of a **Car** object:
- **State:** color, model, speed, fuel level, engine status
- **Behavior:** start(), stop(), accelerate(), brake(), honk()

### Object Characteristics

1. **Identity:** Each object has a unique identity in memory
2. **State:** Current values of the object's attributes
3. **Behavior:** Set of operations the object can perform
4. **Encapsulation:** State and behavior are bundled together

```java
// Example: A Car object in action
Car myCar = new Car("Toyota", "Camry", "Blue");
myCar.start();           // Behavior: starting the engine
myCar.accelerate(30);    // Behavior: increasing speed
String color = myCar.getColor(); // Accessing state: getting color
```

---

## Classes vs Objects

Understanding the distinction between classes and objects is fundamental to OOP.

### Class
A **class** is a blueprint, template, or prototype that defines:
- What attributes objects of this type will have
- What behaviors (methods) objects of this type can perform
- How objects should be constructed

### Object
An **object** is a specific instance of a class:
- Has actual values for the attributes defined in the class
- Can perform the behaviors defined in the class
- Exists in memory during program execution

### Analogy: House Blueprint vs Actual Houses

```java
/**
 * Class = Blueprint for houses
 * Defines what every house should have
 */
public class House {
    // Attributes every house has
    private String address;
    private int numberOfRooms;
    private double squareFootage;
    private boolean hasGarage;
    
    // Behaviors every house can have
    public void turnOnLights() { }
    public void lockDoors() { }
    public double calculatePropertyTax() { return 0.0; }
}

/**
 * Objects = Actual houses built from the blueprint
 */
public class HouseExample {
    public static void main(String[] args) {
        // Creating objects (actual houses) from the class (blueprint)
        House house1 = new House("123 Main St", 3, 1500.0, true);
        House house2 = new House("456 Oak Ave", 4, 2200.0, false);
        House house3 = new House("789 Pine Rd", 2, 1200.0, true);
        
        // Each object has its own state
        house1.turnOnLights(); // Only house1's lights turn on
        house2.lockDoors();    // Only house2's doors lock
    }
}
```

### Key Differences Table

| Aspect | Class | Object |
|--------|-------|--------|
| **Definition** | Template/Blueprint | Instance of a class |
| **Memory** | No memory allocated | Memory allocated at runtime |
| **Existence** | Exists at compile time | Exists at runtime |
| **Values** | No actual values | Has actual values |
| **Quantity** | One class definition | Many objects from one class |

---

## Defining Classes

A class definition specifies the structure and behavior of objects that will be created from it.

### Basic Class Structure

```java
[access_modifier] class ClassName {
    // 1. Fields (attributes/instance variables)
    private dataType fieldName;
    
    // 2. Constructors
    public ClassName(parameters) {
        // initialization code
    }
    
    // 3. Methods (behaviors)
    public returnType methodName(parameters) {
        // method implementation
    }
    
    // 4. Getters and Setters
    public dataType getFieldName() {
        return fieldName;
    }
    
    public void setFieldName(dataType value) {
        this.fieldName = value;
    }
}
```

### Comprehensive Class Example

```java
/**
 * Represents a bank account with basic banking operations.
 * Demonstrates proper class design with encapsulation.
 */
public class BankAccount {
    
    // 1. FIELDS (State) - Private for encapsulation
    private String accountNumber;
    private String ownerName;
    private double balance;
    private String accountType;
    private boolean isActive;
    
    // Class variable (shared by all instances)
    private static int totalAccounts = 0;
    
    // Constants
    private static final double MINIMUM_BALANCE = 10.0;
    private static final double OVERDRAFT_FEE = 35.0;
    
    // 2. CONSTRUCTORS
    
    /**
     * Default constructor - creates account with zero balance
     */
    public BankAccount() {
        this("UNKNOWN", "Anonymous", "SAVINGS");
    }
    
    /**
     * Constructor with basic information
     */
    public BankAccount(String accountNumber, String ownerName, String accountType) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.accountType = accountType;
        this.balance = 0.0;
        this.isActive = true;
        totalAccounts++; // Increment class variable
    }
    
    /**
     * Constructor with initial balance
     */
    public BankAccount(String accountNumber, String ownerName, 
                      String accountType, double initialBalance) {
        this(accountNumber, ownerName, accountType); // Call other constructor
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        }
    }
    
    // 3. METHODS (Behavior)
    
    /**
     * Deposits money into the account
     */
    public boolean deposit(double amount) {
        if (!isActive) {
            System.out.println("Cannot deposit to inactive account");
            return false;
        }
        
        if (amount <= 0) {
            System.out.println("Deposit amount must be positive");
            return false;
        }
        
        balance += amount;
        System.out.println("Deposited $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    /**
     * Withdraws money from the account
     */
    public boolean withdraw(double amount) {
        if (!isActive) {
            System.out.println("Cannot withdraw from inactive account");
            return false;
        }
        
        if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive");
            return false;
        }
        
        if (balance >= amount) {
            balance -= amount;
            System.out.println("Withdrew $" + amount + ". New balance: $" + balance);
            return true;
        } else {
            System.out.println("Insufficient funds. Current balance: $" + balance);
            // Apply overdraft fee for checking accounts
            if ("CHECKING".equals(accountType)) {
                balance -= OVERDRAFT_FEE;
                System.out.println("Overdraft fee applied: $" + OVERDRAFT_FEE);
            }
            return false;
        }
    }
    
    /**
     * Transfers money to another account
     */
    public boolean transferTo(BankAccount targetAccount, double amount) {
        if (this.withdraw(amount)) {
            return targetAccount.deposit(amount);
        }
        return false;
    }
    
    /**
     * Calculates monthly interest (simplified)
     */
    public void applyMonthlyInterest() {
        if (isActive && balance > 0) {
            double interestRate = "SAVINGS".equals(accountType) ? 0.02 : 0.01;
            double monthlyInterest = balance * (interestRate / 12);
            balance += monthlyInterest;
            System.out.println("Interest applied: $" + monthlyInterest);
        }
    }
    
    /**
     * Closes the account
     */
    public void closeAccount() {
        if (balance > 0) {
            System.out.println("Cannot close account with remaining balance: $" + balance);
        } else {
            isActive = false;
            System.out.println("Account " + accountNumber + " has been closed");
        }
    }
    
    // 4. GETTERS (Accessors)
    
    public String getAccountNumber() {
        return accountNumber;
    }
    
    public String getOwnerName() {
        return ownerName;
    }
    
    public double getBalance() {
        return balance;
    }
    
    public String getAccountType() {
        return accountType;
    }
    
    public boolean isActive() {
        return isActive;
    }
    
    // 5. SETTERS (Mutators) - Limited for security
    
    public void setOwnerName(String ownerName) {
        if (ownerName != null && !ownerName.trim().isEmpty()) {
            this.ownerName = ownerName;
        }
    }
    
    // 6. UTILITY METHODS
    
    /**
     * Returns account information as a formatted string
     */
    public String getAccountInfo() {
        return String.format("Account: %s | Owner: %s | Type: %s | Balance: $%.2f | Status: %s",
                accountNumber, ownerName, accountType, balance, 
                isActive ? "Active" : "Closed");
    }
    
    /**
     * Checks if account meets minimum balance requirement
     */
    public boolean meetsMinimumBalance() {
        return balance >= MINIMUM_BALANCE;
    }
    
    // 7. STATIC METHODS (Class-level behavior)
    
    /**
     * Returns total number of accounts created
     */
    public static int getTotalAccounts() {
        return totalAccounts;
    }
    
    /**
     * Validates account number format
     */
    public static boolean isValidAccountNumber(String accountNumber) {
        return accountNumber != null && 
               accountNumber.matches("\\d{10}"); // 10 digits
    }
    
    // 8. OBJECT METHODS (from Object class)
    
    @Override
    public String toString() {
        return getAccountInfo();
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        BankAccount other = (BankAccount) obj;
        return accountNumber.equals(other.accountNumber);
    }
}
```

### Class Design Best Practices

1. **Single Responsibility:** Each class should have one primary purpose
2. **Encapsulation:** Keep fields private, provide controlled access through methods
3. **Meaningful Names:** Use descriptive names for classes, methods, and fields
4. **Constructor Overloading:** Provide multiple ways to create objects
5. **Input Validation:** Validate parameters in methods and constructors
6. **Documentation:** Use Javadoc comments for public methods

---

## Working with Objects

Once you have defined a class, you can create and manipulate objects (instances) of that class.

### Object Creation Process

```java
// Step 1: Declaration - creates reference variable
BankAccount account;

// Step 2: Instantiation - creates object in memory
account = new BankAccount();

// Combined: Declaration + Instantiation
BankAccount account = new BankAccount("1234567890", "John Doe", "CHECKING");
```

### Object Lifecycle Example

```java
public class ObjectLifecycleDemo {
    
    public static void main(String[] args) {
        // 1. OBJECT CREATION
        System.out.println("=== Creating Objects ===");
        
        BankAccount johnAccount = new BankAccount("1111111111", "John Smith", "CHECKING", 1000.0);
        BankAccount janeAccount = new BankAccount("2222222222", "Jane Doe", "SAVINGS");
        BankAccount bobAccount = new BankAccount(); // Default constructor
        
        System.out.println("Total accounts created: " + BankAccount.getTotalAccounts());
        
        // 2. OBJECT INTERACTION
        System.out.println("\n=== Object Interactions ===");
        
        // Accessing object state
        System.out.println("John's balance: $" + johnAccount.getBalance());
        System.out.println("Jane's account type: " + janeAccount.getAccountType());
        
        // Modifying object state through behavior
        johnAccount.withdraw(200.0);
        janeAccount.deposit(500.0);
        
        // Objects interacting with each other
        johnAccount.transferTo(janeAccount, 100.0);
        
        // 3. OBJECT STATE COMPARISON
        System.out.println("\n=== Current States ===");
        System.out.println(johnAccount.getAccountInfo());
        System.out.println(janeAccount.getAccountInfo());
        System.out.println(bobAccount.getAccountInfo());
        
        // 4. OBJECT ARRAY MANAGEMENT
        System.out.println("\n=== Managing Multiple Objects ===");
        
        BankAccount[] accounts = {johnAccount, janeAccount, bobAccount};
        
        // Apply interest to all savings accounts
        for (BankAccount account : accounts) {
            if ("SAVINGS".equals(account.getAccountType())) {
                account.applyMonthlyInterest();
            }
        }
        
        // Find accounts with low balance
        System.out.println("\nAccounts needing attention:");
        for (BankAccount account : accounts) {
            if (!account.meetsMinimumBalance() && account.isActive()) {
                System.out.println("Low balance: " + account.getAccountInfo());
            }
        }
        
        // 5. OBJECT REFERENCES
        System.out.println("\n=== Object References ===");
        
        BankAccount aliasAccount = johnAccount; // Same object, different reference
        aliasAccount.deposit(50.0); // Affects johnAccount too
        
        System.out.println("John's account balance: " + johnAccount.getBalance());
        System.out.println("Alias account balance: " + aliasAccount.getBalance());
        System.out.println("Are they the same object? " + (johnAccount == aliasAccount));
        
        // 6. NULL REFERENCES
        System.out.println("\n=== Null Reference Handling ===");
        
        BankAccount nullAccount = null;
        if (nullAccount != null) {
            nullAccount.deposit(100.0); // Would cause NullPointerException
        } else {
            System.out.println("Cannot operate on null account reference");
        }
    }
}
```

### Object Method Invocation

```java
public class MethodInvocationExamples {
    
    public void demonstrateMethodCalls() {
        BankAccount account = new BankAccount("5555555555", "Alice Johnson", "CHECKING", 500.0);
        
        // 1. VOID METHODS (no return value)
        account.deposit(100.0);           // Performs action, no return
        account.setOwnerName("Alice J."); // Changes state, no return
        
        // 2. METHODS WITH RETURN VALUES
        double balance = account.getBalance();        // Returns double
        String info = account.getAccountInfo();      // Returns String
        boolean isActive = account.isActive();       // Returns boolean
        
        // 3. METHODS THAT RETURN OBJECTS
        String accountType = account.getAccountType(); // Returns String object
        
        // 4. CHAINING METHOD CALLS (if methods return objects)
        String upperCaseName = account.getOwnerName().toUpperCase().trim();
        
        // 5. CONDITIONAL METHOD CALLS
        if (account.getBalance() > 100) {
            account.withdraw(50.0);
        }
        
        // 6. STATIC METHOD CALLS (on class, not object)
        int totalAccounts = BankAccount.getTotalAccounts();
        boolean validNumber = BankAccount.isValidAccountNumber("1234567890");
    }
}
```

---

## Object State and Behavior

Understanding how objects maintain state and expose behavior is crucial for effective OOP.

### State Management

**State** represents the current condition of an object, stored in its fields/attributes.

```java
public class StateManagementExample {
    
    /**
     * Demonstrates different types of state in objects
     */
    public static class Student {
        // INSTANCE STATE (unique to each object)
        private String name;
        private int age;
        private double gpa;
        private List<String> courses;
        private boolean isEnrolled;
        
        // CLASS STATE (shared by all objects)
        private static int totalStudents = 0;
        private static final String SCHOOL_NAME = "Tech University";
        
        public Student(String name, int age) {
            this.name = name;
            this.age = age;
            this.gpa = 0.0;
            this.courses = new ArrayList<>();
            this.isEnrolled = true;
            totalStudents++;
        }
        
        // STATE MODIFICATION METHODS
        public void enrollInCourse(String courseName) {
            if (isEnrolled && !courses.contains(courseName)) {
                courses.add(courseName);
                System.out.println(name + " enrolled in " + courseName);
            }
        }
        
        public void updateGPA(double newGpa) {
            if (newGpa >= 0.0 && newGpa <= 4.0) {
                this.gpa = newGpa;
                checkAcademicStanding();
            }
        }
        
        // STATE-DEPENDENT BEHAVIOR
        private void checkAcademicStanding() {
            if (gpa < 2.0) {
                System.out.println(name + " is on academic probation");
            } else if (gpa >= 3.5) {
                System.out.println(name + " is on the honor roll");
            }
        }
        
        // STATE ACCESS METHODS
        public String getAcademicStatus() {
            if (!isEnrolled) return "Not Enrolled";
            if (gpa >= 3.8) return "Summa Cum Laude";
            if (gpa >= 3.5) return "Magna Cum Laude";
            if (gpa >= 3.2) return "Cum Laude";
            if (gpa >= 2.0) return "Good Standing";
            return "Academic Probation";
        }
        
        // Getters for state access
        public String getName() { return name; }
        public int getAge() { return age; }
        public double getGpa() { return gpa; }
        public List<String> getCourses() { return new ArrayList<>(courses); } // Defensive copy
        public boolean isEnrolled() { return isEnrolled; }
        public static int getTotalStudents() { return totalStudents; }
    }
    
    public static void demonstrateStateChanges() {
        Student alice = new Student("Alice", 20);
        Student bob = new Student("Bob", 19);
        
        // Initial state
        System.out.println("Initial state:");
        System.out.println("Alice GPA: " + alice.getGpa());
        System.out.println("Alice courses: " + alice.getCourses().size());
        
        // State changes through behavior
        alice.enrollInCourse("Computer Science 101");
        alice.enrollInCourse("Mathematics 201");
        alice.updateGPA(3.7);
        
        bob.enrollInCourse("Physics 101");
        bob.updateGPA(2.1);
        
        // State after changes
        System.out.println("\nAfter changes:");
        System.out.println("Alice status: " + alice.getAcademicStatus());
        System.out.println("Bob status: " + bob.getAcademicStatus());
        System.out.println("Total students: " + Student.getTotalStudents());
    }
}
```

### Behavior Implementation

**Behavior** is implemented through methods that can:
- Query object state (accessors/getters)
- Modify object state (mutators/setters)
- Perform calculations based on state
- Interact with other objects

```java
public class BehaviorExamples {
    
    /**
     * Shopping cart demonstrating various types of behavior
     */
    public static class ShoppingCart {
        private List<Item> items;
        private String customerName;
        private double taxRate;
        
        public ShoppingCart(String customerName, double taxRate) {
            this.customerName = customerName;
            this.taxRate = taxRate;
            this.items = new ArrayList<>();
        }
        
        // 1. STATE MODIFICATION BEHAVIOR
        public void addItem(String name, double price, int quantity) {
            Item newItem = new Item(name, price, quantity);
            
            // Check if item already exists
            for (Item item : items) {
                if (item.getName().equals(name)) {
                    item.increaseQuantity(quantity);
                    return;
                }
            }
            
            items.add(newItem);
            System.out.println("Added: " + quantity + "x " + name);
        }
        
        public boolean removeItem(String name) {
            return items.removeIf(item -> item.getName().equals(name));
        }
        
        public void updateItemQuantity(String name, int newQuantity) {
            for (Item item : items) {
                if (item.getName().equals(name)) {
                    item.setQuantity(newQuantity);
                    return;
                }
            }
        }
        
        // 2. CALCULATION BEHAVIOR (based on state)
        public double calculateSubtotal() {
            double subtotal = 0.0;
            for (Item item : items) {
                subtotal += item.getTotalPrice();
            }
            return subtotal;
        }
        
        public double calculateTax() {
            return calculateSubtotal() * taxRate;
        }
        
        public double calculateTotal() {
            return calculateSubtotal() + calculateTax();
        }
        
        // 3. QUERY BEHAVIOR (state access)
        public int getTotalItems() {
            int total = 0;
            for (Item item : items) {
                total += item.getQuantity();
            }
            return total;
        }
        
        public List<Item> getExpensiveItems(double threshold) {
            List<Item> expensive = new ArrayList<>();
            for (Item item : items) {
                if (item.getPrice() > threshold) {
                    expensive.add(item);
                }
            }
            return expensive;
        }
        
        // 4. VALIDATION BEHAVIOR
        public boolean isEmpty() {
            return items.isEmpty();
        }
        
        public boolean hasItem(String name) {
            return items.stream().anyMatch(item -> item.getName().equals(name));
        }
        
        // 5. UTILITY BEHAVIOR
        public void printReceipt() {
            System.out.println("\n=== RECEIPT ===");
            System.out.println("Customer: " + customerName);
            System.out.println("Items:");
            
            for (Item item : items) {
                System.out.printf("  %dx %s @ $%.2f = $%.2f%n",
                    item.getQuantity(), item.getName(), 
                    item.getPrice(), item.getTotalPrice());
            }
            
            System.out.printf("Subtotal: $%.2f%n", calculateSubtotal());
            System.out.printf("Tax: $%.2f%n", calculateTax());
            System.out.printf("TOTAL: $%.2f%n", calculateTotal());
            System.out.println("===============");
        }
        
        public void clearCart() {
            items.clear();
            System.out.println("Cart cleared for " + customerName);
        }
        
        // 6. OBJECT INTERACTION BEHAVIOR
        public void mergeWith(ShoppingCart otherCart) {
            for (Item item : otherCart.items) {
                addItem(item.getName(), item.getPrice(), item.getQuantity());
            }
        }
    }
    
    // Supporting class
    public static class Item {
        private String name;
        private double price;
        private int quantity;
        
        public Item(String name, double price, int quantity) {
            this.name = name;
            this.price = price;
            this.quantity = quantity;
        }
        
        public double getTotalPrice() {
            return price * quantity;
        }
        
        public void increaseQuantity(int amount) {
            this.quantity += amount;
        }
        
        // Getters and setters
        public String getName() { return name; }
        public double getPrice() { return price; }
        public int getQuantity() { return quantity; }
        public void setQuantity(int quantity) { this.quantity = quantity; }
    }
}
```

---

## Garbage Collection

Garbage Collection (GC) is Java's automatic memory management system that reclaims memory used by objects that are no longer reachable or referenced.

### How Garbage Collection Works

```java
public class GarbageCollectionDemo {
    
    public static void demonstrateGarbageCollection() {
        
        // 1. OBJECT CREATION (memory allocated)
        BankAccount account1 = new BankAccount("1111", "John", "CHECKING");
        BankAccount account2 = new BankAccount("2222", "Jane", "SAVINGS");
        
        System.out.println("Objects created - memory allocated");
        
        // 2. OBJECTS ARE REACHABLE (not eligible for GC)
        account1.deposit(1000);
        account2.deposit(500);
        
        // 3. REMOVING REFERENCES
        account1 = null; // Object becomes unreachable
        // The BankAccount object "1111" is now eligible for garbage collection
        
        // 4. REASSIGNING REFERENCES
        account2 = new BankAccount("3333", "Bob", "CHECKING");
        // The original account2 object "2222" is now eligible for GC
        
        // 5. SCOPE-BASED CLEANUP
        {
            BankAccount tempAccount = new BankAccount("TEMP", "Temporary", "SAVINGS");
            tempAccount.deposit(100);
            // tempAccount goes out of scope when this block ends
        } // tempAccount is now eligible for GC
        
        // 6. SUGGESTING GARBAGE COLLECTION (not guaranteed)
        System.gc(); // Suggests JVM to run garbage collection
        
        System.out.println("Garbage collection demonstration complete");
    }
}
```

### Memory Management Best Practices

```java
public class MemoryManagementBestPractices {
    
    /**
     * Demonstrates memory-efficient programming practices
     */
    public static class ResourceManager {
        private List<String> cache;
        
        public ResourceManager() {
            this.cache = new ArrayList<>();
        }
        
        // 1. EXPLICIT NULL ASSIGNMENT
        public void clearCache() {
            cache.clear();      // Remove elements
            cache = null;       // Remove reference to help GC
        }
        
        // 2. LOCAL VARIABLE SCOPE MANAGEMENT
        public void processLargeDataset() {
            // Keep large objects in limited scope
            {
                List<String> largeList = new ArrayList<>();
                for (int i = 0; i < 1000000; i++) {
                    largeList.add("Item " + i);
                }
                
                // Process the large list
                processData(largeList);
                
                // largeList goes out of scope here
                // and becomes eligible for GC
            }
            
            // Smaller operations continue here
            System.out.println("Large dataset processed");
        }
        
        private void processData(List<String> data) {
            // Process data without storing reference
            for (String item : data) {
                // Process each item
            }
            // No reference stored, parameter goes out of scope
        }
        
        // 3. AVOID MEMORY LEAKS
        public static class MemoryLeakExample {
            private static List<Object> staticList = new ArrayList<>();
            
            public void potentialMemoryLeak() {
                Object obj = new Object();
                staticList.add(obj); // Object will never be GC'd!
            }
            
            public void avoidMemoryLeak() {
                Object obj = new Object();
                // Process object without storing in static collection
                // obj becomes eligible for GC when method ends
            }
        }
        
        // 4. WEAK REFERENCES (Advanced)
        public void demonstrateWeakReferences() {
            // Regular reference prevents GC
            BankAccount strongRef = new BankAccount("STRONG", "Strong Reference", "CHECKING");
            
            // Weak reference allows GC even if reference exists
            java.lang.ref.WeakReference<BankAccount> weakRef = 
                new java.lang.ref.WeakReference<>(strongRef);
            
            strongRef = null; // Remove strong reference
            
            // Object may be garbage collected even though weakRef exists
            System.gc();
            
            BankAccount retrieved = weakRef.get();
            if (retrieved == null) {
                System.out.println("Object was garbage collected");
            } else {
                System.out.println("Object still available: " + retrieved.getAccountNumber());
            }
        }
    }
    
    // 5. FINALIZATION (Rarely used, generally discouraged)
    public static class ResourceWithCleanup {
        private String resourceName;
        
        public ResourceWithCleanup(String name) {
            this.resourceName = name;
            System.out.println("Resource " + name + " created");
        }
        
        // Called by GC before object is destroyed (not guaranteed when)
        @Override
        protected void finalize() throws Throwable {
            try {
                System.out.println("Resource " + resourceName + " being finalized");
            } finally {
                super.finalize();
            }
        }
    }
}
```

### Garbage Collection Types and Tuning

```java
public class GarbageCollectionTypes {
    
    /**
     * Information about different GC algorithms
     * (This is informational - you cannot directly control which GC runs)
     */
    public static void printGCInfo() {
        
        System.out.println("=== Garbage Collection Information ===");
        
        // Get memory information
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        
        System.out.println("Max Memory: " + maxMemory / (1024 * 1024) + " MB");
        System.out.println("Total Memory: " + totalMemory / (1024 * 1024) + " MB");
        System.out.println("Free Memory: " + freeMemory / (1024 * 1024) + " MB");
        System.out.println("Used Memory: " + usedMemory / (1024 * 1024) + " MB");
        
        // Force garbage collection and measure
        long beforeGC = usedMemory;
        System.gc();
        
        // Wait a moment for GC to potentially run
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        long afterGC = (runtime.totalMemory() - runtime.freeMemory());
        long freedMemory = beforeGC - afterGC;
        
        System.out.println("Memory potentially freed by GC: " + freedMemory / 1024 + " KB");
    }
}
```

### When Objects Become Eligible for GC

1. **No more references:** When all references to an object are removed
2. **Out of scope:** When variables go out of scope
3. **Reassignment:** When reference variables are assigned to new objects
4. **Null assignment:** When references are explicitly set to null
5. **Method completion:** When local variables' scope ends

---

## Advanced OOP Concepts

### Object Identity and Equality

```java
public class ObjectIdentityExample {
    
    public static void demonstrateObjectIdentity() {
        
        // 1. REFERENCE EQUALITY (==)
        BankAccount account1 = new BankAccount("1111", "John", "CHECKING");
        BankAccount account2 = new BankAccount("1111", "John", "CHECKING");
        BankAccount account3 = account1; // Same reference
        
        System.out.println("Reference equality:");
        System.out.println("account1 == account2: " + (account1 == account2)); // false
        System.out.println("account1 == account3: " + (account1 == account3)); // true
        
        // 2. OBJECT EQUALITY (.equals())
        System.out.println("\nObject equality:");
        System.out.println("account1.equals(account2): " + account1.equals(account2)); // true (if equals is overridden)
        System.out.println("account1.equals(account3): " + account1.equals(account3)); // true
        
        // 3. HASH CODES
        System.out.println("\nHash codes:");
        System.out.println("account1.hashCode(): " + account1.hashCode());
        System.out.println("account2.hashCode(): " + account2.hashCode());
        System.out.println("account3.hashCode(): " + account3.hashCode());
    }
}
```

### Object Composition

```java
/**
 * Demonstrates composition - objects containing other objects
 */
public class CompositionExample {
    
    public static class Address {
        private String street;
        private String city;
        private String state;
        private String zipCode;
        
        public Address(String street, String city, String state, String zipCode) {
            this.street = street;
            this.city = city;
            this.state = state;
            this.zipCode = zipCode;
        }
        
        public String getFullAddress() {
            return street + ", " + city + ", " + state + " " + zipCode;
        }
        
        // Getters
        public String getStreet() { return street; }
        public String getCity() { return city; }
        public String getState() { return state; }
        public String getZipCode() { return zipCode; }
    }
    
    public static class Person {
        private String name;
        private int age;
        private Address homeAddress;    // Composition
        private Address workAddress;    // Composition
        private List<BankAccount> accounts; // Composition with collection
        
        public Person(String name, int age, Address homeAddress) {
            this.name = name;
            this.age = age;
            this.homeAddress = homeAddress;
            this.accounts = new ArrayList<>();
        }
        
        // Delegating to composed objects
        public String getHomeCity() {
            return homeAddress.getCity();
        }
        
        public void addBankAccount(BankAccount account) {
            accounts.add(account);
        }
        
        public double getTotalAssets() {
            double total = 0.0;
            for (BankAccount account : accounts) {
                total += account.getBalance();
            }
            return total;
        }
        
        public void setWorkAddress(Address workAddress) {
            this.workAddress = workAddress;
        }
        
        public String getPersonInfo() {
            StringBuilder info = new StringBuilder();
            info.append("Name: ").append(name).append("\n");
            info.append("Age: ").append(age).append("\n");
            info.append("Home: ").append(homeAddress.getFullAddress()).append("\n");
            
            if (workAddress != null) {
                info.append("Work: ").append(workAddress.getFullAddress()).append("\n");
            }
            
            info.append("Total Assets: $").append(getTotalAssets()).append("\n");
            
            return info.toString();
        }
    }
}
```

---

## Summary

This manual covers the fundamental concepts of Object-Oriented Programming in Java:

### Key Takeaways

1. **Objects vs Classes:** Classes are blueprints; objects are instances with actual state and behavior
2. **Encapsulation:** Keep state private and provide controlled access through methods
3. **State Management:** Objects maintain their condition through fields/attributes
4. **Behavior Implementation:** Methods define what objects can do and how they interact
5. **Memory Management:** Java automatically handles memory through garbage collection
6. **Object Lifecycle:** Objects are created, used, and eventually garbage collected

### Comparing to Python

- **Memory Management:** Java has automatic GC vs Python's reference counting + GC
- **Type Safety:** Java's compile-time type checking vs Python's runtime typing
- **Encapsulation:** Java enforces access modifiers vs Python's naming conventions
- **Object Creation:** Java requires explicit constructors vs Python's `__init__` method

As you lead your development team, these OOP principles will be essential for designing clean, maintainable Java and Spring Boot applications. The concepts of proper encapsulation, clear object responsibilities, and effective memory management will help you guide your developers toward better code architecture.
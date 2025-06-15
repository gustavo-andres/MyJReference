# Java Advanced OOP Reference Manual

## Table of Contents
1. [Packages](#packages)
2. [Inheritance](#inheritance)
3. [Abstract Classes](#abstract-classes)
4. [Interfaces](#interfaces)
5. [Polymorphism](#polymorphism)
6. [Advanced OOP Patterns](#advanced-oop-patterns)
7. [Best Practices](#best-practices)

---

## Packages

Packages are Java's way of organizing related classes and interfaces into namespaces, similar to folders in a file system. They provide organization, access control, and help avoid naming conflicts.

### Package Declaration and Structure

```java
// File: com/company/banking/Account.java
package com.company.banking;

// File: com/company/banking/services/TransactionService.java
package com.company.banking.services;

// File: com/company/banking/models/Customer.java
package com.company.banking.models;
```

### Common Package Naming Conventions

```java
// Company domain reversed + project + module
com.company.projectname.modulename

// Examples:
com.techcorp.bankingsystem.accounts
com.techcorp.bankingsystem.transactions
com.techcorp.bankingsystem.customers
com.techcorp.bankingsystem.services
com.techcorp.bankingsystem.utils
```

### Complete Package Example

```java
// File: com/company/banking/models/Account.java
package com.company.banking.models;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

/**
 * Base account class in the banking models package
 */
public class Account {
    protected String accountNumber;
    protected String ownerName;
    protected double balance;
    protected LocalDateTime createdDate;
    
    public Account(String accountNumber, String ownerName) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = 0.0;
        this.createdDate = LocalDateTime.now();
    }
    
    // Methods...
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }
    
    // Getters
    public String getAccountNumber() { return accountNumber; }
    public String getOwnerName() { return ownerName; }
    public double getBalance() { return balance; }
    public LocalDateTime getCreatedDate() { return createdDate; }
}
```

```java
// File: com/company/banking/services/BankingService.java
package com.company.banking.services;

import com.company.banking.models.Account;
import com.company.banking.models.Customer;
import java.util.HashMap;
import java.util.Map;

/**
 * Banking service class in the services package
 */
public class BankingService {
    private Map<String, Account> accounts;
    private Map<String, Customer> customers;
    
    public BankingService() {
        this.accounts = new HashMap<>();
        this.customers = new HashMap<>();
    }
    
    public void createAccount(String accountNumber, String customerName) {
        Account account = new Account(accountNumber, customerName);
        accounts.put(accountNumber, account);
    }
    
    public Account getAccount(String accountNumber) {
        return accounts.get(accountNumber);
    }
    
    public boolean transferFunds(String fromAccount, String toAccount, double amount) {
        Account from = accounts.get(fromAccount);
        Account to = accounts.get(toAccount);
        
        if (from != null && to != null && from.withdraw(amount)) {
            to.deposit(amount);
            return true;
        }
        return false;
    }
}
```

```java
// File: com/company/banking/utils/ValidationUtils.java
package com.company.banking.utils;

/**
 * Utility class for common validation operations
 */
public class ValidationUtils {
    
    // Private constructor to prevent instantiation
    private ValidationUtils() {}
    
    public static boolean isValidAccountNumber(String accountNumber) {
        return accountNumber != null && 
               accountNumber.matches("\\d{10}") && 
               !accountNumber.equals("0000000000");
    }
    
    public static boolean isValidAmount(double amount) {
        return amount > 0 && amount <= 1000000;
    }
    
    public static boolean isValidName(String name) {
        return name != null && 
               name.trim().length() >= 2 && 
               name.matches("[a-zA-Z\\s]+");
    }
}
```

### Import Statements

```java
// File: com/company/banking/Main.java
package com.company.banking;

// Import specific classes
import com.company.banking.models.Account;
import com.company.banking.services.BankingService;
import com.company.banking.utils.ValidationUtils;

// Import all classes from a package (less preferred)
import com.company.banking.models.*;

// Import static methods
import static com.company.banking.utils.ValidationUtils.isValidAccountNumber;
import static java.lang.Math.PI;
import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        // Using imported classes
        BankingService service = new BankingService();
        
        // Using static import
        if (isValidAccountNumber("1234567890")) {
            service.createAccount("1234567890", "John Doe");
        }
        
        // Using static import for System.out
        out.println("Banking system initialized");
    }
}
```

### Package Access Control

```java
// Classes in the same package can access package-private members
package com.company.banking.models;

public class Account {
    private String accountNumber;     // Only this class
    protected double balance;         // This class + subclasses + package
    String ownerName;                // Package-private (default)
    public String accountType;       // Everyone
    
    void packageMethod() {           // Package-private method
        // Only classes in com.company.banking.models can call this
    }
}

class AccountHelper {               // Package-private class
    void helpAccount(Account account) {
        // Can access package-private and protected members
        account.ownerName = "Updated";     // ✓ Package-private
        account.balance = 1000.0;          // ✓ Protected
        account.packageMethod();           // ✓ Package-private method
    }
}
```

---

## Inheritance

Inheritance allows a class to inherit properties and methods from another class, creating a parent-child relationship. The child class (subclass) extends the parent class (superclass).

### Basic Inheritance Syntax

```java
public class ParentClass {
    // Parent class members
}

public class ChildClass extends ParentClass {
    // Child class inherits all non-private members from ParentClass
    // Child class can add its own members
}
```

### Comprehensive Inheritance Example

```java
// File: com/company/banking/models/Account.java
package com.company.banking.models;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

/**
 * Base Account class - Parent/Superclass
 */
public class Account {
    protected String accountNumber;
    protected String ownerName;
    protected double balance;
    protected LocalDateTime createdDate;
    protected List<String> transactionHistory;
    
    // Parent constructor
    public Account(String accountNumber, String ownerName) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = 0.0;
        this.createdDate = LocalDateTime.now();
        this.transactionHistory = new ArrayList<>();
    }
    
    // Parent methods
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            addTransaction("Deposit: $" + amount);
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            addTransaction("Withdrawal: $" + amount);
            return true;
        }
        return false;
    }
    
    protected void addTransaction(String transaction) {
        transactionHistory.add(LocalDateTime.now() + ": " + transaction);
    }
    
    public double getBalance() { return balance; }
    public String getAccountNumber() { return accountNumber; }
    public String getOwnerName() { return ownerName; }
    public List<String> getTransactionHistory() { 
        return new ArrayList<>(transactionHistory); 
    }
    
    // Method that can be overridden
    public double calculateMonthlyFee() {
        return 5.0; // Default fee
    }
    
    public String getAccountInfo() {
        return String.format("Account: %s, Owner: %s, Balance: $%.2f", 
                           accountNumber, ownerName, balance);
    }
}
```

```java
// File: com/company/banking/models/CheckingAccount.java
package com.company.banking.models;

/**
 * CheckingAccount - Child/Subclass of Account
 */
public class CheckingAccount extends Account {
    private int freeTransactionsPerMonth;
    private int transactionCount;
    private double overdraftLimit;
    
    // Child constructor must call parent constructor
    public CheckingAccount(String accountNumber, String ownerName, double overdraftLimit) {
        super(accountNumber, ownerName); // Call parent constructor
        this.freeTransactionsPerMonth = 10;
        this.transactionCount = 0;
        this.overdraftLimit = overdraftLimit;
    }
    
    // Overriding parent method to add specific behavior
    @Override
    public boolean withdraw(double amount) {
        // Check overdraft limit
        if (amount > 0 && (balance + overdraftLimit) >= amount) {
            balance -= amount;
            transactionCount++;
            addTransaction("Withdrawal: $" + amount);
            
            // Apply overdraft fee if necessary
            if (balance < 0) {
                addTransaction("Overdraft fee applied");
            }
            
            return true;
        }
        return false;
    }
    
    // Overriding parent method
    @Override
    public void deposit(double amount) {
        super.deposit(amount); // Call parent implementation
        transactionCount++;    // Add child-specific behavior
    }
    
    // New method specific to CheckingAccount
    public void writeCheck(double amount, String payee) {
        if (withdraw(amount)) {
            addTransaction("Check written to " + payee + ": $" + amount);
        }
    }
    
    // Overriding parent method for specific fee calculation
    @Override
    public double calculateMonthlyFee() {
        if (transactionCount > freeTransactionsPerMonth) {
            return (transactionCount - freeTransactionsPerMonth) * 1.50;
        }
        return 0.0; // No fee if under limit
    }
    
    // Additional getters
    public double getOverdraftLimit() { return overdraftLimit; }
    public int getTransactionCount() { return transactionCount; }
    
    public void resetMonthlyTransactionCount() {
        transactionCount = 0;
    }
}
```

```java
// File: com/company/banking/models/SavingsAccount.java
package com.company.banking.models;

/**
 * SavingsAccount - Another child of Account
 */
public class SavingsAccount extends Account {
    private double interestRate;
    private int withdrawalsThisMonth;
    private static final int MAX_WITHDRAWALS_PER_MONTH = 6;
    
    public SavingsAccount(String accountNumber, String ownerName, double interestRate) {
        super(accountNumber, ownerName);
        this.interestRate = interestRate;
        this.withdrawalsThisMonth = 0;
    }
    
    // Overriding with restrictions
    @Override
    public boolean withdraw(double amount) {
        if (withdrawalsThisMonth >= MAX_WITHDRAWALS_PER_MONTH) {
            addTransaction("Withdrawal denied: Monthly limit exceeded");
            return false;
        }
        
        if (super.withdraw(amount)) { // Call parent implementation
            withdrawalsThisMonth++;
            return true;
        }
        return false;
    }
    
    // New behavior specific to savings
    public void applyMonthlyInterest() {
        double interest = balance * (interestRate / 12);
        balance += interest;
        addTransaction("Interest earned: $" + interest);
    }
    
    @Override
    public double calculateMonthlyFee() {
        // No monthly fee for savings accounts with minimum balance
        return balance < 100 ? 2.0 : 0.0;
    }
    
    // Savings-specific methods
    public double getInterestRate() { return interestRate; }
    public int getWithdrawalsThisMonth() { return withdrawalsThisMonth; }
    
    public void resetMonthlyWithdrawalCount() {
        withdrawalsThisMonth = 0;
    }
    
    public double projectedAnnualInterest() {
        return balance * interestRate;
    }
}
```

### Method Overriding Rules

```java
public class OverridingExample {
    
    // Parent class
    public static class Animal {
        protected String name;
        
        public Animal(String name) {
            this.name = name;
        }
        
        // Method to be overridden
        public void makeSound() {
            System.out.println(name + " makes a sound");
        }
        
        // Final method cannot be overridden
        public final void breathe() {
            System.out.println(name + " is breathing");
        }
        
        // Static method (hidden, not overridden)
        public static void staticMethod() {
            System.out.println("Animal static method");
        }
        
        public String getInfo() {
            return "Animal: " + name;
        }
    }
    
    // Child class
    public static class Dog extends Animal {
        private String breed;
        
        public Dog(String name, String breed) {
            super(name); // Must call parent constructor
            this.breed = breed;
        }
        
        // Overriding parent method
        @Override
        public void makeSound() {
            System.out.println(name + " barks: Woof!");
        }
        
        // Overriding with additional behavior
        @Override
        public String getInfo() {
            return super.getInfo() + ", Breed: " + breed; // Call parent method
        }
        
        // Static method (hides parent static method)
        public static void staticMethod() {
            System.out.println("Dog static method");
        }
        
        // New method specific to Dog
        public void wagTail() {
            System.out.println(name + " is wagging tail");
        }
    }
}
```

### Super Keyword Usage

```java
public class SuperKeywordExamples {
    
    public static class Vehicle {
        protected String make;
        protected String model;
        protected int year;
        
        public Vehicle(String make, String model, int year) {
            this.make = make;
            this.model = model;
            this.year = year;
        }
        
        public void start() {
            System.out.println("Vehicle is starting...");
        }
        
        public String getDescription() {
            return year + " " + make + " " + model;
        }
    }
    
    public static class Car extends Vehicle {
        private int doors;
        
        public Car(String make, String model, int year, int doors) {
            super(make, model, year); // 1. Call parent constructor
            this.doors = doors;
        }
        
        @Override
        public void start() {
            super.start(); // 2. Call parent method
            System.out.println("Car engine is running");
        }
        
        @Override
        public String getDescription() {
            return super.getDescription() + " with " + doors + " doors"; // 3. Call parent method
        }
        
        public void demonstrateSuper() {
            // 4. Access parent fields (if accessible)
            System.out.println("Make from parent: " + super.make);
            
            // Call parent method explicitly
            String parentDesc = super.getDescription();
            System.out.println("Parent description: " + parentDesc);
        }
    }
}
```

---

## Abstract Classes

Abstract classes are classes that cannot be instantiated directly but serve as blueprints for other classes. They can contain both abstract methods (without implementation) and concrete methods (with implementation).

### Abstract Class Syntax

```java
public abstract class AbstractClassName {
    // Concrete fields
    protected int concreteField;
    
    // Abstract method (no implementation)
    public abstract void abstractMethod();
    
    // Concrete method (with implementation)
    public void concreteMethod() {
        // Implementation
    }
}
```

### Comprehensive Abstract Class Example

```java
// File: com/company/banking/models/Investment.java
package com.company.banking.models;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

/**
 * Abstract base class for all investment types
 * Cannot be instantiated directly
 */
public abstract class Investment {
    protected String investmentId;
    protected String investorName;
    protected double principalAmount;
    protected LocalDate startDate;
    protected List<String> transactionLog;
    
    // Constructor for abstract class
    public Investment(String investmentId, String investorName, double principalAmount) {
        this.investmentId = investmentId;
        this.investorName = investorName;
        this.principalAmount = principalAmount;
        this.startDate = LocalDate.now();
        this.transactionLog = new ArrayList<>();
        logTransaction("Investment created with principal: $" + principalAmount);
    }
    
    // Abstract methods - must be implemented by subclasses
    public abstract double calculateCurrentValue();
    public abstract double calculateYearlyReturn();
    public abstract double calculateRisk(); // Returns risk percentage
    public abstract String getInvestmentType();
    
    // Concrete methods - shared by all subclasses
    public void addFunds(double amount) {
        if (amount > 0) {
            principalAmount += amount;
            logTransaction("Added funds: $" + amount);
        }
    }
    
    public boolean withdrawFunds(double amount) {
        double currentValue = calculateCurrentValue();
        if (amount > 0 && currentValue >= amount) {
            principalAmount -= amount;
            logTransaction("Withdrew funds: $" + amount);
            return true;
        }
        return false;
    }
    
    protected void logTransaction(String transaction) {
        transactionLog.add(LocalDate.now() + ": " + transaction);
    }
    
    // Template method pattern - uses abstract methods
    public String generateInvestmentReport() {
        StringBuilder report = new StringBuilder();
        report.append("=== Investment Report ===\n");
        report.append("Type: ").append(getInvestmentType()).append("\n");
        report.append("ID: ").append(investmentId).append("\n");
        report.append("Investor: ").append(investorName).append("\n");
        report.append("Principal: $").append(String.format("%.2f", principalAmount)).append("\n");
        report.append("Current Value: $").append(String.format("%.2f", calculateCurrentValue())).append("\n");
        report.append("Yearly Return: ").append(String.format("%.2f%%", calculateYearlyReturn())).append("\n");
        report.append("Risk Level: ").append(String.format("%.1f%%", calculateRisk())).append("\n");
        report.append("Start Date: ").append(startDate).append("\n");
        return report.toString();
    }
    
    // Getters
    public String getInvestmentId() { return investmentId; }
    public String getInvestorName() { return investorName; }
    public double getPrincipalAmount() { return principalAmount; }
    public LocalDate getStartDate() { return startDate; }
    public List<String> getTransactionLog() { return new ArrayList<>(transactionLog); }
}
```

```java
// File: com/company/banking/models/StockInvestment.java
package com.company.banking.models;

/**
 * Concrete implementation of Investment for stocks
 */
public class StockInvestment extends Investment {
    private String stockSymbol;
    private double sharesOwned;
    private double currentPricePerShare;
    private double dividendYield;
    
    public StockInvestment(String investmentId, String investorName, 
                          double principalAmount, String stockSymbol, double shares) {
        super(investmentId, investorName, principalAmount);
        this.stockSymbol = stockSymbol;
        this.sharesOwned = shares;
        this.currentPricePerShare = principalAmount / shares; // Initial price
        this.dividendYield = 0.02; // Default 2%
    }
    
    // Implementing abstract methods
    @Override
    public double calculateCurrentValue() {
        return sharesOwned * currentPricePerShare;
    }
    
    @Override
    public double calculateYearlyReturn() {
        double currentValue = calculateCurrentValue();
        double capitalGain = (currentValue - principalAmount) / principalAmount * 100;
        double dividendReturn = dividendYield * 100;
        return capitalGain + dividendReturn;
    }
    
    @Override
    public double calculateRisk() {
        // Simplified risk calculation based on stock volatility
        return 15.0; // Medium-high risk
    }
    
    @Override
    public String getInvestmentType() {
        return "Stock Investment (" + stockSymbol + ")";
    }
    
    // Stock-specific methods
    public void updateStockPrice(double newPrice) {
        this.currentPricePerShare = newPrice;
        logTransaction("Stock price updated to $" + newPrice + " per share");
    }
    
    public void buyMoreShares(double shares, double pricePerShare) {
        double cost = shares * pricePerShare;
        addFunds(cost);
        this.sharesOwned += shares;
        this.currentPricePerShare = pricePerShare;
        logTransaction("Bought " + shares + " shares at $" + pricePerShare + " each");
    }
    
    public boolean sellShares(double shares) {
        if (shares <= sharesOwned) {
            double proceeds = shares * currentPricePerShare;
            sharesOwned -= shares;
            withdrawFunds(proceeds);
            logTransaction("Sold " + shares + " shares at $" + currentPricePerShare + " each");
            return true;
        }
        return false;
    }
    
    // Getters
    public String getStockSymbol() { return stockSymbol; }
    public double getSharesOwned() { return sharesOwned; }
    public double getCurrentPricePerShare() { return currentPricePerShare; }
    public double getDividendYield() { return dividendYield; }
}
```

```java
// File: com/company/banking/models/BondInvestment.java
package com.company.banking.models;

/**
 * Concrete implementation of Investment for bonds
 */
public class BondInvestment extends Investment {
    private double interestRate;
    private int termInYears;
    private LocalDate maturityDate;
    private String bondType;
    
    public BondInvestment(String investmentId, String investorName, 
                         double principalAmount, double interestRate, 
                         int termInYears, String bondType) {
        super(investmentId, investorName, principalAmount);
        this.interestRate = interestRate;
        this.termInYears = termInYears;
        this.maturityDate = startDate.plusYears(termInYears);
        this.bondType = bondType;
    }
    
    // Implementing abstract methods
    @Override
    public double calculateCurrentValue() {
        // Simplified bond valuation
        long yearsHeld = java.time.temporal.ChronoUnit.YEARS.between(startDate, LocalDate.now());
        double accruedInterest = principalAmount * interestRate * yearsHeld;
        return principalAmount + accruedInterest;
    }
    
    @Override
    public double calculateYearlyReturn() {
        return interestRate * 100; // Interest rate as percentage
    }
    
    @Override
    public double calculateRisk() {
        // Bonds generally have lower risk
        if ("Government".equals(bondType)) {
            return 2.0; // Very low risk
        } else if ("Corporate".equals(bondType)) {
            return 8.0; // Medium risk
        } else {
            return 12.0; // Higher risk for other bond types
        }
    }
    
    @Override
    public String getInvestmentType() {
        return "Bond Investment (" + bondType + ")";
    }
    
    // Bond-specific methods
    public boolean isMatured() {
        return LocalDate.now().isAfter(maturityDate) || LocalDate.now().equals(maturityDate);
    }
    
    public double calculateMaturityValue() {
        return principalAmount + (principalAmount * interestRate * termInYears);
    }
    
    public long daysUntilMaturity() {
        return java.time.temporal.ChronoUnit.DAYS.between(LocalDate.now(), maturityDate);
    }
    
    // Getters
    public double getInterestRate() { return interestRate; }
    public int getTermInYears() { return termInYears; }
    public LocalDate getMaturityDate() { return maturityDate; }
    public String getBondType() { return bondType; }
}
```

### Abstract Class Usage Example

```java
// File: com/company/banking/services/InvestmentPortfolioService.java
package com.company.banking.services;

import com.company.banking.models.Investment;
import com.company.banking.models.StockInvestment;
import com.company.banking.models.BondInvestment;
import java.util.ArrayList;
import java.util.List;

public class InvestmentPortfolioService {
    private List<Investment> portfolio;
    
    public InvestmentPortfolioService() {
        this.portfolio = new ArrayList<>();
    }
    
    public void addInvestment(Investment investment) {
        portfolio.add(investment);
    }
    
    public double calculateTotalPortfolioValue() {
        double total = 0.0;
        for (Investment investment : portfolio) {
            total += investment.calculateCurrentValue(); // Polymorphic call
        }
        return total;
    }
    
    public double calculateAverageReturn() {
        if (portfolio.isEmpty()) return 0.0;
        
        double totalReturn = 0.0;
        for (Investment investment : portfolio) {
            totalReturn += investment.calculateYearlyReturn(); // Polymorphic call
        }
        return totalReturn / portfolio.size();
    }
    
    public void generatePortfolioReport() {
        System.out.println("=== PORTFOLIO REPORT ===");
        System.out.printf("Total Value: $%.2f%n", calculateTotalPortfolioValue());
        System.out.printf("Average Return: %.2f%%%n", calculateAverageReturn());
        System.out.println("\nIndividual Investments:");
        
        for (Investment investment : portfolio) {
            System.out.println(investment.generateInvestmentReport());
            System.out.println("---");
        }
    }
}
```

---

## Interfaces

Interfaces define contracts that classes must follow. They support decoupling by defining what a class must do without specifying how it should do it. Classes can implement multiple interfaces.

### Interface Syntax

```java
public interface InterfaceName {
    // Constants (implicitly public, static, final)
    int CONSTANT_VALUE = 100;
    
    // Abstract methods (implicitly public and abstract)
    void abstractMethod();
    
    // Default methods (Java 8+)
    default void defaultMethod() {
        // Implementation
    }
    
    // Static methods (Java 8+)
    static void staticMethod() {
        // Implementation
    }
}
```

### Comprehensive Interface Examples

```java
// File: com/company/banking/interfaces/Transferable.java
package com.company.banking.interfaces;

/**
 * Interface for accounts that support transfers
 */
public interface Transferable {
    
    // Constants
    double MAX_DAILY_TRANSFER = 10000.0;
    double MIN_TRANSFER_AMOUNT = 1.0;
    
    // Abstract methods
    boolean transferTo(Transferable destination, double amount);
    boolean receiveTransfer(double amount, String sourceAccount);
    String getTransferIdentifier();
    
    // Default method
    default boolean isValidTransferAmount(double amount) {
        return amount >= MIN_TRANSFER_AMOUNT && amount <= MAX_DAILY_TRANSFER;
    }
    
    // Static method
    static String generateTransferReference() {
        return "TXN" + System.currentTimeMillis();
    }
}
```

```java
// File: com/company/banking/interfaces/InterestEarning.java
package com.company.banking.interfaces;

/**
 * Interface for accounts that earn interest
 */
public interface InterestEarning {
    
    double getInterestRate();
    void setInterestRate(double rate);
    double calculateInterest();
    void applyInterest();
    
    // Default method with implementation
    default double calculateCompoundInterest(int periods) {
        double principal = getCurrentBalance();
        double rate = getInterestRate();
        return principal * Math.pow(1 + rate / periods, periods) - principal;
    }
    
    // Abstract method for getting current balance
    double getCurrentBalance();
}
```

```java
// File: com/company/banking/interfaces/Reportable.java
package com.company.banking.interfaces;

import java.time.LocalDate;
import java.util.List;

/**
 * Interface for objects that can generate reports
 */
public interface Reportable {
    
    String generateSummaryReport();
    String generateDetailedReport();
    List<String> getTransactionHistory();
    
    // Default method for date-range reporting
    default String generateDateRangeReport(LocalDate start, LocalDate end) {
        return "Report from " + start + " to " + end + ":\n" + generateDetailedReport();
    }
    
    // Default method for export functionality
    default String exportToCSV() {
        StringBuilder csv = new StringBuilder();
        csv.append("Date,Description,Amount\n");
        
        for (String transaction : getTransactionHistory()) {
            csv.append(transaction).append("\n");
        }
        
        return csv.toString();
    }
}
```

### Multiple Interface Implementation

```java
// File: com/company/banking/models/PremiumAccount.java
package com.company.banking.models;

import com.company.banking.interfaces.Transferable;
import com.company.banking.interfaces.InterestEarning;
import com.company.banking.interfaces.Reportable;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

/**
 * Premium account implementing multiple interfaces
 */
public class PremiumAccount extends Account 
    implements Transferable, InterestEarning, Reportable {
    
    private double interestRate;
    private double dailyTransferTotal;
    private LocalDate lastTransferReset;
    private List<String> detailedTransactions;
    
    public PremiumAccount(String accountNumber, String ownerName, double interestRate) {
        super(accountNumber, ownerName);
        this.interestRate = interestRate;
        this.dailyTransferTotal = 0.0;
        this.lastTransferReset = LocalDate.now();
        this.detailedTransactions = new ArrayList<>();
    }
    
    // Implementing Transferable interface
    @Override
    public boolean transferTo(Transferable destination, double amount) {
        if (!isValidTransferAmount(amount)) {
            return false;
        }
        
        // Check daily transfer limit
        resetDailyTransferIfNeeded();
        if (dailyTransferTotal + amount > MAX_DAILY_TRANSFER) {
            addTransaction("Transfer denied: Daily limit exceeded");
            return false;
        }
        
        if (withdraw(amount)) {
            String reference = Transferable.generateTransferReference();
            destination.receiveTransfer(amount, this.getTransferIdentifier());
            dailyTransferTotal += amount;
            
            String detail = String.format("%s,Transfer to %s,$%.2f,%s", 
                LocalDate.now(), destination.getTransferIdentifier(), amount, reference);
            detailedTransactions.add(detail);
            
            return true;
        }
        return false;
    }
    
    @Override
    public boolean receiveTransfer(double amount, String sourceAccount) {
        deposit(amount);
        String detail = String.format("%s,Transfer from %s,$%.2f", 
            LocalDate.now(), sourceAccount, amount);
        detailedTransactions.add(detail);
        return true;
    }
    
    @Override
    public String getTransferIdentifier() {
        return accountNumber;
    }
    
    // Implementing InterestEarning interface
    @Override
    public double getInterestRate() {
        return interestRate;
    }
    
    @Override
    public void setInterestRate(double rate) {
        if (rate >= 0 && rate <= 0.10) { // Max 10%
            this.interestRate = rate;
            addTransaction("Interest rate updated to " + (rate * 100) + "%");
        }
    }
    
    @Override
    public double calculateInterest() {
        return balance * (interestRate / 12); // Monthly interest
    }
    
    @Override
    public void applyInterest() {
        double interest = calculateInterest();
        balance += interest;
        String detail = String.format("%s,Interest Earned,$%.2f", 
            LocalDate.now(), interest);
        detailedTransactions.add(detail);
        addTransaction("Interest applied: $" + String.format("%.2f", interest));
    }
    
    @Override
    public double getCurrentBalance() {
        return balance;
    }
    
    // Implementing Reportable interface
    @Override
    public String generateSummaryReport() {
        return String.format(
            "Premium Account Summary\n" +
            "Account: %s\n" +
            "Owner: %s\n" +
            "Balance: $%.2f\n" +
            "Interest Rate: %.2f%%\n" +
            "Daily Transfers: $%.2f / $%.2f\n",
            accountNumber, ownerName, balance, 
            interestRate * 100, dailyTransferTotal, MAX_DAILY_TRANSFER
        );
    }
    
    @Override
    public String generateDetailedReport() {
        StringBuilder report = new StringBuilder();
        report.append(generateSummaryReport());
        report.append("\nDetailed Transactions:\n");
        
        for (String transaction : detailedTransactions) {
            report.append(transaction).append("\n");
        }
        
        return report.toString();
    }
    
    @Override
    public List<String> getTransactionHistory() {
        return new ArrayList<>(detailedTransactions);
    }
    
    // Helper methods
    private void resetDailyTransferIfNeeded() {
        LocalDate today = LocalDate.now();
        if (!today.equals(lastTransferReset)) {
            dailyTransferTotal = 0.0;
            lastTransferReset = today;
        }
    }
    
    // Additional premium features
    public void requestCreditLineIncrease(double amount) {
        addTransaction("Credit line increase requested: $" + amount);
        // Implementation would involve approval process
    }
    
    public double calculateProjectedInterest(int months) {
        return balance * interestRate * (months / 12.0);
    }
}
```

### Functional Interfaces (Java 8+)

```java
// File: com/company/banking/interfaces/TransactionProcessor.java
package com.company.banking.interfaces;

/**
 * Functional interface for processing transactions
 * Can be used with lambda expressions
 */
@FunctionalInterface
public interface TransactionProcessor {
    boolean processTransaction(double amount, String description);
    
    // Default methods are allowed in functional interfaces
    default String formatAmount(double amount) {
        return String.format("$%.2f", amount);
    }
}
```

```java
// Usage of functional interfaces with lambdas
public class FunctionalInterfaceExample {
    
    public void demonstrateLambdas() {
        PremiumAccount account = new PremiumAccount("PREM001", "VIP Customer", 0.03);
        
        // Lambda expression implementing TransactionProcessor
        TransactionProcessor depositProcessor = (amount, description) -> {
            if (amount > 0) {
                account.deposit(amount);
                return true;
            }
            return false;
        };
        
        TransactionProcessor withdrawalProcessor = (amount, description) -> {
            return account.withdraw(amount);
        };
        
        // Using the processors
        depositProcessor.processTransaction(1000.0, "Initial deposit");
        withdrawalProcessor.processTransaction(200.0, "ATM withdrawal");
        
        // Method reference
        TransactionProcessor interestProcessor = account::applyInterest;
    }
}
```

---

## Polymorphism

Polymorphism allows objects of different types to be treated as objects of a common base type, while still maintaining their specific behavior. It's the ability for an object to take on many forms.

### Types of Polymorphism

1. **Runtime Polymorphism (Method Overriding)**
2. **Compile-time Polymorphism (Method Overloading)**
3. **Interface Polymorphism**

### Runtime Polymorphism Example

```java
// File: com/company/banking/demo/PolymorphismDemo.java
package com.company.banking.demo;

import com.company.banking.models.*;
import com.company.banking.interfaces.*;
import java.util.ArrayList;
import java.util.List;

public class PolymorphismDemo {
    
    /**
     * Demonstrates polymorphism through inheritance
     */
    public static void demonstrateInheritancePolymorphism() {
        System.out.println("=== Inheritance Polymorphism ===");
        
        // Array of Account references pointing to different subclass objects
        Account[] accounts = {
            new CheckingAccount("CHK001", "John Doe", 500.0),
            new SavingsAccount("SAV001", "Jane Smith", 0.025),
            new PremiumAccount("PREM001", "Bob Johnson", 0.035)
        };
        
        // Polymorphic method calls
        for (Account account : accounts) {
            System.out.println("Processing: " + account.getClass().getSimpleName());
            
            // Each object responds according to its specific implementation
            account.deposit(1000.0);                    // Different implementations
            double fee = account.calculateMonthlyFee(); // Different calculations
            
            System.out.println("Monthly fee: $" + fee);
            System.out.println("Balance: $" + account.getBalance());
            System.out.println("---");
        }
    }
    
    /**
     * Demonstrates polymorphism through interfaces
     */
    public static void demonstrateInterfacePolymorphism() {
        System.out.println("=== Interface Polymorphism ===");
        
        // Different objects implementing the same interface
        List<Transferable> transferableAccounts = new ArrayList<>();
        transferableAccounts.add(new PremiumAccount("PREM002", "Alice", 0.04));
        transferableAccounts.add(new CheckingAccount("CHK002", "Charlie", 1000.0));
        
        // Polymorphic interface method calls
        for (Transferable account : transferableAccounts) {
            System.out.println("Transfer identifier: " + account.getTransferIdentifier());
            
            // Each implementation handles validation differently
            boolean valid = account.isValidTransferAmount(500.0);
            System.out.println("Can transfer $500: " + valid);
        }
        
        // Demonstrate transfer between different account types
        Transferable from = transferableAccounts.get(0);
        Transferable to = transferableAccounts.get(1);
        
        boolean success = from.transferTo(to, 250.0);
        System.out.println("Transfer successful: " + success);
    }
    
    /**
     * Demonstrates polymorphism with collections
     */
    public static void demonstrateCollectionPolymorphism() {
        System.out.println("=== Collection Polymorphism ===");
        
        List<Investment> portfolio = new ArrayList<>();
        portfolio.add(new StockInvestment("STK001", "Investor1", 5000.0, "AAPL", 50));
        portfolio.add(new BondInvestment("BND001", "Investor2", 10000.0, 0.04, 5, "Government"));
        portfolio.add(new StockInvestment("STK002", "Investor3", 3000.0, "GOOGL", 10));
        
        // Polymorphic processing of different investment types
        double totalValue = 0.0;
        double totalRisk = 0.0;
        
        for (Investment investment : portfolio) {
            // Each investment type calculates these differently
            double value = investment.calculateCurrentValue();
            double risk = investment.calculateRisk();
            double yearlyReturn = investment.calculateYearlyReturn();
            
            totalValue += value;
            totalRisk += risk;
            
            System.out.printf("%s: Value=$%.2f, Risk=%.1f%%, Return=%.2f%%%n",
                investment.getInvestmentType(), value, risk, yearlyReturn);
        }
        
        System.out.printf("Portfolio Total: $%.2f, Average Risk: %.1f%%%n", 
            totalValue, totalRisk / portfolio.size());
    }
    
    /**
     * Demonstrates method overloading (compile-time polymorphism)
     */
    public static class OverloadingExample {
        
        // Same method name, different parameters
        public void processTransaction(double amount) {
            System.out.println("Processing amount: $" + amount);
        }
        
        public void processTransaction(double amount, String description) {
            System.out.println("Processing $" + amount + " for: " + description);
        }
        
        public void processTransaction(Account account, double amount) {
            account.deposit(amount);
            System.out.println("Deposited $" + amount + " to " + account.getAccountNumber());
        }
        
        public void processTransaction(Account from, Account to, double amount) {
            if (from.withdraw(amount)) {
                to.deposit(amount);
                System.out.println("Transferred $" + amount + " from " + 
                    from.getAccountNumber() + " to " + to.getAccountNumber());
            }
        }
    }
    
    /**
     * Demonstrates instanceof and type checking
     */
    public static void demonstrateTypeChecking() {
        System.out.println("=== Type Checking ===");
        
        Account[] accounts = {
            new CheckingAccount("CHK003", "Test1", 500.0),
            new SavingsAccount("SAV003", "Test2", 0.03),
            new PremiumAccount("PREM003", "Test3", 0.04)
        };
        
        for (Account account : accounts) {
            System.out.println("Account type: " + account.getClass().getSimpleName());
            
            // Type checking and casting
            if (account instanceof InterestEarning) {
                InterestEarning interestAccount = (InterestEarning) account;
                double interest = interestAccount.calculateInterest();
                System.out.println("Monthly interest: $" + String.format("%.2f", interest));
            }
            
            if (account instanceof Transferable) {
                Transferable transferable = (Transferable) account;
                System.out.println("Can transfer: Yes, ID: " + transferable.getTransferIdentifier());
            } else {
                System.out.println("Can transfer: No");
            }
            
            // Specific behavior for each type
            if (account instanceof CheckingAccount) {
                CheckingAccount checking = (CheckingAccount) account;
                checking.writeCheck(100.0, "Utility Bill");
            } else if (account instanceof SavingsAccount) {
                SavingsAccount savings = (SavingsAccount) account;
                savings.applyMonthlyInterest();
            }
            
            System.out.println("---");
        }
    }
}
```

### Polymorphism Best Practices

```java
public class PolymorphismBestPractices {
    
    /**
     * Good: Using polymorphism to avoid type checking
     */
    public void processAccountsPolymorphically(List<Account> accounts) {
        for (Account account : accounts) {
            // Let each account handle its own fee calculation
            double fee = account.calculateMonthlyFee();
            
            // Apply fee if necessary
            if (fee > 0) {
                account.withdraw(fee);
            }
        }
    }
    
    /**
     * Avoid: Excessive type checking defeats polymorphism
     */
    public void processAccountsWithTypeChecking(List<Account> accounts) {
        for (Account account : accounts) {
            if (account instanceof CheckingAccount) {
                // Specific logic for checking accounts
                CheckingAccount checking = (CheckingAccount) account;
                double fee = checking.getTransactionCount() > 10 ? 5.0 : 0.0;
                checking.withdraw(fee);
            } else if (account instanceof SavingsAccount) {
                // Specific logic for savings accounts
                SavingsAccount savings = (SavingsAccount) account;
                double fee = savings.getBalance() < 100 ? 2.0 : 0.0;
                savings.withdraw(fee);
            }
            // This approach is harder to maintain and extend
        }
    }
    
    /**
     * Good: Factory pattern with polymorphism
     */
    public static class AccountFactory {
        public static Account createAccount(String type, String accountNumber, 
                                          String ownerName, double... params) {
            switch (type.toUpperCase()) {
                case "CHECKING":
                    double overdraftLimit = params.length > 0 ? params[0] : 500.0;
                    return new CheckingAccount(accountNumber, ownerName, overdraftLimit);
                    
                case "SAVINGS":
                    double interestRate = params.length > 0 ? params[0] : 0.02;
                    return new SavingsAccount(accountNumber, ownerName, interestRate);
                    
                case "PREMIUM":
                    double premiumRate = params.length > 0 ? params[0] : 0.035;
                    return new PremiumAccount(accountNumber, ownerName, premiumRate);
                    
                default:
                    throw new IllegalArgumentException("Unknown account type: " + type);
            }
        }
    }
    
    /**
     * Using strategy pattern with interfaces
     */
    public interface FeeCalculationStrategy {
        double calculateFee(Account account);
    }
    
    public static class BasicFeeStrategy implements FeeCalculationStrategy {
        @Override
        public double calculateFee(Account account) {
            return account.getBalance() < 500 ? 10.0 : 0.0;
        }
    }
    
    public static class PremiumFeeStrategy implements FeeCalculationStrategy {
        @Override
        public double calculateFee(Account account) {
            return 0.0; // No fees for premium accounts
        }
    }
    
    public void applyFeesWithStrategy(List<Account> accounts, FeeCalculationStrategy strategy) {
        for (Account account : accounts) {
            double fee = strategy.calculateFee(account);
            if (fee > 0) {
                account.withdraw(fee);
            }
        }
    }
}
```

---

## Advanced OOP Patterns

### Template Method Pattern (using Abstract Classes)

```java
public abstract class LoanProcessor {
    
    // Template method - defines the algorithm structure
    public final boolean processLoan(double amount, String applicantName) {
        if (!validateInput(amount, applicantName)) {
            return false;
        }
        
        double creditScore = checkCreditScore(applicantName);
        if (!isEligible(creditScore, amount)) {
            return false;
        }
        
        double interestRate = calculateInterestRate(creditScore, amount);
        createLoanDocument(amount, interestRate, applicantName);
        
        return approveLoan();
    }
    
    // Concrete methods
    private boolean validateInput(double amount, String applicantName) {
        return amount > 0 && applicantName != null && !applicantName.trim().isEmpty();
    }
    
    // Abstract methods - subclasses must implement
    protected abstract double checkCreditScore(String applicantName);
    protected abstract boolean isEligible(double creditScore, double amount);
    protected abstract double calculateInterestRate(double creditScore, double amount);
    protected abstract void createLoanDocument(double amount, double rate, String applicant);
    protected abstract boolean approveLoan();
}

public class MortgageLoanProcessor extends LoanProcessor {
    
    @Override
    protected double checkCreditScore(String applicantName) {
        // Mortgage-specific credit check
        return 750.0; // Simplified
    }
    
    @Override
    protected boolean isEligible(double creditScore, double amount) {
        return creditScore >= 650 && amount <= 500000;
    }
    
    @Override
    protected double calculateInterestRate(double creditScore, double amount) {
        if (creditScore >= 750) return 0.035;
        if (creditScore >= 700) return 0.045;
        return 0.055;
    }
    
    @Override
    protected void createLoanDocument(double amount, double rate, String applicant) {
        System.out.println("Creating mortgage document for " + applicant);
    }
    
    @Override
    protected boolean approveLoan() {
        return true; // Simplified approval
    }
}
```

---

## Best Practices

### Package Organization

```java
// Typical enterprise package structure
com.company.banking.
├── models/           // Domain objects
│   ├── Account.java
│   ├── Customer.java
│   └── Transaction.java
├── interfaces/       // Contracts
│   ├── Transferable.java
│   └── InterestEarning.java
├── services/         // Business logic
│   ├── AccountService.java
│   └── TransactionService.java
├── repositories/     // Data access
│   └── AccountRepository.java
├── controllers/      // Web/API layer
│   └── AccountController.java
├── utils/           // Utilities
│   └── ValidationUtils.java
└── exceptions/      // Custom exceptions
    └── InsufficientFundsException.java
```

### Design Principles

**1. Favor Composition over Inheritance**
```java
// Good: Composition
public class Account {
    private TransactionHistory history; // Has-a relationship
    private InterestCalculator calculator; // Has-a relationship
}

// Avoid: Deep inheritance hierarchies
public class SavingsAccount extends Account extends BankProduct extends FinancialInstrument { }
```

**2. Program to Interfaces, not Implementations**
```java
// Good
List<Account> accounts = new ArrayList<>();
Transferable transferService = new WireTransferService();

// Avoid
ArrayList<Account> accounts = new ArrayList<>();
WireTransferService transferService = new WireTransferService();
```

**3. Use Abstract Classes for Shared Implementation**
```java
// When you have common code and want to force implementation of certain methods
public abstract class Investment {
    // Common implementation
    protected void logTransaction(String transaction) { }
    
    // Force subclasses to implement
    public abstract double calculateReturn();
}
```

**4. Use Interfaces for Contracts**
```java
// When you want to define what classes must do, not how
public interface Auditable {
    List<String> getAuditTrail();
    void addAuditEntry(String entry);
}
```

---

## Summary

This manual covers the essential advanced OOP concepts that form the backbone of enterprise Java development:

### Key Takeaways

1. **Packages** - Organize code logically and control access
2. **Inheritance** - Share code and create specialized versions of classes
3. **Abstract Classes** - Provide partial implementations with enforced contracts
4. **Interfaces** - Define contracts for decoupling and multiple inheritance
5. **Polymorphism** - Write flexible code that works with multiple types

### Comparing to Python

- **Packages** - Similar to Python modules, but more structured
- **Inheritance** - Java requires explicit `extends`, Python uses parentheses
- **Abstract Classes** - Java enforces with `abstract` keyword, Python uses conventions
- **Interfaces** - Java has true interfaces, Python uses duck typing and protocols
- **Polymorphism** - Java compile-time checking vs Python runtime flexibility

### Spring Boot Connection

These concepts are crucial for Spring Boot development:
- **Dependency Injection** relies heavily on interfaces
- **Component scanning** uses package organization
- **Service layers** often use abstract base classes
- **Repository patterns** leverage interface polymorphism

As a tech lead, these patterns will help you design maintainable, scalable applications and guide your team toward clean architecture principles.

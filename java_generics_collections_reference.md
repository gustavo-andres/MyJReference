# Java Generics and Collections Reference Manual

## Table of Contents
1. [Generics Fundamentals](#generics-fundamentals)
2. [Generic Classes and Interfaces](#generic-classes-and-interfaces)
3. [Generic Methods](#generic-methods)
4. [Wildcards and Bounded Types](#wildcards-and-bounded-types)
5. [Collections Framework](#collections-framework)
6. [List Collections](#list-collections)
7. [Set Collections](#set-collections)
8. [Map Collections](#map-collections)
9. [Queue Collections](#queue-collections)
10. [Collection Algorithms](#collection-algorithms)
11. [Performance and Best Practices](#performance-and-best-practices)

---

## Generics Fundamentals

Generics provide type safety by allowing you to parameterize types. They eliminate the need for casting, provide stronger compile-time type checking, and enable you to write more generic and reusable code.

### Before Generics (Raw Types)

```java
// Pre-generics approach (Java 1.4 and earlier)
public class PreGenericsExample {
    
    public void demonstrateProblems() {
        // Raw List - no type safety
        List accounts = new ArrayList();
        
        accounts.add(new CheckingAccount("CHK001", "John", 500.0));
        accounts.add(new SavingsAccount("SAV001", "Jane", 0.025));
        accounts.add("Invalid String"); // Compiler doesn't catch this!
        
        // Requires casting and can fail at runtime
        for (Object obj : accounts) {
            try {
                Account account = (Account) obj; // ClassCastException possible!
                System.out.println(account.getBalance());
            } catch (ClassCastException e) {
                System.err.println("Invalid object in list: " + obj);
            }
        }
    }
}
```

### With Generics (Type Safe)

```java
// Modern approach with generics
public class GenericsExample {
    
    public void demonstrateTypeSafety() {
        // Type-safe List - compiler enforces type
        List<Account> accounts = new ArrayList<>();
        
        accounts.add(new CheckingAccount("CHK001", "John", 500.0));
        accounts.add(new SavingsAccount("SAV001", "Jane", 0.025));
        // accounts.add("Invalid String"); // Compiler error!
        
        // No casting required, type safety guaranteed
        for (Account account : accounts) {
            System.out.println(account.getBalance()); // No casting needed
        }
    }
}
```

### Generic Type Syntax

```java
// Generic type parameter naming conventions
public class GenericNamingConventions {
    
    // T = Type (most common)
    // E = Element (used by collections)
    // K = Key (maps)
    // V = Value (maps)
    // N = Number
    // ? = Wildcard
    
    // Examples:
    List<Account> accounts;              // T = Account
    Map<String, Account> accountMap;     // K = String, V = Account
    Set<TransactionType> types;          // E = TransactionType
    Queue<Transaction> pending;          // E = Transaction
}
```

---

## Generic Classes and Interfaces

Generic classes and interfaces allow you to define classes that work with different types while maintaining type safety.

### Generic Class Definition

```java
// File: com/company/banking/generics/Repository.java
package com.company.banking.generics;

import java.util.*;
import java.util.function.Predicate;

/**
 * Generic repository for managing entities of type T
 * @param <T> the type of entities this repository manages
 * @param <ID> the type of the entity's identifier
 */
public class Repository<T, ID> {
    
    private final Map<ID, T> entities;
    private final Class<T> entityType;
    
    public Repository(Class<T> entityType) {
        this.entities = new HashMap<>();
        this.entityType = entityType;
    }
    
    // Generic methods using type parameters
    public void save(ID id, T entity) {
        if (id == null || entity == null) {
            throw new IllegalArgumentException("ID and entity cannot be null");
        }
        entities.put(id, entity);
    }
    
    public Optional<T> findById(ID id) {
        return Optional.ofNullable(entities.get(id));
    }
    
    public List<T> findAll() {
        return new ArrayList<>(entities.values());
    }
    
    public boolean deleteById(ID id) {
        return entities.remove(id) != null;
    }
    
    public List<T> findByPredicate(Predicate<T> predicate) {
        return entities.values().stream()
                .filter(predicate)
                .collect(ArrayList::new, 
                        (list, item) -> list.add(item), 
                        (list1, list2) -> list1.addAll(list2));
    }
    
    public int size() {
        return entities.size();
    }
    
    public boolean isEmpty() {
        return entities.isEmpty();
    }
    
    public Class<T> getEntityType() {
        return entityType;
    }
    
    // Bulk operations
    public void saveAll(Map<ID, T> entityMap) {
        entities.putAll(entityMap);
    }
    
    public void clear() {
        entities.clear();
    }
    
    public Set<ID> getAllIds() {
        return new HashSet<>(entities.keySet());
    }
}
```

### Specialized Repository Implementations

```java
// File: com/company/banking/repositories/AccountRepository.java
package com.company.banking.repositories;

import com.company.banking.models.Account;
import com.company.banking.generics.Repository;
import java.util.List;
import java.util.stream.Collectors;

/**
 * Specialized repository for Account entities
 */
public class AccountRepository extends Repository<Account, String> {
    
    public AccountRepository() {
        super(Account.class);
    }
    
    // Domain-specific methods
    public List<Account> findByOwnerName(String ownerName) {
        return findByPredicate(account -> 
            account.getOwnerName().equalsIgnoreCase(ownerName));
    }
    
    public List<Account> findByMinimumBalance(double minimumBalance) {
        return findByPredicate(account -> 
            account.getBalance() >= minimumBalance);
    }
    
    public List<Account> findActiveAccounts() {
        return findByPredicate(Account::isActive);
    }
    
    public double getTotalBalance() {
        return findAll().stream()
                .mapToDouble(Account::getBalance)
                .sum();
    }
    
    public List<Account> findByAccountType(Class<? extends Account> accountType) {
        return findByPredicate(account -> 
            accountType.isInstance(account));
    }
}

// File: com/company/banking/repositories/TransactionRepository.java
package com.company.banking.repositories;

import com.company.banking.models.Transaction;
import com.company.banking.enums.TransactionType;
import com.company.banking.generics.Repository;
import java.time.LocalDateTime;
import java.util.List;

/**
 * Specialized repository for Transaction entities
 */
public class TransactionRepository extends Repository<Transaction, String> {
    
    public TransactionRepository() {
        super(Transaction.class);
    }
    
    public List<Transaction> findByAccountNumber(String accountNumber) {
        return findByPredicate(transaction -> 
            transaction.getAccountNumber().equals(accountNumber));
    }
    
    public List<Transaction> findByType(TransactionType type) {
        return findByPredicate(transaction -> 
            transaction.getType() == type);
    }
    
    public List<Transaction> findByDateRange(LocalDateTime start, LocalDateTime end) {
        return findByPredicate(transaction -> {
            LocalDateTime txTime = transaction.getTimestamp();
            return !txTime.isBefore(start) && !txTime.isAfter(end);
        });
    }
    
    public List<Transaction> findByAmountRange(double minAmount, double maxAmount) {
        return findByPredicate(transaction -> {
            double amount = transaction.getAmount();
            return amount >= minAmount && amount <= maxAmount;
        });
    }
}
```

### Generic Interfaces

```java
// File: com/company/banking/interfaces/Cacheable.java
package com.company.banking.interfaces;

import java.util.Optional;

/**
 * Generic interface for cacheable operations
 * @param <K> the type of cache keys
 * @param <V> the type of cached values
 */
public interface Cacheable<K, V> {
    
    void put(K key, V value);
    Optional<V> get(K key);
    void remove(K key);
    void clear();
    int size();
    boolean containsKey(K key);
    
    // Default methods with generic implementation
    default boolean isEmpty() {
        return size() == 0;
    }
    
    default V getOrDefault(K key, V defaultValue) {
        return get(key).orElse(defaultValue);
    }
}

// File: com/company/banking/interfaces/Auditable.java
package com.company.banking.interfaces;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Generic interface for auditable entities
 * @param <T> the type of audit event
 */
public interface Auditable<T> {
    
    void addAuditEvent(T event);
    List<T> getAuditTrail();
    LocalDateTime getLastModified();
    String getLastModifiedBy();
    
    default boolean hasAuditTrail() {
        return !getAuditTrail().isEmpty();
    }
}
```

### Complex Generic Class Example

```java
// File: com/company/banking/services/GenericTransactionProcessor.java
package com.company.banking.services;

import com.company.banking.interfaces.Cacheable;
import com.company.banking.interfaces.Auditable;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.time.LocalDateTime;
import java.util.function.Function;
import java.util.function.Consumer;

/**
 * Generic transaction processor with caching and auditing
 * @param <T> transaction type
 * @param <R> result type
 * @param <A> audit event type
 */
public class GenericTransactionProcessor<T, R, A> 
    implements Cacheable<String, R>, Auditable<A> {
    
    private final Map<String, R> cache;
    private final List<A> auditTrail;
    private final Function<T, R> processor;
    private final Function<T, String> keyGenerator;
    private final Consumer<A> auditLogger;
    private LocalDateTime lastModified;
    private String lastModifiedBy;
    
    public GenericTransactionProcessor(Function<T, R> processor,
                                     Function<T, String> keyGenerator,
                                     Consumer<A> auditLogger) {
        this.cache = new ConcurrentHashMap<>();
        this.auditTrail = new ArrayList<>();
        this.processor = processor;
        this.keyGenerator = keyGenerator;
        this.auditLogger = auditLogger;
        this.lastModified = LocalDateTime.now();
        this.lastModifiedBy = "SYSTEM";
    }
    
    // Main processing method
    public R processTransaction(T transaction, String userId) {
        String key = keyGenerator.apply(transaction);
        
        // Check cache first
        Optional<R> cached = get(key);
        if (cached.isPresent()) {
            return cached.get();
        }
        
        // Process transaction
        R result = processor.apply(transaction);
        
        // Cache result
        put(key, result);
        
        // Update audit info
        lastModified = LocalDateTime.now();
        lastModifiedBy = userId;
        
        return result;
    }
    
    // Implement Cacheable interface
    @Override
    public void put(String key, R value) {
        cache.put(key, value);
    }
    
    @Override
    public Optional<R> get(String key) {
        return Optional.ofNullable(cache.get(key));
    }
    
    @Override
    public void remove(String key) {
        cache.remove(key);
    }
    
    @Override
    public void clear() {
        cache.clear();
        lastModified = LocalDateTime.now();
    }
    
    @Override
    public int size() {
        return cache.size();
    }
    
    @Override
    public boolean containsKey(String key) {
        return cache.containsKey(key);
    }
    
    // Implement Auditable interface
    @Override
    public void addAuditEvent(A event) {
        auditTrail.add(event);
        auditLogger.accept(event);
        lastModified = LocalDateTime.now();
    }
    
    @Override
    public List<A> getAuditTrail() {
        return new ArrayList<>(auditTrail);
    }
    
    @Override
    public LocalDateTime getLastModified() {
        return lastModified;
    }
    
    @Override
    public String getLastModifiedBy() {
        return lastModifiedBy;
    }
    
    // Additional generic utility methods
    public Map<String, R> getCacheSnapshot() {
        return new HashMap<>(cache);
    }
    
    public void processMultiple(List<T> transactions, String userId) {
        transactions.forEach(transaction -> processTransaction(transaction, userId));
    }
}
```

---

## Generic Methods

Generic methods allow you to parameterize individual methods with type parameters, providing flexibility within non-generic classes.

### Basic Generic Methods

```java
// File: com/company/banking/utils/GenericUtils.java
package com.company.banking.utils;

import java.util.*;
import java.util.function.Function;
import java.util.function.Predicate;

public class GenericUtils {
    
    /**
     * Generic method to swap two elements in a list
     * @param <T> the type of elements in the list
     * @param list the list containing elements to swap
     * @param i first index
     * @param j second index
     */
    public static <T> void swap(List<T> list, int i, int j) {
        if (i < 0 || i >= list.size() || j < 0 || j >= list.size()) {
            throw new IndexOutOfBoundsException("Invalid indices for swap operation");
        }
        
        T temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }
    
    /**
     * Generic method to find the maximum element
     * @param <T> the type of elements (must be Comparable)
     * @param list the list to search
     * @return the maximum element
     */
    public static <T extends Comparable<T>> T findMax(List<T> list) {
        if (list == null || list.isEmpty()) {
            throw new IllegalArgumentException("List cannot be null or empty");
        }
        
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }
    
    /**
     * Generic method to filter a collection
     * @param <T> the type of elements
     * @param collection the source collection
     * @param predicate the filter condition
     * @return filtered list
     */
    public static <T> List<T> filter(Collection<T> collection, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T element : collection) {
            if (predicate.test(element)) {
                result.add(element);
            }
        }
        return result;
    }
    
    /**
     * Generic method to transform a collection
     * @param <T> the input type
     * @param <R> the output type
     * @param collection the source collection
     * @param mapper the transformation function
     * @return transformed list
     */
    public static <T, R> List<R> map(Collection<T> collection, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T element : collection) {
            result.add(mapper.apply(element));
        }
        return result;
    }
    
    /**
     * Generic method to partition a list
     * @param <T> the type of elements
     * @param list the source list
     * @param predicate the partitioning condition
     * @return map with "true" and "false" keys containing matching and non-matching elements
     */
    public static <T> Map<Boolean, List<T>> partition(List<T> list, Predicate<T> predicate) {
        Map<Boolean, List<T>> result = new HashMap<>();
        result.put(true, new ArrayList<>());
        result.put(false, new ArrayList<>());
        
        for (T element : list) {
            result.get(predicate.test(element)).add(element);
        }
        
        return result;
    }
    
    /**
     * Generic method to create a safe copy of a collection
     * @param <T> the type of elements
     * @param source the source collection
     * @return defensive copy
     */
    public static <T> List<T> defensiveCopy(Collection<T> source) {
        if (source == null) {
            return new ArrayList<>();
        }
        return new ArrayList<>(source);
    }
    
    /**
     * Generic method to combine multiple collections
     * @param <T> the type of elements
     * @param collections variable number of collections
     * @return combined list
     */
    @SafeVarargs
    public static <T> List<T> combine(Collection<T>... collections) {
        List<T> result = new ArrayList<>();
        for (Collection<T> collection : collections) {
            if (collection != null) {
                result.addAll(collection);
            }
        }
        return result;
    }
}
```

### Banking-Specific Generic Methods

```java
// File: com/company/banking/services/AccountAnalysisService.java
package com.company.banking.services;

import com.company.banking.models.*;
import com.company.banking.utils.GenericUtils;
import java.util.*;
import java.util.function.Function;
import java.util.function.ToDoubleFunction;

public class AccountAnalysisService {
    
    /**
     * Generic method to calculate statistics for any numeric property
     * @param <T> the type of objects being analyzed
     * @param items the collection of items
     * @param valueExtractor function to extract numeric value
     * @return statistics object
     */
    public static <T> Statistics calculateStatistics(Collection<T> items, 
                                                   ToDoubleFunction<T> valueExtractor) {
        if (items == null || items.isEmpty()) {
            return new Statistics(0, 0, 0, 0, 0);
        }
        
        DoubleSummaryStatistics stats = items.stream()
                .mapToDouble(valueExtractor)
                .summaryStatistics();
        
        return new Statistics(
            stats.getCount(),
            stats.getSum(),
            stats.getAverage(),
            stats.getMin(),
            stats.getMax()
        );
    }
    
    /**
     * Generic grouping method
     * @param <T> the type of items
     * @param <K> the type of grouping key
     * @param items the items to group
     * @param keyExtractor function to extract grouping key
     * @return map of grouped items
     */
    public static <T, K> Map<K, List<T>> groupBy(Collection<T> items, 
                                               Function<T, K> keyExtractor) {
        Map<K, List<T>> groups = new HashMap<>();
        
        for (T item : items) {
            K key = keyExtractor.apply(item);
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(item);
        }
        
        return groups;
    }
    
    /**
     * Generic method to find top N items by some criteria
     * @param <T> the type of items
     * @param items the collection to search
     * @param comparator the comparison criteria
     * @param n number of top items to return
     * @return list of top N items
     */
    public static <T> List<T> findTopN(Collection<T> items, 
                                     Comparator<T> comparator, 
                                     int n) {
        if (items == null || items.isEmpty() || n <= 0) {
            return new ArrayList<>();
        }
        
        return items.stream()
                .sorted(comparator.reversed())
                .limit(n)
                .collect(ArrayList::new, 
                        (list, item) -> list.add(item), 
                        (list1, list2) -> list1.addAll(list2));
    }
    
    // Usage examples with accounts
    public void demonstrateGenericMethods() {
        List<Account> accounts = Arrays.asList(
            new CheckingAccount("CHK001", "John", 1500.0),
            new SavingsAccount("SAV001", "Jane", 0.025),
            new CheckingAccount("CHK002", "Bob", 750.0)
        );
        
        // Calculate balance statistics
        Statistics balanceStats = calculateStatistics(accounts, Account::getBalance);
        System.out.printf("Balance Stats - Count: %d, Average: $%.2f, Max: $%.2f%n",
                         balanceStats.count, balanceStats.average, balanceStats.max);
        
        // Group accounts by type
        Map<Class<?>, List<Account>> accountsByType = groupBy(accounts, 
            account -> account.getClass());
        
        // Find top 3 accounts by balance
        List<Account> topAccounts = findTopN(accounts, 
            Comparator.comparing(Account::getBalance), 3);
        
        // Filter accounts with high balance
        List<Account> highBalanceAccounts = GenericUtils.filter(accounts,
            account -> account.getBalance() > 1000.0);
        
        // Transform accounts to account numbers
        List<String> accountNumbers = GenericUtils.map(accounts, 
            Account::getAccountNumber);
    }
    
    // Supporting class for statistics
    public static class Statistics {
        public final long count;
        public final double sum;
        public final double average;
        public final double min;
        public final double max;
        
        public Statistics(long count, double sum, double average, double min, double max) {
            this.count = count;
            this.sum = sum;
            this.average = average;
            this.min = min;
            this.max = max;
        }
        
        @Override
        public String toString() {
            return String.format("Statistics{count=%d, sum=%.2f, avg=%.2f, min=%.2f, max=%.2f}",
                               count, sum, average, min, max);
        }
    }
}
```

---

## Wildcards and Bounded Types

Wildcards provide flexibility when working with generic types, allowing you to accept or return types that are related through inheritance.

### Wildcard Types

```java
// File: com/company/banking/services/CollectionService.java
package com.company.banking.services;

import com.company.banking.models.*;
import java.util.*;

public class CollectionService {
    
    /**
     * Upper bounded wildcard (? extends T)
     * Use when you only need to READ from the collection
     */
    
    // Can accept List<CheckingAccount>, List<SavingsAccount>, List<Account>, etc.
    public static double calculateTotalBalance(List<? extends Account> accounts) {
        double total = 0.0;
        for (Account account : accounts) {  // Can read as Account
            total += account.getBalance();
        }
        return total;
        // accounts.add(new CheckingAccount(...)); // Compile error! Can't add
    }
    
    /**
     * Lower bounded wildcard (? super T)
     * Use when you need to WRITE to the collection
     */
    
    // Can accept List<Account>, List<Object>, but not List<CheckingAccount>
    public static void addCheckingAccounts(List<? super CheckingAccount> accounts) {
        // Can add CheckingAccount or any subtype
        accounts.add(new CheckingAccount("CHK001", "John", 500.0));
        accounts.add(new PremiumCheckingAccount("PRM001", "Jane", 1000.0, 0.02));
        
        // Object obj = accounts.get(0); // Can only read as Object
    }
    
    /**
     * Unbounded wildcard (?)
     * Use when you don't care about the type or only use Object methods
     */
    public static int getSize(List<?> list) {
        return list.size(); // Only Object methods available
    }
    
    public static void printAll(List<?> list) {
        for (Object item : list) {
            System.out.println(item.toString()); // Only Object methods
        }
    }
}
```

### Bounded Type Parameters

```java
// File: com/company/banking/comparisons/GenericComparator.java
package com.company.banking.comparisons;

import java.util.*;

/**
 * Generic class with bounded type parameters
 */
public class GenericComparator {
    
    /**
     * Method with bounded type parameter
     * T must implement Comparable<T>
     */
    public static <T extends Comparable<T>> T findMaximum(T a, T b, T c) {
        T max = a;
        if (b.compareTo(max) > 0) {
            max = b;
        }
        if (c.compareTo(max) > 0) {
            max = c;
        }
        return max;
    }
    
    /**
     * Method with multiple bounds
     * T must extend Account AND implement Comparable<T>
     */
    public static <T extends Account & Comparable<T>> void sortAccounts(List<T> accounts) {
        Collections.sort(accounts); // Works because T implements Comparable
    }
    
    /**
     * Method with bounded wildcard for processing numeric values
     */
    public static double sumNumbers(List<? extends Number> numbers) {
        double sum = 0.0;
        for (Number number : numbers) {
            sum += number.doubleValue();
        }
        return sum;
    }
    
    /**
     * Complex generic method with multiple type parameters and bounds
     */
    public static <K extends Comparable<K>, V extends Number> 
           Map<K, Double> convertValues(Map<K, V> source) {
        
        Map<K, Double> result = new TreeMap<>(); // TreeMap requires Comparable keys
        for (Map.Entry<K, V> entry : source.entrySet()) {
            result.put(entry.getKey(), entry.getValue().doubleValue());
        }
        return result;
    }
}
```

### Advanced Wildcard Examples

```java
// File: com/company/banking/processors/TransactionProcessor.java
package com.company.banking.processors;

import com.company.banking.models.*;
import java.util.*;
import java.util.function.Function;

public class TransactionProcessor {
    
    /**
     * PECS Principle demonstration
     * Producer Extends, Consumer Super
     */
    
    // Producer - extends wildcard (reading from source)
    public static <T> List<T> copyFrom(Collection<? extends T> source) {
        List<T> result = new ArrayList<>();
        for (T item : source) { // Can read as T
            result.add(item);
        }
        return result;
    }
    
    // Consumer - super wildcard (writing to destination)
    public static <T> void copyTo(Collection<T> source, Collection<? super T> destination) {
        for (T item : source) {
            destination.add(item); // Can write T to destination
        }
    }
    
    /**
     * Complex example: Generic transformation with wildcards
     */
    public static <T, R> void transformAndAdd(
            Collection<? extends T> source,
            Function<? super T, ? extends R> transformer,
            Collection<? super R> destination) {
        
        for (T item : source) {
            R transformed = transformer.apply(item);
            destination.add(transformed);
        }
    }
    
    /**
     * Real banking example: Process different account types
     */
    public void processAccountBalances() {
        // Different specific account lists
        List<CheckingAccount> checkingAccounts = Arrays.asList(
            new CheckingAccount("CHK001", "John", 500.0),
            new CheckingAccount("CHK002", "Jane", 750.0)
        );
        
        List<SavingsAccount> savingsAccounts = Arrays.asList(
            new SavingsAccount("SAV001", "Bob", 0.025),
            new SavingsAccount("SAV002", "Alice", 0.03)
        );
        
        // Can calculate total for any type of account list
        double checkingTotal = CollectionService.calculateTotalBalance(checkingAccounts);
        double savingsTotal = CollectionService.calculateTotalBalance(savingsAccounts);
        
        // Combined processing
        List<Account> allAccounts = new ArrayList<>();
        CollectionService.copyTo(checkingAccounts, allAccounts); // CheckingAccount -> Account
        CollectionService.copyTo(savingsAccounts, allAccounts);  // SavingsAccount -> Account
        
        // Transform accounts to account numbers
        List<String> accountNumbers = new ArrayList<>();
        transformAndAdd(allAccounts, Account::getAccountNumber, accountNumbers);
    }
    
    /**
     * Wildcard capture helper method
     */
    public static void processUnknownList(List<?> list) {
        processListHelper(list);
    }
    
    private static <T> void processListHelper(List<T> list) {
        // Now we can work with T as a concrete type
        if (!list.isEmpty()) {
            T first = list.get(0);
            T last = list.get(list.size() - 1);
            // Can perform operations that require knowing the type
        }
    }
}
```

---

## Collections Framework

The Java Collections Framework provides a unified architecture for representing and manipulating collections of objects, offering improved performance and interoperability.

### Collection Hierarchy Overview

```java
/**
 * Collection Framework Hierarchy:
 * 
 * Collection<E>
 * ├── List<E>
 * │   ├── ArrayList<E>
 * │   ├── LinkedList<E>
 * │   └── Vector<E>
 * ├── Set<E>
 * │   ├── HashSet<E>
 * │   ├── LinkedHashSet<E>
 * │   └── TreeSet<E> (SortedSet)
 * └── Queue<E>
 *     ├── LinkedList<E>
 *     ├── PriorityQueue<E>
 *     └── ArrayDeque<E>
 * 
 * Map<K,V> (separate hierarchy)
 * ├── HashMap<K,V>
 * ├── LinkedHashMap<K,V>
 * └── TreeMap<K,V> (SortedMap)
 */
```

### Common Collection Operations

```java
// File: com/company/banking/examples/CollectionOperations.java
package com.company.banking.examples;

import com.company.banking.models.*;
import java.util.*;
import java.util.function.Predicate;

public class CollectionOperations {
    
    /**
     * Demonstrate common operations across all collection types
     */
    public void demonstrateBasicOperations() {
        
        // Creating collections
        List<Account> accountList = new ArrayList<>();
        Set<String> accountNumbers = new HashSet<>();
        Map<String, Account> accountMap = new HashMap<>();
        Queue<Transaction> pendingTransactions = new LinkedList<>();
        
        // Basic operations available on all collections
        Account account1 = new CheckingAccount("CHK001", "John", 500.0);
        Account account2 = new SavingsAccount("SAV001", "Jane", 0.025);
        
        // Adding elements
        accountList.add(account1);
        accountList.add(account2);
        
        accountNumbers.add(account1.getAccountNumber());
        accountNumbers.add(account2.getAccountNumber());
        
        accountMap.put(account1.getAccountNumber(), account1);
        accountMap.put(account2.getAccountNumber(), account2);
        
        // Checking size and emptiness
        System.out.println("List size: " + accountList.size());
        System.out.println("Is list empty: " + accountList.isEmpty());
        
        // Checking containment
        boolean hasAccount = accountList.contains(account1);
        boolean hasAccountNumber = accountNumbers.contains("CHK001");
        boolean hasKey = accountMap.containsKey("CHK001");
        boolean hasValue = accountMap.containsValue(account1);
        
        // Removing elements
        accountList.remove(account1);           // Remove by object
        accountNumbers.remove("CHK001");        // Remove by value
        accountMap.remove("CHK001");            // Remove by key
        
        // Clearing collections
        accountList.clear();
        accountNumbers.clear();
        accountMap.clear();
    }
    
    /**
     * Bulk operations
     */
    public void demonstrateBulkOperations() {
        List<Account> sourceAccounts = Arrays.asList(
            new CheckingAccount("CHK001", "John", 500.0),
            new SavingsAccount("SAV001", "Jane", 0.025),
            new CheckingAccount("CHK002", "Bob", 750.0)
        );
        
        List<Account> targetAccounts = new ArrayList<>();
        
        // Add all elements from one collection to another
        targetAccounts.addAll(sourceAccounts);
        
        // Check if collection contains all elements from another
        boolean containsAll = targetAccounts.containsAll(sourceAccounts);
        
        // Remove all elements that exist in another collection
        List<Account> toRemove = Arrays.asList(sourceAccounts.get(0));
        targetAccounts.removeAll(toRemove);
        
        // Retain only elements that exist in another collection
        List<Account> toRetain = Arrays.asList(sourceAccounts.get(1), sourceAccounts.get(2));
        targetAccounts.retainAll(toRetain);
        
        // Remove elements based on condition (Java 8+)
        targetAccounts.removeIf(account -> account.getBalance() < 100);
    }
    
    /**
     * Collection conversion and interoperability
     */
    public void demonstrateConversion() {
        // Array to Collection
        Account[] accountArray = {
            new CheckingAccount("CHK001", "John", 500.0),
            new SavingsAccount("SAV001", "Jane", 0.025)
        };
        
        List<Account> listFromArray = Arrays.asList(accountArray);
        List<Account> mutableList = new ArrayList<>(Arrays.asList(accountArray));
        
        // Collection to Array
        Account[] arrayFromList = mutableList.toArray(new Account[0]);
        
        // Between different collection types
        Set<Account> accountSet = new HashSet<>(mutableList);
        List<Account> listFromSet = new ArrayList<>(accountSet);
        
        // Using Collections utility methods
        List<Account> synchronizedList = Collections.synchronizedList(new ArrayList<>());
        List<Account> unmodifiableList = Collections.unmodifiableList(mutableList);
        List<Account> singletonList = Collections.singletonList(accountArray[0]);
    }
}
```

---

## List Collections

Lists are ordered collections that allow duplicate elements and provide indexed access to elements.

### ArrayList vs LinkedList vs Vector

```java
// File: com/company/banking/examples/ListComparison.java
package com.company.banking.examples;

import com.company.banking.models.*;
import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

public class ListComparison {
    
    /**
     * ArrayList - Dynamic array implementation
     * - Best for: Random access, frequent reading
     * - Worst for: Frequent insertions/deletions in middle
     */
    public void demonstrateArrayList() {
        List<Account> accounts = new ArrayList<>();
        
        // Efficient operations
        accounts.add(new CheckingAccount("CHK001", "John", 500.0));    // O(1) amortized
        Account account = accounts.get(0);                             // O(1) random access
        accounts.set(0, new SavingsAccount("SAV001", "Jane", 0.025)); // O(1) replacement
        
        // Less efficient operations
        accounts.add(0, new CheckingAccount("CHK000", "Bob", 300.0));  // O(n) - shifts elements
        accounts.remove(0);                                            // O(n) - shifts elements
        
        // ArrayList with initial capacity (performance optimization)
        List<Account> optimizedAccounts = new ArrayList<>(1000);
        
        // Bulk operations
        List<Account> moreAccounts = Arrays.asList(
            new CheckingAccount("CHK002", "Alice", 600.0),
            new SavingsAccount("SAV002", "Charlie", 0.03)
        );
        accounts.addAll(moreAccounts); // Efficient bulk addition
    }
    
    /**
     * LinkedList - Doubly-linked list implementation
     * - Best for: Frequent insertions/deletions, queue operations
     * - Worst for: Random access
     */
    public void demonstrateLinkedList() {
        LinkedList<Transaction> transactions = new LinkedList<>();
        
        // Efficient operations
        transactions.addFirst(new Transaction("TXN001", TransactionType.DEPOSIT, 500.0, "CHK001", "Initial deposit"));     // O(1)
        transactions.addLast(new Transaction("TXN002", TransactionType.WITHDRAWAL, 100.0, "CHK001", "ATM withdrawal"));    // O(1)
        
        Transaction first = transactions.removeFirst();  // O(1)
        Transaction last = transactions.removeLast();    // O(1)
        
        // Less efficient operations
        Transaction middle = transactions.get(transactions.size() / 2);  // O(n) - must traverse
        
        // LinkedList as Queue
        Queue<Transaction> transactionQueue = new LinkedList<>();
        transactionQueue.offer(new Transaction("TXN003", TransactionType.TRANSFER_IN, 200.0, "CHK001", "Wire transfer"));
        Transaction processed = transactionQueue.poll();
        
        // LinkedList as Deque (double-ended queue)
        Deque<Transaction> transactionDeque = new LinkedList<>();
        transactionDeque.addFirst(new Transaction("TXN004", TransactionType.DEPOSIT, 300.0, "CHK001", "Mobile deposit"));
        transactionDeque.addLast(new Transaction("TXN005", TransactionType.FEE, 25.0, "CHK001", "Monthly fee"));
    }
    
    /**
     * Vector - Synchronized dynamic array (legacy)
     * - Similar to ArrayList but synchronized
     * - Generally prefer ArrayList with external synchronization
     */
    public void demonstrateVector() {
        Vector<Account> accounts = new Vector<>();
        
        // All methods are synchronized
        accounts.add(new CheckingAccount("CHK001", "John", 500.0));
        Account account = accounts.get(0);
        
        // Specific Vector methods
        accounts.addElement(new SavingsAccount("SAV001", "Jane", 0.025));
        Account firstElement = accounts.firstElement();
        Account lastElement = accounts.lastElement();
        
        // Enumeration (legacy iteration)
        Enumeration<Account> enumeration = accounts.elements();
        while (enumeration.hasMoreElements()) {
            Account current = enumeration.nextElement();
            System.out.println(current.getAccountNumber());
        }
    }
    
    /**
     * Modern thread-safe alternatives
     */
    public void demonstrateThreadSafeAlternatives() {
        
        // CopyOnWriteArrayList - Good for read-heavy scenarios
        List<Account> readHeavyList = new CopyOnWriteArrayList<>();
        readHeavyList.add(new CheckingAccount("CHK001", "John", 500.0));
        
        // Synchronized wrapper
        List<Account> synchronizedList = Collections.synchronizedList(new ArrayList<>());
        
        // Manual synchronization for complex operations
        synchronized(synchronizedList) {
            for (Account account : synchronizedList) {
                // Synchronized iteration
                System.out.println(account.getBalance());
            }
        }
    }
    
    /**
     * Performance comparison example
     */
    public void performanceComparison() {
        int size = 100000;
        
        // ArrayList performance test
        List<Integer> arrayList = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < size; i++) {
            arrayList.add(i);
        }
        long arrayListAddTime = System.nanoTime() - start;
        
        // LinkedList performance test
        List<Integer> linkedList = new LinkedList<>();
        start = System.nanoTime();
        for (int i = 0; i < size; i++) {
            linkedList.add(i);
        }
        long linkedListAddTime = System.nanoTime() - start;
        
        // Random access comparison
        start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            arrayList.get(size / 2);
        }
        long arrayListAccessTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            linkedList.get(size / 2);
        }
        long linkedListAccessTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList add: %d ns, access: %d ns%n", arrayListAddTime, arrayListAccessTime);
        System.out.printf("LinkedList add: %d ns, access: %d ns%n", linkedListAddTime, linkedListAccessTime);
    }
}
```

### List-Specific Operations

```java
// File: com/company/banking/services/AccountListService.java
package com.company.banking.services;

import com.company.banking.models.*;
import java.util.*;
import java.util.function.UnaryOperator;

public class AccountListService {
    
    private List<Account> accounts;
    
    public AccountListService() {
        this.accounts = new ArrayList<>();
    }
    
    /**
     * Indexed operations
     */
    public void addAccountAtPosition(int index, Account account) {
        if (index < 0 || index > accounts.size()) {
            throw new IndexOutOfBoundsException("Invalid index: " + index);
        }
        accounts.add(index, account);
    }
    
    public Account getAccountAtPosition(int index) {
        return accounts.get(index);
    }
    
    public Account setAccountAtPosition(int index, Account account) {
        return accounts.set(index, account);
    }
    
    public Account removeAccountAtPosition(int index) {
        return accounts.remove(index);
    }
    
    /**
     * Search operations
     */
    public int findAccountIndex(String accountNumber) {
        for (int i = 0; i < accounts.size(); i++) {
            if (accounts.get(i).getAccountNumber().equals(accountNumber)) {
                return i;
            }
        }
        return -1;
    }
    
    public int findLastOccurrence(Account account) {
        return accounts.lastIndexOf(account);
    }
    
    /**
     * Range operations
     */
    public List<Account> getAccountRange(int fromIndex, int toIndex) {
        return accounts.subList(fromIndex, toIndex);
    }
    
    public void replaceAccountsInRange(int fromIndex, int toIndex, List<Account> newAccounts) {
        List<Account> subList = accounts.subList(fromIndex, toIndex);
        subList.clear();
        subList.addAll(newAccounts);
    }
    
    /**
     * Sorting operations
     */
    public void sortAccountsByBalance() {
        accounts.sort(Comparator.comparing(Account::getBalance));
    }
    
    public void sortAccountsByOwnerName() {
        accounts.sort(Comparator.comparing(Account::getOwnerName));
    }
    
    public void sortAccounts(Comparator<Account> comparator) {
        accounts.sort(comparator);
    }
    
    /**
     * Advanced list operations
     */
    public void replaceAllAccounts(UnaryOperator<Account> operator) {
        accounts.replaceAll(operator);
    }
    
    public void reverseAccountOrder() {
        Collections.reverse(accounts);
    }
    
    public void shuffleAccounts() {
        Collections.shuffle(accounts);
    }
    
    public void rotateAccounts(int distance) {
        Collections.rotate(accounts, distance);
    }
    
    /**
     * Binary search (requires sorted list)
     */
    public int binarySearchByBalance(double targetBalance) {
        // First ensure list is sorted by balance
        sortAccountsByBalance();
        
        return Collections.binarySearch(accounts, null, 
            (account1, account2) -> {
                if (account1 == null) {
                    return Double.compare(targetBalance, account2.getBalance());
                }
                return Double.compare(account1.getBalance(), account2.getBalance());
            });
    }
    
    /**
     * List iteration patterns
     */
    public void demonstrateIteration() {
        
        // Traditional for loop (when you need index)
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            System.out.printf("Account %d: %s%n", i, account.getAccountNumber());
        }
        
        // Enhanced for loop (when you don't need index)
        for (Account account : accounts) {
            System.out.println(account.getBalance());
        }
        
        // Iterator (when you might need to remove during iteration)
        Iterator<Account> iterator = accounts.iterator();
        while (iterator.hasNext()) {
            Account account = iterator.next();
            if (account.getBalance() < 0) {
                iterator.remove(); // Safe removal during iteration
            }
        }
        
        // ListIterator (bidirectional, allows modification)
        ListIterator<Account> listIterator = accounts.listIterator();
        while (listIterator.hasNext()) {
            Account account = listIterator.next();
            if (account.getBalance() > 10000) {
                // Replace with premium account
                listIterator.set(new PremiumAccount(
                    account.getAccountNumber(), 
                    account.getOwnerName(), 
                    0.035
                ));
            }
        }
    }
    
    // Getters and utility methods
    public List<Account> getAccounts() {
        return new ArrayList<>(accounts); // Defensive copy
    }
    
    public int size() {
        return accounts.size();
    }
    
    public boolean isEmpty() {
        return accounts.isEmpty();
    }
}
```

---

## Set Collections

Sets are collections that do not allow duplicate elements. Different implementations provide different ordering guarantees and performance characteristics.

### HashSet vs LinkedHashSet vs TreeSet

```java
// File: com/company/banking/examples/SetComparison.java
package com.company.banking.examples;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import java.util.*;

public class SetComparison {
    
    /**
     * HashSet - Hash table implementation
     * - Best performance: O(1) for basic operations
     * - No ordering guarantee
     * - Requires good hashCode() and equals() implementation
     */
    public void demonstrateHashSet() {
        Set<String> accountNumbers = new HashSet<>();
        
        // Fast operations - O(1) average case
        accountNumbers.add("CHK001");
        accountNumbers.add("SAV001");
        accountNumbers.add("CHK001"); // Duplicate - ignored
        
        boolean contains = accountNumbers.contains("CHK001"); // O(1)
        boolean removed = accountNumbers.remove("SAV001");    // O(1)
        
        System.out.println("HashSet size: " + accountNumbers.size()); // 1
        
        // No predictable iteration order
        for (String accountNumber : accountNumbers) {
            System.out.println("Account: " + accountNumber);
        }
        
        // Working with custom objects
        Set<Account> uniqueAccounts = new HashSet<>();
        uniqueAccounts.add(new CheckingAccount("CHK001", "John", 500.0));
        uniqueAccounts.add(new CheckingAccount("CHK001", "John", 600.0)); // Same account number
        
        // Only one account if equals() is properly implemented
        System.out.println("Unique accounts: " + uniqueAccounts.size());
    }
    
    /**
     * LinkedHashSet - Hash table + linked list implementation
     * - Maintains insertion order
     * - Slightly slower than HashSet due to linked list overhead
     * - Predictable iteration order
     */
    public void demonstrateLinkedHashSet() {
        Set<TransactionType> transactionTypes = new LinkedHashSet<>();
        
        // Add in specific order
        transactionTypes.add(TransactionType.DEPOSIT);
        transactionTypes.add(TransactionType.WITHDRAWAL);
        transactionTypes.add(TransactionType.TRANSFER_IN);
        transactionTypes.add(TransactionType.TRANSFER_OUT);
        transactionTypes.add(TransactionType.DEPOSIT); // Duplicate - ignored
        
        // Iteration maintains insertion order
        System.out.println("Transaction types in insertion order:");
        for (TransactionType type : transactionTypes) {
            System.out.println("- " + type.getDescription());
        }
        
        // Useful for maintaining order in configuration or menu items
        Set<String> menuOptions = new LinkedHashSet<>();
        menuOptions.add("View Balance");
        menuOptions.add("Make Deposit");
        menuOptions.add("Make Withdrawal");
        menuOptions.add("Transfer Funds");
        menuOptions.add("View History");
    }
    
    /**
     * TreeSet - Red-black tree implementation (SortedSet)
     * - Elements are sorted according to natural order or Comparator
     * - O(log n) for basic operations
     * - Implements SortedSet and NavigableSet interfaces
     */
    public void demonstrateTreeSet() {
        
        // Natural ordering (requires Comparable)
        Set<String> sortedAccountNumbers = new TreeSet<>();
        sortedAccountNumbers.add("CHK003");
        sortedAccountNumbers.add("CHK001");
        sortedAccountNumbers.add("SAV002");
        sortedAccountNumbers.add("CHK002");
        
        // Automatically sorted
        System.out.println("Sorted account numbers:");
        for (String accountNumber : sortedAccountNumbers) {
            System.out.println("- " + accountNumber);
        }
        
        // Custom comparator for complex objects
        Set<Account> accountsByBalance = new TreeSet<>(
            Comparator.comparing(Account::getBalance)
        );
        
        accountsByBalance.add(new CheckingAccount("CHK001", "John", 500.0));
        accountsByBalance.add(new SavingsAccount("SAV001", "Jane", 0.025)); // Balance: 0.025
        accountsByBalance.add(new CheckingAccount("CHK002", "Bob", 750.0));
        
        System.out.println("Accounts sorted by balance:");
        for (Account account : accountsByBalance) {
            System.out.printf("- %s: $%.2f%n", 
                account.getAccountNumber(), account.getBalance());
        }
        
        // NavigableSet operations (TreeSet-specific)
        NavigableSet<String> navigableSet = new TreeSet<>(sortedAccountNumbers);
        
        String first = navigableSet.first();           // Lowest element
        String last = navigableSet.last();            // Highest element
        String lower = navigableSet.lower("CHK002");   // Largest element < "CHK002"
        String higher = navigableSet.higher("CHK002"); // Smallest element > "CHK002"
        String floor = navigableSet.floor("CHK001");   // Largest element <= "CHK001"
        String ceiling = navigableSet.ceiling("CHK001"); // Smallest element >= "CHK001"
        
        // Range operations
        SortedSet<String> chkAccounts = navigableSet.subSet("CHK000", "CHK999");
        SortedSet<String> headSet = navigableSet.headSet("CHK002"); // Elements < "CHK002"
        SortedSet<String> tailSet = navigableSet.tailSet("CHK002"); // Elements >= "CHK002"
    }
    
    /**
     * EnumSet - Specialized set for enum types
     * - Extremely efficient for enum types
     * - Compact bit vector representation
     */
    public void demonstrateEnumSet() {
        
        // Create EnumSet with specific values
        Set<TransactionType> feeTransactions = EnumSet.of(
            TransactionType.WITHDRAWAL,
            TransactionType.ATM_WITHDRAWAL,
            TransactionType.CHECK
        );
        
        // Create EnumSet with all values
        Set<TransactionType> allTransactions = EnumSet.allOf(TransactionType.class);
        
        // Create EnumSet with none (empty)
        Set<TransactionType> noTransactions = EnumSet.noneOf(TransactionType.class);
        
        // Create EnumSet with range
        Set<AccountTier> middleTiers = EnumSet.range(AccountTier.STANDARD, AccountTier.PREMIUM);
        
        // Complement set
        Set<TransactionType> nonFeeTransactions = EnumSet.complementOf(feeTransactions);
        
        // Very efficient operations
        boolean hasFee = feeTransactions.contains(TransactionType.WITHDRAWAL);
        feeTransactions.add(TransactionType.TRANSFER_OUT);
        
        System.out.println("Fee-based transactions:");
        for (TransactionType type : feeTransactions) {
            System.out.println("- " + type.getDescription() + ": $" + type.getStandardFee());
        }
    }
    
    /**
     * Set operations (mathematical set operations)
     */
    public void demonstrateSetOperations() {
        Set<String> checkingAccounts = new HashSet<>(Arrays.asList(
            "CHK001", "CHK002", "CHK003", "CHK004"
        ));
        
        Set<String> highBalanceAccounts = new HashSet<>(Arrays.asList(
            "CHK002", "CHK004", "SAV001", "SAV002"
        ));
        
        // Union (all elements from both sets)
        Set<String> union = new HashSet<>(checkingAccounts);
        union.addAll(highBalanceAccounts);
        System.out.println("Union: " + union);
        
        // Intersection (common elements)
        Set<String> intersection = new HashSet<>(checkingAccounts);
        intersection.retainAll(highBalanceAccounts);
        System.out.println("Intersection: " + intersection);
        
        // Difference (elements in first set but not in second)
        Set<String> difference = new HashSet<>(checkingAccounts);
        difference.removeAll(highBalanceAccounts);
        System.out.println("Difference: " + difference);
        
        // Symmetric difference (elements in either set but not in both)
        Set<String> symmetricDifference = new HashSet<>(union);
        symmetricDifference.removeAll(intersection);
        System.out.println("Symmetric difference: " + symmetricDifference);
        
        // Check subset relationship
        boolean isSubset = union.containsAll(checkingAccounts);
        System.out.println("Checking accounts is subset of union: " + isSubset);
    }
}
```

### Set-Based Services

```java
// File: com/company/banking/services/UniqueElementService.java
package com.company.banking.services;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import java.util.*;
import java.time.LocalDate;

public class UniqueElementService {
    
    /**
     * Track unique customers
     */
    public static class CustomerTracker {
        private final Set<String> uniqueCustomers = new LinkedHashSet<>();
        private final Map<String, Set<String>> customerAccounts = new HashMap<>();
        
        public void addCustomerAccount(String customerName, String accountNumber) {
            uniqueCustomers.add(customerName);
            customerAccounts.computeIfAbsent(customerName, k -> new HashSet<>())
                           .add(accountNumber);
        }
        
        public Set<String> getUniqueCustomers() {
            return new LinkedHashSet<>(uniqueCustomers); // Preserve insertion order
        }
        
        public Set<String> getAccountsForCustomer(String customerName) {
            return new HashSet<>(customerAccounts.getOrDefault(customerName, Collections.emptySet()));
        }
        
        public int getUniqueCustomerCount() {
            return uniqueCustomers.size();
        }
        
        public Set<String> getCustomersWithMultipleAccounts() {
            return customerAccounts.entrySet().stream()
                    .filter(entry -> entry.getValue().size() > 1)
                    .map(Map.Entry::getKey)
                    .collect(HashSet::new, Set::add, Set::addAll);
        }
    }
    
    /**
     * Transaction type analytics
     */
    public static class TransactionAnalytics {
        private final Set<TransactionType> observedTypes = EnumSet.noneOf(TransactionType.class);
        private final Map<LocalDate, Set<TransactionType>> dailyTypes = new HashMap<>();
        
        public void recordTransaction(Transaction transaction) {
            TransactionType type = transaction.getType();
            LocalDate date = transaction.getTimestamp().toLocalDate();
            
            observedTypes.add(type);
            dailyTypes.computeIfAbsent(date, k -> EnumSet.noneOf(TransactionType.class))
                     .add(type);
        }
        
        public Set<TransactionType> getAllObservedTypes() {
            return EnumSet.copyOf(observedTypes);
        }
        
        public Set<TransactionType> getTypesForDate(LocalDate date) {
            return EnumSet.copyOf(dailyTypes.getOrDefault(date, EnumSet.noneOf(TransactionType.class)));
        }
        
        public Set<TransactionType> getUnusedTypes() {
            Set<TransactionType> allTypes = EnumSet.allOf(TransactionType.class);
            allTypes.removeAll(observedTypes);
            return allTypes;
        }
        
        public Map<LocalDate, Integer> getTransactionTypeDiversity() {
            Map<LocalDate, Integer> diversity = new HashMap<>();
            for (Map.Entry<LocalDate, Set<TransactionType>> entry : dailyTypes.entrySet()) {
                diversity.put(entry.getKey(), entry.getValue().size());
            }
            return diversity;
        }
    }
    
    /**
     * Account number validation and uniqueness
     */
    public static class AccountNumberValidator {
        private final Set<String> usedAccountNumbers = new TreeSet<>();
        private final Set<String> reservedPrefixes = new HashSet<>(Arrays.asList(
            "ADM", "SYS", "TST", "TMP"
        ));
        
        public boolean isAccountNumberAvailable(String accountNumber) {
            if (accountNumber == null || accountNumber.length() < 6) {
                return false;
            }
            
            String prefix = accountNumber.substring(0, 3).toUpperCase();
            if (reservedPrefixes.contains(prefix)) {
                return false;
            }
            
            return !usedAccountNumbers.contains(accountNumber);
        }
        
        public String reserveAccountNumber(String accountNumber) {
            if (!isAccountNumberAvailable(accountNumber)) {
                throw new IllegalArgumentException("Account number not available: " + accountNumber);
            }
            
            usedAccountNumbers.add(accountNumber);
            return accountNumber;
        }
        
        public void releaseAccountNumber(String accountNumber) {
            usedAccountNumbers.remove(accountNumber);
        }
        
        public Set<String> getAccountNumbersWithPrefix(String prefix) {
            return usedAccountNumbers.stream()
                    .filter(number -> number.startsWith(prefix.toUpperCase()))
                    .collect(TreeSet::new, Set::add, Set::addAll);
        }
        
        public String generateNextAccountNumber(String prefix) {
            Set<String> existing = getAccountNumbersWithPrefix(prefix);
            
            for (int i = 1; i <= 9999999; i++) {
                String candidate = prefix.toUpperCase() + String.format("%07d", i);
                if (!existing.contains(candidate)) {
                    return reserveAccountNumber(candidate);
                }
            }
            
            throw new RuntimeException("No available account numbers for prefix: " + prefix);
        }
    }
    
    /**
     * Feature flag management using sets
     */
    public static class FeatureManager {
        private final Set<String> enabledFeatures = new LinkedHashSet<>();
        private final Map<String, Set<String>> userFeatures = new HashMap<>();
        private final Set<String> experimentalFeatures = new HashSet<>();
        
        public void enableFeature(String feature) {
            enabledFeatures.add(feature);
        }
        
        public void disableFeature(String feature) {
            enabledFeatures.remove(feature);
            // Also remove from all users
            userFeatures.values().forEach(userSet -> userSet.remove(feature));
        }
        
        public boolean isFeatureEnabled(String feature) {
            return enabledFeatures.contains(feature);
        }
        
        public void enableFeatureForUser(String userId, String feature) {
            if (!isFeatureEnabled(feature)) {
                throw new IllegalArgumentException("Feature not globally enabled: " + feature);
            }
            
            userFeatures.computeIfAbsent(userId, k -> new HashSet<>()).add(feature);
        }
        
        public boolean isFeatureEnabledForUser(String userId, String feature) {
            return isFeatureEnabled(feature) && 
                   userFeatures.getOrDefault(userId, Collections.emptySet()).contains(feature);
        }
        
        public Set<String> getEnabledFeatures() {
            return new LinkedHashSet<>(enabledFeatures);
        }
        
        public Set<String> getFeaturesForUser(String userId) {
            return new HashSet<>(userFeatures.getOrDefault(userId, Collections.emptySet()));
        }
        
        public void markAsExperimental(String feature) {
            experimentalFeatures.add(feature);
        }
        
        public Set<String> getExperimentalFeatures() {
            Set<String> experimental = new HashSet<>(experimentalFeatures);
            experimental.retainAll(enabledFeatures); // Only enabled experimental features
            return experimental;
        }
    }
}
```

---

## Map Collections

Maps store key-value pairs and provide efficient lookup based on keys. Different implementations offer different performance characteristics and ordering guarantees.

### HashMap vs LinkedHashMap vs TreeMap

```java
// File: com/company/banking/examples/MapComparison.java
package com.company.banking.examples;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class MapComparison {
    
    /**
     * HashMap - Hash table implementation
     * - Best performance: O(1) for basic operations
     * - No ordering guarantee
     * - Allows one null key and multiple null values
     */
    public void demonstrateHashMap() {
        Map<String, Account> accountMap = new HashMap<>();
        
        // Fast operations - O(1) average case
        accountMap.put("CHK001", new CheckingAccount("CHK001", "John", 500.0));
        accountMap.put("SAV001", new SavingsAccount("SAV001", "Jane", 0.025));
        accountMap.put("CHK002", new CheckingAccount("CHK002", "Bob", 750.0));
        
        // Key operations
        Account account = accountMap.get("CHK001");           // O(1) retrieval
        boolean hasKey = accountMap.containsKey("SAV001");    // O(1) key check
        boolean hasValue = accountMap.containsValue(account); // O(n) value check
        Account removed = accountMap.remove("CHK002");        // O(1) removal
        
        // Replace operations
        Account replaced = accountMap.replace("CHK001", 
            new CheckingAccount("CHK001", "John Smith", 600.0));
        
        accountMap.replaceAll((key, value) -> 
            new CheckingAccount(key, value.getOwnerName() + " (Updated)", value.getBalance()));
        
        // Iteration (no guaranteed order)
        for (Map.Entry<String, Account> entry : accountMap.entrySet()) {
            System.out.printf("Key: %s, Account: %s%n", 
                entry.getKey(), entry.getValue().getOwnerName());
        }
        
        // Separate iteration over keys and values
        for (String key : accountMap.keySet()) {
            System.out.println("Account number: " + key);
        }
        
        for (Account value : accountMap.values()) {
            System.out.println("Balance: " + value.getBalance());
        }
    }
    
    /**
     * LinkedHashMap - Hash table + linked list implementation
     * - Maintains insertion order or access order
     * - Slightly slower than HashMap due to linked list overhead
     * - Useful for caching (with access-order mode)
     */
    public void demonstrateLinkedHashMap() {
        
        // Insertion-order LinkedHashMap (default)
        Map<String, Account> insertionOrderMap = new LinkedHashMap<>();
        insertionOrderMap.put("CHK003", new CheckingAccount("CHK003", "Charlie", 300.0));
        insertionOrderMap.put("SAV002", new SavingsAccount("SAV002", "Diana", 0.03));
        insertionOrderMap.put("CHK001", new CheckingAccount("CHK001", "Alice", 800.0));
        
        System.out.println("Insertion order iteration:");
        for (String key : insertionOrderMap.keySet()) {
            System.out.println("- " + key);
        }
        
        // Access-order LinkedHashMap (for LRU cache)
        Map<String, Account> lruCache = new LinkedHashMap<String, Account>(16, 0.75f, true) {
            private static final int MAX_ENTRIES = 3;
            
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, Account> eldest) {
                return size() > MAX_ENTRIES;
            }
        };
        
        lruCache.put("ACC1", new CheckingAccount("ACC1", "User1", 100.0));
        lruCache.put("ACC2", new CheckingAccount("ACC2", "User2", 200.0));
        lruCache.put("ACC3", new CheckingAccount("ACC3", "User3", 300.0));
        
        // Access ACC1 (moves it to end in access order)
        lruCache.get("ACC1");
        
        // Add new entry (should remove ACC2 as it's least recently used)
        lruCache.put("ACC4", new CheckingAccount("ACC4", "User4", 400.0));
        
        System.out.println("LRU Cache contents:");
        for (String key : lruCache.keySet()) {
            System.out.println("- " + key);
        }
    }
    
    /**
     * TreeMap - Red-black tree implementation (SortedMap)
     * - Keys are sorted according to natural order or Comparator
     * - O(log n) for basic operations
     * - Implements SortedMap and NavigableMap interfaces
     */
    public void demonstrateTreeMap() {
        
        // Natural ordering (String keys)
        Map<String, Account> sortedMap = new TreeMap<>();
        sortedMap.put("CHK003", new CheckingAccount("CHK003", "Charlie", 300.0));
        sortedMap.put("CHK001", new CheckingAccount("CHK001", "Alice", 800.0));
        sortedMap.put("SAV002", new SavingsAccount("SAV002", "Diana", 0.03));
        sortedMap.put("CHK002", new CheckingAccount("CHK002", "Bob", 600.0));
        
        System.out.println("Sorted keys iteration:");
        for (String key : sortedMap.keySet()) {
            System.out.println("- " + key);
        }
        
        // Custom comparator (sort by balance)
        Map<Account, String> balanceSortedMap = new TreeMap<>(
            Comparator.comparing(Account::getBalance)
        );
        
        Account acc1 = new CheckingAccount("CHK001", "John", 500.0);
        Account acc2 = new SavingsAccount("SAV001", "Jane", 0.025);
        Account acc3 = new CheckingAccount("CHK002", "Bob", 750.0);
        
        balanceSortedMap.put(acc1, "John's Account");
        balanceSortedMap.put(acc2, "Jane's Savings");
        balanceSortedMap.put(acc3, "Bob's Account");
        
        System.out.println("Accounts sorted by balance:");
        for (Map.Entry<Account, String> entry : balanceSortedMap.entrySet()) {
            System.out.printf("- $%.2f: %s%n", 
                entry.getKey().getBalance(), entry.getValue());
        }
        
        // NavigableMap operations (TreeMap-specific)
        NavigableMap<String, Account> navigableMap = new TreeMap<>(sortedMap);
        
        Map.Entry<String, Account> firstEntry = navigableMap.firstEntry();
        Map.Entry<String, Account> lastEntry = navigableMap.lastEntry();
        Map.Entry<String, Account> lowerEntry = navigableMap.lowerEntry("CHK002");
        Map.Entry<String, Account> higherEntry = navigableMap.higherEntry("CHK002");
        
        // Range operations
        SortedMap<String, Account> chkAccounts = navigableMap.subMap("CHK000", "CHK999");
        SortedMap<String, Account> headMap = navigableMap.headMap("CHK002");
        SortedMap<String, Account> tailMap = navigableMap.tailMap("CHK002");
        
        // Descending map
        NavigableMap<String, Account> descendingMap = navigableMap.descendingMap();
    }
    
    /**
     * EnumMap - Specialized map for enum keys
     * - Extremely efficient for enum keys
     * - Compact array representation
     * - Natural ordering of enum constants
     */
    public void demonstrateEnumMap() {
        
        // Account tier statistics
        Map<AccountTier, Integer> tierCounts = new EnumMap<>(AccountTier.class);
        tierCounts.put(AccountTier.BASIC, 150);
        tierCounts.put(AccountTier.STANDARD, 300);
        tierCounts.put(AccountTier.PREMIUM, 75);
        
        // Transaction type fees
        Map<TransactionType, Double> transactionFees = new EnumMap<>(TransactionType.class);
        for (TransactionType type : TransactionType.values()) {
            transactionFees.put(type, type.getStandardFee());
        }
        
        // Efficient operations
        double withdrawalFee = transactionFees.get(TransactionType.WITHDRAWAL);
        transactionFees.put(TransactionType.ATM_WITHDRAWAL, 3.50); // Update fee
        
        System.out.println("Transaction fees by type:");
        for (Map.Entry<TransactionType, Double> entry : transactionFees.entrySet()) {
            System.out.printf("- %s: $%.2f%n", 
                entry.getKey().getDescription(), entry.getValue());
        }
    }
    
    /**
     * ConcurrentHashMap - Thread-safe hash map
     * - High performance in concurrent environments
     * - Segment-based locking (Java 7) or node-based (Java 8+)
     * - Atomic operations
     */
    public void demonstrateConcurrentHashMap() {
        Map<String, Account> concurrentMap = new ConcurrentHashMap<>();
        
        // Thread-safe operations
        concurrentMap.put("CHK001", new CheckingAccount("CHK001", "John", 500.0));
        
        // Atomic operations
        Account existing = concurrentMap.putIfAbsent("CHK001", 
            new CheckingAccount("CHK001", "Jane", 600.0)); // Won't replace
        
        // Atomic replacement
        boolean replaced = concurrentMap.replace("CHK001", existing,
            new CheckingAccount("CHK001", "John Updated", 550.0));
        
        // Compute operations (atomic)
        concurrentMap.compute("CHK001", (key, account) -> {
            if (account != null) {
                account.deposit(100.0);
            }
            return account;
        });
        
        concurrentMap.computeIfAbsent("CHK002", key -> 
            new CheckingAccount(key, "Default User", 0.0));
        
        concurrentMap.computeIfPresent("CHK001", (key, account) -> {
            account.withdraw(50.0);
            return account;
        });
        
        // Merge operation
        concurrentMap.merge("CHK001", new CheckingAccount("CHK001", "Temp", 25.0),
            (existing, update) -> {
                existing.deposit(update.getBalance());
                return existing;
            });
    }
    
    /**
     * Map utility operations and patterns
     */
    public void demonstrateMapOperations() {
        Map<String, Account> accounts = new HashMap<>();
        accounts.put("CHK001", new CheckingAccount("CHK001", "John", 500.0));
        accounts.put("SAV001", new SavingsAccount("SAV001", "Jane", 0.025));
        
        // Default value operations
        Account defaultAccount = accounts.getOrDefault("CHK999", 
            new CheckingAccount("DEFAULT", "Unknown", 0.0));
        
        // Conditional operations
        accounts.putIfAbsent("CHK002", new CheckingAccount("CHK002", "Bob", 300.0));
        
        // Bulk operations
        Map<String, Account> moreAccounts = Map.of(
            "CHK003", new CheckingAccount("CHK003", "Alice", 700.0),
            "SAV002", new SavingsAccount("SAV002", "Charlie", 0.03)
        );
        accounts.putAll(moreAccounts);
        
        // forEach operation
        accounts.forEach((key, account) -> {
            System.out.printf("Account %s belongs to %s with balance $%.2f%n",
                key, account.getOwnerName(), account.getBalance());
        });
        
        // Clear and size operations
        System.out.println("Map size: " + accounts.size());
        System.out.println("Is empty: " + accounts.isEmpty());
        accounts.clear();
    }
}
```

### Map-Based Services

```java
// File: com/company/banking/services/AccountCacheService.java
package com.company.banking.services;

import com.company.banking.models.*;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.time.LocalDateTime;
import java.time.Duration;

public class AccountCacheService {
    
    /**
     * Multi-level caching using different map types
     */
    public static class MultiLevelCache {
        // L1 Cache - Recent access (LinkedHashMap with access order)
        private final Map<String, CacheEntry<Account>> l1Cache;
        
        // L2 Cache - Larger capacity (HashMap)
        private final Map<String, CacheEntry<Account>> l2Cache;
        
        // Statistics (EnumMap)
        private final Map<CacheLevel, CacheStats> stats;
        
        private static final int L1_CAPACITY = 100;
        private static final int L2_CAPACITY = 1000;
        private static final Duration CACHE_TTL = Duration.ofMinutes(30);
        
        public MultiLevelCache() {
            // L1 Cache with LRU eviction
            this.l1Cache = new LinkedHashMap<String, CacheEntry<Account>>(
                L1_CAPACITY, 0.75f, true) {
                @Override
                protected boolean removeEldestEntry(Map.Entry<String, CacheEntry<Account>> eldest) {
                    return size() > L1_CAPACITY;
                }
            };
            
            this.l2Cache = new HashMap<>();
            this.stats = new EnumMap<>(CacheLevel.class);
            
            // Initialize statistics
            for (CacheLevel level : CacheLevel.values()) {
                stats.put(level, new CacheStats());
            }
        }
        
        public Account get(String accountId) {
            // Try L1 cache first
            CacheEntry<Account> entry = l1Cache.get(accountId);
            if (entry != null && !entry.isExpired()) {
                stats.get(CacheLevel.L1).recordHit();
                return entry.getValue();
            }
            stats.get(CacheLevel.L1).recordMiss();
            
            // Try L2 cache
            entry = l2Cache.get(accountId);
            if (entry != null && !entry.isExpired()) {
                stats.get(CacheLevel.L2).recordHit();
                // Promote to L1
                l1Cache.put(accountId, entry);
                return entry.getValue();
            }
            stats.get(CacheLevel.L2).recordMiss();
            
            return null; // Cache miss
        }
        
        public void put(String accountId, Account account) {
            CacheEntry<Account> entry = new CacheEntry<>(account, CACHE_TTL);
            l1Cache.put(accountId, entry);
            l2Cache.put(accountId, entry);
        }
        
        public void evictExpired() {
            LocalDateTime now = LocalDateTime.now();
            l1Cache.entrySet().removeIf(entry -> entry.getValue().isExpired());
            l2Cache.entrySet().removeIf(entry -> entry.getValue().isExpired());
        }
        
        public Map<CacheLevel, CacheStats> getStatistics() {
            return new EnumMap<>(stats);
        }
        
        private static class CacheEntry<T> {
            private final T value;
            private final LocalDateTime expiryTime;
            
            public CacheEntry(T value, Duration ttl) {
                this.value = value;
                this.expiryTime = LocalDateTime.now().plus(ttl);
            }
            
            public T getValue() { return value; }
            
            public boolean isExpired() {
                return LocalDateTime.now().isAfter(expiryTime);
            }
        }
        
        public enum CacheLevel {
            L1, L2
        }
        
        public static class CacheStats {
            private long hits = 0;
            private long misses = 0;
            
            public void recordHit() { hits++; }
            public void recordMiss() { misses++; }
            
            public long getHits() { return hits; }
            public long getMisses() { return misses; }
            public double getHitRatio() {
                long total = hits + misses;
                return total == 0 ? 0.0 : (double) hits / total;
            }
        }
    }
    
    /**
     * Account grouping and indexing
     */
    public static class AccountIndex {
        private final Map<String, Account> primaryIndex; // accountNumber -> Account
        private final Map<String, Set<Account>> ownerIndex; // ownerName -> Set<Account>
        private final Map<Class<?>, Set<Account>> typeIndex; // Class -> Set<Account>
        private final NavigableMap<Double, Set<Account>> balanceIndex; // balance -> Set<Account>
        
        public AccountIndex() {
            this.primaryIndex = new ConcurrentHashMap<>();
            this.ownerIndex = new ConcurrentHashMap<>();
            this.typeIndex = new ConcurrentHashMap<>();
            this.balanceIndex = new TreeMap<>();
        }
        
        public void addAccount(Account account) {
            String accountNumber = account.getAccountNumber();
            
            // Primary index
            primaryIndex.put(accountNumber, account);
            
            // Owner index
            ownerIndex.computeIfAbsent(account.getOwnerName(), k -> ConcurrentHashMap.newKeySet())
                     .add(account);
            
            // Type index
            typeIndex.computeIfAbsent(account.getClass(), k -> ConcurrentHashMap.newKeySet())
                    .add(account);
            
            // Balance index
            synchronized (balanceIndex) {
                balanceIndex.computeIfAbsent(account.getBalance(), k -> ConcurrentHashMap.newKeySet())
                           .add(account);
            }
        }
        
        public Account getByAccountNumber(String accountNumber) {
            return primaryIndex.get(accountNumber);
        }
        
        public Set<Account> getByOwner(String ownerName) {
            return new HashSet<>(ownerIndex.getOrDefault(ownerName, Collections.emptySet()));
        }
        
        public Set<Account> getByType(Class<? extends Account> accountType) {
            return new HashSet<>(typeIndex.getOrDefault(accountType, Collections.emptySet()));
        }
        
        public Set<Account> getByBalanceRange(double minBalance, double maxBalance) {
            Set<Account> result = new HashSet<>();
            synchronized (balanceIndex) {
                NavigableMap<Double, Set<Account>> range = balanceIndex.subMap(minBalance, true, maxBalance, true);
                for (Set<Account> accounts : range.values()) {
                    result.addAll(accounts);
                }
            }
            return result;
        }
        
        public void removeAccount(String accountNumber) {
            Account account = primaryIndex.remove(accountNumber);
            if (account != null) {
                // Remove from secondary indexes
                Set<Account> ownerAccounts = ownerIndex.get(account.getOwnerName());
                if (ownerAccounts != null) {
                    ownerAccounts.remove(account);
                    if (ownerAccounts.isEmpty()) {
                        ownerIndex.remove(account.getOwnerName());
                    }
                }
                
                Set<Account> typeAccounts = typeIndex.get(account.getClass());
                if (typeAccounts != null) {
                    typeAccounts.remove(account);
                    if (typeAccounts.isEmpty()) {
                        typeIndex.remove(account.getClass());
                    }
                }
                
                synchronized (balanceIndex) {
                    Set<Account> balanceAccounts = balanceIndex.get(account.getBalance());
                    if (balanceAccounts != null) {
                        balanceAccounts.remove(account);
                        if (balanceAccounts.isEmpty()) {
                            balanceIndex.remove(account.getBalance());
                        }
                    }
                }
            }
        }
        
        public Map<String, Integer> getOwnerAccountCounts() {
            Map<String, Integer> counts = new HashMap<>();
            ownerIndex.forEach((owner, accounts) -> counts.put(owner, accounts.size()));
            return counts;
        }
        
        public Map<Class<?>, Integer> getTypeDistribution() {
            Map<Class<?>, Integer> distribution = new HashMap<>();
            typeIndex.forEach((type, accounts) -> distribution.put(type, accounts.size()));
            return distribution;
        }
    }
    
    /**
     * Configuration management using maps
     */
    public static class ConfigurationManager {
        private final Map<String, String> globalConfig = new ConcurrentHashMap<>();
        private final Map<String, Map<String, String>> userConfig = new ConcurrentHashMap<>();
        private final Map<String, Map<String, String>> groupConfig = new ConcurrentHashMap<>();
        
        // Configuration hierarchy: user -> group -> global
        public String getConfig(String userId, String groupId, String key) {
            // Check user-specific config first
            Map<String, String> userMap = userConfig.get(userId);
            if (userMap != null && userMap.containsKey(key)) {
                return userMap.get(key);
            }
            
            // Check group config
            Map<String, String> groupMap = groupConfig.get(groupId);
            if (groupMap != null && groupMap.containsKey(key)) {
                return groupMap.get(key);
            }
            
            // Fallback to global config
            return globalConfig.get(key);
        }
        
        public void setGlobalConfig(String key, String value) {
            globalConfig.put(key, value);
        }
        
        public void setUserConfig(String userId, String key, String value) {
            userConfig.computeIfAbsent(userId, k -> new ConcurrentHashMap<>()).put(key, value);
        }
        
        public void setGroupConfig(String groupId, String key, String value) {
            groupConfig.computeIfAbsent(groupId, k -> new ConcurrentHashMap<>()).put(key, value);
        }
        
        public Map<String, String> getAllConfigForUser(String userId, String groupId) {
            Map<String, String> result = new LinkedHashMap<>();
            
            // Start with global config
            result.putAll(globalConfig);
            
            // Override with group config
            Map<String, String> groupMap = groupConfig.get(groupId);
            if (groupMap != null) {
                result.putAll(groupMap);
            }
            
            // Override with user config
            Map<String, String> userMap = userConfig.get(userId);
            if (userMap != null) {
                result.putAll(userMap);
            }
            
            return result;
        }
    }
}
```

---

## Queue Collections

Queues represent collections designed for holding elements prior to processing, supporting various ordering disciplines.

### Queue Implementations

```java
// File: com/company/banking/examples/QueueComparison.java
package com.company.banking.examples;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import java.util.*;
import java.util.concurrent.*;

public class QueueComparison {
    
    /**
     * LinkedList as Queue - FIFO implementation
     * - Good general-purpose queue
     * - Supports both Queue and Deque operations
     */
    public void demonstrateLinkedListQueue() {
        Queue<Transaction> transactionQueue = new LinkedList<>();
        
        // Queue operations
        transactionQueue.offer(new Transaction("TXN001", TransactionType.DEPOSIT, 500.0, "CHK001", "Initial deposit"));
        transactionQueue.offer(new Transaction("TXN002", TransactionType.WITHDRAWAL, 100.0, "CHK001", "ATM withdrawal"));
        transactionQueue.offer(new Transaction("TXN003", TransactionType.TRANSFER_IN, 200.0, "CHK001", "Wire transfer"));
        
        // Process transactions in FIFO order
        while (!transactionQueue.isEmpty()) {
            Transaction transaction = transactionQueue.poll(); // Remove and return head
            if (transaction != null) {
                System.out.println("Processing: " + transaction.getTransactionId());
            }
        }
        
        // Deque operations (double-ended queue)
        Deque<Transaction> deque = new LinkedList<>();
        deque.addFirst(new Transaction("TXN004", TransactionType.DEPOSIT, 300.0, "CHK001", "Front deposit"));
        deque.addLast(new Transaction("TXN005", TransactionType.DEPOSIT, 400.0, "CHK001", "Back deposit"));
        
        Transaction fromFront = deque.removeFirst();
        Transaction fromBack = deque.removeLast();
    }
    
    /**
     * PriorityQueue - Heap-based priority queue
     * - Elements are ordered according to natural ordering or Comparator
     * - Not thread-safe
     * - Useful for processing items by priority
     */
    public void demonstratePriorityQueue() {
        
        // Priority queue ordered by transaction amount (highest first)
        Queue<Transaction> priorityQueue = new PriorityQueue<>(
            Comparator.comparing(Transaction::getAmount).reversed()
        );
        
        priorityQueue.offer(new Transaction("TXN001", TransactionType.DEPOSIT, 100.0, "CHK001", "Small deposit"));
        priorityQueue.offer(new Transaction("TXN002", TransactionType.DEPOSIT, 1000.0, "CHK001", "Large deposit"));
        priorityQueue.offer(new Transaction("TXN003", TransactionType.DEPOSIT, 500.0, "CHK001", "Medium deposit"));
        
        System.out.println("Processing transactions by amount (highest first):");
        while (!priorityQueue.isEmpty()) {
            Transaction transaction = priorityQueue.poll();
            System.out.printf("Processing $%.2f transaction: %s%n", 
                transaction.getAmount(), transaction.getTransactionId());
        }
        
        // Custom priority logic for account processing
        Queue<Account> accountPriorityQueue = new PriorityQueue<>(
            Comparator.comparing((Account a) -> a instanceof PremiumAccount ? 0 : 1)
                     .thenComparing(Account::getBalance, Comparator.reverseOrder())
        );
        
        accountPriorityQueue.offer(new CheckingAccount("CHK001", "John", 500.0));
        accountPriorityQueue.offer(new PremiumAccount("PRM001", "VIP", 0.035));
        accountPriorityQueue.offer(new CheckingAccount("CHK002", "Jane", 1000.0));
        accountPriorityQueue.offer(new SavingsAccount("SAV001", "Bob", 0.025));
        
        System.out.println("Processing accounts by priority (Premium first, then by balance):");
        while (!accountPriorityQueue.isEmpty()) {
            Account account = accountPriorityQueue.poll();
            System.out.printf("Processing %s: %s ($%.2f)%n",
                account.getClass().getSimpleName(),
                account.getAccountNumber(),
                account.getBalance());
        }
    }
    
    /**
     * ArrayDeque - Resizable array implementation of Deque
     * - More efficient than LinkedList for most queue operations
     * - Good general-purpose deque implementation
     */
    public void demonstrateArrayDeque() {
        Deque<String> processingQueue = new ArrayDeque<>();
        
        // Use as a stack (LIFO)
        processingQueue.push("Task 1");
        processingQueue.push("Task 2");
        processingQueue.push("Task 3");
        
        System.out.println("Stack processing (LIFO):");
        while (!processingQueue.isEmpty()) {
            String task = processingQueue.pop();
            System.out.println("Processing: " + task);
        }
        
        // Use as a queue (FIFO)
        processingQueue.offer("Request 1");
        processingQueue.offer("Request 2");
        processingQueue.offer("Request 3");
        
        System.out.println("Queue processing (FIFO):");
        while (!processingQueue.isEmpty()) {
            String request = processingQueue.poll();
            System.out.println("Processing: " + request);
        }
        
        // Deque-specific operations
        Deque<Transaction> transactionDeque = new ArrayDeque<>();
        
        // Add to both ends
        transactionDeque.addFirst(new Transaction("TXN001", TransactionType.DEPOSIT, 500.0, "CHK001", "Priority deposit"));
        transactionDeque.addLast(new Transaction("TXN002", TransactionType.WITHDRAWAL, 100.0, "CHK001", "Regular withdrawal"));
        
        // Peek at both ends without removing
        Transaction first = transactionDeque.peekFirst();
        Transaction last = transactionDeque.peekLast();
        
        // Remove from both ends
        Transaction removedFirst = transactionDeque.removeFirst();
        Transaction removedLast = transactionDeque.removeLast();
    }
    
    /**
     * Concurrent queue implementations
     */
    public void demonstrateConcurrentQueues() {
        
        // ConcurrentLinkedQueue - Thread-safe FIFO queue
        Queue<Transaction> concurrentQueue = new ConcurrentLinkedQueue<>();
        
        // Thread-safe operations
        concurrentQueue.offer(new Transaction("TXN001", TransactionType.DEPOSIT, 500.0, "CHK001", "Concurrent deposit"));
        Transaction transaction = concurrentQueue.poll();
        
        // LinkedBlockingQueue - Bounded blocking queue
        BlockingQueue<Transaction> blockingQueue = new LinkedBlockingQueue<>(100);
        
        try {
            // Blocking operations
            blockingQueue.put(new Transaction("TXN002", TransactionType.WITHDRAWAL, 100.0, "CHK001", "Blocking withdrawal"));
            Transaction blockedTransaction = blockingQueue.take(); // Blocks if empty
            
            // Timed operations
            boolean offered = blockingQueue.offer(
                new Transaction("TXN003", TransactionType.TRANSFER_IN, 200.0, "CHK001", "Timed transfer"),
                1, TimeUnit.SECONDS
            );
            
            Transaction timedTransaction = blockingQueue.poll(1, TimeUnit.SECONDS);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // PriorityBlockingQueue - Unbounded blocking priority queue
        BlockingQueue<Transaction> priorityBlockingQueue = new PriorityBlockingQueue<>(
            10, Comparator.comparing(Transaction::getAmount).reversed()
        );
        
        // ArrayBlockingQueue - Bounded blocking queue with array backing
        BlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(10);
        
        // DelayQueue - Queue of delayed elements
        DelayQueue<DelayedTransaction> delayQueue = new DelayQueue<>();
        delayQueue.offer(new DelayedTransaction("TXN004", 5000)); // 5 second delay
    }
    
    /**
     * Custom delayed transaction for DelayQueue
     */
    private static class DelayedTransaction implements Delayed {
        private final String transactionId;
        private final long executeTime;
        
        public DelayedTransaction(String transactionId, long delayMillis) {
            this.transactionId = transactionId;
            this.executeTime = System.currentTimeMillis() + delayMillis;
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long remaining = executeTime - System.currentTimeMillis();
            return unit.convert(remaining, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed other) {
            DelayedTransaction otherTransaction = (DelayedTransaction) other;
            return Long.compare(this.executeTime, otherTransaction.executeTime);
        }
        
        public String getTransactionId() {
            return transactionId;
        }
    }
}
```

### Queue-Based Services

```java
// File: com/company/banking/services/TransactionProcessingService.java
package com.company.banking.services;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import java.util.*;
import java.util.concurrent.*;
import java.time.LocalDateTime;
import java.time.Duration;

public class TransactionProcessingService {
    
    /**
     * Multi-tier transaction processing system
     */
    public static class TransactionProcessor {
        private final PriorityQueue<PriorityTransaction> highPriorityQueue;
        private final Queue<Transaction> normalQueue;
        private final Queue<Transaction> retryQueue;
        private final DelayQueue<DelayedRetryTransaction> delayedRetryQueue;
        
        public TransactionProcessor() {
            this.highPriorityQueue = new PriorityQueue<>(
                Comparator.comparing(PriorityTransaction::getPriority)
                         .thenComparing(pt -> pt.getTransaction().getTimestamp())
            );
            this.normalQueue = new LinkedList<>();
            this.retryQueue = new LinkedList<>();
            this.delayedRetryQueue = new DelayQueue<>();
        }
        
        public void submitTransaction(Transaction transaction) {
            TransactionPriority priority = determinePriority(transaction);
            
            switch (priority) {
                case HIGH:
                    highPriorityQueue.offer(new PriorityTransaction(transaction, priority));
                    break;
                case NORMAL:
                    normalQueue.offer(transaction);
                    break;
                case LOW:
                    // Add to normal queue but will be processed after others
                    normalQueue.offer(transaction);
                    break;
            }
        }
        
        public Transaction getNextTransaction() {
            // Process high priority first
            PriorityTransaction highPriority = highPriorityQueue.poll();
            if (highPriority != null) {
                return highPriority.getTransaction();
            }
            
            // Check for delayed retries that are ready
            DelayedRetryTransaction delayedRetry = delayedRetryQueue.poll();
            if (delayedRetry != null) {
                return delayedRetry.getTransaction();
            }
            
            // Process retry queue
            Transaction retry = retryQueue.poll();
            if (retry != null) {
                return retry;
            }
            
            // Process normal queue
            return normalQueue.poll();
        }
        
        public void scheduleRetry(Transaction transaction, Duration delay) {
            DelayedRetryTransaction delayedRetry = new DelayedRetryTransaction(transaction, delay);
            delayedRetryQueue.offer(delayedRetry);
        }
        
        public void addToRetryQueue(Transaction transaction) {
            retryQueue.offer(transaction);
        }
        
        private TransactionPriority determinePriority(Transaction transaction) {
            // VIP customers get high priority
            if (transaction.getDescription().contains("VIP")) {
                return TransactionPriority.HIGH;
            }
            
            // Large amounts get high priority
            if (transaction.getAmount() > 10000) {
                return TransactionPriority.HIGH;
            }
            
            // Wire transfers get normal priority
            if (transaction.getType() == TransactionType.TRANSFER_IN || 
                transaction.getType() == TransactionType.TRANSFER_OUT) {
                return TransactionPriority.NORMAL;
            }
            
            return TransactionPriority.LOW;
        }
        
        public int getTotalPendingTransactions() {
            return highPriorityQueue.size() + normalQueue.size() + 
                   retryQueue.size() + delayedRetryQueue.size();
        }
        
        public Map<TransactionPriority, Integer> getQueueSizes() {
            Map<TransactionPriority, Integer> sizes = new EnumMap<>(TransactionPriority.class);
            sizes.put(TransactionPriority.HIGH, highPriorityQueue.size());
            sizes.put(TransactionPriority.NORMAL, normalQueue.size());
            sizes.put(TransactionPriority.LOW, retryQueue.size() + delayedRetryQueue.size());
            return sizes;
        }
    }
    
    /**
     * Batch processing with work stealing queues
     */
    public static class BatchProcessor {
        private final Map<String, Queue<Transaction>> customerQueues;
        private final Queue<String> availableCustomers;
        
        public BatchProcessor() {
            this.customerQueues = new ConcurrentHashMap<>();
            this.availableCustomers = new ConcurrentLinkedQueue<>();
        }
        
        public void addTransaction(Transaction transaction) {
            String customerId = extractCustomerId(transaction);
            
            Queue<Transaction> customerQueue = customerQueues.computeIfAbsent(
                customerId, k -> new ConcurrentLinkedQueue<>()
            );
            
            customerQueue.offer(transaction);
            
            // Mark customer as having work available
            if (!availableCustomers.contains(customerId)) {
                availableCustomers.offer(customerId);
            }
        }
        
        public List<Transaction> getNextBatch(int batchSize) {
            List<Transaction> batch = new ArrayList<>();
            
            String customerId = availableCustomers.poll();
            if (customerId != null) {
                Queue<Transaction> customerQueue = customerQueues.get(customerId);
                
                if (customerQueue != null) {
                    for (int i = 0; i < batchSize && !customerQueue.isEmpty(); i++) {
                        Transaction transaction = customerQueue.poll();
                        if (transaction != null) {
                            batch.add(transaction);
                        }
                    }
                    
                    // If customer still has transactions, put back in available queue
                    if (!customerQueue.isEmpty()) {
                        availableCustomers.offer(customerId);
                    }
                }
            }
            
            return batch;
        }
        
        private String extractCustomerId(Transaction transaction) {
            // Extract customer ID from account number or description
            return transaction.getAccountNumber().substring(0, 3);
        }
        
        public int getTotalPendingTransactions() {
            return customerQueues.values().stream()
                    .mapToInt(Queue::size)
                    .sum();
        }
    }
    
    /**
     * Rate limiting using token bucket pattern with queues
     */
    public static class RateLimitedProcessor {
        private final Queue<Transaction> pendingTransactions;
        private final Queue<Long> tokenTimestamps;
        private final int maxRequestsPerWindow;
        private final Duration timeWindow;
        
        public RateLimitedProcessor(int maxRequestsPerWindow, Duration timeWindow) {
            this.pendingTransactions = new LinkedList<>();
            this.tokenTimestamps = new LinkedList<>();
            this.maxRequestsPerWindow = maxRequestsPerWindow;
            this.timeWindow = timeWindow;
        }
        
        public boolean submitTransaction(Transaction transaction) {
            if (isRateLimitExceeded()) {
                pendingTransactions.offer(transaction);
                return false;
            }
            
            processTransaction(transaction);
            recordTokenUsage();
            return true;
        }
        
        public List<Transaction> processWaitingTransactions() {
            List<Transaction> processed = new ArrayList<>();
            
            while (!pendingTransactions.isEmpty() && !isRateLimitExceeded()) {
                Transaction transaction = pendingTransactions.poll();
                processTransaction(transaction);
                recordTokenUsage();
                processed.add(transaction);
            }
            
            return processed;
        }
        
        private boolean isRateLimitExceeded() {
            long currentTime = System.currentTimeMillis();
            long windowStart = currentTime - timeWindow.toMillis();
            
            // Remove old timestamps
            while (!tokenTimestamps.isEmpty() && tokenTimestamps.peek() < windowStart) {
                tokenTimestamps.poll();
            }
            
            return tokenTimestamps.size() >= maxRequestsPerWindow;
        }
        
        private void recordTokenUsage() {
            tokenTimestamps.offer(System.currentTimeMillis());
        }
        
        private void processTransaction(Transaction transaction) {
            System.out.println("Processing transaction: " + transaction.getTransactionId());
        }
        
        public int getPendingCount() {
            return pendingTransactions.size();
        }
        
        public int getCurrentTokenCount() {
            long currentTime = System.currentTimeMillis();
            long windowStart = currentTime - timeWindow.toMillis();
            
            // Count tokens in current window
            return (int) tokenTimestamps.stream()
                    .filter(timestamp -> timestamp >= windowStart)
                    .count();
        }
    }
    
    // Supporting classes
    private static class PriorityTransaction {
        private final Transaction transaction;
        private final TransactionPriority priority;
        
        public PriorityTransaction(Transaction transaction, TransactionPriority priority) {
            this.transaction = transaction;
            this.priority = priority;
        }
        
        public Transaction getTransaction() { return transaction; }
        public TransactionPriority getPriority() { return priority; }
    }
    
    private static class DelayedRetryTransaction implements Delayed {
        private final Transaction transaction;
        private final long executeTime;
        
        public DelayedRetryTransaction(Transaction transaction, Duration delay) {
            this.transaction = transaction;
            this.executeTime = System.currentTimeMillis() + delay.toMillis();
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long remaining = executeTime - System.currentTimeMillis();
            return unit.convert(remaining, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed other) {
            DelayedRetryTransaction otherTransaction = (DelayedRetryTransaction) other;
            return Long.compare(this.executeTime, otherTransaction.executeTime);
        }
        
        public Transaction getTransaction() { return transaction; }
    }
    
    private enum TransactionPriority {
        HIGH(1), NORMAL(2), LOW(3);
        
        private final int value;
        
        TransactionPriority(int value) {
            this.value = value;
        }
        
        public int getValue() { return value; }
    }
}
```

---

## Collection Algorithms

The Collections utility class provides various algorithms for working with collections.

### Collections Utility Methods

```java
// File: com/company/banking/utils/CollectionAlgorithms.java
package com.company.banking.utils;

import com.company.banking.models.*;
import java.util.*;
import java.util.function.Function;

public class CollectionAlgorithms {
    
    /**
     * Sorting algorithms
     */
    public static class SortingExamples {
        
        public static void demonstrateSorting() {
            List<Account> accounts = Arrays.asList(
                new CheckingAccount("CHK003", "Charlie", 300.0),
                new CheckingAccount("CHK001", "Alice", 800.0),
                new SavingsAccount("SAV002", "Bob", 0.03),
                new CheckingAccount("CHK002", "Diana", 600.0)
            );
            
            // Natural ordering (requires Comparable implementation)
            // Collections.sort(accounts); // Would need Account to implement Comparable
            
            // Sort by balance
            Collections.sort(accounts, Comparator.comparing(Account::getBalance));
            System.out.println("Sorted by balance:");
            accounts.forEach(account -> 
                System.out.printf("- %s: $%.2f%n", account.getAccountNumber(), account.getBalance()));
            
            // Sort by owner name (case-insensitive)
            Collections.sort(accounts, 
                Comparator.comparing(Account::getOwnerName, String.CASE_INSENSITIVE_ORDER));
            
            // Complex sorting with multiple criteria
            Collections.sort(accounts, 
                Comparator.comparing((Account a) -> a.getClass().getSimpleName())
                         .thenComparing(Account::getBalance, Comparator.reverseOrder())
                         .thenComparing(Account::getOwnerName));
            
            // Reverse order
            Collections.reverse(accounts);
            
            // Stable sort (maintains relative order of equal elements)
            accounts.sort(Comparator.comparing(Account::getBalance));
        }
        
        public static <T> void customSort(List<T> list, Function<T, ? extends Comparable> keyExtractor) {
            list.sort(Comparator.comparing(keyExtractor));
        }
        
        public static <T> void multiKeySort(List<T> list, 
                                          Function<T, ? extends Comparable>... keyExtractors) {
            Comparator<T> comparator = null;
            
            for (Function<T, ? extends Comparable> extractor : keyExtractors) {
                if (comparator == null) {
                    comparator = Comparator.comparing(extractor);
                } else {
                    comparator = comparator.thenComparing(extractor);
                }
            }
            
            list.sort(comparator);
        }
    }
    
    /**
     * Searching algorithms
     */
    public static class SearchingExamples {
        
        public static void demonstrateSearching() {
            List<String> accountNumbers = Arrays.asList(
                "CHK001", "CHK002", "CHK003", "SAV001", "SAV002"
            );
            
            // Linear search
            int index = Collections.binarySearch(accountNumbers, "CHK002");
            System.out.println("Binary search result: " + index);
            
            // Custom binary search with comparator
            List<Account> accounts = Arrays.asList(
                new CheckingAccount("CHK001", "Alice", 300.0),
                new CheckingAccount("CHK002", "Bob", 600.0),
                new CheckingAccount("CHK003", "Charlie", 900.0)
            );
            
            // Must be sorted by the same criteria used for comparison
            Collections.sort(accounts, Comparator.comparing(Account::getBalance));
            
            int accountIndex = Collections.binarySearch(accounts, null, 
                (a1, a2) -> {
                    if (a1 == null) return Double.compare(600.0, a2.getBalance());
                    return Double.compare(a1.getBalance(), a2.getBalance());
                });
            
            // Find minimum and maximum
            Account minBalanceAccount = Collections.min(accounts, 
                Comparator.comparing(Account::getBalance));
            Account maxBalanceAccount = Collections.max(accounts, 
                Comparator.comparing(Account::getBalance));
            
            System.out.printf("Min balance: $%.2f, Max balance: $%.2f%n",
                minBalanceAccount.getBalance(), maxBalanceAccount.getBalance());
        }
        
        public static <T> Optional<T> findFirst(Collection<T> collection, 
                                              java.util.function.Predicate<T> predicate) {
            for (T item : collection) {
                if (predicate.test(item)) {
                    return Optional.of(item);
                }
            }
            return Optional.empty();
        }
        
        public static <T> List<T> findAll(Collection<T> collection, 
                                        java.util.function.Predicate<T> predicate) {
            List<T> results = new ArrayList<>();
            for (T item : collection) {
                if (predicate.test(item)) {
                    results.add(item);
                }
            }
            return results;
        }
    }
    
    /**
     * Collection manipulation algorithms
     */
    public static class ManipulationExamples {
        
        public static void demonstrateManipulation() {
            List<Account> accounts = new ArrayList<>(Arrays.asList(
                new CheckingAccount("CHK001", "Alice", 800.0),
                new SavingsAccount("SAV001", "Bob", 0.03),
                new CheckingAccount("CHK002", "Charlie", 600.0)
            ));
            
            // Shuffle
            Collections.shuffle(accounts);
            Collections.shuffle(accounts, new Random(42)); // With seed for reproducibility
            
            // Rotate (shift elements)
            Collections.rotate(accounts, 1); // Shift right by 1
            Collections.rotate(accounts, -1); // Shift left by 1
            
            // Reverse
            Collections.reverse(accounts);
            
            // Fill
            List<String> messages = new ArrayList<>(Collections.nCopies(5, ""));
            Collections.fill(messages, "Processing...");
            
            // Copy
            List<Account> accountsCopy = new ArrayList<>(Collections.nCopies(accounts.size(), null));
            Collections.copy(accountsCopy, accounts);
            
            // Swap elements
            Collections.swap(accounts, 0, accounts.size() - 1);
            
            // Replace all occurrences
            List<String> statuses = new ArrayList<>(Arrays.asList("ACTIVE", "INACTIVE", "ACTIVE"));
            Collections.replaceAll(statuses, "INACTIVE", "SUSPENDED");
        }
        
        public static <T> void rotateLeft(List<T> list, int positions) {
            Collections.rotate(list, -positions);
        }
        
        public static <T> void rotateRight(List<T> list, int positions) {
            Collections.rotate(list, positions);
        }
        
        public static <T> List<T> concatenate(List<T>... lists) {
            List<T> result = new ArrayList<>();
            for (List<T> list : lists) {
                result.addAll(list);
            }
            return result;
        }
    }
    
    /**
     * Frequency and counting algorithms
     */
    public static class CountingExamples {
        
        public static void demonstrateCounting() {
            List<String> accountTypes = Arrays.asList(
                "CHECKING", "SAVINGS", "CHECKING", "PREMIUM", "CHECKING", "SAVINGS"
            );
            
            // Count frequency of each element
            int checkingCount = Collections.frequency(accountTypes, "CHECKING");
            System.out.println("Checking accounts: " + checkingCount);
            
            // Check if collections have no common elements
            List<String> group1 = Arrays.asList("CHK001", "CHK002");
            List<String> group2 = Arrays.asList("SAV001", "SAV002");
            boolean disjoint = Collections.disjoint(group1, group2);
            System.out.println("Groups are disjoint: " + disjoint);
            
            // Custom frequency counting
            Map<String, Integer> frequencyMap = getFrequencyMap(accountTypes);
            System.out.println("Frequency map: " + frequencyMap);
        }
        
        public static <T> Map<T, Integer> getFrequencyMap(Collection<T> collection) {
            Map<T, Integer> frequencyMap = new HashMap<>();
            for (T item : collection) {
                frequencyMap.put(item, frequencyMap.getOrDefault(item, 0) + 1);
            }
            return frequencyMap;
        }
        
        public static <T> T getMostFrequent(Collection<T> collection) {
            Map<T, Integer> frequencyMap = getFrequencyMap(collection);
            return frequencyMap.entrySet().stream()
                    .max(Map.Entry.comparingByValue())
                    .map(Map.Entry::getKey)
                    .orElse(null);
        }
        
        public static <T> List<T> getMostFrequent(Collection<T> collection, int n) {
            Map<T, Integer> frequencyMap = getFrequencyMap(collection);
            return frequencyMap.entrySet().stream()
                    .sorted(Map.Entry.<T, Integer>comparingByValue().reversed())
                    .limit(n)
                    .map(Map.Entry::getKey)
                    .collect(ArrayList::new, (list, item) -> list.add(item), List::addAll);
        }
    }
    
    /**
     * Immutable and synchronized collections
     */
    public static class WrapperExamples {
        
        public static void demonstrateWrappers() {
            List<Account> accounts = new ArrayList<>(Arrays.asList(
                new CheckingAccount("CHK001", "Alice", 800.0),
                new SavingsAccount("SAV001", "Bob", 0.03)
            ));
            
            // Unmodifiable collections
            List<Account> unmodifiableList = Collections.unmodifiableList(accounts);
            Set<Account> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>(accounts));
            Map<String, Account> accountMap = new HashMap<>();
            accounts.forEach(account -> accountMap.put(account.getAccountNumber(), account));
            Map<String, Account> unmodifiableMap = Collections.unmodifiableMap(accountMap);
            
            // Synchronized collections
            List<Account> synchronizedList = Collections.synchronizedList(new ArrayList<>());
            Set<Account> synchronizedSet = Collections.synchronizedSet(new HashSet<>());
            Map<String, Account> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
            
            // Manual synchronization for iteration
            synchronized(synchronizedList) {
                for (Account account : synchronizedList) {
                    System.out.println(account.getAccountNumber());
                }
            }
            
            // Checked collections (runtime type checking)
            List<Account> checkedList = Collections.checkedList(new ArrayList<>(), Account.class);
            Set<Account> checkedSet = Collections.checkedSet(new HashSet<>(), Account.class);
            Map<String, Account> checkedMap = Collections.checkedMap(new HashMap<>(), String.class, Account.class);
            
            // Empty collections
            List<Account> emptyList = Collections.emptyList();
            Set<Account> emptySet = Collections.emptySet();
            Map<String, Account> emptyMap = Collections.emptyMap();
            
            // Singleton collections
            List<Account> singletonList = Collections.singletonList(accounts.get(0));
            Set<Account> singletonSet = Collections.singleton(accounts.get(0));
            Map<String, Account> singletonMap = Collections.singletonMap("CHK001", accounts.get(0));
        }
    }
}
```

---

## Performance and Best Practices

### Performance Comparison

```java
// File: com/company/banking/performance/CollectionPerformance.java
package com.company.banking.performance;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class CollectionPerformance {
    
    /**
     * Performance characteristics of different collections
     */
    public static class PerformanceAnalysis {
        
        public static void compareListPerformance() {
            int size = 100000;
            int iterations = 1000;
            
            System.out.println("=== List Performance Comparison ===");
            
            // ArrayList vs LinkedList insertion performance
            long start = System.nanoTime();
            List<Integer> arrayList = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                arrayList.add(i);
            }
            long arrayListInsertTime = System.nanoTime() - start;
            
            start = System.nanoTime();
            List<Integer> linkedList = new LinkedList<>();
            for (int i = 0; i < size; i++) {
                linkedList.add(i);
            }
            long linkedListInsertTime = System.nanoTime() - start;
            
            // Random access performance
            Random random = new Random(42);
            start = System.nanoTime();
            for (int i = 0; i < iterations; i++) {
                arrayList.get(random.nextInt(size));
            }
            long arrayListAccessTime = System.nanoTime() - start;
            
            start = System.nanoTime();
            for (int i = 0; i < iterations; i++) {
                linkedList.get(random.nextInt(size));
            }
            long linkedListAccessTime = System.nanoTime() - start;
            
            // Middle insertion performance
            start = System.nanoTime();
            for (int i = 0; i < iterations; i++) {
                arrayList.add(size / 2, i);
            }
            long arrayListMiddleInsertTime = System.nanoTime() - start;
            
            start = System.nanoTime();
            for (int i = 0; i < iterations; i++) {
                linkedList.add(size / 2, i);
            }
            long linkedListMiddleInsertTime = System.nanoTime() - start;
            
            System.out.printf("ArrayList - Insert: %,d ns, Access: %,d ns, Middle Insert: %,d ns%n",
                arrayListInsertTime, arrayListAccessTime, arrayListMiddleInsertTime);
            System.out.printf("LinkedList - Insert: %,d ns, Access: %,d ns, Middle Insert: %,d ns%n",
                linkedListInsertTime, linkedListAccessTime, linkedListMiddleInsertTime);
        }
        
        public static void compareSetPerformance() {
            int size = 100000;
            int iterations = 10000;
            
            System.out.println("=== Set Performance Comparison ===");
            
            // Different set implementations
            Set<Integer> hashSet = new HashSet<>();
            Set<Integer> linkedHashSet = new LinkedHashSet<>();
            Set<Integer> treeSet = new TreeSet<>();
            
            // Insertion performance
            long[] insertTimes = new long[3];
            Set<Integer>[] sets = new Set[]{hashSet, linkedHashSet, treeSet};
            String[] names = {"HashSet", "LinkedHashSet", "TreeSet"};
            
            for (int setIndex = 0; setIndex < sets.length; setIndex++) {
                long start = System.nanoTime();
                for (int i = 0; i < size; i++) {
                    sets[setIndex].add(i);
                }
                insertTimes[setIndex] = System.nanoTime() - start;
            }
            
            // Lookup performance
            Random random = new Random(42);
            long[] lookupTimes = new long[3];
            
            for (int setIndex = 0; setIndex < sets.length; setIndex++) {
                long start = System.nanoTime();
                for (int i = 0; i < iterations; i++) {
                    sets[setIndex].contains(random.nextInt(size));
                }
                lookupTimes[setIndex] = System.nanoTime() - start;
            }
            
            for (int i = 0; i < names.length; i++) {
                System.out.printf("%s - Insert: %,d ns, Lookup: %,d ns%n",
                    names[i], insertTimes[i], lookupTimes[i]);
            }
        }
        
        public static void compareMapPerformance() {
            int size = 100000;
            int iterations = 10000;
            
            System.out.println("=== Map Performance Comparison ===");
            
            Map<Integer, String> hashMap = new HashMap<>();
            Map<Integer, String> linkedHashMap = new LinkedHashMap<>();
            Map<Integer, String> treeMap = new TreeMap<>();
            Map<Integer, String> concurrentHashMap = new ConcurrentHashMap<>();
            
            Map<Integer, String>[] maps = new Map[]{hashMap, linkedHashMap, treeMap, concurrentHashMap};
            String[] names = {"HashMap", "LinkedHashMap", "TreeMap", "ConcurrentHashMap"};
            
            // Insertion performance
            for (int mapIndex = 0; mapIndex < maps.length; mapIndex++) {
                long start = System.nanoTime();
                for (int i = 0; i < size; i++) {
                    maps[mapIndex].put(i, "Value" + i);
                }
                long insertTime = System.nanoTime() - start;
                
                // Lookup performance
                Random random = new Random(42);
                start = System.nanoTime();
                for (int i = 0; i < iterations; i++) {
                    maps[mapIndex].get(random.nextInt(size));
                }
                long lookupTime = System.nanoTime() - start;
                
                System.out.printf("%s - Insert: %,d ns, Lookup: %,d ns%n",
                    names[mapIndex], insertTime, lookupTime);
            }
        }
    }
    
    /**
     * Memory usage analysis
     */
    public static class MemoryAnalysis {
        
        public static void analyzeMemoryUsage() {
            Runtime runtime = Runtime.getRuntime();
            
            // Measure ArrayList memory
            runtime.gc();
            long beforeArrayList = runtime.totalMemory() - runtime.freeMemory();
            
            List<Integer> arrayList = new ArrayList<>();
            for (int i = 0; i < 100000; i++) {
                arrayList.add(i);
            }
            
            runtime.gc();
            long afterArrayList = runtime.totalMemory() - runtime.freeMemory();
            long arrayListMemory = afterArrayList - beforeArrayList;
            
            // Measure LinkedList memory
            arrayList = null;
            runtime.gc();
            long beforeLinkedList = runtime.totalMemory() - runtime.freeMemory();
            
            List<Integer> linkedList = new LinkedList<>();
            for (int i = 0; i < 100000; i++) {
                linkedList.add(i);
            }
            
            runtime.gc();
            long afterLinkedList = runtime.totalMemory() - runtime.freeMemory();
            long linkedListMemory = afterLinkedList - beforeLinkedList;
            
            System.out.printf("ArrayList memory usage: %,d bytes%n", arrayListMemory);
            System.out.printf("LinkedList memory usage: %,d bytes%n", linkedListMemory);
            System.out.printf("LinkedList overhead: %.2fx%n", (double) linkedListMemory / arrayListMemory);
        }
        
        public static void demonstrateCapacityManagement() {
            System.out.println("=== Capacity Management ===");
            
            // ArrayList with default capacity
            List<Integer> defaultList = new ArrayList<>();
            
            // ArrayList with initial capacity
            List<Integer> optimizedList = new ArrayList<>(100000);
            
            // Measure time for default capacity (multiple resizes)
            long start = System.nanoTime();
            for (int i = 0; i < 100000; i++) {
                defaultList.add(i);
            }
            long defaultTime = System.nanoTime() - start;
            
            // Measure time for pre-sized capacity (no resizes)
            start = System.nanoTime();
            for (int i = 0; i < 100000; i++) {
                optimizedList.add(i);
            }
            long optimizedTime = System.nanoTime() - start;
            
            System.out.printf("Default capacity: %,d ns%n", defaultTime);
            System.out.printf("Pre-sized capacity: %,d ns%n", optimizedTime);
            System.out.printf("Performance improvement: %.2fx%n", (double) defaultTime / optimizedTime);
        }
    }
    
    /**
     * Best practices demonstrations
     */
    public static class BestPractices {
        
        public static void demonstrateInitialCapacity() {
            // BAD: Default capacity with known size
            List<String> badList = new ArrayList<>();
            for (int i = 0; i < 10000; i++) {
                badList.add("Item " + i);
            }
            
            // GOOD: Initial capacity to avoid resizing
            List<String> goodList = new ArrayList<>(10000);
            for (int i = 0; i < 10000; i++) {
                goodList.add("Item " + i);
            }
            
            // GOOD: Use factory methods for small, known collections
            List<String> smallList = List.of("Item1", "Item2", "Item3");
            Set<String> smallSet = Set.of("A", "B", "C");
            Map<String, Integer> smallMap = Map.of("one", 1, "two", 2, "three", 3);
        }
        
        public static void demonstrateCollectionChoice() {
            // Use ArrayList for random access and iteration
            List<String> randomAccess = new ArrayList<>();
            
            // Use LinkedList for frequent insertions/deletions at ends
            Deque<String> queue = new LinkedList<>();
            
            // Use HashSet for uniqueness without ordering
            Set<String> uniqueItems = new HashSet<>();
            
            // Use LinkedHashSet for uniqueness with insertion order
            Set<String> orderedUniqueItems = new LinkedHashSet<>();
            
            // Use TreeSet for uniqueness with natural ordering
            Set<String> sortedUniqueItems = new TreeSet<>();
            
            // Use HashMap for key-value mapping
            Map<String, String> keyValueMap = new HashMap<>();
            
            // Use TreeMap for sorted key-value mapping
            Map<String, String> sortedKeyValueMap = new TreeMap<>();
            
            // Use EnumSet for enum collections
            Set<TransactionType> transactionTypes = EnumSet.allOf(TransactionType.class);
            
            // Use EnumMap for enum keys
            Map<TransactionType, String> typeDescriptions = new EnumMap<>(TransactionType.class);
        }
        
        public static void demonstrateIterationBestPractices() {
            List<Account> accounts = Arrays.asList(
                new CheckingAccount("CHK001", "Alice", 800.0),
                new SavingsAccount("SAV001", "Bob", 0.03)
            );
            
            // GOOD: Enhanced for loop for simple iteration
            for (Account account : accounts) {
                System.out.println(account.getAccountNumber());
            }
            
            // GOOD: Iterator for safe removal during iteration
            Iterator<Account> iterator = accounts.iterator();
            while (iterator.hasNext()) {
                Account account = iterator.next();
                if (account.getBalance() < 100) {
                    iterator.remove(); // Safe removal
                }
            }
            
            // GOOD: Stream API for functional operations
            accounts.stream()
                    .filter(account -> account.getBalance() > 500)
                    .forEach(account -> System.out.println(account.getOwnerName()));
            
            // BAD: Traditional for loop unless you need the index
            for (int i = 0; i < accounts.size(); i++) {
                Account account = accounts.get(i);
                // Only use this pattern when you need the index
            }
        }
        
        public static void demonstrateThreadSafety() {
            // BAD: Unsynchronized collection in concurrent environment
            Map<String, Account> unsafeMap = new HashMap<>();
            
            // BETTER: Synchronized wrapper
            Map<String, Account> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
            
            // BEST: Concurrent collection
            Map<String, Account> concurrentMap = new ConcurrentHashMap<>();
            
            // Thread-safe iteration with synchronized wrapper
            synchronized(synchronizedMap) {
                for (Map.Entry<String, Account> entry : synchronizedMap.entrySet()) {
                    // Process entry
                }
            }
            
            // Thread-safe iteration with concurrent collection (no synchronization needed)
            for (Map.Entry<String, Account> entry : concurrentMap.entrySet()) {
                // Process entry
            }
        }
    }
}

/**
 * Comprehensive banking example using all concepts
 */
// File: com/company/banking/examples/ComprehensiveBankingExample.java
package com.company.banking.examples;

import com.company.banking.models.*;
import com.company.banking.enums.*;
import com.company.banking.generics.Repository;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.stream.Collectors;

public class ComprehensiveBankingExample {
    
    // Generic repositories
    private final Repository<Account, String> accountRepository = new Repository<>(Account.class);
    private final Repository<Transaction, String> transactionRepository = new Repository<>(Transaction.class);
    private final Repository<Customer, String> customerRepository = new Repository<>(Customer.class);
    
    // Specialized collections for different use cases
    private final Map<String, Set<Account>> customerAccounts = new ConcurrentHashMap<>();
    private final Queue<Transaction> pendingTransactions = new PriorityQueue<>(
        Comparator.comparing(Transaction::getAmount).reversed()
    );
    private final Map<TransactionType, List<Transaction>> transactionsByType = new EnumMap<>(TransactionType.class);
    private final NavigableSet<Account> accountsByBalance = new TreeSet<>(
        Comparator.comparing(Account::getBalance)
    );
    
    /**
     * Demonstrates comprehensive use of generics and collections
     */
    public void demonstrateComprehensiveExample() {
        // Initialize data
        setupInitialData();
        
        // Demonstrate various operations
        demonstrateAccountManagement();
        demonstrateTransactionProcessing();
        demonstrateReporting();
        demonstrateAnalytics();
    }
    
    private void setupInitialData() {
        // Create customers using generic repository
        Customer customer1 = new Customer("CUST001", "Alice Johnson", "alice@email.com");
        Customer customer2 = new Customer("CUST002", "Bob Smith", "bob@email.com");
        Customer customer3 = new Customer("CUST003", "Charlie Brown", "charlie@email.com");
        
        customerRepository.save(customer1.getCustomerId(), customer1);
        customerRepository.save(customer2.getCustomerId(), customer2);
        customerRepository.save(customer3.getCustomerId(), customer3);
        
        // Create accounts using polymorphism and generics
        List<Account> accounts = Arrays.asList(
            new CheckingAccount("CHK001", "Alice Johnson", 1500.0),
            new SavingsAccount("SAV001", "Alice Johnson", 0.025),
            new CheckingAccount("CHK002", "Bob Smith", 750.0),
            new PremiumAccount("PRM001", "Charlie Brown", 0.035)
        );
        
        // Store accounts in multiple collections for different access patterns
        for (Account account : accounts) {
            accountRepository.save(account.getAccountNumber(), account);
            accountsByBalance.add(account);
            
            customerAccounts.computeIfAbsent(account.getOwnerName(), k -> new LinkedHashSet<>())
                           .add(account);
        }
        
        // Create sample transactions
        List<Transaction> transactions = Arrays.asList(
            new Transaction("TXN001", TransactionType.DEPOSIT, 500.0, "CHK001", "Initial deposit"),
            new Transaction("TXN002", TransactionType.WITHDRAWAL, 100.0, "CHK001", "ATM withdrawal"),
            new Transaction("TXN003", TransactionType.TRANSFER_IN, 250.0, "SAV001", "Transfer from checking"),
            new Transaction("TXN004", TransactionType.INTEREST, 15.50, "SAV001", "Monthly interest"),
            new Transaction("TXN005", TransactionType.FEE, 25.0, "CHK002", "Overdraft fee")
        );
        
        for (Transaction transaction : transactions) {
            transactionRepository.save(transaction.getTransactionId(), transaction);
            pendingTransactions.offer(transaction);
            
            transactionsByType.computeIfAbsent(transaction.getType(), k -> new ArrayList<>())
                             .add(transaction);
        }
    }
    
    private void demonstrateAccountManagement() {
        System.out.println("=== Account Management ===");
        
        // Generic repository operations
        Optional<Account> account = accountRepository.findById("CHK001");
        account.ifPresent(acc -> System.out.println("Found account: " + acc.getAccountNumber()));
        
        // Find accounts by criteria using generic methods
        List<Account> highBalanceAccounts = accountRepository.findByPredicate(
            acc -> acc.getBalance() > 1000.0
        );
        
        System.out.println("High balance accounts: " + highBalanceAccounts.size());
        
        // Use NavigableSet for range queries
        Account targetBalance = new CheckingAccount("TEMP", "TEMP", 800.0);
        Set<Account> accountsAbove800 = accountsByBalance.tailSet(targetBalance, true);
        System.out.println("Accounts with balance >= $800: " + accountsAbove800.size());
        
        // Customer account lookup using Map
        Set<Account> aliceAccounts = customerAccounts.get("Alice Johnson");
        if (aliceAccounts != null) {
            System.out.println("Alice has " + aliceAccounts.size() + " accounts");
        }
    }
    
    private void demonstrateTransactionProcessing() {
        System.out.println("=== Transaction Processing ===");
        
        // Process transactions by priority (using PriorityQueue)
        System.out.println("Processing transactions by amount (highest first):");
        while (!pendingTransactions.isEmpty()) {
            Transaction transaction = pendingTransactions.poll();
            System.out.printf("Processing $%.2f %s transaction%n", 
                transaction.getAmount(), transaction.getType().getDescription());
        }
        
        // Group transactions by type using EnumMap
        System.out.println("\nTransaction counts by type:");
        for (Map.Entry<TransactionType, List<Transaction>> entry : transactionsByType.entrySet()) {
            System.out.printf("%s: %d transactions%n", 
                entry.getKey().getDescription(), entry.getValue().size());
        }
        
        // Calculate total fees using stream operations
        double totalFees = transactionsByType.getOrDefault(TransactionType.FEE, Collections.emptyList())
                .stream()
                .mapToDouble(Transaction::getAmount)
                .sum();
        System.out.printf("Total fees collected: $%.2f%n", totalFees);
    }
    
    private void demonstrateReporting() {
        System.out.println("=== Reporting ===");
        
        // Account summary using various collection operations
        List<Account> allAccounts = accountRepository.findAll();
        
        // Group by account type
        Map<Class<?>, List<Account>> accountsByType = allAccounts.stream()
                .collect(Collectors.groupingBy(Account::getClass));
        
        System.out.println("Accounts by type:");
        accountsByType.forEach((type, accounts) -> 
            System.out.printf("- %s: %d accounts%n", type.getSimpleName(), accounts.size()));
        
        // Calculate statistics
        DoubleSummaryStatistics balanceStats = allAccounts.stream()
                .mapToDouble(Account::getBalance)
                .summaryStatistics();
        
        System.out.printf("Balance statistics - Count: %d, Average: $%.2f, Max: $%.2f%n",
            balanceStats.getCount(), balanceStats.getAverage(), balanceStats.getMax());
        
        // Top accounts by balance using sorted collection
        System.out.println("Top 3 accounts by balance:");
        accountsByBalance.descendingSet().stream()
                .limit(3)
                .forEach(account -> System.out.printf("- %s: $%.2f%n", 
                    account.getAccountNumber(), account.getBalance()));
    }
    
    private void demonstrateAnalytics() {
        System.out.println("=== Analytics ===");
        
        // Customer analytics using complex collection operations
        Map<String, Integer> customerAccountCounts = customerAccounts.entrySet().stream()
                .collect(Collectors.toMap(
                    Map.Entry::getKey,
                    entry -> entry.getValue().size()
                ));
        
        System.out.println("Customer account counts:");
        customerAccountCounts.entrySet().stream()
                .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                .forEach(entry -> System.out.printf("- %s: %d accounts%n", 
                    entry.getKey(), entry.getValue()));
        
        // Transaction type analysis using EnumMap and streams
        Map<TransactionType, Double> totalAmountsByType = new EnumMap<>(TransactionType.class);
        
        for (TransactionType type : TransactionType.values()) {
            double total = transactionsByType.getOrDefault(type, Collections.emptyList())
                    .stream()
                    .mapToDouble(Transaction::getAmount)
                    .sum();
            totalAmountsByType.put(type, total);
        }
        
        System.out.println("Total amounts by transaction type:");
        totalAmountsByType.entrySet().stream()
                .sorted(Map.Entry.<TransactionType, Double>comparingByValue().reversed())
                .forEach(entry -> System.out.printf("- %s: $%.2f%n", 
                    entry.getKey().getDescription(), entry.getValue()));
        
        // Performance monitoring using concurrent collections
        Map<String, Long> operationCounts = new ConcurrentHashMap<>();
        operationCounts.put("account_lookups", 1250L);
        operationCounts.put("transaction_processing", 875L);
        operationCounts.put("balance_updates", 450L);
        
        System.out.println("Operation performance metrics:");
        operationCounts.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .forEach(entry -> System.out.printf("- %s: %,d operations%n", 
                    entry.getKey(), entry.getValue()));
    }
    
    // Supporting Customer class for the example
    private static class Customer {
        private final String customerId;
        private final String name;
        private final String email;
        
        public Customer(String customerId, String name, String email) {
            this.customerId = customerId;
            this.name = name;
            this.email = email;
        }
        
        public String getCustomerId() { return customerId; }
        public String getName() { return name; }
        public String getEmail() { return email; }
    }
}

---

## Summary

This comprehensive manual covers the fundamental concepts of Generics and Collections that form the backbone of modern Java development:

### Key Takeaways

**Generics:**
1. **Type Safety** - Eliminate casting and catch errors at compile time
2. **Code Reusability** - Write algorithms that work with different types
3. **Performance** - No boxing/unboxing overhead, better JVM optimizations
4. **API Clarity** - Method signatures clearly indicate expected types

**Collections:**
1. **Framework Design** - Unified architecture for data structures
2. **Performance Characteristics** - Choose the right collection for your use case
3. **Thread Safety** - Understand concurrent vs synchronized collections
4. **Memory Efficiency** - Proper capacity management and collection choice

### Enterprise Development Impact

These concepts are crucial for:
- **Spring Boot Applications** - Dependency injection, configuration management, data handling
- **Data Processing** - ETL operations, business logic, reporting systems  
- **Performance Optimization** - Efficient algorithms, proper collection choice, memory management
- **Concurrent Programming** - Thread-safe collections, producer-consumer patterns
- **API Design** - Type-safe interfaces, generic utility methods

### Integration with Previous Concepts

- **Generics enhance OOP** - Type-safe inheritance, polymorphic collections
- **Collections use interfaces** - Abstraction, implementation flexibility
- **Exception handling** - Proper error management in collection operations
- **Performance considerations** - Big O notation, memory usage, algorithmic complexity

### Best Practices Summary

1. **Always use generics** - Avoid raw types for type safety
2. **Choose appropriate collections** - Consider performance characteristics
3. **Initialize with capacity** - Avoid unnecessary resizing operations
4. **Use concurrent collections** - For multi-threaded environments
5. **Prefer composition** - Build complex functionality with simple collections
6. **Apply PECS principle** - Producer Extends, Consumer Super for wildcards

### Spring Boot Connection

These collections concepts directly support Spring Boot development:
- **Configuration properties** - Maps and lists for application.yml/properties
- **Dependency injection** - Collections of beans, conditional configurations  
- **Data transfer objects** - Generic DTOs, pagination responses
- **Repository patterns** - Generic CRUD operations, query results
- **Caching strategies** - Map-based caches, LRU implementations

As a tech lead, mastering these concepts will help you design efficient, maintainable systems and guide your team toward performant solutions. The type safety and performance benefits of generics and collections are essential for enterprise-grade applications.


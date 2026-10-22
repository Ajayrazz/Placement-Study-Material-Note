# Object-Oriented Programming (OOP) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master concepts, internal mechanics, real-world analogies, and exact interview responses.

---

## Table of Contents
1. [Paradigm Shift: Why OOP?](#1-paradigm-shift-why-oop)
2. [Classes vs. Objects (Memory & Mechanics)](#2-classes-vs-objects-memory--mechanics)
3. [The Four Pillars of OOP (Deep Dive)](#3-the-four-pillars-of-oop)
   - [Pillar 1: Encapsulation](#pillar-1-encapsulation)
   - [Pillar 2: Abstraction](#pillar-2-abstraction)
   - [Pillar 3: Inheritance](#pillar-3-inheritance)
   - [Pillar 4: Polymorphism](#pillar-4-polymorphism)
4. [Critical Interview Face-Offs](#4-critical-interview-face-offs)
   - [Encapsulation vs. Abstraction](#encapsulation-vs-abstraction)
   - [Abstract Class vs. Interface](#abstract-class-vs-interface)
   - [Inheritance (IS-A) vs. Composition (HAS-A)](#inheritance-is-a-vs-composition-has-a)
   - [Shallow Copy vs. Deep Copy](#shallow-copy-vs-deep-copy)
5. [Relationships Between Objects (UML Level)](#5-relationships-between-objects)
6. [SOLID Principles (Interview Edition)](#6-solid-principles-interview-edition)
7. [Under The Hood: Virtual Tables (VTable & VPTR)](#7-under-the-hood-virtual-tables-vtable--vptr)
8. [Real-World Case Study: Notification System (LLD)](#8-real-world-case-study-notification-system-lld)
9. [Top 15 Tricky Interview Questions & Model Answers](#9-top-15-tricky-interview-questions--model-answers)

---

## 1. Paradigm Shift: Why OOP?

### The Core Problem with Procedural Programming (e.g., C)
In procedural programming, code is organized around functions that operate on passive data structures. 
- **Global State Pollution:** Any function can modify shared global data, making debugging unpredictable in large codebases.
- **Tight Coupling:** Changing a data structure requires modifying every single function that accesses it.
- **Poor Reusability:** Logic is tied directly to specific data layouts.

### Why OOP Came to the Rescue
OOP bundles **data (state)** and **functions (behavior)** into a single unit called an **Object**.
- **Modularity:** Internal object state is shielded from unintended external tampering.
- **Extensibility:** Systems grow by adding new classes without rewriting existing, tested code.
- **Maintainability:** Bugs are localized to specific classes rather than rippling across an entire codebase.

```
+-------------------------------------------------------+
| Procedural Approach:                                  |
|   Data (Global / Unprotected) <---> Function A()      |
|                               <---> Function B()      |
|                               <---> Function C()      |
+-------------------------------------------------------+
                            vs
+-------------------------------------------------------+
| OOP Approach:                                         |
|   [ Object A: State + Methods ]                       |
|          |                                            |
|          | (Controlled Messages / Method Calls)       |
|          v                                            |
|   [ Object B: State + Methods ]                       |
+-------------------------------------------------------+
```

---

## 2. Classes vs. Objects (Memory & Mechanics)

| Feature | Class | Object |
| :--- | :--- | :--- |
| **Definition** | A user-defined blueprint, template, or prototype. | A concrete instance of a class created at runtime. |
| **Memory Allocation** | Takes **no heap memory** for data fields (only metadata/bytecode loaded in Method Area / Text segment). | Takes **actual heap memory** for its instance variables. |
| **Existence** | Logical construct. | Physical entity in memory. |
| **Declaration** | Declared using the `class` keyword. | Created using `new` (in Java/C++) or constructor call. |

### Memory Allocation: Stack vs. Heap
When you execute:
```java
BankAccount acc = new BankAccount("Alice", 5000);
```
1. **Stack Memory:** Holds the reference variable `acc` (a 4-byte or 8-byte memory pointer).
2. **Heap Memory:** Allocates the actual `BankAccount` object containing the instance variables (`ownerName = "Alice"`, `balance = 5000`).
3. **Metaspace / Code Segment:** Stores class metadata, bytecode, and method instructions.

---

## 3. The Four Pillars of OOP

```
                +---------------------------------------+
                |           THE 4 PILLARS OF OOP        |
                +---------------------------------------+
                   |           |           |          |
         +---------+     +-----+-----+     +----+     +---------+
         |               |                 |                    |
    Encapsulation   Abstraction       Inheritance          Polymorphism
   (Data Hiding)    (Simplicity)     (Code Reusability)   (Flexibility)
```

---

### Pillar 1: Encapsulation

#### 1. What is it?
Encapsulation is the bundling of data (attributes) and the methods (behaviors) that manipulate that data into a single unit (class), while **restricting direct access** to the internal state from the outside world.

#### 2. The Real-World Interview Analogy
> **Analogy:** A **Medical Capsule** or an **ATM Machine**.  
> An ATM encapsulates the vault, currency counters, and transaction ledger. You cannot open the vault doors directly to pull cash. You must interact through a secure, validated interface (screen, keypad, PIN). The machine validates your balance and limits before updating its internal ledger.

#### 3. How to Explain in an Interview
> *"Encapsulation is about protective wrapping. We make fields `private` and expose controlled access via `public` getters and setters. This gives us complete control over data validation, prevents invalid state transitions, and decouples the internal representation from the public API."*

#### 4. Code Example
```java
public class BankAccount {
    // 1. Private data: Hidden from direct external modification
    private String accountNumber;
    private double balance;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        } else {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
    }

    // 2. Controlled access: Business validation rules enforced
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > this.balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        this.balance -= amount;
    }

    // Read-only getter for balance
    public double getBalance() {
        return this.balance;
    }
}
```

#### 5. Industry Application
- **Domain Entities:** In microservices (e.g., Order Service), an `Order` object ensures its `status` cannot transition directly from `PLACED` to `REFUNDED` without passing through `CANCELLED` or `RETURNED`.

---

### Pillar 2: Abstraction

#### 1. What is it?
Abstraction is the concept of hiding complex, low-level implementation details and exposing only the essential, high-level features or interfaces to the user.

#### 2. The Real-World Interview Analogy
> **Analogy:** A **Car's Accelerator Pedal** or a **Television Remote**.  
> When you press the accelerator pedal, the car speeds up. You do not need to know fuel injection timing, throttle position sensor voltages, or combustion cylinder mechanics. The pedal provides an **abstract interface** to a complex mechanical system.

#### 3. How to Explain in an Interview
> *"Abstraction focuses on **WHAT** an object does rather than **HOW** it does it. It reduces cognitive complexity and enables loose coupling. We achieve abstraction through Interfaces and Abstract Classes. The caller depends on an abstract contract, allowing the underlying implementation to change without breaking consumers."*

#### 4. Code Example (Payment Gateway)
```java
// Contract: WHAT needs to be done
public interface PaymentGateway {
    PaymentResponse processPayment(double amount, String currency);
    boolean refundPayment(String transactionId, double amount);
}

// HOW it is done for Stripe
public class StripePaymentGateway implements PaymentGateway {
    @Override
    public PaymentResponse processPayment(double amount, String currency) {
        // Complex Stripe API handshake, tokenization, TLS handshake
        System.out.println("Processing $" + amount + " through Stripe API...");
        return new PaymentResponse("SUCCESS", "STRIPE_TX_12345");
    }

    @Override
    public boolean refundPayment(String transactionId, double amount) {
        // Stripe-specific refund logic
        return true;
    }
}

// HOW it is done for PayPal
public class PayPalPaymentGateway implements PaymentGateway {
    @Override
    public PaymentResponse processPayment(double amount, String currency) {
        // Complex PayPal OAuth & REST API calls
        System.out.println("Processing $" + amount + " through PayPal API...");
        return new PaymentResponse("SUCCESS", "PAYPAL_TX_98765");
    }

    @Override
    public boolean refundPayment(String transactionId, double amount) {
        // PayPal-specific refund logic
        return true;
    }
}

// Client depends ONLY on the abstraction
public class CheckoutService {
    private final PaymentGateway paymentGateway;

    // Injected at runtime (Dependency Injection)
    public CheckoutService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void completeOrder(double total) {
        PaymentResponse response = paymentGateway.processPayment(total, "USD");
        System.out.println("Order status: " + response.getStatus());
    }
}
```

#### 5. Industry Application
- **JDBC / Database Drivers:** Your application code calls `connection.prepareStatement()`. Whether the database is PostgreSQL, MySQL, or Oracle, the interface is identical; the vendor driver implements the internal logic.

---

### Pillar 3: Inheritance

#### 1. What is it?
Inheritance is the mechanism by which one class (derived/subclass/child) acquires the properties and behaviors of another class (base/superclass/parent), enabling code reusability and establishing an **IS-A** relationship.

#### 2. Types of Inheritance
1. **Single Inheritance:** `A -> B`
2. **Multilevel Inheritance:** `A -> B -> C`
3. **Hierarchical Inheritance:** `A -> B` and `A -> C`
4. **Multiple Inheritance:** `A, B -> C` *(Supported directly in C++; supported via Interfaces in Java to prevent ambiguity)*
5. **Hybrid Inheritance:** Combination of two or more types.

```
Single:            Multilevel:         Hierarchical:          Multiple:
  [Parent]           [Base]               [Parent]          [Parent A]  [Parent B]
     |                 |                   /    \                 \        /
     v                 v                  v      v                 v      v
  [Child]          [Derived 1]        [Child 1] [Child 2]           [Child]
                       |                                       (Diamond Problem)
                       v
                  [Derived 2]
```

#### 3. The Dreaded "Diamond Problem"
Occurs when class `D` inherits from both `B` and `C`, and both `B` and `C` inherit from class `A`. If `A` has a method `foo()`, and both `B` and `C` override `foo()`, which version does `D` inherit?

```
       [Class A]
       foo() { ... }
        /        \
       /          \
   [Class B]    [Class C]
   foo() {...}  foo() {...}
       \          /
        \        /
        [Class D]  <-- Which foo() does D call? Ambiguity!
```

- **In C++:** Resolved using `virtual` base classes (`class B : virtual public A`).
- **In Java:** Classes can inherit from only one superclass (`extends`), eliminating multiple inheritance of state. Multiple inheritance of type is supported via `interfaces`. If two interfaces provide conflicting `default` methods, Java forces the implementing class to explicitly override and resolve the conflict.

#### 4. The Real-World Interview Analogy
> **Analogy:** **Vehicle Hierarchy** in a Logistics Platform.  
> A `Vehicle` base class defines `licensePlate`, `fuelCapacity`, and `startEngine()`. An `ElectricTruck` inherits from `Truck`, which inherits from `Vehicle`. It reuses engine and registration logic while adding its own `batteryCapacity` and `regenerativeBraking()`.

#### 5. Interview Pitch & Golden Rule
> *"Inheritance gives us code reuse, but tight coupling is a significant risk. If a parent class changes, children can break. Therefore, modern software engineering favors **Composition over Inheritance** ('HAS-A' over 'IS-A') unless there is a genuine subtyping hierarchy."*

---

### Pillar 4: Polymorphism

#### 1. What is it?
Derived from Greek meaning *"many forms"*, polymorphism is the ability of an object, function, or operator to take on multiple forms depending on the context.

#### 2. The Two Primary Types

```
                           POLYMORPHISM
                                |
        +-----------------------+-----------------------+
        |                                               |
  Compile-Time Polymorphism                    Run-Time Polymorphism
  (Static / Early Binding)                    (Dynamic / Late Binding)
        |                                               |
  - Method Overloading                         - Method Overriding
  - Operator Overloading (C++/Python)          - Virtual Functions (C++)
```

| Dimension | Compile-Time (Static) | Run-Time (Dynamic) |
| :--- | :--- | :--- |
| **Resolution Time** | Determined by compiler at compile time. | Determined by JVM / runtime based on actual object. |
| **Mechanism** | Method signature matching (name + parameter types). | Virtual Method Table (`vtable`) lookup via object pointer. |
| **Performance** | Faster (no runtime lookup overhead). | Slight overhead due to dynamic dispatch table lookup. |
| **Flexibility** | High speed, less dynamic. | Enables true runtime extensibility (plugins, strategies). |

#### 3. Method Overloading vs. Overriding

```java
// Compile-time Polymorphism (Overloading): Same method name, different parameter signature
public class MathUtils {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

// Run-time Polymorphism (Overriding): Subclass provides specific implementation of parent method
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a generic sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Dog barks: Woof Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows: Meow!");
    }
}

// Runtime Usage:
Animal myPet = new Dog(); // Upcasting
myPet.makeSound();        // Output: "Dog barks: Woof Woof!" (Resolved at runtime)
```

#### 4. The Real-World Interview Analogy
> **Analogy:** A **USB Port** or a **Person's Identity**.  
> - **Real-world:** You are a single person, but you act as an *Employee* at the office, a *Customer* at a grocery store, and a *Driver* in your car. Your behavior changes based on context.  
> - **Software system:** A `NotificationService` calls `send(message)`. At runtime, the receiver can be `EmailNotification`, `SMSNotification`, or `PushNotification`. The calling service treats them uniformly as `Notification`.

---

## 4. Critical Interview Face-Offs

### Encapsulation vs. Abstraction
*This is the most common conceptual trap in SDE-1 interviews.*

| Feature | Encapsulation | Abstraction |
| :--- | :--- | :--- |
| **Core Question** | **HOW** is data protected and managed? | **WHAT** does the object do? |
| **Primary Goal** | **Data Hiding & Security:** Bundles state and methods, preventing unauthorized tampering. | **Complexity Reduction:** Exposes the interface and hides implementation mechanics. |
| **Implementation** | Access modifiers (`private`, `protected`, `public`), getters/setters. | Interfaces and Abstract classes. |
| **Focus** | Internal implementation level. | Outer architectural / contract level. |
| **Example** | Making `balance` private and validating deposits. | Exposing `processPayment()` without revealing the API calls. |

---

### Abstract Class vs. Interface

```
+--------------------------------------------------------------------------+
|                        ABSTRACT CLASS vs. INTERFACE                      |
+--------------------------------------------------------------------------+
| Feature             | Abstract Class               | Interface           |
|---------------------|------------------------------|---------------------|
| Multiple Inheritance| Single class inheritance     | Multiple interfaces |
| State / Fields      | Can have instance fields     | Only public static  |
|                     | with mutable state           | final constants     |
| Constructors        | Has constructors             | Cannot have         |
|                     | (called via super())         | constructors        |
| Methods             | Concrete, abstract, final,   | Abstract methods,   |
|                     | or static methods            | default/static (Java)|
| Speed               | Faster (direct dispatch)     | Slight table search |
| Relationship        | IS-A (Identity)              | CAN-DO (Capability) |
+--------------------------------------------------------------------------+
```

> **Interview Pro-Tip:** Use an **Interface** when you want to define a contract of *capabilities* across unrelated classes (e.g., `Comparable`, `Serializable`, `Payable`). Use an **Abstract Class** when classes share a common identity, core state, and non-trivial code implementation (e.g., `BaseController`, `AbstractRepository`).

---

### Inheritance (IS-A) vs. Composition (HAS-A)

```java
// Inheritance: Tightly coupled (Fragile Base Class problem)
public class SuperQueue extends ArrayList { ... } 

// Composition: Loosely coupled (Design Best Practice)
public class SuperQueue {
    private final List<Object> internalList = new ArrayList<>(); // HAS-A
    
    public void enqueue(Object item) {
        internalList.add(item);
    }
}
```

#### Why favor Composition over Inheritance?
1. **Encapsulation Preservation:** Inheritance exposes parent internals to subclasses; composition respects encapsulation through clean interfaces.
2. **Runtime Flexibility:** Composition allows switching behavior at runtime by swapping injected references (Strategy Pattern).
3. **Avoids Class Explosion:** Avoids massive, brittle deep inheritance hierarchies (`Vehicle -> LandVehicle -> WheeledLandVehicle -> Car`).

---

### Shallow Copy vs. Deep Copy

```
Original Object: [ Person: name="Bob", addressRef=0xABCD ]
                                            |
                                            v
                                [ Address: city="Seattle" ]

Shallow Copy:    [ Person: name="Bob", addressRef=0xABCD ] 
                 (Points to the exact SAME address object!)

Deep Copy:       [ Person: name="Bob", addressRef=0x9999 ] 
                                            |
                                            v
                                [ Address: city="Seattle" ] 
                 (Cloned into a fresh, distinct memory block!)
```

```java
public class Person implements Cloneable {
    String name;
    Address address;

    // Shallow Copy
    public Person shallowCopy() {
        return new Person(this.name, this.address);
    }

    // Deep Copy
    public Person deepCopy() {
        Address copiedAddress = new Address(this.address.getCity(), this.address.getZip());
        return new Person(this.name, copiedAddress);
    }
}
```
- **Bug Scenario:** In a shallow copy, changing the city on the copied person mutates the original person's city, leading to critical state corruption.

---

## 5. Relationships Between Objects

Understanding these relationships is essential for Low-Level Design (LLD) interviews:

```
[Dependency]  ---> Uses temporarily (method parameter)
      |
[Association] ---> Knows / interacts with (field reference)
      |
[Aggregation] ---> HAS-A relationship (Weak: Child survives without Parent)
      |
[Composition] ---> HAS-A relationship (Strong: Child dies with Parent)
      |
[Inheritance] ---> IS-A relationship
```

1. **Dependency ("Uses-A"):** A class uses another temporarily inside a method.
   ```java
   class Printer {
       void print(Document doc) { /* Document passed as parameter */ }
   }
   ```
2. **Association ("Knows-A"):** Long-term link between two independent classes (e.g., `Teacher` and `Student`). Both exist independently.
3. **Aggregation ("Has-A" - Weak):** Whole-part relationship where parts can exist independently.
   * *Example:* A `Department` has `Professors`. If the department is closed, the professors still exist.
4. **Composition ("Has-A" - Strong):** Whole-part relationship where parts **cannot** exist without the whole.
   * *Example:* A `House` has `Rooms`. If the house is demolished, the rooms cease to exist.

---

## 6. SOLID Principles (Interview Edition)

The SOLID principles are OOP best practices that ensure software is scalable, maintainable, and testable.

### 1. Single Responsibility Principle (SRP)
> *"A class should have one, and only one, reason to change."*
- **Violation:** A `UserRegistrationService` that validates input, inserts records into MySQL, and formats & sends welcome emails.
- **Fix:** Split into `UserValidator`, `UserRepository`, and `EmailNotificationService`.

### 2. Open/Closed Principle (OCP)
> *"Software entities should be open for extension, but closed for modification."*
- **Violation:** Using a giant `switch-case` statement over payment types inside `processPayment()`. Adding a new payment type requires modifying existing, tested code.
- **Fix:** Define a `PaymentMethod` interface and implement new classes (`CreditCardPayment`, `CryptoPayment`).

### 3. Liskov Substitution Principle (LSP)
> *"Subtypes must be substitutable for their base types without altering program correctness."*
- **Classic Violation:** The **Square-Rectangle Problem**. A `Square` extends `Rectangle`. Overriding `setWidth(w)` also modifies height, breaking callers who assume width and height are independent.
- **Fix:** Do not inherit `Square` from `Rectangle`. Have both implement a common `Shape` interface.

### 4. Interface Segregation Principle (ISP)
> *"Clients should not be forced to depend on interfaces they do not use."*
- **Violation:** A massive `Worker` interface with `work()`, `eat()`, and `sleep()`. A `Robot` class implementing `Worker` is forced to implement dummy `eat()` and `sleep()` methods.
- **Fix:** Split into `Workable`, `Feedable`, and `Restable`.

### 5. Dependency Inversion Principle (DIP)
> *"High-level modules should not depend on low-level modules. Both should depend on abstractions."*
- **Violation:** `NotificationManager` directly instantiates `new TwilioSmsSender()`.
- **Fix:** `NotificationManager` depends on `MessageSender` interface; `TwilioSmsSender` implements it and is injected via constructor.

---

## 7. Under The Hood: Virtual Tables (VTable & VPTR)

*Interviewers ask this when interviewing in C++ or probing deep language mechanics.*

### How Dynamic Dispatch Actually Works
When a language encounters:
```cpp
Base* obj = new Derived();
obj->calculate(); // How does it know to call Derived::calculate()?
```

1. **VTable (Virtual Table):** A static array of function pointers constructed by the compiler for every class that declares or inherits virtual functions.
2. **VPTR (Virtual Pointer):** A hidden pointer injected as the first member of every object instance of that class.
3. **Execution Steps:**
   - Program inspects the object at runtime.
   - Accesses the hidden `vptr` stored in the object.
   - Follows `vptr` to the class's `vtable`.
   - Fetches the function pointer at the specific offset and executes `Derived::calculate()`.

```
Instance on Heap:
[ VPTR ] -----------> VTable for Derived Class:
[ field1 ]             [0] &Derived::calculate()
[ field2 ]             [1] &Derived::render()
```

---

## 8. Real-World Case Study: Notification System (LLD)

Let's assemble all 4 pillars and SOLID principles into a clean Low-Level Design that you can sketch on an interview whiteboard.

### Problem Statement
Design an enterprise-grade notification service capable of dispatching alerts via **Email**, **SMS**, and **Push Notification** with rate limiting and recipient validation.

```
                  +-------------------------+
                  |  <<interface>>          |
                  |     NotificationChannel |
                  +-------------------------+
                  | + send(msg, recipient)  |
                  +-------------------------+
                               ^
         +---------------------+---------------------+
         |                     |                     |
+-----------------+   +-----------------+   +-----------------+
|  EmailChannel   |   |   SMSChannel    |   |   PushChannel   |
+-----------------+   +-----------------+   +-----------------+
| - smtpClient    |   | - twilioClient  |   | - fcmClient     |
+-----------------+   +-----------------+   +-----------------+
| + send(...)     |   | + send(...)     |   | + send(...)     |
+-----------------+   +-----------------+   +-----------------+
         ^
         | HAS-A (Composition)
+-------------------------+
|   NotificationService   |
+-------------------------+
| - channels: List        |
+-------------------------+
| + notifyAll(...)        |
+-------------------------+
```

### Implementation (Java)
```java
import java.util.List;
import java.util.ArrayList;

// 1. Abstraction: Contract for all notification media
public interface NotificationChannel {
    void sendNotification(String message, String recipient);
    String getChannelType();
}

// 2. Polymorphic Implementation: Email
public class EmailNotificationChannel implements NotificationChannel {
    @Override
    public void sendNotification(String message, String recipient) {
        // Validation + Encapsulated logic for SMTP
        System.out.println("[EMAIL] Sent to " + recipient + ": " + message);
    }

    @Override
    public String getChannelType() {
        return "EMAIL";
    }
}

// 3. Polymorphic Implementation: SMS
public class SmsNotificationChannel implements NotificationChannel {
    @Override
    public void sendNotification(String message, String recipient) {
        // Encapsulated logic for SMS Gateway (e.g. Twilio)
        System.out.println("[SMS] Sent to " + recipient + ": " + message);
    }

    @Override
    public String getChannelType() {
        return "SMS";
    }
}

// 4. Encapsulation & Dependency Inversion: Service coordinates channels
public class NotificationService {
    // Composition over Inheritance: Holds a list of channels
    private final List<NotificationChannel> channels;

    public NotificationService() {
        this.channels = new ArrayList<>();
    }

    // Dynamic registration of channels (Open/Closed Principle)
    public void registerChannel(NotificationChannel channel) {
        this.channels.add(channel);
    }

    public void broadcast(String message, String recipient) {
        for (NotificationChannel channel : channels) {
            channel.sendNotification(message, recipient);
        }
    }
}

// Driver Execution
public class Main {
    public static void main(String[] args) {
        NotificationService service = new NotificationService();
        
        // Polymorphic injection
        service.registerChannel(new EmailNotificationChannel());
        service.registerChannel(new SmsNotificationChannel());

        service.broadcast("Your OTP is 492011", "user@example.com / +19876543210");
    }
}
```

---

## 9. Top 15 Tricky Interview Questions & Model Answers

### Q1: Can constructors be private? Why would you make one private?
**Answer:** Yes. A constructor is made `private` to restrict direct instantiation from outside the class.
- **Primary Use Cases:**
  1. **Singleton Pattern:** Ensure only a single instance of a class exists (`getInstance()`).
  2. **Utility Classes:** Classes with only static methods (e.g., `java.lang.Math`, `java.util.Collections`) should not be instantiated.
  3. **Factory Method Pattern:** Force object construction through descriptive static factory methods (e.g., `ComplexNumber.fromCartesian(x, y)` vs `ComplexNumber.fromPolar(r, theta)`).

---

### Q2: Can you override a `static` or `private` method in Java?
**Answer:** No.
- **Private methods** are not visible to child classes; hence, they cannot be overridden. If a child defines a method with the same name, it is a completely independent new method.
- **Static methods** belong to the class, not the instance. If a subclass declares a static method with the exact same signature, it is called **Method Hiding**, not method overriding. Resolution is static (compile-time) based on the reference type, not runtime dynamic dispatch.

---

### Q3: Why is multiple inheritance of classes not supported in Java, but allowed for interfaces?
**Answer:** To eliminate the **Diamond Problem** and ambiguity in state resolution. When two classes inherit common state (fields), a subclass inheriting from both would receive conflicting duplicate copies of variables. Interfaces historically carried no state (no instance variables) and only defined abstract methods, meaning there is only one method implementation to execute regardless of how many interfaces declare the signature.

---

### Q4: What is the difference between early binding and late binding?
**Answer:**
- **Early Binding (Static Binding):** Function call is bound to code at compile time based on the reference type. Occurs for `static`, `private`, and `final` methods, as well as overloaded methods.
- **Late Binding (Dynamic Binding):** Function call is resolved at runtime based on the actual object instance in memory using virtual method tables. Occurs for overridden virtual methods.

---

### Q5: Why should a destructor in C++ be declared `virtual` if a class is intended for inheritance?
**Answer:** If a base class pointer points to a derived class object on the heap, deleting the base pointer (`delete basePtr;`) without a virtual destructor causes **undefined behavior** and a **memory leak**. Only the base class destructor will be invoked, leaving the derived class's allocated resources uncleaned. A `virtual` destructor ensures the destructor chain starts at the derived class and proceeds up to the base class.

---

### Q6: Can an abstract class have a constructor?
**Answer:** Yes. Although you cannot instantiate an abstract class directly using `new AbstractClass()`, its constructor is called during subclass instantiation via `super()` to initialize the abstract class's internal fields and enforce invariant setup.

---

### Q7: What is the difference between Method Overloading and Method Overriding?
**Answer:**
- **Overloading:** Happens within the same class (or subclass) where methods share the same name but differ in their **parameter list** (number, types, or order). Return type does not matter. It is compile-time polymorphism.
- **Overriding:** Occurs across inheritance where a subclass provides a specific implementation of a method already declared in its superclass with the **exact same signature and return type** (or covariant return). It is runtime polymorphism.

---

### Q8: What is Object Object Slicing in C++?
**Answer:** Object slicing occurs when a derived class object is assigned by value to a base class object (`Base b = derivedObj;`). The derived-specific members and vptr are "sliced off", and only the base sub-object is copied, losing polymorphic behavior. It is avoided by using pointers or references (`Base& b = derivedObj;`).

---

### Q9: Can an interface implement another interface? Can an interface extend a class?
**Answer:** An interface can `extend` another interface (and can extend multiple interfaces simultaneously). An interface **cannot** implement another interface (it doesn't provide implementation), and an interface **cannot** extend a class (interfaces define contracts, not class state).

---

### Q10: What is the Liskov Substitution Principle (LSP) and how is it tested in interviews?
**Answer:** LSP states that any function expecting a base class must be able to use a derived class without knowing it and without breaking correctness. Interviewers test this with the **Rectangle vs. Square** problem. A Square violates LSP if derived from Rectangle because setting the width to 5 should not automatically alter the height to 5 in an entity masquerading as a Rectangle.

---

### Q11: What is the difference between Aggregation and Composition?
**Answer:** Both represent "HAS-A" relationships, but differ in lifecycle ownership:
- **Composition (Strong):** The child object's lifecycle is strictly tied to the parent's lifecycle (e.g., `Order` and `OrderItem`). If the parent is destroyed, the child is destroyed.
- **Aggregation (Weak):** The child object has an independent lifecycle outside the parent (e.g., `Department` and `Professor`). If the department is disbanded, the professor continues to exist.

---

### Q12: Can you create an instance of an abstract class using reflection or any other trick?
**Answer:** No. Language runtimes (like the JVM) explicitly reject direct instantiation instructions for classes marked with the abstract modifier flags, throwing an `InstantiationException`. You can only instantiate concrete anonymous or concrete named subclasses.

---

### Q13: What is Cohesion and Coupling in OOP design?
**Answer:**
- **Cohesion:** Measures how focused and closely related the responsibilities inside a single module/class are. We always strive for **High Cohesion** (e.g., a class does one job thoroughly).
- **Coupling:** Measures the degree of interdependence between different modules/classes. We always strive for **Low (Loose) Coupling** (e.g., classes interact via interfaces rather than direct references).

---

### Q14: What is the purpose of the `final` keyword in Java in the context of OOP?
**Answer:**
- **Final Variable:** Value cannot be reassigned once initialized (creates constants).
- **Final Method:** Cannot be overridden by subclasses (prevents altering critical behavior).
- **Final Class:** Cannot be extended/subclassed (e.g., `java.lang.String` is final to preserve immutability and security).

---

### Q15: How would you explain OOP to a 10-year-old? (A favorite soft-skill & communication check)
**Answer:**
> *"Imagine you love LEGOs. A **Class** is the instruction booklet with the blueprint drawing. An **Object** is the actual toy spaceship you snapped together using the plastic bricks. **Encapsulation** means keeping the fragile motors and gears inside the hull so you don't break them with your fingers. **Inheritance** means you can take your spaceship blueprint, add two extra wings, and build a super-fighter without starting from scratch. **Polymorphism** means you have a remote control with a button that says 'GO'—when you point it at your spaceship it flies, and when you point it at your LEGO car it drives!"*

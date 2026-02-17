# SOLID Principles 

SOLID principles - the foundation of writing clean, maintainable, and scalable code.

---

## What are SOLID Principles?

**SOLID** is an acronym for five design principles that make software more:
-  **Understandable**
-  **Flexible**
-  **Maintainable**
-  **Scalable**

### The Five Principles

| Letter | Principle | What it means |
|--------|-----------|---------------|
| **S** | Single Responsibility | One class = One job |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Child classes should work like parent classes |
| **I** | Interface Segregation | Don't force classes to implement unused methods |
| **D** | Dependency Inversion | Depend on abstractions, not concrete classes |

---

## S - Single Responsibility Principle (SRP)

###  Definition

> **A class should have only ONE reason to change.**

Each class should do **ONE thing** and do it well.

###  Bad Example (Violates SRP)

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def save_to_database(self):
        # Saves user to database
        print(f"Saving {self.name} to database...")
    
    def send_email(self):
        # Sends email to user
        print(f"Sending email to {self.email}...")
    
    def generate_report(self):
        # Generates user report
        print(f"Generating report for {self.name}...")
```

**Problem:** This class has **3 responsibilities**:
1. Database operations
2. Email operations
3. Report generation

If email logic changes, we have to modify the User class!

###  Good Example (Follows SRP)

```python
class User:
    """Responsible ONLY for user data"""
    def __init__(self, name, email):
        self.name = name
        self.email = email


class UserDatabase:
    """Responsible ONLY for database operations"""
    def save(self, user):
        print(f"Saving {user.name} to database...")


class EmailService:
    """Responsible ONLY for email operations"""
    def send_email(self, user):
        print(f"Sending email to {user.email}...")


class ReportGenerator:
    """Responsible ONLY for generating reports"""
    def generate(self, user):
        print(f"Generating report for {user.name}...")


# Usage
user = User("Alice", "alice@example.com")
db = UserDatabase()
email = EmailService()
report = ReportGenerator()

db.save(user)
email.send_email(user)
report.generate(user)
```

###  Benefits

-  Easier to test
-  Easier to understand
-  Changes in one area don't affect others
-  Code is more organized

---

## O - Open/Closed Principle (OCP)

### 📖 Definition

> **Classes should be OPEN for extension but CLOSED for modification.**

You should be able to add new functionality **without changing existing code**.

###  Bad Example (Violates OCP)

```python
class PaymentProcessor:
    def process_payment(self, payment_type, amount):
        if payment_type == "credit_card":
            print(f"Processing ${amount} via Credit Card")
        elif payment_type == "paypal":
            print(f"Processing ${amount} via PayPal")
        elif payment_type == "bitcoin":
            print(f"Processing ${amount} via Bitcoin")
        # Every time we add a new payment method, we modify this class!


# Usage
processor = PaymentProcessor()
processor.process_payment("credit_card", 100)
```

**Problem:** Every time we add a new payment method, we have to **modify** this class.

###  Good Example (Follows OCP)

```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    """Abstract base class"""
    @abstractmethod
    def process(self, amount):
        pass


class CreditCardPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing ${amount} via Credit Card")


class PayPalPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing ${amount} via PayPal")


class BitcoinPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing ${amount} via Bitcoin")


class PaymentProcessor:
    def process_payment(self, payment_method: PaymentMethod, amount):
        payment_method.process(amount)


# Usage
processor = PaymentProcessor()
processor.process_payment(CreditCardPayment(), 100)
processor.process_payment(PayPalPayment(), 200)
processor.process_payment(BitcoinPayment(), 300)

# Adding new payment method - NO MODIFICATION needed!
class GooglePayPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing ${amount} via Google Pay")

processor.process_payment(GooglePayPayment(), 150)
```

###  Benefits

-  Add new features without breaking existing code
-  Reduces bugs
-  Makes code more flexible

---

## L - Liskov Substitution Principle (LSP)

### 📖 Definition

> **Objects of a parent class should be replaceable with objects of a child class without breaking the application.**

Child classes must be able to do **everything** the parent class can do.

###  Bad Example (Violates LSP)

```python
class Bird:
    def fly(self):
        print("Flying in the sky")


class Sparrow(Bird):
    def fly(self):
        print("Sparrow flying")


class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly!")
        # This breaks LSP - Penguin can't replace Bird


# Usage
def make_bird_fly(bird: Bird):
    bird.fly()

sparrow = Sparrow()
penguin = Penguin()

make_bird_fly(sparrow)  #  Works
make_bird_fly(penguin)  #  Crashes!
```

**Problem:** Penguin is a Bird but can't fly! This breaks the contract.

###  Good Example (Follows LSP)

```python
class Bird:
    def move(self):
        pass


class FlyingBird(Bird):
    def move(self):
        self.fly()
    
    def fly(self):
        print("Flying in the sky")


class Sparrow(FlyingBird):
    def fly(self):
        print("Sparrow flying")


class Penguin(Bird):
    def move(self):
        self.swim()
    
    def swim(self):
        print("Penguin swimming")


# Usage
def make_bird_move(bird: Bird):
    bird.move()

sparrow = Sparrow()
penguin = Penguin()

make_bird_move(sparrow)  #  Sparrow flies
make_bird_move(penguin)  #  Penguin swims
```

###  Real-World Analogy

Think of a **remote control**:
- Any remote (child) should work with the TV (parent)
- If a remote doesn't have a volume button when all remotes should have one, it violates LSP

###  Benefits

-  Predictable behavior
-  Fewer surprises and bugs
-  Code is more reliable

---

## I - Interface Segregation Principle (ISP)

### 📖 Definition

> **No class should be forced to implement methods it doesn't use.**

Better to have many **small, specific interfaces** than one large, general interface.

###  Bad Example (Violates ISP)

```python
from abc import ABC, abstractmethod

class Worker(ABC):
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass


class Human(Worker):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating lunch")


class Robot(Worker):
    def work(self):
        print("Robot working")
    
    def eat(self):
        # Robots don't eat! But we're forced to implement this
        raise Exception("Robots don't eat!")
```

**Problem:** Robot is forced to implement `eat()` even though robots don't eat!

###  Good Example (Follows ISP)

```python
from abc import ABC, abstractmethod

class Workable(ABC):
    @abstractmethod
    def work(self):
        pass


class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass


class Human(Workable, Eatable):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating lunch")


class Robot(Workable):
    def work(self):
        print("Robot working")
    # No need to implement eat() - Robot only implements what it needs!


# Usage
def make_work(worker: Workable):
    worker.work()

human = Human()
robot = Robot()

make_work(human)  #  Works
make_work(robot)  #  Works
```

###  Real-World Analogy

Think of a **multi-function printer**:
- Some printers can: print, scan, fax
- Some printers can only: print

Don't force a simple printer to have fax capability it doesn't need!

###  Benefits

-  Classes only implement what they need
-  Smaller, focused interfaces
-  Easier to understand and maintain

---

## D - Dependency Inversion Principle (DIP)

### 📖 Definition

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

Don't depend on **concrete classes**, depend on **interfaces/abstractions**.

###  Bad Example (Violates DIP)

```python
class MySQLDatabase:
    def save(self, data):
        print(f"Saving {data} to MySQL database")


class UserService:
    def __init__(self):
        # UserService directly depends on MySQLDatabase (concrete class)
        self.database = MySQLDatabase()
    
    def save_user(self, user):
        self.database.save(user)


# Usage
service = UserService()
service.save_user("Alice")

# Problem: If we want to switch to PostgreSQL, we have to modify UserService!
```

**Problem:** `UserService` is tightly coupled to `MySQLDatabase`. Hard to switch databases!

###  Good Example (Follows DIP)

```python
from abc import ABC, abstractmethod

# Abstraction (Interface)
class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass


# Concrete implementations
class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saving {data} to MySQL database")


class PostgreSQLDatabase(Database):
    def save(self, data):
        print(f"Saving {data} to PostgreSQL database")


class MongoDBDatabase(Database):
    def save(self, data):
        print(f"Saving {data} to MongoDB database")


# High-level module depends on abstraction
class UserService:
    def __init__(self, database: Database):
        self.database = database
    
    def save_user(self, user):
        self.database.save(user)


# Usage - Easy to switch databases!
mysql_db = MySQLDatabase()
postgres_db = PostgreSQLDatabase()
mongo_db = MongoDBDatabase()

service1 = UserService(mysql_db)
service1.save_user("Alice")

service2 = UserService(postgres_db)
service2.save_user("Bob")

service3 = UserService(mongo_db)
service3.save_user("Charlie")
```

### Real-World Analogy

Think of a **USB port**:
- Your laptop doesn't depend on a specific mouse or keyboard
- It depends on the **USB interface**
- Any USB device works!

###  Benefits

-  Easy to swap implementations
-  Loose coupling
-  Better testability (can use mock objects)
-  More flexible code

---

## Quick Reference

### SOLID in One Sentence Each

| Principle | One-Liner |
|-----------|-----------|
| **S** - Single Responsibility | One class, one job |
| **O** - Open/Closed | Add features without changing existing code |
| **L** - Liskov Substitution | Child classes should work like parent classes |
| **I** - Interface Segregation | Don't force unused methods on classes |
| **D** - Dependency Inversion | Depend on interfaces, not concrete classes |

### Quick Decision Tree

```
Are you writing a new class
├─ Does it do ONE thing? ➜ SRP
├─ Can you extend it without modifying? ➜ OCP 
└─ Is it an interface/abstraction? ➜ DIP 

Are you creating a child class?
└─ Can it replace the parent everywhere? ➜ LSP 

Are you creating an interface?
└─ Does it have only necessary methods? ➜ ISP 
```

---

## Benefits of SOLID

###  Why Follow SOLID Principles?

| Benefit | Description |
|---------|-------------|
| **Maintainability** | Code is easier to update and fix |
| **Scalability** | Easy to add new features |
| **Testability** | Easier to write unit tests |
| **Reusability** | Components can be reused in different contexts |
| **Readability** | Code is cleaner and easier to understand |
| **Flexibility** | Easy to swap implementations |
| **Reduced Bugs** | Changes in one area don't break others |

### Before vs After SOLID

| Without SOLID | With SOLID |
|---------------|------------|
|  Hard to understand |  Easy to understand |
|  Many bugs |  Fewer bugs |
|  Tightly coupled |  Loosely coupled |
|  Hard to test |  Easy to test |
|  Rigid code |  Flexible code |
|  Complex changes |  Simple changes |

---

## Complete Example: E-Commerce System

Let's see how all SOLID principles work together:

###  Without SOLID

```python
class OrderProcessor:
    def process_order(self, order_type, items, email):
        # Calculate total
        total = sum([item['price'] for item in items])
        
        # Apply discount
        if order_type == "premium":
            total *= 0.9
        elif order_type == "regular":
            total *= 0.95
        
        # Save to database
        print(f"Saving order to MySQL: ${total}")
        
        # Send email
        print(f"Sending email to {email}")
        
        # Process payment
        print("Processing credit card payment")
        
        return total
```

**Problems:**
-  Does too many things (SRP)
-  Hard to add new discount types (OCP)
-  Tightly coupled to MySQL and email (DIP)

###  With SOLID

```python
from abc import ABC, abstractmethod

# S - Each class has ONE responsibility
# D - Using abstractions

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate_discount(self, total):
        pass


class PremiumDiscount(DiscountStrategy):
    def calculate_discount(self, total):
        return total * 0.9


class RegularDiscount(DiscountStrategy):
    def calculate_discount(self, total):
        return total * 0.95


class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass


class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saving to MySQL: {data}")


class NotificationService(ABC):
    @abstractmethod
    def send(self, recipient, message):
        pass


class EmailService(NotificationService):
    def send(self, recipient, message):
        print(f"Sending email to {recipient}: {message}")


class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount):
        pass


class CreditCardProcessor(PaymentProcessor):
    def process(self, amount):
        print(f"Processing ${amount} via Credit Card")


# O - Easy to extend with new functionality
# L - Can substitute different implementations
# I - Each interface has only relevant methods
# D - Depends on abstractions

class OrderService:
    def __init__(
        self,
        database: Database,
        notification: NotificationService,
        payment: PaymentProcessor
    ):
        self.database = database
        self.notification = notification
        self.payment = payment
    
    def process_order(self, items, discount: DiscountStrategy, email):
        total = sum([item['price'] for item in items])
        discounted_total = discount.calculate_discount(total)
        
        self.database.save(f"Order: ${discounted_total}")
        self.notification.send(email, f"Order confirmed: ${discounted_total}")
        self.payment.process(discounted_total)
        
        return discounted_total


# Usage - Everything is pluggable!
order_service = OrderService(
    database=MySQLDatabase(),
    notification=EmailService(),
    payment=CreditCardProcessor()
)

items = [{'name': 'Book', 'price': 20}, {'name': 'Pen', 'price': 5}]
order_service.process_order(items, PremiumDiscount(), "customer@example.com")

# Easy to switch implementations!
# Want PostgreSQL? Just pass PostgreSQLDatabase()
# Want SMS? Just pass SMSService()
# Want PayPal? Just pass PayPalProcessor()
```



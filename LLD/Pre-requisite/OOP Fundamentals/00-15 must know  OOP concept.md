In this chapter, we'll break down the 15 most important OOP concepts using simple explanations, practical examples, and code.

To make them easier to understand, the concepts are divided into three groups:

The basic building blocks
The four pillars of OOP
Class relationships


What is Object-Oriented Programming?
Object-oriented programming is a way of organizing software around objects. Each object contains two main things:

State: the data it stores
Behavior: the actions it can perform

For example, an Order object might store its items and current status. It might also provide methods to calculate the total, add an item, or cancel the order.

A complete application is built from many such objects working together, with each object handling a specific part of the system.

With that basic idea in mind, let's start with the building blocks you'll see in almost every object-oriented codebase.


Building Blocks

1. Class
A class is a blueprint for creating objects. It defines the data an object stores and the actions it can perform


class User {
    String name;
    String email;

    void login() {
        System.out.println(name + " logged in");
    }
}

This class defines that every user will have a name, an email, and a login behavior.

But a class is only a blueprint. It does not represent an actual user until we create something from it. That brings us to objects.


2. Object
An object is an instance of a class. If a class is the blueprint, an object is the actual thing created from that blueprint.

User user1 = new User();
user1.name = "Alice";
user1.email = "alice@example.com";

User user2 = new User();
user2.name = "Bob";
user2.email = "bob@example.com";

user1.login(); // Alice logged in
user2.login(); // Bob logged in

Here, both user1 and user2 are objects of the same User class. They have the same structure and behavior, but they store different data.

So, a class defines the template, while objects are the actual instances created from it.


3. Constructor
Setting every field manually while creating an object can become repetitive and may lead to creating incomplete objects:


User user1 = new User();
user1.name = "Alice";
user1.email = "alice@example.com";

User user2 = new User();
user2.name = "Bob";
// forgot user2.email

This is where constructors help.

A constructor is a special method that runs when an object is created. Its main purpose is to initialize the object with the required data.

class User {
    String name;
    String email;

    User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    void login() {
        System.out.println(name + " logged in");
    }
}

Now creating both users takes a single line each, and neither can be left incomplete:

User user1 = new User("Alice", "alice@example.com");

User user2 = new User("Bob", "bob@example.com");

Constructors also help ensure that objects start in a valid state. We can add validation inside the constructor to prevent invalid data from being used:


User(String name, String email) {
    if (email == null || !email.contains("@")) {
        throw new IllegalArgumentException("Invalid email: " + email);
    }
    this.name = name;
    this.email = email;
}


4. Enum
So far, we have looked at creating objects and storing data inside them. But sometimes, a field should only accept a small set of predefined values.

For that, we can use enums. An enum is useful when a value can only be one of a fixed set of options.

For example, think about an order in an e-commerce application. An order might be pending, shipped, delivered, or cancelled.

We could represent the status using a string:

String status = "pending";

// somewhere else in the code
status = "shiped"; // typo, but the code still compiles

But strings are easy to mistype, and the compiler will not catch this mistake.

A safer approach is to use an enum:

enum OrderStatus {
    PENDING, SHIPPED, DELIVERED, CANCELLED
}

OrderStatus status = OrderStatus.PENDING;
status = OrderStatus.SHIPPED; // only valid values are allowed

This prevents unsupported values and makes the code easier to understand.

Whenever you have a fixed set of values, such as OrderStatus, PaymentStatus, UserRole, or DifficultyLevel, enums are usually a good choice


5. Access Modifiers
Now that we can define data, we also need to control which parts of a class should be visible from outside. That is where access modifiers come in.

Access modifiers control where a class, field, constructor, or method can be accessed.

In Java, the three access modifiers you will use most often are public, private, and protected.

Modifier	Accessible From
public	Anywhere in the application
private	Only inside the same class
protected	The same package and child classes


class UserAccount {
    private String email;

    public String getEmail() {
        return email;
    }

    public void updateEmail(String newEmail) {
        if (isValidEmail(newEmail)) {
            email = newEmail;
            logChange();
        }
    }

    protected void logChange() {
        System.out.println("Account updated");
    }

    private boolean isValidEmail(String email) {
        return email.contains("@");
    }
}


Here, getEmail and updateEmail are public because other parts of the application need to use them.

isValidEmail is private because it is an internal implementation detail.

logChange is protected, which means a child class could reuse or customize it.

In general, expose only what other parts of the application actually need. Keep everything else private whenever possible.


6. Interface
Access modifiers control visibility within a class. But in larger applications, we also need a way to define behavior that different classes can provide in their own way.

That brings us to interfaces.

An interface defines a contract. It tells us what a class must do without deciding exactly how it should do it.

interface FileStorage {
    void save(String fileName, byte[] data);
}

This interface says that every file-storage implementation must provide a save method.

We can now create different implementations:

class LocalDiskStorage implements FileStorage {
    public void save(String fileName, byte[] data) {
        System.out.println("Saving " + fileName + " to local disk");
    }
}

class CloudStorage implements FileStorage {
    public void save(String fileName, byte[] data) {
        System.out.println("Uploading " + fileName + " to cloud storage");
    }
}


The rest of the application can depend on the interface instead of a specific storage system:


class BackupService {
    private FileStorage storage;

    BackupService(FileStorage storage) {
        this.storage = storage;
    }

    void backup(String fileName, byte[] data) {
        storage.save(fileName, data);
    }
}


Now BackupService can work with local storage, cloud storage, or any future implementation of FileStorage.

We can change the storage provider without rewriting the backup logic. That makes the code more flexible and easier to test.


7. Abstract Class
Interfaces work well when different classes need to follow the same contract. But sometimes, related classes also need to share common code or state.

That is where abstract classes become useful.

An abstract class is a class that cannot be instantiated directly. It can provide shared fields and methods while leaving some behavior for child classes to implement.


For example, imagine we need to generate reports in different formats:

abstract class ReportGenerator {

    void generate(List<String> data) {
        String content = format(data);
        save(content);
    }

    void save(String content) {
        System.out.println("Saving report: " + content);
    }

    abstract String format(List<String> data);
}

class CsvReportGenerator extends ReportGenerator {
    String format(List<String> data) {
        return String.join(",", data);
    }
}

class JsonReportGenerator extends ReportGenerator {
    String format(List<String> data) {
        return "[\"" + String.join("\", \"", data) + "\"]";
    }
}


The generate and save methods contain behavior shared by every report generator. But the format method is abstract because each report type formats its data differently.

Both child classes reuse the common report-generation flow while providing their own formatting logic.

When to Use Interfaces vs Abstract Classes
Rule of Thumb

Use an interface when you want to define a contract or capability.
Use an abstract class when closely related classes need to share common code or state.
Java interfaces can also contain default methods, so the difference is not always absolute. But this rule is a useful starting point.

8. Encapsulation
The first pillar is encapsulation.

Encapsulation means keeping data and the methods that operate on that data together inside a class, while controlling how that data can be accessed or modified from outside.

In simple terms, an object should protect its internal state.


For example, imagine we have a BankAccount class that exposes its balance directly:


class BankAccount {

    public String owner;
    public double balance;

    BankAccount(String owner) {
        this.owner = owner;
    }

    boolean isOverdrawn() {
        return balance < 0;
    }
}


Now any part of the application can modify the balance:


BankAccount account = new BankAccount("Alice");
account.balance = 500;

// checkout service
account.balance -= 800;

// nightly refund job
account.balance += 250;

// an internal admin tool
account.balance = -1000;

account.isOverdrawn(); // true


This is dangerous because the object can easily end up in an invalid state.

A better design is to keep the balance private and provide controlled methods for modifying it:


class BankAccount {

    private double balance;

    void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("deposit must be positive");
        }
        balance += amount;
    }

    void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("invalid withdrawal");
        }
        balance -= amount;
    }

    boolean isOverdrawn() {
        return balance < 0;
    }
}


Now the balance cannot be changed directly. It can only be modified through deposit and withdraw, where we can enforce the necessary rules.

That is encapsulation. It protects objects from invalid states and keeps the logic for managing their data in one place.

Access modifiers we discussed earlier, such as private and protected, are the tools that help us implement encapsulation.


9. Abstraction
Encapsulation controls access to internal data. But sometimes, we also want to hide the internal complexity of an operation or method.

That brings us to abstraction.

Think about a video player library and its clients. To play a video, the caller should not need to understand networking, buffering, codecs, or frame decoding. It should only need to call a simple method:

class VideoPlayer {

    public void play(String fileName) {
        connect(fileName);
        buffer();
        decodeFrames();
        render();
    }

    private void connect(String fileName) { /* networking */ }
    private void buffer() { /* buffering strategy */ }
    private void decodeFrames() { /* codecs and decoding */ }
    private void render() { /* frame rendering */ }
}

// The caller only needs this:
VideoPlayer player = new VideoPlayer();
player.play("movie.mp4");



The caller knows what the play method does, but it does not need to know every detail of how it works. That complexity is hidden behind a simple public API.

This is abstraction. It reduces the amount of information developers need to understand at one time.

Interfaces and abstract classes are two common ways to create abstractions, but abstraction is a broader idea. Any well-designed class or method can provide abstraction by exposing a simple API and hiding unnecessary details.


10. Inheritance
Now that we know how to hide complexity, let's look at how one class can build on another.

Inheritance allows one class to reuse and extend the fields and behavior of another class.

It represents an is-a relationship. For example, an email notification is a type of notification, and an SMS notification is also a type of notification.


We can represent their common behavior in a parent class:

class Notification {
    protected String recipient;

    Notification(String recipient) {
        this.recipient = recipient;
    }

    void send(String message) {
        System.out.println("Sending notification to " + recipient);
    }

    void log(String message) {
        System.out.println("Sent: " + message);
    }
}

The child classes can inherit from Notification:


Both child classes inherit the recipient field and the log method from the parent class. They also provide their own implementations of the send method.

Method Overriding
This is called method overriding. Method overriding happens when a child class provides a new implementation of a method inherited from its parent. The method name and parameters remain the same, but the behavior changes.

In Java, the @Override annotation tells the compiler and other developers that we intentionally want to replace the inherited behavior. It also helps the compiler catch mistakes. For example, if the method signature does not match the parent method, the code will not compile. C++, C#, and TypeScript provide an override keyword that serves the same purpose.


Use Inheritance Carefully
Inheritance can be useful, but it should be used carefully. A common mistake is extending a class only to reuse some of its code.

For example, an OrderService should not extend EmailSender just because it needs to send confirmation emails. An order service is not a type of email sender.

A better design is to give the service an EmailSender object:


class OrderService {
    private EmailSender emailSender;

    OrderService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }

    void placeOrder() {
        // process the order
        emailSender.send("Your order has been placed");
    }
}

This is composition. Instead of inheriting behavior, OrderService contains and uses another object.

Composition usually creates looser coupling because the dependency can be replaced without changing the class hierarchy.

Rule of Thumb

Use inheritance when one class is genuinely a specialized version of another.
Prefer composition when one class simply needs to use another class's behavior.
We will explore composition in more detail later in the chapter.


11. Polymorphism
The fourth pillar is polymorphism. It allows different object types to be used through the same interface or parent type.

In simple terms, the same method call can behave differently depending on the actual object being used.

Let's continue with the notification example:

Notification email = new EmailNotification("alice@example.com");
Notification sms = new SmsNotification("+1-555-0142");

Both variables have the parent type Notification, but they refer to different child objects.

Now we can call the same method:

Both variables have the parent type Notification, but they refer to different child objects.

Now we can call the same method:

email.send("Your order has shipped");
// Sending email to alice@example.com: Your order has shipped

sms.send("Your order has shipped");
// Sending SMS to +1-555-0142: Your order has shipped

The method call is the same, but the actual behavior depends on whether the object is an EmailNotification or an SmsNotification.

We can also process different notification types together:


List<Notification> notifications = List.of(
    new EmailNotification("alice@example.com"),
    new SmsNotification("+1-555-0142")
);

for (Notification notification : notifications) {
    notification.send("Your order has shipped");
}

The language runtime chooses the correct method based on the actual object. That is runtime polymorphism.

It allows high-level code to work with a common abstraction without knowing every specific implementation.

If we add a new PushNotification class later, the loop does not need to change. This makes the code easier to extend.


Class Relationships
Now that we've covered the building blocks and the four pillars, let's look at another important part of OOP: relationships between classes.

In real-world applications, classes rarely work alone:

A Customer places an Order.
A Team contains multiple developers.
An Order owns its OrderItems.
An InvoiceService uses a TaxCalculator.

Class relationships describe how these objects are connected and interact with one another. The four important relationships we'll cover are association, aggregation, composition, and dependency.


12. Association
The first relationship is association.

Association simply means that one class is connected to another. In other words, one object knows about another object.

For example, an Order can be associated with a Customer:

class Customer {
    private String name;
    private String email;

    Customer(String name, String email) {
        this.name = name;
        this.email = email;
    }
}

class Order {
    private String orderId;
    private Customer customer;

    Order(String orderId, Customer customer) {
        this.orderId = orderId;
        this.customer = customer;
    }
}

Here, the Order class stores a reference to a Customer. This tells us that every order is connected to a customer.

Associations can take different forms:

Form	Example
One-to-one	A User has a Profile
One-to-many	One Customer can place many Orders
Many-to-one	Many Students learn from one Teacher
Many-to-many	One Student can join many Courses, and each Course can contain many Students

Association is the most general relationship between classes. It simply tells us that two classes are connected in some way.

13. Aggregation
Sometimes, however, one object does more than just know about other objects. It may also contain or group them.

That brings us to aggregation.

Aggregation is a special type of association that represents a has-a relationship with weak ownership. One object contains or groups other objects, but those objects can still exist independently.

For example, consider an engineering team and its developers:


class EngineeringTeam {
    private String teamName;
    private List<Developer> developers;

    EngineeringTeam(String teamName, List<Developer> developers) {
        this.teamName = teamName;
        this.developers = developers;
    }
}


An engineering team has developers, but it does not fully own them. The developers are created outside the team and then passed to it:


Developer developer1 = new Developer("Alice");
Developer developer2 = new Developer("Bob");

EngineeringTeam team = new EngineeringTeam(
    "Payments",
    List.of(developer1, developer2)
);

If the team is removed, the developers still exist. They can move to another team or remain without one.

That is aggregation. The key idea is that the contained objects have an independent lifecycle.

But sometimes, the parent object has much stronger ownership over the objects it contains. That relationship is called composition.

14. Composition
Composition is a stricter has-a association relationship representing strong ownership. The child object is an important part of the parent and usually does not exist independently from it.

For example, an Order can contain multiple OrderItems:


class OrderItem {
    private String productName;
    private int quantity;

    OrderItem(String productName, int quantity) {
        this.productName = productName;
        this.quantity = quantity;
    }
}

class Order {
    private String orderId;
    private List<OrderItem> items = new ArrayList<>();

    Order(String orderId) {
        this.orderId = orderId;
    }

    void addItem(String productName, int quantity) {
        items.add(new OrderItem(productName, quantity));
    }
}


Here, the Order creates and manages its OrderItem objects. Each order item belongs to a specific order and does not have much meaning without it.


Aggregation vs Composition
Aggregation and composition are both types of association. The main difference is ownership and lifecycle.


Aspect	Aggregation	Composition
Ownership	Weak	Strong
Child lifecycle	Independent of the parent	Tied to the parent
Example	A team has developers	An order owns its order items

In aggregation, the child can exist independently. In composition, the child is strongly owned by the parent.

So: a team has developers. That is aggregation. An order owns its order items. That is composition.

In most languages, aggregation and composition do not use different keywords. The difference comes from how the objects are created, owned, and managed in the design.

Composition is also the idea we discussed in the inheritance section. Instead of extending another class just to reuse its behavior, a class can contain another object and delegate work to it. This often produces code that is more flexible than a deep inheritance hierarchy.


15. Dependency
So far, these relationships describe objects that are usually stored inside another object. But sometimes, a class simply needs another class to perform some work.

That is a dependency.

A dependency exists when one class uses another class to perform its work. It is generally a weaker relationship than association.

In an association, one class often keeps a reference to another object as part of its state:


class InvoiceService {

    private TaxCalculator taxCalculator;

    InvoiceService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }

    void createInvoice(int amount) {
        int tax = taxCalculator.calculateTax(amount);
        int total = amount + tax;
        System.out.println("Invoice total: " + total);
    }
}

Here, InvoiceService depends on TaxCalculator to calculate the tax. It does not create or own the calculator. It simply uses it to complete the operation. That is dependency.

Dependencies are common in real-world applications:

An OrderService may depend on an inventory service.
A UserService may depend on an email sender.
A ReportService may depend on a PDF generator.
Dependencies need to be managed carefully. When a class creates and depends directly on many concrete implementations, the code becomes tightly coupled and harder to test.

That is why good object-oriented designs often depend on interfaces and receive dependencies through constructors or method parameters. This allows implementations to be replaced without rewriting the high-level logic.


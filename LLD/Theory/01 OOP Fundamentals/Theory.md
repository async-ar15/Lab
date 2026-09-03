What is Object-Oriented Programming?
Object-oriented programming is a way of organizing software around objects. Each object contains two main things:

State: the data it stores
Behavior: the actions it can perform


1. Class
A class is a blueprint for creating objects. It defines the data an object stores and the actions it can perform.

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

Whenever you have a fixed set of values, such as OrderStatus, PaymentStatus, UserRole, or DifficultyLevel, enums are usually a good choice.

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

For example, imagine that our application needs to store files:

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


#### did not understand 

The generate and save methods contain behavior shared by every report generator. But the format method is abstract because each report type formats its data differently.

Both child classes reuse the common report-generation flow while providing their own formatting logic.


When to Use Interfaces vs Abstract Classes

Use an interface when you want to define a contract or capability.
Use an abstract class when closely related classes need to share common code or state.

Java interfaces can also contain default methods, so the difference is not always absolute. But this rule is a useful starting point.

# The Four Pillars of OOP

8. Encapsulation

The first pillar is encapsulation.

Encapsulation means keeping data and the methods that operate on that data together inside a class, while controlling how that data can be accessed or modified from outside.

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


Encapsulation vs Abstraction

A simple way to distinguish the two:

Encapsulation controls who can access or modify something.
Abstraction controls which details the caller needs to see.

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

class EmailNotification extends Notification {
    EmailNotification(String recipient) {
        super(recipient);
    }

    @Override
    void send(String message) {
        System.out.println("Sending email to " + recipient + ": " + message);
    }
}

class SmsNotification extends Notification {
    SmsNotification(String recipient) {
        super(recipient);
    }

    @Override
    void send(String message) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
    }
}


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

11. Polymorphism
The fourth pillar is polymorphism. It allows different object types to be used through the same interface or parent type.

In simple terms, the same method call can behave differently depending on the actual object being used.

Let's continue with the notification example:

Notification email = new EmailNotification("alice@example.com");
Notification sms = new SmsNotification("+1-555-0142");


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




# Class Relationships

Now that we've covered the building blocks and the four pillars, let's look at another important part of OOP: relationships between classes.

In real-world applications, classes rarely work alone:

A Customer places an Order.
A Team contains multiple developers.
An Order owns its OrderItems.
An InvoiceService uses a TaxCalculator.

Class relationships describe how these objects are connected and interact with one another. The four important relationships we'll cover are association, aggregation, composition, and dependency.


2. Association
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

3. Aggregation
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

With a dependency, the other object may only appear as a method parameter, local variable, or return type:

class InvoiceService {

    void createInvoice(int amount, TaxCalculator taxCalculator) {
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

These concepts form the foundation for everything that follows in low-level design: SOLID principles, design patterns, and interview problems that build on them.


# 15 Must-Know Design Patterns

23 Gang of Four Patterns

Creational
how objects are created

Structural
how objects are composed

Behavioral
how objects communicat

15 Design Patterns

Creational
Singleton, Builder,
Factory Method

Structural
Adapter, Facade, Proxy,
Decorator, Composite

Behavioral
Strategy, Observer, State, Command,
Template Method, Iterator,
Chain of Responsibility

Creational Patterns

1. Singleton
Sometimes, an application needs exactly one shared instance of a class. Common examples include a configuration manager, logger, cache manager, or thread pool. Creating multiple instances of these objects can waste resources or lead to inconsistent behavior.

The Singleton Pattern solves this by ensuring that only one instance of a class exists, and providing a consistent way to access it.



enum AppConfig {
    INSTANCE;

    private String environment = "production";

    public String getEnvironment() {
        return environment;
    }
}

We can now access the same instance from anywhere in the application:

// in OrderService
String env = AppConfig.INSTANCE.getEnvironment();

// in EmailWorker
log(AppConfig.INSTANCE.getEnvironment());

However, Singleton should be used carefully. Because it is globally accessible, it can hide dependencies and make testing harder.

In many cases, creating the object once and passing it explicitly to the classes that need it is cleaner:

// created once, then passed in
AppConfig config = new AppConfig("production");

OrderService orderService = new OrderService(config);
EmailWorker emailWorker = new EmailWorker(config);


Now the dependency is visible in the constructor, and a test can pass a different AppConfig.


The core idea

Use Singleton when the application genuinely requires one shared instance, not simply because global access is convenient.

Singleton ensures that only one object of a class is ever created. But what if creating a single object is itself complicated to construct? That brings us to the Builder Pattern.


2. Builder
Consider a UserProfile class. The name and email are required, while fields like age, location, bio, and notification preferences are optional:

class UserProfile {
    String name;
    String email;
    int age;
    String city;
    String bio;
    boolean notifications;
}


For creating a new user profile object, one simple approach is to pass everything through the constructor:


UserProfile profile = new UserProfile(
        "Alice",
        "alice@example.com",
        29,
        "Berlin",
        null,
        true
);

This works, but it is difficult to read. Without checking the constructor definition, it is not obvious what each value represents, and the problem gets worse as more optional fields are added.

You could create multiple overloaded constructors, but that often leads to another problem called the telescoping constructor, where you end up maintaining many constructor variations with different combinations of parameters.

The Builder Pattern solves this by letting us construct the object step by step using clearly named methods:


UserProfile profile =
    new UserProfile.Builder("Alice", "alice@example.com")
        .age(29)
        .city("Berlin")
        .build();


Now the required fields are provided first, and optional fields are added only when needed.

A simplified implementation looks like this:

public static class Builder {
    private final String name;
    private final String email;
    private int age;
    private String city;

    public Builder(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public Builder age(int age) {
        this.age = age;
        return this;
    }

    public Builder city(String city) {
        this.city = city;
        return this;
    }

    public UserProfile build() {
        return new UserProfile(this);
    }
}


Internally, each builder method updates one property and returns the same builder, which allows us to chain multiple calls together. Finally, the build method creates the complete UserProfile object.


Builder
name, email

.age 29

.city Berlin

.build

UserProfile

Builder helps us construct one complex object. But what if we need to create objects from multiple related classes without exposing their exact creation logic to the client? That brings us to the next pattern: Factory Method.

3. Factory Method
Consider a notification system that supports email, SMS, and push notifications. Each notification follows the same interface:

interface Notification {
    void send(String msg);
}

Each notification channel class provides its own implementation, along with its own setup:

class EmailNotification implements Notification {
    EmailNotification(String sender) { ... }
    void setRetryPolicy(RetryPolicy p) { ... }
    public void send(String msg) { sendEmail(msg); }
}

class SMSNotification implements Notification {
    SMSNotification(String gateway) { ... }
    void setRateLimit(RateLimit limit) { ... }
    public void send(String msg) { sendSms(msg); }
}

class PushNotification implements Notification {
    PushNotification(String token) { ... }
    void setPriority(Priority priority) { ... }
    public void send(String msg) { sendPush(msg); }
}


Creating these objects directly would tightly couple the client to a specific notification class:

var email = new EmailNotification(sender);
email.setRetryPolicy(RetryPolicy.EXPONENTIAL);

email.send("Your order has shipped");


If the type of notification changes, the client must also change its object creation logic. And if the same creation logic appears in several places, it can quickly become duplicated across the application.

The Factory Method Pattern moves that object creation behind a dedicated method.

We start by creating an abstract notification factory. The sendNotification method defines the overall workflow for creating and sending a notification, but it does not know which concrete notification object will be created. That decision is delegated to the createNotification factory method:


abstract class NotificationFactory {
    protected abstract Notification createNotification();

    public void sendNotification(String msg) {
        Notification notification = createNotification();
        notification.send(msg);
    }
}

Subclasses extend from this abstract class and decide how and which notification object to return:

class EmailNotificationFactory extends NotificationFactory {
    protected Notification createNotification() {
        var email = new EmailNotification(sender);
        email.setRetryPolicy(RetryPolicy.EXPONENTIAL);
        return email;
    }
}

class SMSNotificationFactory extends NotificationFactory {
    protected Notification createNotification() {
        var sms = new SMSNotification(gateway);
        sms.setRateLimit(RateLimit.perMinute(60));
        return sms;
    }
}

class PushNotificationFactory extends NotificationFactory {
    protected Notification createNotification() {
        var push = new PushNotification(deviceToken);
        push.setPriority(Priority.HIGH);
        return push;
    }
}

The client can now work with the base factory class:

NotificationFactory factory = new EmailNotificationFactory();
factory.sendNotification("Your order has shipped");

The client uses the common workflow without knowing the internal details of how the notification object is created.

To switch from email to SMS, we can simply provide a different factory:
NotificationFactory factory = new SMSNotificationFactory();
factory.sendNotification("Your order has shipped");

The rest of the client code remains unchanged.


![alt text](image.png)

Structural Patterns
So far, we have focused on how objects are created. But once those objects exist, we also need effective ways to connect, organize, and combine them. That brings us to the next category: Structural Design Patterns.

In this section, we'll cover five important structural patterns: Adapter, Facade, Proxy, Decorator, and Composite.

Let's begin with the Adapter Pattern.


4. Adapter
The Adapter Pattern helps two components work together when their interfaces do not match.

Think of a travel adapter. If your charger and the wall socket use different plugs, the adapter can translate between them. The same idea applies in software.

Consider a checkout system for an e-commerce application. It expects to implement this interface for every payment provider:


Java

interface PaymentProcessor {
    void pay(int amountInCents);
}





123
But an external third-party gateway that we need to support exposes a different method name, and expects the amount in dollars:


Java

class ExternalPaymentGateway {

    public void makePayment(double amountInDollars) {
        System.out.println("Charging $" + amountInDollars);
    }
}






123456
The interfaces do not match. This is where the Adapter Pattern helps.

Instead of changing the gateway or rewriting our checkout system, we can create an adapter that implements the interface our application expects:


Java

class ExternalPaymentAdapter implements PaymentProcessor {

    private final ExternalPaymentGateway gateway;

    public ExternalPaymentAdapter(ExternalPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(int amountInCents) {
        double amountInDollars = amountInCents / 100.0;
        gateway.makePayment(amountInDollars);
    }
}





1234567891011121314
The adapter receives the request in the format our application understands. It then converts the amount from cents to dollars and calls the method provided by the legacy gateway.


![alt text](image-1.png)

pay 4999 cents

makePayment 49.99 dollars

Checkout

ExternalPaymentAdapter
implements PaymentProcessor

ExternalPaymentGateway

Now the checkout system can use the external payment provider through the standard PaymentProcessor interface:


Java

PaymentProcessor processor =
        new ExternalPaymentAdapter(
                new ExternalPaymentGateway()
        );

processor.pay(4999);







123456
It does not need to know that the adapter is translating method names, data formats, or units internally.

The Adapter Pattern helps incompatible components work together. But what if the system works correctly and is simply too complicated to use? That brings us to the Facade pattern.


5. Facade
The Facade Pattern provides a simple interface to a complex subsystem.

Consider building a client for a video publishing platform like YouTube. Publishing a video may involve several steps. Without a facade, the client must coordinate every service itself:


Java

String video = compressor.compress("clip.mp4");
String thumb = thumbnails.generate(video);

String videoUrl = storage.upload(video);
String thumbUrl = storage.upload(thumb);

repository.save(videoUrl, thumbUrl);
notifier.notifyPublished(videoUrl);






12345678
This forces the client to understand the complete workflow, including which services to call and in what order. If this workflow is needed in multiple places, the same coordination logic may also get duplicated across the application.

A facade hides those details behind one simple method provided by the video publishing platform:


Java


class VideoPublishingFacade {

    public void publish(String videoFile) {
        String video = compressor.compress(videoFile);
        String thumb = thumbnails.generate(video);

        String videoUrl = storage.upload(video);
        String thumbUrl = storage.upload(thumb);

        repository.save(videoUrl, thumbUrl);
        notifier.notifyPublished(videoUrl);
    }
}




12345678910111213
Now the client only needs to make one simple call:


Java

VideoPublishingFacade facade = new VideoPublishingFacade();

facade.publish("clip.mp4");







123
The client no longer needs to know how compression, storage, metadata, or notifications work internally.

![alt text](image-2.png)




UploadClient

VideoPublishingFacade
publish

Compressor

ThumbnailService

Storage

VideoRepository

Notifier

Facade gives us a simpler way to interact with a complex system. But what if we need to control access to an object instead? That brings us to the Proxy pattern.


6. Proxy
The Proxy Pattern places a substitute object in front of a real object to control access to it. Both the real object and the proxy expose the same interface, so the client can use the proxy just like the original object.

Consider an image gallery application where the high-resolution image class loads the image in the constructor when its object is created:


Java

class HighResolutionImage implements Image {

    private final String fileName;

    public HighResolutionImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading " + fileName);
    }

    @Override
    public void display() {
        System.out.println("Showing " + fileName);
    }
}






123456789101112131415161718
Creating every image immediately would waste memory and processing time, especially if the client never opens most of them by calling the display method:


Java


Image image1 = new HighResolutionImage("photo-1.jpg");
Image image2 = new HighResolutionImage("photo-2.jpg");
Image image3 = new HighResolutionImage("photo-3.jpg");

image1.display();   // the only one actually opened




12345
If you are not allowed to modify the high-resolution image class, a proxy can delay loading until the image is actually needed by the client.

We create a proxy class called ImageProxy. It wraps the HighResolutionImage class and starts as a lightweight object. It creates and loads the real image only when the display method is called:


Java

class ImageProxy implements Image {
    private final String fileName;
    private HighResolutionImage realImage;

    public ImageProxy(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new HighResolutionImage(fileName);
        }

        realImage.display();
    }
}






1234567891011121314151617
The client now uses the proxy class instead of the original high-resolution image class:


Java

Image image3 = new ImageProxy("photo-3.jpg");

image3.display();








123
Most of the client code stays the same, since both the proxy and the original object implement the same interface.


![](image-3.png)



display

creates on first call

display

Client

ImageProxy

HighResolutionImage

Proxy controls access, while the next pattern, Decorator, adds new behavior to an object.


7. Decorator
The Decorator Pattern lets us add new behavior to an object dynamically without modifying its original class.

Consider a rich-text editor that starts with a basic component that renders plain text:


Java

interface TextView {
    String render();
}

class PlainTextView implements TextView {
    private final String text;

    public PlainTextView(String text) {
        this.text = text;
    }

    public String render() {
        return text;
    }
}





123456789101112131415
We can use it like this:


Java

TextView plain = new PlainTextView("Design Patterns");

System.out.println(plain.render());   // Design Patterns





123
This works perfectly for simple text without any formatting.

But now we also need to support bold, italic, underline, and different combinations of these styles. Creating a subclass for every combination would quickly lead to a subclass explosion.

Instead, we create small decorators that wrap another TextView and add one style. We first create a base decorator called TextDecorator. It implements the same TextView interface as PlainTextView and wraps another TextView object:


Java

abstract class TextDecorator implements TextView {
    protected final TextView wrapped;

    public TextDecorator(TextView wrapped) {
        this.wrapped = wrapped;
    }
}





1234567
We can then create separate decorators for bold, italic, and underlined text:


Java

class BoldDecorator extends TextDecorator {
    public BoldDecorator(TextView t) { super(t); }
    public String render() { return "<b>" + wrapped.render() + "</b>"; }
}

class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(TextView t) { super(t); }
    public String render() { return "<i>" + wrapped.render() + "</i>"; }
}

class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(TextView t) { super(t); }
    public String render() { return "<u>" + wrapped.render() + "</u>"; }
}





1234567891011121314
Now we can combine these decorators however we need. The client applies multiple formatting styles by wrapping them around one another:


Java
TextView plain =
        new PlainTextView("Design Patterns");

TextView bold =
        new BoldDecorator(plain);

TextView boldItalic =
        new ItalicDecorator(
                new BoldDecorator(plain));

TextView boldItalicUnderlined =
        new UnderlineDecorator(
                new ItalicDecorator(
                        new BoldDecorator(plain)));






1234567891011121314

![alt text](image-4.png)



UnderlineDecorator

ItalicDecorator

BoldDecorator

PlainTextView
Design Patterns

Each decorator adds one responsibility and then delegates the remaining work to the object it wraps. And because every decorator implements the same TextView interface, the client interacts with the text in exactly the same way, regardless of how many decorators have been applied.

A Decorator wraps one object and adds behavior around it. But what if an object needs to contain other objects? That brings us to the next pattern: Composite.


8. Composite
The Composite Pattern lets us treat individual objects and groups of objects through the same interface.

A file system is a perfect example. A file is a single object, while a folder can contain files and other folders. But we still want to perform operations like calculating size or printing details on both.

Without the Composite Pattern, the client may need separate logic for files and folders:


Java


long total = 0;

if (item instanceof FileItem) {
    FileItem file = (FileItem) item;
    total += file.getSize();
} else if (item instanceof Folder) {
    Folder folder = (Folder) item;
    total += sizeOf(folder);
}




123456789
The client needs to check which type of object it received and handle each one differently. This becomes even more complicated because folders can contain other folders, which may contain even more files and folders:


Java

for (Object child : folder.getChildren()) {
    if (child instanceof FileItem) {
        total += ((FileItem) child).getSize();
    } else if (child instanceof Folder) {
        Folder inner = (Folder) child;

        for (Object item : inner.getChildren()) {
            if (item instanceof FileItem) {
                total += ((FileItem) item).getSize();
            } else if (item instanceof Folder) {
                // ... and again, one level down
            }
        }
    }
}





123456789101112131415
This is where the Composite Pattern helps. We start with a common interface called FileSystemItem that represents anything in the file system:


Java







12345
Now both individual files and folders can implement the same interface. A file is the simplest type of object and simply returns its own size and prints its own name:


Java

class FileItem implements FileSystemItem {
    private final String name;
    private final long size;

    public long getSize() {
        return size;
    }

    public void print(String indent) {
        System.out.println(indent + "- " + name);
    }
}





123456789101112
A folder contains other file-system items and delegates the operation to them:


Java
class Folder implements FileSystemItem {
    private final String name;
    private final List<FileSystemItem> children = new ArrayList<>();

    public void add(FileSystemItem item) {
        children.add(item);
    }

    public long getSize() {
        long total = 0;
        for (FileSystemItem child : children) {
            total += child.getSize();
        }
        return total;
    }

    public void print(String indent) {
        System.out.println(indent + "+ " + name);
        for (FileSystemItem child : children) {
            child.print(indent + "  ");
        }
    }
}






1234567891011121314151617181920212223
When we ask a folder for its size, it asks every child for its size and adds the results together. This recursive structure is what makes the pattern so powerful.

![alt text](image-5.png)




Folder: project

File: README.md

Folder: src

File: Main.java

Folder: lib

File: Utils.java

Now we can build an entire directory tree:


Java

Folder project = new Folder("project");
project.add(new FileItem("README.md", 2_000));

Folder src = new Folder("src");
src.add(new FileItem("Main.java", 8_000));
project.add(src);





123456
The client can now treat the entire folder tree exactly like a single file-system item:


Java
FileSystemItem item = project;

System.out.println(item.getSize());
item.print("");






1234
Those last two lines would work identically if item were a single file.


Behavioral Patterns
So far, we have looked at how objects are created and how they are connected. But software is not just about structure. Objects also need to communicate, respond to events, switch behavior, and coordinate workflows. That brings us to the final category: Behavioral Design Patterns.

In this section, we'll cover seven important behavioral patterns: Strategy, Observer, State, Command, Template Method, Iterator, and Chain of Responsibility.

Let's begin with one of the most commonly used patterns: the Strategy Pattern.


9. Strategy
The Strategy Pattern lets us define multiple ways to perform the same task and switch between them when needed.

Consider a navigation app that calculates routes for driving, walking, cycling, or public transport. One way to implement this is to have a RoutePlanner class with a large if-else block that selects the routing logic based on the travel mode:


Java


class RoutePlanner {
    public void buildRoute(
            String start,
            String destination,
            String travelMode) {

        if (travelMode.equals("DRIVING")) {
            buildDrivingRoute(start, destination);
        } else if (travelMode.equals("WALKING")) {
            buildWalkingRoute(start, destination);
        } else if (travelMode.equals("CYCLING")) {
            buildCyclingRoute(start, destination);
        } else if (travelMode.equals("TRANSIT")) {
            buildTransitRoute(start, destination);
        }
    }
}




1234567891011121314151617
But this approach does not scale well. As new travel modes are added, the conditional logic keeps growing, making the class harder to understand, test, and maintain:


Java


} else if (travelMode.equals("RIDESHARE")) {
    buildRideshareRoute(start, destination);
} else if (travelMode.equals("SCOOTER")) {
    buildScooterRoute(start, destination);
} else if (travelMode.equals("FERRY")) {
    buildFerryRoute(start, destination);
}




1234567
It also means modifying the RoutePlanner every time we introduce a new routing algorithm, which tightly couples the class to all possible routing strategies.

This is where the Strategy Pattern helps. We start by defining a common interface for building routes:


Java

interface RouteStrategy {
    void buildRoute(String start, String destination);
}





123
Each routing algorithm becomes a separate strategy class implementing the interface:


Java

class DrivingRouteStrategy implements RouteStrategy {
    public void buildRoute(String start, String destination) {
        System.out.println("Fastest driving route");
    }
}

class WalkingRouteStrategy implements RouteStrategy {
    public void buildRoute(String start, String destination) {
        System.out.println("Pedestrian-friendly route");
    }
}

class CyclingRouteStrategy implements RouteStrategy {
    public void buildRoute(String start, String destination) {
        System.out.println("Cycling route");
    }
}

class TransitRouteStrategy implements RouteStrategy {
    public void buildRoute(String start, String destination) {
        System.out.println("Public transport route");
    }
}





1234567891011121314151617181920212223
Now the route planner no longer needs to know how each algorithm works. It simply delegates the task to the selected strategy:


Java

class RoutePlanner {
    private RouteStrategy strategy;

    public RoutePlanner(RouteStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(RouteStrategy strategy) {
        this.strategy = strategy;
    }

    public void buildRoute(String start, String destination) {
        strategy.buildRoute(start, destination);
    }
}

![alt text](image-6.png)





123456789101112131415



RoutePlanner

RouteStrategy

DrivingRouteStrategy

WalkingRouteStrategy

CyclingRouteStrategy

TransitRouteStrategy

The client can choose the appropriate strategy when creating the route planner:


Java

RoutePlanner planner = new RoutePlanner(new DrivingRouteStrategy());
planner.buildRoute("Berlin", "Munich");






12
And if the user switches from driving to walking, we can change the strategy at runtime:


Java


planner.setStrategy(new WalkingRouteStrategy());
planner.buildRoute("Berlin", "Munich");




12
The RoutePlanner remains unchanged. Only the selected algorithm changes.

Strategy allows one object to choose between different behaviors. But sometimes we need several objects to react when something happens. That brings us to the Observer pattern.


10. Observer
The Observer Pattern is useful when multiple objects need to react to the same event.

Consider an e-commerce system. When an order is shipped, we may need to send an email, update the inventory, and reward the customer. Calling each service directly would tightly couple the order service to all of these actions:


Java

class OrderService {
    private final EmailService emailService;
    private final InventoryService inventoryService;
    private final AnalyticsService analyticsService;
    private final LoyaltyService loyaltyService;

    public void shipOrder(Order order) {
        order.markShipped();

        emailService.sendShippingEmail(order);
        inventoryService.updateInventory(order);
        analyticsService.trackShipment(order);
        loyaltyService.addRewardPoints(order);
    }
}





123456789101112131415
The order service is now responsible not only for shipping the order, but also for knowing everything that should happen afterward.

This is where the Observer Pattern helps. Instead of calling every component directly, the order service simply announces that an event has occurred. Any interested component can subscribe and react to that event.

We begin with a common observer interface:


Java

interface OrderObserver {
    void onOrderShipped(Order order);
}





123
Each observer handles one specific reaction:


Java



class EmailObserver implements OrderObserver {
    public void onOrderShipped(Order order) {
        System.out.println("Sending shipping email");
    }
}

class InventoryObserver implements OrderObserver {
    public void onOrderShipped(Order order) {
        System.out.println("Updating inventory");
    }
}

class LoyaltyObserver implements OrderObserver {
    public void onOrderShipped(Order order) {
        System.out.println("Adding reward points");
    }
}



1234567891011121314151617
The order service maintains a list of subscribers and notifies them when the event occurs:


Java

class OrderService {
    private final List<OrderObserver> observers = new ArrayList<>();

    public void subscribe(OrderObserver observer) {
        observers.add(observer);
    }

    public void shipOrder(Order order) {
        order.markShipped();

        for (OrderObserver observer : observers) {
            observer.onOrderShipped(order);
        }
    }
}





123456789101112131415

![alt text](image-7.png)



notify

notify

notify

OrderService
SHIPPED

EmailObserver

InventoryObserver

LoyaltyObserver

The observers are wired up once, and shipOrder no longer names any of them:


Java
OrderService orderService = new OrderService();

orderService.subscribe(new EmailObserver());
orderService.subscribe(new InventoryObserver());
orderService.subscribe(new LoyaltyObserver());

orderService.shipOrder(order);






1234567

![alt text](image-8.png)

Observer allows several objects to react when something happens. But what if one object changes its own behavior based on its current condition? That brings us to the State pattern.


11. State
The State Pattern allows an object to change its behavior when its internal state changes.

Consider an e-commerce system. An order can move through several states: New, Paid, Shipped, Delivered, or Cancelled. The operations allowed on the order depend on its current state. A new order can be paid. A paid order can be shipped. A shipped order can be delivered. But a delivered order cannot be shipped again.


![alt text](image-9.png)




Order created

pay

cancel

ship

cancel

deliver

NEW

PAID

CANCELLED

SHIPPED

DELIVERED

A simple approach is to store the current status and use conditionals throughout the Order class:


Java
class Order {
    private String status = "NEW";

    public void pay() {
        if (status.equals("NEW")) {
            status = "PAID";
        } else if (status.equals("PAID")) {
            System.out.println("Already paid");
        } else {
            System.out.println("Cannot pay this order");
        }
    }

    public void ship() {
        if (status.equals("PAID")) {
            status = "SHIPPED";
        } else if (status.equals("NEW")) {
            System.out.println("Pay before shipping");
        } else {
            System.out.println("Cannot ship this order");
        }
    }
}






1234567891011121314151617181920212223
This may look manageable with only a few states. But as the order lifecycle grows, every method starts accumulating more if-else or switch statements. Soon, the Order class becomes responsible for the behavior of every possible state and every valid transition between them.

This is where the State Pattern helps. Instead of representing the state using a string or enum alone, we represent each state as a separate object.

We begin with a common interface for order state:


Java

interface OrderState {
    void pay(Order order);
    void ship(Order order);
    void deliver(Order order);
}





12345
Each concrete state defines which operations are valid and what should happen next:


Java

class NewOrderState implements OrderState {
    public void pay(Order order) {
        System.out.println("Payment completed");
        order.setState(new PaidOrderState());
    }
}

class PaidOrderState implements OrderState {
    public void ship(Order order) {
        System.out.println("Order shipped");
        order.setState(new ShippedOrderState());
    }
}

class ShippedOrderState implements OrderState {
    public void deliver(Order order) {
        System.out.println("Order delivered");
        order.setState(new DeliveredOrderState());
    }
}








1234567891011121314151617181920
The Order class now delegates its behavior to the current state object:


Java
class Order {
    private OrderState state = new NewOrderState();

    public void setState(OrderState s) { state = s; }

    public void pay() { state.pay(this); }
    public void ship() { state.ship(this); }
    public void deliver() { state.deliver(this); }
}






123456789
The client can use the order without writing any state-specific conditions:


Java

Order order = new Order();

order.pay();        // Payment completed
order.ship();       // Order shipped
order.deliver();    // Order delivered





12345
Internally, the same method call behaves differently depending on the current state object. For example, calling ship on a new order produces an error. Calling ship after payment moves the order to the shipped state. And calling it after delivery tells us that the order has already been shipped.

Each state owns its own behavior and controls the valid transitions from that state. This makes it easier to add a new state, such as CancelledOrderState or ReturnedOrderState, without filling the main Order class with even more conditions.

State changes how an object behaves over time. But sometimes, instead of performing an action immediately, we want to represent that action as an object so it can be queued, logged, retried, or undone. That brings us to the next pattern: Command.


12. Command
The Command Pattern turns a request or action into a separate object.

Why is that useful? Because once an action becomes an object, we can store it, queue it, retry it, keep a history, or undo it later.

Consider a text editor with multiple options in the toolbar, where users can add or delete text, copy and paste, and undo the most recent operation. A simple approach is to put all of this logic directly inside the toolbar:


Java

class Toolbar {
    private final TextEditor editor;

    public void onButtonClick(String action) {
        if (action.equals("ADD")) {
            editor.addText("Hello");
        } else if (action.equals("DELETE")) {
            editor.deleteText(5);
        } else if (action.equals("UNDO")) {
            // what exactly do we reverse?
        }
    }
}





12345678910111213
This works for executing actions, but undo becomes difficult. The toolbar knows that a button was clicked, but it does not have enough information to reverse the action. As more actions are added, the toolbar also becomes tightly coupled to every operation supported by the editor.

This is where the Command Pattern helps. Instead of executing actions directly, we represent each action as a command object.

Here is the TextEditor class that performs the actual work of adding and deleting text:


Java


class TextEditor {
    private final StringBuilder content = new StringBuilder();

    public void addText(String text) {
        content.append(text);
    }

    public String deleteText(int length) {
        int start = Math.max(0, content.length() - length);
        String deleted = content.substring(start);
        content.delete(start, content.length());
        return deleted;
    }

    public String getContent() {
        return content.toString();
    }
}




123456789101112131415161718
To turn these operations into commands, we first define a common Command interface:


Java

interface Command {
    void execute();
    void undo();
}





1234
We then create concrete command classes for each type of action. Each command stores the information needed to execute and reverse the operation, and wraps the TextEditor object to perform the actual work:


Java

class AddTextCommand implements Command {
    private final TextEditor editor;
    private final String text;

    public AddTextCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.text = text;
    }

    public void execute() { editor.addText(text); }
    public void undo() { editor.deleteText(text.length()); }
}





123456789101112
We also need a way to keep track of executed commands so they can be undone in the correct order. That is the responsibility of the CommandManager class. It maintains the command history, delegates execution to the appropriate command, and handles undo operations:


Java

class CommandManager {
    private final Stack<Command> history = new Stack<>();

    public void execute(Command command) {
        command.execute();
        history.push(command);
    }

    public void undo() {
        if (!history.isEmpty()) history.pop().undo();
    }
}





123456789101112

![alt text](image-10.png)

execute

execute / undo

addText / deleteText

Toolbar

CommandManager
history

AddTextCommand

TextEditor

The client can now use it like this:


Java


TextEditor editor = new TextEditor();
CommandManager manager = new CommandManager();

manager.execute(new AddTextCommand(editor, "Design "));
manager.execute(new AddTextCommand(editor, "Patterns"));

System.out.println(editor.getContent());   // Design Patterns




1234567
Now suppose the user presses Undo. This reverses the most recent command:


Java


manager.undo();

System.out.println(editor.getContent());   // Design




123
Command lets us package an individual action as an object. But sometimes, we have an entire process made up of several steps, where the overall workflow stays the same but some steps need to vary. That brings us to the next pattern: Template Method.


13. Template Method
The Template Method Pattern is useful when several processes follow the same overall workflow, but a few steps vary.

Consider a data import system that needs to support different file formats, such as CSV and JSON. All formats follow the same sequence: read the file, parse the content, validate the records, save them, and generate a report.

If each importer implements the full workflow separately, most of the code gets duplicated:


Java


class CSVDataImporter {

    public void importData(String filePath) {
        String content = readFile(filePath);
        List<Record> records = parseCSV(content);
        validate(records);
        save(records);
        generateReport(records);
    }
}

class JSONDataImporter {

    public void importData(String filePath) {
        String content = readFile(filePath);
        List<Record> records = parseJSON(content);
        validate(records);
        save(records);
        generateReport(records);
    }
}




123456789101112131415161718192021
The only major difference is how the content is parsed.

This is where the Template Method Pattern helps. We move the common workflow into a base class:


Java

abstract class DataImporter {

    public final void importData(String filePath) {
        String content = readFile(filePath);
        List<Record> records = parse(content);
        validate(records);
        save(records);
        generateReport(records);
    }

    protected abstract List<Record> parse(String content);

    private String readFile(String path) { ... }
    protected void validate(List<Record> records) { ... }
    protected void save(List<Record> records) { ... }
    protected void generateReport(List<Record> records) { ... }
}





1234567891011121314151617
The importData method is the template method. It defines the overall algorithm and fixes the order of the steps, while subclasses customize only what differs.

Now every importer follows the same workflow while providing its own parsing logic:


Java

class CSVDataImporter extends DataImporter {
    @Override
    protected List<Record> parse(String content) {
        System.out.println("Parsing CSV content");
        return new ArrayList<>();
    }
}

class JSONDataImporter extends DataImporter {
    @Override
    protected List<Record> parse(String content) {
        System.out.println("Parsing JSON content");
        return new ArrayList<>();
    }
}





123456789101112131415

![alt text](image-11.png)

DataImporter.importData

readFile

parse
varies by subclass

validate

save

generateReport

CSVDataImporter.parse

JSONDataImporter.parse

The base class controls the structure of the algorithm, while subclasses customize specific steps. The client can now use either importer through the same process:


Java

DataImporter csv = new CSVDataImporter();
csv.importData("users.csv");

DataImporter json = new JSONDataImporter();
json.importData("users.json");





12345
Template Method defines the structure of an algorithm while letting us customize specific steps. But what if we need to traverse a collection without caring how its data is stored? That brings us to the Iterator pattern.


14. Iterator
The Iterator Pattern lets us move through a collection without exposing how its elements are stored.

Consider a music application with a playlist. Internally, the playlist might store songs in an array, a linked list, a database, or even fetch them from an external service.

Initially, the playlist class might expose its internal data structure directly to the client:


Java

class Playlist {
    private final List<Song> songs = new ArrayList<>();

    public List<Song> getSongs() {
        return songs;
    }
}






1234567
The client then traverses it by index:


Java

List<Song> songs = playlist.getSongs();

for (int i = 0; i < songs.size(); i++) {
    Song song = songs.get(i);
    System.out.println(song.getTitle());
}






123456
This works, but it creates a few problems. The client now knows that songs are stored in a List and accessed using an index. If we later update the Playlist and replace the data structure with a linked list, a tree, or a remote data source, the traversal logic may also need to change on the client side.

This is where the Iterator Pattern helps. Instead of exposing the internal collection, the playlist provides an iterator.

An iterator usually supports two basic operations: hasNext, which tells us whether another element is available, and next, which returns that element and moves the iterator forward:


Java

interface Iterator<T> {
    boolean hasNext();
    T next();
}






1234
The iterator keeps track of the current position internally:


Java

class PlaylistIterator implements Iterator<Song> {
    private final List<Song> songs;
    private int position = 0;

    public PlaylistIterator(List<Song> songs) {
        this.songs = songs;
    }

    public boolean hasNext() {
        return position < songs.size();
    }

    public Song next() {
        if (!hasNext()) throw new NoSuchElementException();
        return songs.get(position++);
    }
}






1234567891011121314151617
The playlist provides an iterator without exposing its internal list:


Java

class Playlist implements Iterable<Song> {
    private final List<Song> songs = new ArrayList<>();

    public void addSong(Song song) { ... }

    public Iterator<Song> iterator() {
        return new PlaylistIterator(songs);
    }
}







123456789



hasNext / next

iterator

Client

PlaylistIterator
position

Song 1

Song 2

Song 3

Playlist

Here is what it looks like in code. The client can now traverse the playlist by repeatedly calling hasNext and next until every song has been visited:


Java

Playlist playlist = new Playlist();
playlist.addSong(new Song("Nightcall"));
playlist.addSong(new Song("Resonance"));

Iterator<Song> it = playlist.iterator();
while (it.hasNext()) {
    Song song = it.next();
    System.out.println(song.getTitle());
}





123456789

![alt text](image-12.png)
The client no longer needs to manage indexes or know how the playlist stores its songs. And because Playlist implements Iterable, it also works with the enhanced for loop, which is Java's own use of this pattern:


Java

for (Song song : playlist) {
    System.out.println(song.getTitle());
}






123

15. Chain of Responsibility
The final pattern in the chain is the Chain of Responsibility Pattern. This pattern lets you pass a request through a sequence of independent handlers. Each handler performs one part of the processing and either stops the request or forwards it to the next handler.

Consider an API request that must pass authentication, authorization, rate limiting, and validation. Putting every check inside one method creates a long conditional that is difficult to reuse or rearrange:


Java


class APIService {
    public void handle(Request request) {
        if (!request.isAuthenticated()) {
            System.out.println("Authentication failed");
            return;
        }
        if (!request.isAuthorized()) {
            System.out.println("Access denied");
            return;
        }
        if (request.hasExceededRateLimit()) {
            System.out.println("Rate limit exceeded");
            return;
        }
        if (!request.isValid()) {
            System.out.println("Invalid request");
            return;
        }

        System.out.println("Processing request");
    }
}





12345678910111213141516171819202122
This is where the Chain of Responsibility Pattern helps. We begin with a base handler:


Java

abstract class RequestHandler {
    private RequestHandler next;

    public RequestHandler setNext(RequestHandler next) {
        this.next = next;
        return next;
    }

    public final void handle(Request r) {
        if (!process(r)) return;
        if (next != null) next.handle(r);
    }

    protected abstract boolean process(Request r);
}






123456789101112131415
Each handler implements the process method and handles one responsibility. If the handler returns true, the request continues to the next handler. If it returns false, the chain stops immediately:


Java



class AuthenticationHandler extends RequestHandler {
    protected boolean process(Request r) { return r.isAuthenticated(); }
}

class AuthorizationHandler extends RequestHandler {
    protected boolean process(Request r) { return r.isAuthorized(); }
}

class RateLimitHandler extends RequestHandler {
    protected boolean process(Request r) { return r.isUnderLimit(); }
}

class ValidationHandler extends RequestHandler {
    protected boolean process(Request r) { return r.isValid(); }
}



123456789101112131415
We can then build the pipeline dynamically by connecting these handlers into a chain:


Java

RequestHandler authentication = new AuthenticationHandler();

authentication
        .setNext(new AuthorizationHandler())
        .setNext(new RateLimitHandler())
        .setNext(new ValidationHandler());







123456

![alt text](image-13.png)

Request

AuthenticationHandler

AuthorizationHandler

RateLimitHandler

ValidationHandler

Processed

To process a request, the sender only needs to know where the chain begins:


Java

Request request = new Request(user, "/api/orders");

authentication.handle(request);
// stops at the rate limit





1234
Another advantage is that the chain can be configured differently for different situations. For example, a public endpoint may only require rate limiting and validation:


Java

RequestHandler publicChain = new RateLimitHandler();
publicChain.setNext(new ValidationHandler());





12
An admin endpoint may require the full chain:


Java

RequestHandler adminChain = new AuthenticationHandler();

adminChain
        .setNext(new AuthorizationHandler())
        .setNext(new RateLimitHandler())
        .setNext(new ValidationHandler());





123456
We can also change the order of the handlers without modifying their internal logic.


Choosing the Right Pattern
You do not need to remember every class or implementation you saw in this chapter. Instead, focus on the problem each pattern is designed to solve. When you recognize the problem, the right pattern becomes much easier to identify.

Pattern	Use it when
Singleton	The application genuinely requires one shared instance of a class
Builder	An object has many optional fields and its constructor is hard to read
Factory Method	Subclasses should decide which concrete object gets created
Adapter	Two components must work together but their interfaces do not match
Facade	A working subsystem is too complicated for clients to use directly
Proxy	Access to an object needs to be controlled, delayed, or checked
Decorator	Behavior must be added to an object without modifying its class
Composite	Individual objects and groups of them should be treated the same way
Strategy	One task has multiple interchangeable algorithms
Observer	Several objects need to react to the same event
State	An object's behavior depends on the state it is currently in
Command	An action must be stored, queued, logged, retried, or undone
Template Method	Several processes share a workflow but differ in a few steps
Iterator	A collection must be traversed without exposing how it stores data
Chain of Responsibility	A request should pass through a configurable series of handlers
Several of these patterns look similar in a diagram and differ in intent. Adapter and Decorator both wrap an object, but an adapter changes the interface while a decorator keeps it and adds behavior. Proxy also keeps the interface, and controls access rather than extending it. Strategy and State both delegate to an interchangeable object, but a strategy is chosen by the client while a state is chosen by the object itself as part of its own lifecycle.
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


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



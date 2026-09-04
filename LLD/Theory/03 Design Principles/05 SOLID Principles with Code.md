Writing clean, maintainable code is just as important as writing code that works.

The SOLID principles provide a blueprint for writing code that’s easy to adjust, extend, and maintain over time.

It was introduced by Robert C. Martin (Uncle Bob) in the early 2000s.

In this article, we will explore each of the 5 principles with real world examples and code:

If you’re enjoying this newsletter and want to get even more value, consider becoming a paid subscriber.

As a paid subscriber, you'll unlock all premium articles and gain full access to all premium courses on algomaster.io.

Unlock Full Access

S: Single Responsibility Principle (SRP)
A class should have one, and only one, reason to change.

This means that a class must have only one responsibility.

When a class performs just one task, it contains a small number of methods and member variables making them more usable and easier to maintain.

If a class has multiple responsibilities, it becomes harder to understand, maintain, and modify and increases the potential for bugs because changes to one responsibility could affect the others.

Code Example:
Imagine you have a class called UserManager that handles user authentication, user profile management, and email notifications.




This class violates the SRP because it has multiple responsibilities: authentication, profile management, and email notifications.

If you need to change the way user authentication is handled, you might inadvertently affect the email notification logic, or vice versa.

To adhere to the SRP, we can split this class into three separate classes, each with a single responsibility:




Now, each class has a single, well-defined responsibility. Changes to user authentication won't affect the email notification logic, and vice versa, improving maintainability and reducing the risk of unintended side effects.

O: Open/Closed Principle (OCP)
Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

This means the design of a software entity should be such that you can introduce new functionality or behavior without modifying the existing code since changing the existing code might introduce bugs.

Code Example:
Let's say you have a ShapeCalculator class that calculates the area and perimeter of different shapes like rectangles and circles.




If we want to add support for a new shape, like a triangle, we would have to modify the calculate_area and calculate_perimeter methods, violating the Open/Closed Principle.

To adhere to the OCP, we can create an abstract base class for shapes and separate concrete classes for each shape type:




By introducing an abstraction (Shape class) and separating the concrete implementations (Rectangle and Circle classes), we can add new shapes without modifying the existing code.

The ShapeCalculator class can now work with any shape that implements the Shape interface, allowing for easy extensibility.

L: Liskov Substitution Principle (LSP)
Objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program.

This means if you have a base class and a derived class, you should be able to use instances of the derived class wherever instances of the base class are expected, without breaking the application.

Code Example:
Let's consider a scenario where we have a base class Vehicle and two derived classes Car and Bicycle.

Without following the LSP, the code might look like this:




In this example, the Bicycle class violates the LSP because it provides an implementation for the start_engine method, which doesn't make sense for a bicycle.

If we try to substitute a Bicycle instance where a Vehicle instance is expected, it might lead to unexpected behavior or errors.

To adhere to the LSP, we can restructure the code as follows:




Here, we've replaced the start_engine method with a more general start method in the base class Vehicle. The Car class implements the start method to start the engine, while the Bicycle class implements the start method to indicate that the rider is pedaling.

Now, instances of Car and Bicycle can be safely substituted for instances of Vehicle without any unexpected behavior or errors.

Share

I: Interface Segregation Principle (ISP)
No client should be forced to depend on interfaces they don't use.

The main idea behind ISP is to prevent the creation of "fat" or "bloated" interfaces that include methods that are not required by all clients.

By segregating interfaces into smaller, more specific ones, clients only depend on the methods they actually need, promoting loose coupling and better code organization.

Code Example:
Let's consider a scenario where we have a media player application that supports different types of media files, such as audio files (MP3, WAV) and video files (MP4, AVI).

Without applying the ISP, we might have a single interface like this:




In this case, any class that implements the MediaPlayer interface would be forced to implement all the methods, even if it doesn't need them.

For example, an audio player would have to implement the play_video, stop_video, and adjust_video_brightness methods, even though they are not relevant for audio playback.

To adhere to the ISP, we can segregate the interface into smaller, more focused interfaces:




Now, we can have separate implementations for audio and video players:




By segregating the interfaces, each class only needs to implement the methods it actually requires. This not only makes the code more maintainable but also prevents clients from being forced to depend on methods they don't use.

If we need a class that supports both audio and video playback, we can create a new class that implements both interfaces:


D: Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level modules; both should depend on abstractions.

This means that a particular class should not depend directly on another class, but on an abstraction (interface) of this class.

Applying this principle reduces dependency on specific implementations and makes our code more reusable.

Code Example:
Let's consider a example where we have a EmailService class that sends emails using a specific email provider (e.g., Gmail).




In this example, the EmailService class directly depends on the GmailClient class, a low-level module that implements the details of sending emails using the Gmail API.

This violates the DIP because the high-level EmailService module is tightly coupled to the low-level GmailClient module.

To adhere to the DIP, we can introduce an abstraction (interface) for email clients:




Now, the EmailService class depends on the EmailClient abstraction, and the low-level email client implementations (GmailClient and OutlookClient) depend on the abstraction.

This follows the DIP, resulting in a more flexible and extensible design.
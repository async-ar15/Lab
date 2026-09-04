    Abstract Factory Design Pattern
Medium Priority
14 min read
Updated July 2, 2026

Listen to this chapter
Unlock Audio




Family 1
Family 2
Client
AbstractFactory
Factory 1
Factory 2
Product A1
Product B1
Product A2
Product B2
The Abstract Factory Design Pattern is a creational pattern that provides an interface for creating families of related or dependent objects without specifying their concrete classes.

It’s particularly useful in situations where:

You need to create objects that must be used together and are part of a consistent family (e.g., GUI elements like buttons, checkboxes, and menus).
Your system must support multiple configurations, environments, or product variants (e.g., light vs. dark themes, Windows vs. macOS look-and-feel).
You want to enforce consistency across related objects, ensuring that they are all created from the same factory.
The Abstract Factory Pattern encapsulates object creation into factory interfaces.

Each concrete factory implements the interface and produces a complete set of related objects. This ensures that your code remains extensible, consistent, and loosely coupled to specific product implementations.

Let’s walk through a real-world example to see how we can apply the Abstract Factory Pattern to build a system that’s flexible, maintainable, and able to support multiple interchangeable product families without conditional logic.


1. The Problem: Platform-Specific UI
Imagine you're building a cross-platform desktop application that must support both Windows and macOS.

To provide a good user experience, your application should render native-looking UI components for each operating system like: Buttons, Checkboxes, Text fields, and Menus.

Naive Implementation: Conditional UI Component Instantiation
In your first attempt, you might implement platform-specific UI components like this:

Windows UI Elements

Java







12345678910111213141516171819
MacOS UI Elements

Java







12345678910111213141516171819
Conditional Client Code
Now, in your application logic, you check the operating system and manually instantiate the correct classes:


Java







1234567891011121314151617
Why This Approach Breaks Down
This setup works when you have two components on two platforms. But it quickly becomes unmanageable.

1. No family consistency enforcement
Nothing stops a developer from writing new WindowsButton() alongside new MacOSCheckbox(). The code compiles, the tests might even pass, and the bug only surfaces when a user sees a visually broken screen.

2. Tight coupling to concrete classes
The client code directly references WindowsButton, MacOSCheckbox, and every other platform-specific class. Every file that creates UI components needs platform-checking logic.

3. No shared interface
You cannot treat all buttons polymorphically. There is no Button type that WindowsButton and MacOSButton both implement. You cannot write a method that accepts "any button."

4. Explosive growth with new platforms or components
Adding Linux means creating LinuxButton, LinuxCheckbox, and updating every conditional block in the codebase. Adding a TextField component means creating WindowsTextField, MacOSTextField, LinuxTextField, and adding more branches everywhere.

5. Violation of Open/Closed Principle
Every new platform or component forces you to modify existing code. You cannot extend the system without editing files that already work.

What We Really Need
A way to group related components by family (all Windows components together, all macOS components together)
Encapsulated creation logic so platform checks happen in exactly one place
Polymorphic products so the client works with Button and Checkbox interfaces, not concrete classes
Structural guarantees that mixing families is impossible, not just discouraged
This is exactly what the Abstract Factory pattern provides.


2. What is Abstract Factory
The Abstract Factory Pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes.

The key word is families. Factory Method deals with creating one product at a time. Abstract Factory deals with creating multiple products that must work together. A GUI factory does not just create buttons. It creates buttons, checkboxes, text fields, and menus that all share the same visual style.


Class Diagram



ConcreteFactory1
+ concreteProductA(): ProductA
+ concreteProductB(): ProductB
<<interface>>
AbstractFactory
+ createProductA(): ProductA
+ createProductB(): ProductB
ConcreteFactory2
+ concreteProductA(): ProductA
+ concreteProductB(): ProductB
ProductA1
<<interface>>
AbstractProductA
ProductA2
ProductB1
<<interface>>
AbstractProductB
ProductB2
The structure involves five participants:

1. Abstract Factory (GUIFactory)
Defines a common interface for creating a family of related products.
Typically includes factory methods like createButton(), createCheckbox(), createTextField(), etc.
Clients rely on this interface to create objects without knowing their concrete types.
2. Concrete Factory (WindowsFactory, MacOSFactory)
Implement the abstract factory interface.
Create concrete product variants that belong to a specific family or platform.
Each factory ensures that all components it produces are compatible (i.e., belong to the same platform/theme).
3. Abstract Product (Button, Checkbox)
Define the interfaces or abstract classes for a set of related components.
All product variants for a given type (e.g., WindowsButton, MacOSButton) will implement these interfaces.
4. Concrete Product (WindowsButton, MacOSCheckbox)
Implement the abstract product interfaces.
Contain platform-specific logic and appearance for the components.
5. Client (Application)
Uses the abstract factory and abstract product interfaces.
Is completely unaware of the concrete classes it is using — it only interacts with the factory and product interfaces.
Can switch entire product families (e.g., from Windows to macOS) by changing the factory without touching UI logic.

3. How It Works
Here is the Abstract Factory workflow, step by step:




Client never references concrete types directly
new WindowsFactory()
new Application(factory)
createButton()
WindowsButton
createCheckbox()
WindowsCheckbox
paint()
"Windows-style button"
paint()
"Windows-style checkbox"
Configuration
Application
WindowsFactory
WindowsButton
WindowsCheckbox
Configuration
Application
WindowsFactory
WindowsButton
WindowsCheckbox




11 / 11
Step 1: Configuration determines the factory
At application startup, a configuration value, environment variable, or runtime check determines which concrete factory to instantiate. This is the only place in the codebase that references concrete factory classes.

Step 2: The factory is injected into the client
The client receives the factory through its constructor. It stores the factory as the abstract type, not a concrete one.

Step 3: The client calls factory methods to create products
When the client needs a button, it calls factory.createButton(). When it needs a checkbox, it calls factory.createCheckbox(). The client has no idea which concrete classes are being instantiated.

Step 4: The factory returns compatible products
Because the factory is a WindowsFactory, both createButton() and createCheckbox() return Windows-specific components. There is no possibility of getting a macOS checkbox from a Windows factory.

Step 5: The client uses products through abstract interfaces
The client calls button.paint() and checkbox.paint(). It does not know or care whether these are Windows or macOS components. The behavior is determined by the factory that was injected in Step 2.


4. Implementing Abstract Factory
Let's implement the Abstract Factory pattern step by step. We will define abstract product interfaces, create concrete products for two platforms, build an abstract factory with concrete implementations, and wire everything together through a client that never touches a concrete class.

Step 1: Define Abstract Product Interfaces
We start with the contracts that all product variants must fulfill. These interfaces are what the client will work with.

Button

Java







1234
Checkbox

Java







1234
Step 2: Create Concrete Products
Each platform provides its own implementation of every product interface.

Windows Products

Java







1234567891011121314151617181920212223
macOS Products

Java







1234567891011121314151617181920212223
Step 3: Define the Abstract Factory
The abstract factory declares one creation method per product type. Any concrete factory must implement all of them.


Java







1234
Step 4: Implement Concrete Factories
Each concrete factory produces a complete, compatible set of products for its platform.

WindowsFactory

Java







1234567891011
MacOSFactory

Java







1234567891011
Step 5: Client Code
The client receives a factory through its constructor and uses only abstract interfaces.


Java







1234567891011121314
Step 6: Wire Everything Together
The entry point is the only place that references concrete factories. It reads the platform, picks the right factory, and injects it into the client.


Java







12345678910111213141516
Output (on macOS)

Output (on Windows)

What We Achieved
Platform independence: Application code never references platform-specific classes
Consistency: Buttons and checkboxes always match the selected OS style
Open/Closed Principle: Add support for Linux or Android without modifying existing factories or components
Testability & Flexibility: Factories can be swapped easily for testing or theming

5. Practical Example: Notification System
Let's build a notification system that supports Email and SMS channels. Each channel produces two related objects: a Message and a Sender. Mixing an email message with an SMS sender would produce garbled output, so family consistency matters.

Architecture



Products

Factories

Client

Application

NotificationFactory

EmailFactory

SmsFactory

Message

Sender

EmailMessage

EmailSender

SmsMessage

SmsSender

Full Implementation

Java

Run







12345678910111213141516171819202122232425262728293031
What we achieved:
Two product types (Message and Sender) that must stay consistent within a notification channel
Zero mixing risk: An EmailFactory can only produce email objects. There is no code path that creates an email message paired with an SMS sender
Easy to extend: Adding push notification support means creating PushFactory, PushMessage, and PushSender. Nothing existing changes
Simple and focused: Each product has a clear, single responsibility, making the pattern easy to understand and test
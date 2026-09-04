Use Case Diagram
Medium Priority
6 min read
Updated July 6, 2026

Listen to this chapter
Unlock Audio

Imagine you’ve been asked to design a new software system—say, an online food delivery app. Before you dive into writing classes, functions, or database schemas, the first question is:

"What exactly should this system do?"

You need to understand the problem from the user’s point of view.

That’s where Use Case Diagrams come in.

In this chapter, we will explore:

What a Use Case Diagram is?
Building Blocks of a Use Case Diagram
How to Draw a Use Case Diagram (Step-by-Step)

1. What is a Use Case Diagram?
A use case diagram is a behavioral diagram from UML that shows the interactions between external entities (called actors) and the functionality a system provides (called use cases).

Think of it as a bird's-eye map of your system's capabilities, drawn from the user's perspective rather than the developer's.

It answers one fundamental question: "Who can do what with this system?"

Notice what it does not answer. It doesn't explain how a feature is implemented, what classes are involved, or what the database schema looks like. A use case diagram deliberately stays at the requirements level. It captures scope and user goals, nothing more.

Use Case Diagrams are typically used in the early phases of software development right after the requirements are gathered and before detailed design begins.

Tip

In LLD interviews, you don't always need to draw a formal use case diagram. But spending some time listing actors and use cases before diving into class design shows structured thinking and saves you time during design and implementation phase.


2. Building Blocks of a Use Case Diagram
A use case diagram has four building blocks: actors, use cases, the system boundary, and relationships. Each one is simple on its own, but combining them gives you a powerful tool for capturing requirements.

Let’s explore each of them in detail.


1. Actors
An actor is anything that interacts with the system from outside. Most commonly actors are people (a customer, an admin, a driver), but they can also be external systems (a payment gateway, a notification service, a third-party API).

Actors are drawn as stick figures along with their label.




Actor
There are two types of actors, and the distinction matters.

Primary actors initiate interactions with the system. They have a goal they want to achieve. A customer who wants to book a ticket is a primary actor. A rider who wants to request a ride is a primary actor. Primary actors are typically drawn on the left side of the diagram.

Secondary actors are called upon by the system to help fulfill a use case, but they don't initiate the interaction. A payment gateway doesn't decide to process a payment on its own; the system calls it during checkout. Secondary actors are typically drawn on the right side.


2. Use Cases
A use case is a specific goal or function that the system provides to an actor. It represents a complete interaction, something that starts, does something meaningful, and ends with a result.

Use cases are drawn as ovals with a descriptive name inside.




Use Case
Good use cases follow three rules:

Start with a verb. "Book Ticket" not "Ticket Booking." The verb makes the action clear.
Represent a complete goal. "Book Ticket" is a complete goal. "Enter Credit Card Number" is a single step within a goal, too granular for a use case diagram.
Deliver a meaningful result. After "Book Ticket" completes, the actor has a confirmed booking. That's a meaningful outcome.

3. System Boundary
The system boundary is a rectangle that encloses all the use cases. It visually defines what's inside your system and what's outside. The system name is written at the top of the rectangle.

Actors sit outside the boundary (they're external to the system). Use cases sit inside (they're functionality your system provides). 




System

4. Relationships
Relationships are the lines connecting actors to use cases, and use cases to each other. There are four types.

a. Association
An association is a solid line connecting an actor to a use case. It means "this actor participates in this use case." It's the most basic relationship and the one you'll draw most often.




Actor
Use Case
b. Include
An include relationship means one use case always triggers another as a mandatory part of its execution. The base use case cannot complete without the included use case running.




«include»
Checkout
ValidatePayment
"Checkout" includes "Validate Payment." Every single time someone does a checkout, payment processing happens. There's no purchase without payment. The dashed arrow with the label <<include>> points from the base use case to the included use case.

c. Extend
An extend relationship means one use case optionally adds behavior to another under certain conditions. The base use case works perfectly fine on its own; the extending use case just enhances it sometimes.

Notation: Dashed arrow with label <<extend>>




«extends»
Book Ticket
Apply Coupon
"Apply Coupon" extends "Book Ticket." A customer can book a ticket without applying a coupon. But if they have a coupon, they can optionally apply it during booking. The dashed arrow points from the extending use case back to the base use case.

d. Generalization
Generalization shows inheritance between actors or between use cases. A child actor inherits all the associations of the parent actor, plus may have additional ones.

Notation: Directed arrow with a triangle arrowhead from child to parent




Admin
User
Example: Admin is also a type of User. They inherit everything a User can do (browse movies, view showtimes), but an Admin might additionally be able to add/edit movies.


3. How to Draw a Use Case Diagram (Step-by-Step)
Let's build a use case diagram from scratch for a Movie Ticket Booking System. We'll go through each step, and build up to a complete diagram at the end.

Step 1: Identify Actors
Start by asking: who or what interacts with this system from outside?

For a movie ticket booking system, the actors are:

Customer (primary)
Admin (primary)
Payment Gateway (secondary)



Customer
Admin
PaymentGateway
Step 2: List the Use Cases
Now ask: what does each actor want to accomplish?

Customer: Browse Movies, Book Ticket, Cancel Ticket, View Booking History
Admin: Add Movie, Update Showtime
System-level: Process Payment (called during booking)
Step 3: Define the System Boundary
Draw a box around all the use cases and label it with the name of your system. This box represents your system and visually defines the scope.

Anything inside the box is your responsibility to implement. Anything outside is external.




Movie Ticket Booking System
Browse Movies
Book Ticket
Cancel Ticket
Make Payment
Add/Edit Movies
Step 4: Draw Associations
Connect each actor to the use cases they participate in with solid lines.

Customer connects to Browse Movies, Book Ticket, Cancel Ticket
Admin connects to Add Movie, Update Showtime
Payment Gateway connects to Make Payment
Step 5: Add Include, Extend, and Generalization Relationships
Look for mandatory dependencies and optional extensions.

Book Ticket includes Make Payment (you can't book without paying)
Browse Movies is extended by Filter by Genre (optional filtering)
Here's the complete use case diagram for the Movie Ticket Booking System, combining all the elements we've identified.




Movie Ticket Booking System
«extend»
«include»
Customer
PaymentGateway
Admin
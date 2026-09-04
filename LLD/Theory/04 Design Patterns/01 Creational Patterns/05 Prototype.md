Prototype Design Pattern
Low Priority
13 min read
Updated July 2, 2026

Listen to this chapter
Unlock Audio




Client
Prototype
Copy 1
Copy 2
Copy 3
clone()
The Prototype Design Pattern is a creational design pattern that lets you create new objects by cloning existing ones, instead of instantiating them from scratch.

It’s particularly useful in situations where:

Creating a new object is expensive, time-consuming, or resource-intensive.
You want to avoid duplicating complex initialization logic.
You need many similar objects with only slight differences.
The Prototype Pattern allows you to create new instances by cloning a pre-configured prototype object, ensuring consistency while reducing boilerplate and complexity.


1. The Challenge of Cloning Objects
Imagine you have an object in your system, and you want to create an exact copy of it. How would you do it?

Your first instinct might be to:

Create a new object of the same class.
Manually copy each field from the original object to the new one.
Simple enough, right?

Well, not quite.

Problem 1: Encapsulation Gets in the Way
This approach assumes that all fields of the object are publicly accessible. But in a well-designed system, many fields are private and hidden behind encapsulation. That means your cloning logic can’t access them directly.

Unless you break encapsulation (which defeats the purpose of object-oriented design), you can’t reliably copy the object this way.

Problem 2: Class-Level Dependency
Even if you could access all the fields, you'd still need to know the concrete class of the object to instantiate a copy.

This tightly couples your cloning logic to the object's class, which introduces problems:

It violates the Open/Closed Principle.
It reduces flexibility if the object's implementation changes.
It becomes harder to scale when you work with polymorphism.
Problem 3: Interface-Only Contexts
In many cases, your code doesn’t work with concrete classes at all, it works with interfaces.

For example:


Here, you know the object implements a certain interface (Shape), but you don’t know what class it is, let alone how to create a new instance of it. You’re stuck unless the object knows how to clone itself.

The Better Way: Let the Object Clone Itself
This is where the Prototype Design Pattern comes in.

Instead of having external code copy or recreate the object, the object itself knows how to create its clone. It exposes a clone() or copy() method that returns a new instance with the same data.

This:

Preserves encapsulation
Eliminates the need to know the concrete class
Makes the system more flexible and extensible
Shallow Copy vs Deep Copy

This is the most important concept to understand when implementing Prototype. Getting it wrong produces bugs that are subtle, intermittent, and painful to debug.

What is Shallow Copy?
A shallow copy duplicates the object itself but shares references to any objects it contains. Primitive fields (int, double, boolean) get copied by value. Reference fields (lists, maps, other objects) get copied by reference, meaning both the original and the clone point to the same underlying object.

What is Deep Copy?
A deep copy duplicates everything: the object, all objects it references, all objects those reference, and so on recursively. The clone is completely independent.

This is what Prototype implementations should do when the object contains mutable reference types.

Let’s walk through a real-world example to see how we can apply the Prototype Pattern to build a more efficient and maintainable object creation workflow.


2. The Problem: Spawning Enemies in a Game
Let’s say you’re developing a 2D shooting game where enemies appear frequently throughout the gameplay.

You have several enemy types with distinct attributes:

BasicEnemy: Low health, slow speed. Used in early levels.
ArmoredEnemy: High health, medium speed. Harder to defeat, appears later.
FlyingEnemy: Medium health, fast speed. Harder to hit, used for surprise attacks.
Each enemy type comes with predefined properties such as:

Health (how much damage they can take)
Speed (how quickly they move across the screen)
Armor (whether they take reduced damage)
Weapon type (e.g., laser, cannon, missile)
Sprite or appearance (the visual representation)
Now, imagine you need to spawn a FlyingEnemy. You might write code like this:


And you’ll do the same for dozens, maybe hundreds, of similar enemies during the game.

But Here’s the Problem
Repetitive Code: You’re duplicating the same instantiation logic again and again.
Scattered Defaults: If the default speed or weapon of FlyingEnemy changes, you need to update it in every single place you created one.
Error-Prone: Forget to set one property? Use a wrong value? Bugs will creep in silently.
Cluttered Codebase: Your main game loop or spawn logic becomes bloated with object construction details.
As your game scales (adding more enemy types, behaviors, or configurations) this naive approach quickly becomes hard to manage and maintain.

You need a clean, centralized, and reusable way to create enemy instances with consistent defaults while allowing minor tweaks.

To avoid repetitive instantiation and duplicated setup logic, we turn to the Prototype Design Pattern.


3. The Prototype Design Pattern
The Prototype pattern specifies the kinds of objects to create using a prototypical instance and creates new objects by copying (cloning) this prototype.

Instead of configuring every new object line-by-line, we define a pre-configured prototype and simply clone it whenever we need a new instance.

Two ideas define the pattern:

Self-cloning: The object itself knows how to create a copy of itself. No external code needs to understand its internal structure.
Decoupled creation: The client does not need to know the concrete class of the object it is cloning. It works through a common interface with a clone() method.

Class Diagram



clone()
Client
<<interface>>
Prototype
+ clone(): Prototype
ConcretePrototypeA
- field1: string
+ ConcretePrototypeA(prototype)
+ clone(): Prototype
ConcretePrototypeB
- field2: string
+ ConcretePrototypeB(prototype)
+ clone(): Prototype
Prototype has four participants, three required and one optional:

1. Prototype (Interface)
Declares the clone() method that all cloneable objects must implement
Defines the contract for self-cloning and allows clients to clone objects without knowing their concrete class
2. ConcretePrototype
Implements the clone() method to produce a copy of itself
Copies all fields to the new instance (shallow or deep, depending on the fields)
3. Client
Creates new objects by asking a prototype to clone itself
Holds a reference to a prototype instance and calls clone() when a new object is needed
Optionally customize the clone after creation
4. Prototype Registry (Optional)
Stores a collection of pre-configured prototypes, indexed by a key (string, enum, etc.)
Returns clones (not originals) when clients request objects by key

4. How It Works
The Prototype workflow follows five steps:




Copy all fieldsfrom original
Client customizesthe clone freely
get("flying")
clone()
new Enemy(this)
clone instance
clone
clone
setHealth(80)
Client
Prototype Registry
Prototype
Clone
Client
Prototype Registry
Prototype
Clone




9 / 9
Step 1: Create Prototype Instances
You configure a set of fully initialized objects that represent the "template" for each type you need. For our game, this means creating one FlyingEnemy with the correct health, speed, armor, and weapon.

Step 2: Register Prototypes (Optional)
Store these prototypes in a registry, keyed by a meaningful name like "flying" or "armored". This centralizes configuration and makes it easy to add new types.

Step 3: Client Requests a Clone
When the game needs to spawn a new enemy, it asks the registry for a clone by key. The registry looks up the prototype and calls its clone() method.

Step 4: Prototype Copies Itself
The prototype creates a new instance and copies its own fields into it. The clone is a separate object in memory with the same values as the original.

Step 5: Client Customizes the Clone
The client can modify the clone without affecting the original. For example, reducing a spawned enemy's health to create a "weakened" variant.

The client never calls a constructor directly. It asks the registry for a clone, gets back a fully configured object, and optionally tweaks it. The original prototype stays untouched for future cloning.


5. Implementing Prototype
Let’s refactor our enemy spawning system in a game using this pattern.

We’ll break the implementation down into 4 clear steps:

Step 1: Define the Prototype Interface
The interface declares a single clone() method. Every cloneable object must implement it.


Java







123
The interface is minimal by design. It says nothing about what fields exist, what types are involved, or how cloning should be performed. That is the concrete class's responsibility.

Step 2: Concrete Prototype with Shallow Clone
The Enemy class implements the interface. This first version performs a shallow clone, which works correctly here because all fields are primitives or immutable strings.


Java







12345678910111213141516171819202122232425262728293031
Key Points:
The clone() method creates a new instance using the copy constructor pattern, passing its own fields as arguments
This is a shallow clone: all fields here are primitives or immutable (strings), so shallow is sufficient
The clone is a completely independent object. Calling setHealth() on the clone does not affect the original
But what happens when we add a mutable reference field? That is where shallow cloning breaks.

Step 3: Deep Clone (Handling Mutable Fields)
Let us add an inventory field, a list of items the enemy carries. With shallow cloning, the original and clone share the same list. To fix this, the clone() method must create a new copy of every mutable reference field:


Java







12345678910111213141516171819202122232425262728293031
Step 4: Prototype Registry
The registry stores pre-configured prototypes and returns clones on request. This centralizes all configuration in one place.


Java







123456789101112131415161718
Notice that get() always returns a clone, never the stored prototype. This is critical. If it returned the original, clients could modify the prototype and corrupt all future clones.

Step 5: Client Code
Here is how everything comes together. The client configures prototypes once, registers them, and then spawns enemies by cloning:


Java







123456789101112131415161718192021222324
Expected Output:







123

6. Practical Example: Email Templates
Let us apply Prototype to a completely different domain. You are building a bulk email system. Your company sends a monthly newsletter, but each department needs a slightly different version with a customized subject line and department-specific recipients.

The base template defines the shared body text and a default recipient list, and you clone it for each department variant.

Without Prototype, you would duplicate the full constructor call for every department email, copying all the shared fields each time. With Prototype, you define the base template once and clone it for each variant.

The key challenge is that RecipientList is a nested mutable object containing two lists: to and cc. A shallow clone would cause all email templates to share the same recipient lists, so adding a recipient to the marketing email would also add them to every other department's email.


Java

Run







12345678910111213141516171819202122232425262728293031
The HR email has an extra recipient and a CC, the marketing and engineering emails each have their own added recipient, but the base template is completely unaffected.

This works because clone() calls recipients.deepCopy(), creating an independent RecipientList for each email. Without the deep copy, adding  to the marketing email would also add it to every other department's email and to the base template itself.
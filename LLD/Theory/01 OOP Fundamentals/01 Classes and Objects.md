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




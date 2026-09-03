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


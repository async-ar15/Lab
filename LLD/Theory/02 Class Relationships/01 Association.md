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


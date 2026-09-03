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



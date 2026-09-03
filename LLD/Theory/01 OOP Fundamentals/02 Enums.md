4. Enum
So far, we have looked at creating objects and storing data inside them. But sometimes, a field should only accept a small set of predefined values.

For that, we can use enums. An enum is useful when a value can only be one of a fixed set of options.

For example, think about an order in an e-commerce application. An order might be pending, shipped, delivered, or cancelled.

We could represent the status using a string:

String status = "pending";

// somewhere else in the code
status = "shiped"; // typo, but the code still compiles

But strings are easy to mistype, and the compiler will not catch this mistake.

A safer approach is to use an enum:

enum OrderStatus {
    PENDING, SHIPPED, DELIVERED, CANCELLED
}

OrderStatus status = OrderStatus.PENDING;
status = OrderStatus.SHIPPED; // only valid values are allowed

This prevents unsupported values and makes the code easier to understand.

Whenever you have a fixed set of values, such as OrderStatus, PaymentStatus, UserRole, or DifficultyLevel, enums are usually a good choice.


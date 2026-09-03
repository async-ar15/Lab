15. Dependency
So far, these relationships describe objects that are usually stored inside another object. But sometimes, a class simply needs another class to perform some work.

That is a dependency.

A dependency exists when one class uses another class to perform its work. It is generally a weaker relationship than association.

In an association, one class often keeps a reference to another object as part of its state:

class InvoiceService {

    private TaxCalculator taxCalculator;

    InvoiceService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }

    void createInvoice(int amount) {
        int tax = taxCalculator.calculateTax(amount);
        int total = amount + tax;
        System.out.println("Invoice total: " + total);
    }
}

With a dependency, the other object may only appear as a method parameter, local variable, or return type:

class InvoiceService {

    void createInvoice(int amount, TaxCalculator taxCalculator) {
        int tax = taxCalculator.calculateTax(amount);
        int total = amount + tax;
        System.out.println("Invoice total: " + total);
    }
}

Here, InvoiceService depends on TaxCalculator to calculate the tax. It does not create or own the calculator. It simply uses it to complete the operation. That is dependency.

Dependencies are common in real-world applications:

An OrderService may depend on an inventory service.
A UserService may depend on an email sender.
A ReportService may depend on a PDF generator.
Dependencies need to be managed carefully. When a class creates and depends directly on many concrete implementations, the code becomes tightly coupled and harder to test.

That is why good object-oriented designs often depend on interfaces and receive dependencies through constructors or method parameters. This allows implementations to be replaced without rewriting the high-level logic.

These concepts form the foundation for everything that follows in low-level design: SOLID principles, design patterns, and interview problems that build on them.


# 15 Must-Know Design Patterns

23 Gang of Four Patterns

Creational
how objects are created

Structural
how objects are composed

Behavioral
how objects communicat

15 Design Patterns

Creational
Singleton, Builder,
Factory Method

Structural
Adapter, Facade, Proxy,
Decorator, Composite

Behavioral
Strategy, Observer, State, Command,
Template Method, Iterator,
Chain of Responsibility

Creational Patterns


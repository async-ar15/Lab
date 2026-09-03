4. Adapter
The Adapter Pattern helps two components work together when their interfaces do not match.

Think of a travel adapter. If your charger and the wall socket use different plugs, the adapter can translate between them. The same idea applies in software.

Consider a checkout system for an e-commerce application. It expects to implement this interface for every payment provider:


Java

interface PaymentProcessor {
    void pay(int amountInCents);
}





123
But an external third-party gateway that we need to support exposes a different method name, and expects the amount in dollars:


Java

class ExternalPaymentGateway {

    public void makePayment(double amountInDollars) {
        System.out.println("Charging $" + amountInDollars);
    }
}






123456
The interfaces do not match. This is where the Adapter Pattern helps.

Instead of changing the gateway or rewriting our checkout system, we can create an adapter that implements the interface our application expects:


Java

class ExternalPaymentAdapter implements PaymentProcessor {

    private final ExternalPaymentGateway gateway;

    public ExternalPaymentAdapter(ExternalPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(int amountInCents) {
        double amountInDollars = amountInCents / 100.0;
        gateway.makePayment(amountInDollars);
    }
}





1234567891011121314
The adapter receives the request in the format our application understands. It then converts the amount from cents to dollars and calls the method provided by the legacy gateway.


```mermaid
graph LR
    Checkout["Checkout"] -- "pay 4999 cents" --> EPA["ExternalPaymentAdapter<br>implements PaymentProcessor"]
    EPA -- "makePayment 49.99<br>dollars" --> EPG["ExternalPaymentGateway"]
```

pay 4999 cents

makePayment 49.99 dollars

Checkout

ExternalPaymentAdapter
implements PaymentProcessor

ExternalPaymentGateway

Now the checkout system can use the external payment provider through the standard PaymentProcessor interface:


Java

PaymentProcessor processor =
        new ExternalPaymentAdapter(
                new ExternalPaymentGateway()
        );

processor.pay(4999);







123456
It does not need to know that the adapter is translating method names, data formats, or units internally.

The Adapter Pattern helps incompatible components work together. But what if the system works correctly and is simply too complicated to use? That brings us to the Facade pattern.



1. Singleton
Sometimes, an application needs exactly one shared instance of a class. Common examples include a configuration manager, logger, cache manager, or thread pool. Creating multiple instances of these objects can waste resources or lead to inconsistent behavior.

The Singleton Pattern solves this by ensuring that only one instance of a class exists, and providing a consistent way to access it.



enum AppConfig {
    INSTANCE;

    private String environment = "production";

    public String getEnvironment() {
        return environment;
    }
}

We can now access the same instance from anywhere in the application:

// in OrderService
String env = AppConfig.INSTANCE.getEnvironment();

// in EmailWorker
log(AppConfig.INSTANCE.getEnvironment());

However, Singleton should be used carefully. Because it is globally accessible, it can hide dependencies and make testing harder.

In many cases, creating the object once and passing it explicitly to the classes that need it is cleaner:

// created once, then passed in
AppConfig config = new AppConfig("production");

OrderService orderService = new OrderService(config);
EmailWorker emailWorker = new EmailWorker(config);


Now the dependency is visible in the constructor, and a test can pass a different AppConfig.


The core idea

Use Singleton when the application genuinely requires one shared instance, not simply because global access is convenient.

Singleton ensures that only one object of a class is ever created. But what if creating a single object is itself complicated to construct? That brings us to the Builder Pattern.



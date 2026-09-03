8. Encapsulation

The first pillar is encapsulation.

Encapsulation means keeping data and the methods that operate on that data together inside a class, while controlling how that data can be accessed or modified from outside.

For example, imagine we have a BankAccount class that exposes its balance directly:

class BankAccount {

    public String owner;
    public double balance;

    BankAccount(String owner) {
        this.owner = owner;
    }

    boolean isOverdrawn() {
        return balance < 0;
    }
}

Now any part of the application can modify the balance:

BankAccount account = new BankAccount("Alice");
account.balance = 500;

// checkout service
account.balance -= 800;

// nightly refund job
account.balance += 250;

// an internal admin tool
account.balance = -1000;

account.isOverdrawn(); // true

This is dangerous because the object can easily end up in an invalid state.

A better design is to keep the balance private and provide controlled methods for modifying it:


class BankAccount {

    private double balance;

    void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("deposit must be positive");
        }
        balance += amount;
    }

    void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("invalid withdrawal");
        }
        balance -= amount;
    }

    boolean isOverdrawn() {
        return balance < 0;
    }
}

Now the balance cannot be changed directly. It can only be modified through deposit and withdraw, where we can enforce the necessary rules.

That is encapsulation. It protects objects from invalid states and keeps the logic for managing their data in one place.

Access modifiers we discussed earlier, such as private and protected, are the tools that help us implement encapsulation.



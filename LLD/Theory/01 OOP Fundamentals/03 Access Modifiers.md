5. Access Modifiers
Now that we can define data, we also need to control which parts of a class should be visible from outside. That is where access modifiers come in.

Access modifiers control where a class, field, constructor, or method can be accessed.

In Java, the three access modifiers you will use most often are public, private, and protected.


Modifier	Accessible From
public	Anywhere in the application
private	Only inside the same class
protected	The same package and child classes



class UserAccount {
    private String email;

    public String getEmail() {
        return email;
    }

    public void updateEmail(String newEmail) {
        if (isValidEmail(newEmail)) {
            email = newEmail;
            logChange();
        }
    }

    protected void logChange() {
        System.out.println("Account updated");
    }

    private boolean isValidEmail(String email) {
        return email.contains("@");
    }
}


Here, getEmail and updateEmail are public because other parts of the application need to use them.

isValidEmail is private because it is an internal implementation detail.

logChange is protected, which means a child class could reuse or customize it.

In general, expose only what other parts of the application actually need. Keep everything else private whenever possible.


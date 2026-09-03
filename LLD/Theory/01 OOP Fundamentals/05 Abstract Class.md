7. Abstract Class

Interfaces work well when different classes need to follow the same contract. But sometimes, related classes also need to share common code or state.

That is where abstract classes become useful.

An abstract class is a class that cannot be instantiated directly. It can provide shared fields and methods while leaving some behavior for child classes to implement. 

For example, imagine we need to generate reports in different formats:


abstract class ReportGenerator {

    void generate(List<String> data) {
        String content = format(data);
        save(content);
    }

    void save(String content) {
        System.out.println("Saving report: " + content);
    }

    abstract String format(List<String> data);
}

class CsvReportGenerator extends ReportGenerator {
    String format(List<String> data) {
        return String.join(",", data);
    }
}

class JsonReportGenerator extends ReportGenerator {
    String format(List<String> data) {
        return "[\"" + String.join("\", \"", data) + "\"]";
    }
}


#### did not understand 

The generate and save methods contain behavior shared by every report generator. But the format method is abstract because each report type formats its data differently.

Both child classes reuse the common report-generation flow while providing their own formatting logic.


When to Use Interfaces vs Abstract Classes

Use an interface when you want to define a contract or capability.
Use an abstract class when closely related classes need to share common code or state.

Java interfaces can also contain default methods, so the difference is not always absolute. But this rule is a useful starting point.

# The Four Pillars of OOP


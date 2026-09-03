7. Decorator
The Decorator Pattern lets us add new behavior to an object dynamically without modifying its original class.

Consider a rich-text editor that starts with a basic component that renders plain text:


Java

interface TextView {
    String render();
}

class PlainTextView implements TextView {
    private final String text;

    public PlainTextView(String text) {
        this.text = text;
    }

    public String render() {
        return text;
    }
}





123456789101112131415
We can use it like this:


Java

TextView plain = new PlainTextView("Design Patterns");

System.out.println(plain.render());   // Design Patterns





123
This works perfectly for simple text without any formatting.

But now we also need to support bold, italic, underline, and different combinations of these styles. Creating a subclass for every combination would quickly lead to a subclass explosion.

Instead, we create small decorators that wrap another TextView and add one style. We first create a base decorator called TextDecorator. It implements the same TextView interface as PlainTextView and wraps another TextView object:


Java

abstract class TextDecorator implements TextView {
    protected final TextView wrapped;

    public TextDecorator(TextView wrapped) {
        this.wrapped = wrapped;
    }
}





1234567
We can then create separate decorators for bold, italic, and underlined text:


Java

class BoldDecorator extends TextDecorator {
    public BoldDecorator(TextView t) { super(t); }
    public String render() { return "<b>" + wrapped.render() + "</b>"; }
}

class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(TextView t) { super(t); }
    public String render() { return "<i>" + wrapped.render() + "</i>"; }
}

class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(TextView t) { super(t); }
    public String render() { return "<u>" + wrapped.render() + "</u>"; }
}





1234567891011121314
Now we can combine these decorators however we need. The client applies multiple formatting styles by wrapping them around one another:


Java
TextView plain =
        new PlainTextView("Design Patterns");

TextView bold =
        new BoldDecorator(plain);

TextView boldItalic =
        new ItalicDecorator(
                new BoldDecorator(plain));

TextView boldItalicUnderlined =
        new UnderlineDecorator(
                new ItalicDecorator(
                        new BoldDecorator(plain)));






1234567891011121314

```mermaid
graph LR
    UD["UnderlineDecorator"] --> ID["ItalicDecorator"]
    ID --> BD["BoldDecorator"]
    BD --> PTV["PlainTextView<br>Design Patterns"]
```



UnderlineDecorator

ItalicDecorator

BoldDecorator

PlainTextView
Design Patterns

Each decorator adds one responsibility and then delegates the remaining work to the object it wraps. And because every decorator implements the same TextView interface, the client interacts with the text in exactly the same way, regardless of how many decorators have been applied.

A Decorator wraps one object and adds behavior around it. But what if an object needs to contain other objects? That brings us to the next pattern: Composite.



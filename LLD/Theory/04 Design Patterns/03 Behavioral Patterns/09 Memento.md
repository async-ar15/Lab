Memento Design Pattern
Low Priority
11 min read
Updated July 2, 2026

Listen to this chapter
Unlock Audio




Caretaker · history
Originator
Memento
Snapshot 1
Snapshot 2
save()
store()
restore()
The Memento Design Pattern is a behavioral design pattern that lets you capture and store an object’s internal state so it can be restored later, without violating encapsulation.

It’s particularly useful in situations where:

You need to implement undo/redo functionality.
You want to support checkpointing or versioning of an object’s state.
You want to separate the concerns of state storage from state management logic.
Let’s walk through a real-world example to see how we can apply the Memento Pattern to solve a problem that involves implementing undo functionality in a text editor.


1. The Problem: Implementing Undo in a Text Editor
Imagine you’re building a simple text editor. The editor supports basic operations like:

type(String text) – appends text to the current document
getContent() – returns the current document text
undo() – reverts to the previous version of the content
Implementing typing and reading is easy. The real challenge is undo.

The Naive Approach: Client Manages Snapshots
The most straightforward approach is to have the client manually capture the editor's state before each operation and feed it back during undo:




Stores "Hello" locally
Stores "Hello World" locally
content = "Hello"
type("Hello")
getContent()
"Hello"
type(" World")
getContent()
"Hello World"
undo("Hello")
Client
TextEditor
Client
TextEditor




10 / 10
The client is doing all the heavy lifting: fetching internal state, storing it, and feeding it back during undo.


Java







123456789101112131415
Here is how the client uses it:


Java







1234567891011121314151617
What’s Wrong with This Design?
While this naive implementation works for very basic undo logic, it introduces several major issues:

1. Encapsulation is Broken
The client must call getContent() to fetch internal state and pass it directly to undo(). This means the client knows that the editor's state is a string called "content."

If the editor later adds cursor position, selection range, or formatting metadata, the client must be updated to snapshot all of those too. The editor's internal structure has leaked into every class that implements undo.

2. Client Bears the Responsibility
The client must remember to take a snapshot before every operation. Miss one, and you have a gap in your undo history. This is manual, error-prone, and scatters undo logic across the entire codebase instead of centralizing it.

3. Not Scalable
What if the editor's state grows to include cursor position, selection range, font formatting, and scroll position? The client would need to capture all of those fields separately, store them in some custom structure, and feed them all back during undo. The snapshot logic balloons in complexity, and it is all in the wrong place, outside the editor instead of inside it.

4. No Separation of Concerns
The same code that handles user interactions is also managing state snapshots. This violates the single responsibility principle and makes both the UI code and the undo logic harder to test, maintain, and extend.

What We Really Need
We need a design that:

Lets the TextEditor capture and restore its own state without exposing its internals
Gives the client a way to manage state history without understanding what is inside each snapshot
Scales cleanly when the editor's internal state grows more complex
Separates undo management from editing logic
This is exactly what the Memento pattern provides.


2. What is the Memento Pattern
The Memento Design Pattern allows an object to save and restore its state without exposing its internal structure. It achieves this by encapsulating the state in a special object called a Memento.

Two characteristics define the Memento pattern:

State capture without exposure. The originator (the object whose state you want to save) creates a memento that contains a snapshot of its private internal state. The memento does not expose that state to the outside world. Only the originator can read it back. This preserves encapsulation while enabling state restoration.
External state management. A separate object called the caretaker stores and manages the mementos. The caretaker decides when to save (before a risky operation) and when to restore (undo). But it never inspects or modifies the memento's contents. It treats each memento as an opaque black box.
Class Diagram



creates
history
Originator
- state: any
+ save(): Memento
+ restore(m: Memento)
Memento
- state: any
+ Memento(state)
+ getState()
Caretaker
- history: Memento[]
+ undo()
Memento has three participants.

1. Originator (e.g., TextEditor)
The object whose internal state you want to capture and restore.

The originator is the only participant that touches its own private fields. It packages them into a memento during save, and unpacks them during restore. No other object needs to know what those fields are or how they are structured.

2. Memento (e.g., TextEditorMemento)
An immutable snapshot of the originator's state at a specific point in time. Store the originator's state in a way that prevents external modification

3. Caretaker (e.g., UndoManager)
The external object that decides when to save and restore state. It manages the lifecycle of mementos. It never examines or modifies the content of a memento, it just treats it as a black box.


3. How It Works
Here is the Memento workflow step by step:




Push onto history stack
state changes
Pop from history stack
State restored
save(editor)
save()
new Memento(state)
memento
type(" World")
undo(editor)
restore(memento)
getState()
Client
UndoManager
TextEditor
Memento
Client
UndoManager
TextEditor
Memento




12 / 12
Step 1: Caretaker requests a save
Before the user performs an operation (like typing, deleting, or formatting), the caretaker asks the originator to save its current state.

Step 2: Originator creates a memento
The originator reads its own private fields, packages them into a new memento object, and returns it to the caretaker.

Step 3: Caretaker stores the memento
The caretaker pushes the memento onto a history stack (or list). It does not open the memento or read its contents.

Step 4: User performs operations
The originator's state changes through normal operations (typing text, moving shapes, etc.).

Step 5: User triggers undo
The caretaker pops the most recent memento from the history stack and passes it to the originator.

Step 6: Originator restores from the memento
The originator reads the state from the memento and overwrites its current fields. The object is now back to exactly how it was when the memento was created.


4. Implementing Memento Pattern
Let’s refactor our naive text editor into a clean, maintainable design using the Memento Pattern. We will create the memento, then the originator, then the caretaker, and finally wire them together in client code.




creates

stores

calls save/restore

TextEditor

-content: String

+type(text: String)

+getContent() : : String

+save() : : TextEditorMemento

+restore(m: TextEditorMemento)

TextEditorMemento

-state: String

+getState() : : String

TextEditorUndoManager

-history: Stack<TextEditorMemento>

+save(editor: TextEditor)

+undo(editor: TextEditor)

Step 1: Create the Memento - TextEditorMemento
The memento stores a snapshot of the TextEditor's internal state. It has three important properties:

Immutable — fields are private final (or readonly) and cannot be changed after creation
Minimal — it stores only what is needed for restoration
Encapsulated — only the originator should read its contents

Java







1234567891011
This class is passive. It does not contain any logic, just a frozen snapshot of the editor's state at the moment it was created.

Step 2: Create the Originator – TextEditor
The originator is the object whose state we want to save and restore. It provides two key methods beyond its normal operations:

save() — creates a memento capturing the current state
restore(memento) — replaces the current state with the state from the memento

Java







12345678910111213141516171819202122
Notice that the save() and restore() methods are the only ones that interact with the memento. The rest of the editor (typing, getting content) works exactly as before. The memento pattern adds state capture without changing how the object normally operates.

Step 3: Create the Caretaker – TextEditorUndoManager
The caretaker manages the history of mementos. It is responsible for:

Asking the originator to save its state at the right times
Storing mementos in a stack (last-in, first-out for undo)
Passing mementos back to the originator for restoration
Never inspecting or modifying memento contents

Java







12345678910111213141516171819
The TextEditorUndoManager allows undo operations without the client managing snapshots directly. Notice that save() and undo() take a TextEditor reference but never call getContent() or any other method that exposes internal state. They only call save() and restore(), which return and accept opaque mementos.

Step 4: Using the Memento from the Client
Now let's put it all together. The client creates an editor and an undo manager, performs operations, saves state at appropriate moments, and undoes when needed.


Java







1234567891011121314151617181920212223242526
Expected Output:

What We Achieved
Encapsulation: Editor’s internal state is never exposed directly to the client
Clean undo logic: The client doesn’t need to manage or interpret state — it just saves and restores
Separation of concerns: The TextEditor handles state, and the TextEditorUndoManager handles history
Scalability: Easy to extend with redo support, multi-level undo, or persistent versioning

5. Evolving the System: Adding Cursor Position
What happens when the product manager says "We need to restore cursor position too, not just content"? With the naive approach, this would be a nightmare. The client would need to capture two fields instead of one, store them in some tuple or object, and know about both when undoing.

With Memento, the change is entirely inside the originator.


Java







12345678910111213141516171819202122232425262728293031
The undo manager did not change at all. Neither did the client code. The only files that changed were the memento and the editor. This is the Open/Closed Principle at work. You extended the system's capability (restoring cursor position) by modifying only the originator and its memento, without touching any external code.

If you later add selection range, scroll position, or font formatting, the same principle applies. The memento and editor grow, but the undo manager and client remain untouched.
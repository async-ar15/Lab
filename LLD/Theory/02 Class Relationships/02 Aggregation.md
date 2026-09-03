3. Aggregation
Sometimes, however, one object does more than just know about other objects. It may also contain or group them.

That brings us to aggregation.

Aggregation is a special type of association that represents a has-a relationship with weak ownership. One object contains or groups other objects, but those objects can still exist independently.


For example, consider an engineering team and its developers:

class EngineeringTeam {
    private String teamName;
    private List<Developer> developers;

    EngineeringTeam(String teamName, List<Developer> developers) {
        this.teamName = teamName;
        this.developers = developers;
    }
}

An engineering team has developers, but it does not fully own them. The developers are created outside the team and then passed to it:

Developer developer1 = new Developer("Alice");
Developer developer2 = new Developer("Bob");

EngineeringTeam team = new EngineeringTeam(
    "Payments",
    List.of(developer1, developer2)
);

If the team is removed, the developers still exist. They can move to another team or remain without one.

That is aggregation. The key idea is that the contained objects have an independent lifecycle.

But sometimes, the parent object has much stronger ownership over the objects it contains. That relationship is called composition.


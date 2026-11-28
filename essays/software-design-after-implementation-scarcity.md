# Software design after implementation scarcity

*Reconsidering familiar design decisions when writing and replacing code become less expensive.*

Imagine an engineer adding support for another supplier's catalogue to a purchasing system. The new supplier provides a CSV file whose structure resembles that of an existing supplier.

There is an obvious opportunity for reuse. Both importers need to interpret product identifiers, read prices, normalize units and deal with missing fields.

The engineer could extract the common functionality into a framework for supplier imports. The next supplier might then require only a small amount of configuration.

This is a familiar way to approach the problem. Writing and maintaining similar implementations has traditionally been expensive.

But suppose an agent can produce a straightforward importer for the new supplier quite cheaply, and the team already has a reliable way to verify its behavior. Does the shared framework still make economic sense?

Perhaps it does. But we should examine why rather than assume the answer.

## Duplication and coupling

The two suppliers represent their prices differently. One publishes prices inclusive of tax. The other publishes prices exclusive of tax. They also have different conventions for unavailable products.

A shared importer might handle these differences with configurable behaviors. As more suppliers are added, the framework accumulates options, extension points and conditional logic.

Every supplier benefits from improvements to the framework. Every supplier is also exposed to mistakes introduced when the framework changes.

There is another possibility: maintain small independent importers, each of which converts its supplier's representation into the purchasing system's established model.

Some implementation details will be duplicated. But the engineer making a change for one supplier need not necessarily understand the behavior of the others.

Cheaper implementation makes this a more attractive trade-off than it once was.

There are still good reasons for sharing code. Suppose all the suppliers use the same external service for obtaining currency exchange rates. The service's authentication and failure handling deserve consistent treatment. Maintaining several versions might create needless operational work.

The distinction concerns which things are expected to change together.

If a correction to one implementation should generally be applied to every implementation, keeping them independent creates recurring work. If they represent genuinely independent concerns, bringing them together creates a recurring need for coordination.

Similarity in code is an imperfect guide to that decision.

## Keep one meaning where one meaning matters

The purchasing system has an important obligation: an accepted purchase order must retain the agreed price, even when the supplier subsequently publishes a different catalogue.

The team can afford to maintain several independent catalogue importers. It should be much more reluctant to maintain several independent interpretations of what an accepted price means.

This distinction becomes increasingly important when software is easy to generate.

An agent could implement a little calculation in each consumer that appears to preserve the agreed price. Each implementation might be simple and individually understandable. Over time, however, they could disagree about taxes, discounts, rounding or the effect of an amended purchase order.

The organization would then need to reconcile different accounts of the same commitment.

Some things deserve centralization because they have one authoritative meaning in the business. Others deserve it because a security guarantee, operational responsibility or external dependency must be handled consistently.

Neither justification depends primarily on the number of lines of code saved.

When deciding whether to reuse something, ask whether the duplication is in the implementation or in the meaning.

Duplicating a file-reading routine may be harmless. Duplicating responsibility for establishing how much a customer owes is a different matter.

## Be more selective about extensibility

Suppose the purchasing team needs to support three kinds of supplier discount.

It could design a general discount engine that accommodates arbitrary conditions and combinations. That might be sensible if discount flexibility is central to the product and customers need to define their own arrangements.

Alternatively, the three discounts could have separate, straightforward implementations.

Under the old economics, the general engine might have been attractive partly because implementing a fourth discount would otherwise require substantial effort.

When implementing another narrow case becomes inexpensive, that justification weakens. The team still has to weigh the cost of understanding a general engine, verifying interactions between its rules and explaining its behavior to customers.

The important question is not how easy it will be to add the next piece of code. It is how easy it will be to understand the resulting software.

A useful generalization makes a collection of situations easier to reason about. A poor one makes ordinary situations difficult to understand because it must accommodate every conceivable variation.

There is no particular virtue in avoiding abstraction. A well-chosen abstraction can substantially reduce the reasoning required to change a system.

But an abstraction that mainly saves implementation effort may deserve reconsideration as that effort becomes less expensive.

## Preserve understanding before preserving implementations

The purchasing team's existing importers contain considerable knowledge of suppliers.

Some of that knowledge is explicit: which identifier is stable, how the supplier represents unavailable stock and whether catalogue prices include tax.

Some of it is buried in conditionals added after previous integration failures.

If the team wants the freedom to replace an importer economically, it should make the relevant understanding accessible independently of the implementation.

Reviewed supplier examples are especially useful. They can establish how a new importer ought to interpret particular situations. The same examples may help support staff diagnose a failed import or product engineers determine whether a proposed common model is adequate.

The original code remains valuable evidence. But it need not remain the only place where the organization can recover what it has learned.

This suggests a different allocation of engineering effort.

Spend less on making every implementation accommodate an uncertain future. Spend more on understanding its obligations and developing reliable ways to determine whether a different implementation fulfills them.

The future implementation may then be easier to produce without the original being particularly extensible.

## Replacement is cheap only within certain boundaries

Consider a service that determines the delivery date for a purchase order.

If the service is a pure calculation with well-understood inputs and outputs, replacing its implementation may be straightforward.

Now suppose its estimates have been stored alongside accepted orders, displayed to customers and used by suppliers to allocate capacity. Changing the calculation may be easy. Changing the meaning of the dates already communicated to people may be impossible without further work.

Persistent state, public interfaces and external commitments have a different relationship with change than internal implementation decisions.

An architecture suitable for inexpensive replacement needs to recognize those differences.

The purchasing system might distinguish a predicted delivery date from a date that has been agreed with a supplier. Prediction methods could then change relatively freely, while agreed dates retain their meaning.

That distinction is useful regardless of which prediction method the organization chooses. It prevents a change in an implementation from silently changing an existing commitment.

[Designing for decisions we haven't made yet](https://grove.github.io/engineering-that-compounds/essays/designing-for-decisions-we-havent-made-yet.html) examines this problem in more detail.

Cheaper implementation can make careful design more important at the boundaries where decisions become difficult to undo.

## Standardize the things that make independent work possible

There are familiar arguments for using the same technology throughout an engineering organization.

People can move between teams more easily. Operational knowledge is shared. Dependencies can be maintained consistently. Security practices are easier to establish.

Some of these benefits may remain substantial even if implementation becomes inexpensive. Others depend on the difficulty of learning and working with unfamiliar code.

An organization should reconsider the reasons for each standard.

Perhaps every service needs to expose operational information in a consistent way because the same people respond to incidents. Perhaps every component needs a reliable means of reproducing its build and running its verification independently.

It is less obvious that every component must use the same internal application framework.

The relevant cost is the total cost of operating with the variation, including security updates, recovery, understanding and maintenance. Being able to generate an implementation in another language is not sufficient justification for introducing that language into production.

Standardization can enable independence when it supplies agreements and capabilities that teams can rely on. It can also prevent independence when it requires every team to coordinate changes through a shared implementation.

Those effects deserve to be distinguished.

## Design for the lifetime the software actually needs

A company may need a small service to coordinate a particular supplier migration. The service will be useful for six weeks. After that, the relevant suppliers will use the normal purchasing system.

It would be strange to design this service as though it must eventually support every possible supplier migration.

Yet organizations often do something resembling this because developing a narrow solution for a temporary need would not historically have justified its cost. The investment needs to serve additional purposes in the future.

As the economics change, a narrower solution may become preferable.

The difficult part is ensuring that the service can genuinely be withdrawn. Any commitments it creates need to be fulfilled. Necessary records must survive. Consumers must have somewhere appropriate to go.

[Software with an end date](https://grove.github.io/reimagining-software-engineering/essays/software-with-an-end-date.html) explores the implications of such deliberately temporary software.

A short-lived implementation can still leave behind enduring engineering capital: an improved understanding of supplier migrations, reviewed examples and a better means of verifying that outstanding orders have been transferred correctly.

There is no requirement that the application itself should survive for its development to have been worthwhile.

## Compare the costs that remain

The original question about the supplier importers can now be approached differently.

- **Introduce a shared implementation.** Which changes must now be coordinated? Which repeated work disappears?
- **Maintain separate implementations.** Which corrections will need to be repeated? Could their behavior diverge in unacceptable ways?
- **Introduce a general framework.** Does it make the domain easier to understand, or primarily make new code easier to write?
- **Keep several narrow solutions.** What operational and maintenance responsibilities multiply?
- **Extend an existing component.** Does the additional behavior fit its existing responsibilities?
- **Replace the component.** Which behavior, state and commitments must survive the transition?

There is no universally cheaper answer.

The point of reconsidering these decisions is that the relative costs may have changed. An implementation that was previously too expensive to duplicate may now be inexpensive. A shared dependency that once saved considerable effort may now primarily impose coordination costs.

Conversely, a common capability for verification may become more valuable because it can support a much wider range of independently produced implementations.

## Make the next change easier to understand

I would be reluctant to replace one collection of familiar software design principles with another.

Always duplicate. Never abstract. Prefer replacement to extension. These prescriptions would be as unhelpful as applying the original principles without regard for the circumstances.

A more durable consideration is whether a design reduces the amount of understanding, evidence and coordination needed to make the next consequential change.

Sometimes the answer is a shared abstraction. Sometimes it is two small independent components. Sometimes it is removing a capability from the product because maintaining it is no longer worthwhile.

The purchasing team might settle on independent supplier importers, a shared representation of purchasing commitments, a small set of reviewed supplier situations and common machinery for observing and verifying imports.

Such an arrangement could allow the organization to add, remove or replace suppliers without repeatedly reasoning about the whole purchasing system.

More importantly, the understanding and verification developed for this work might subsequently help the team introduce a different way of sourcing products or managing supplier relationships.

The economics of writing software influence which designs are sensible. But the lasting value of a design increasingly depends on what the organization can understand, change and become capable of doing because of it.

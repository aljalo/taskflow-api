# ADR-0001: Use a Modular Monolith

## Status

Accepted

## Date

2026-08-23

## Context

TaskFlow is a medium-sized project and task management backend that will
include authentication, users, projects, memberships, tasks, assignments,
comments, search, PostgreSQL persistence, security, testing, monitoring, and
cloud deployment.

The system requires clear business boundaries and maintainable code, but it
does not currently require independent deployment or scaling of individual
business capabilities.

The architecture must support learning and applying professional engineering
practices without introducing operational complexity that is not justified by
the product requirements.

## Decision

TaskFlow will be implemented as a Modular Monolith.

The application will:

- Be built and deployed as one Spring Boot application.
- Use one PostgreSQL application database.
- Be divided into business-focused modules.
- Use package-by-feature organization.
- Keep repositories and persistence details private to their owning modules.
- Prevent controllers from accessing repositories directly.
- Use application-facing interfaces for cross-module collaboration.
- Keep transaction boundaries inside application services.
- Avoid distributed communication between internal modules.

The initial modules are:

- Authentication
- User
- Project
- Membership
- Task
- Comment

Modules will be introduced incrementally when their features are implemented.
Empty package structures will not be created in advance.

## Decision Drivers

- The team needs simple local development.
- The product requires transactions across related business operations.
- The deployment process should remain understandable and inexpensive.
- Module boundaries must be explicit.
- The system must remain testable without distributed infrastructure.
- The architecture should allow later extraction when justified by evidence.
- The project does not currently have independent service teams.

## Considered Options

### Option 1: Layered Monolith

Organize the application using global technical packages such as:

```text
controller
service
repository
entity
dto
```

#### Advantages

- Familiar to many Spring developers.
- Simple for very small applications.
- Easy to generate initially.

#### Disadvantages

- Business features become scattered across the codebase.
- Global packages grow quickly.
- Module ownership is unclear.
- Unrelated features become coupled through shared services and repositories.

This option was rejected because TaskFlow is expected to grow beyond a small
CRUD application.

### Option 2: Modular Monolith

Organize one deployable application into business modules with internal
technical layers.

#### Advantages

- Keeps related business code together.
- Supports atomic database transactions.
- Requires one deployment unit.
- Reduces operational complexity.
- Encourages explicit module ownership.
- Provides a path to future service extraction.

#### Disadvantages

- Module boundaries require discipline.
- The compiler does not automatically prevent every invalid dependency.
- One deployment still releases all modules together.
- One process failure can affect the complete application.

This option was selected because it provides strong structure without
premature distributed-system complexity.

### Option 3: Microservices

Deploy authentication, projects, tasks, comments, and other capabilities as
separate services.

#### Advantages

- Independent deployment.
- Independent scaling.
- Strong runtime boundaries.
- Potential ownership by separate teams.

#### Disadvantages

- Distributed transactions and eventual consistency.
- Network failure handling.
- Multiple deployment pipelines.
- More complex local development.
- Additional monitoring and security requirements.
- Higher Azure hosting cost.
- API versioning between internal services.
- Data ownership and migration complexity.

This option was rejected because the current product scale and team structure
do not justify its operational cost.

## Consequences

### Positive

- Developers run one application locally.
- Related operations can use database transactions.
- Deployment and rollback remain straightforward.
- Business capabilities have explicit ownership.
- Integration testing requires fewer moving parts.
- Infrastructure cost remains suitable for a learning project.
- Module extraction remains possible if boundaries are preserved.

### Negative

- All modules are deployed together.
- One application instance contains all business capabilities.
- Poor dependency discipline could degrade the design into a tightly coupled
  monolith.
- The team must review cross-module dependencies carefully.
- Scaling applies to the complete application rather than one internal module.

## Enforcement Guidelines

- New code is added to the module that owns the business capability.
- A module must not access another module's repository directly.
- A module must not expose its JPA entities as an integration contract.
- REST controllers must not contain business rules.
- REST controllers must not access repositories.
- Cross-module operations use explicit application-facing interfaces.
- Shared code must represent genuinely cross-cutting behavior.
- The `shared` package must not become a place for unrelated utilities.
- Architecture boundaries should be covered by automated tests when the module
  structure exists.

## Reconsideration Triggers

This decision should be reviewed if:

- Different modules require independent deployment schedules.
- One module requires significantly different scaling characteristics.
- Separate teams own independent business capabilities.
- Availability requirements differ between modules.
- A module requires independent data ownership.
- Release coordination becomes a measurable delivery bottleneck.
- Regulatory or security boundaries require process isolation.

A service must not be extracted only because microservices appear more modern.

## Related Documentation

- [TaskFlow Backend Architecture](../architecture.md)
- [TaskFlow Product Scope](../product-scope.md)

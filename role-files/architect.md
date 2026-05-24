# Architect Guidelines & Constraints

First section of this document captures architect constitution principles to be used when generating the project constitution via `speckit.constitution`. Second section captures project-specific overrides and additions. Third section defines the gate checklists. Here there is only one checklist and it applies to the plan speckit step.
Every mandatory bullet with an `ARCH-` or `ARCH-CHECK-` ID in this file is the source of truth and MUST be referenced by the generated constitution and downstream plan checks. Mandatory bullets must not be skipped, merged away, or summarized.
---

## 0. Reference and Source-of-Truth Principles

- **MANDATORY:** [ARCH-REF-001] This role file is the authoritative source for architect governance requirements.
- **MANDATORY:** [ARCH-REF-002] Generated constitution artifacts MUST reference this role file path and revision metadata instead of copying all mandatory bullets.
- **MANDATORY:** [ARCH-REF-003] Mandatory source items must not be skipped, merged, or summarized into generic wording when generating plan checks.
- **MANDATORY:** [ARCH-REF-004] Plan artifacts MUST explicitly cover each `ARCH-CHECK-` item with evidence, sourced from this role file.
- **MANDATORY:** [ARCH-REF-005] Any justified exception to a mandatory source item MUST be explicitly documented with rationale and approval.

---

## 1. Mandatory Constitution principles
### 1. System Architecture Style

- **MANDATORY:** [ARCH-01-01] **Repository structure**: Use Monorepo when a high degree of coupling between components and a single team works on them. Use Polyrepo when components are expected to be independently deployable or reusable across projects or multiple teams work on them.
- **MANDATORY:** [ARCH-01-02] **System components**: System components are applications, queues or databases that must support a separate set of architectural characteristics. Divide the szstem into meaningful set of components and idenfify most important characteristics of every component. 
- **MANDATORY:** [ARCH-01-03] **Overall style**: Always use modular style. Use Modular Monolith for simplicity and ease of development when the system is not expected to grow beyond a certain complexity. Use Microservices for large, complex systems with clear bounded contexts and the need for independent scalability or deployment. Use service based architecture as a middle ground when some modularity, scalability and independent deployment is desired but speed is a priority too.
- **MANDATORY:** [ARCH-01-04] **API style**: Use REST when simple and widely supported approach is needed. GraphQL is beneficial when clients need flexible queries and to minimize over-fetching. gRPC is ideal for high-performance, low-latency communication between internal services. Use hybrid approaches when different API styles are needed for different use cases (e.g., REST for public API, gRPC for internal services).
- **MANDATORY:** [ARCH-01-05] **Frontend/backend coupling**: Use fully decoupled architectures (separate deployments) for larger applications with distinct frontend and backend teams, or when the frontend must be reusable across multiple platforms (web, mobile). Use server-rendered (SSR/hybrid) architectures for smaller applications or when SEO is a concern, but account for the tighter coupling between frontend and backend this introduces.
- **MANDATORY:** [ARCH-01-06] **Background processing**: Use background processing for long-running or resource-intensive tasks that do not need to be completed within the request-response cycle. Choose a task queue based on language and ecosystem compatibility, ease of use, and scalability requirements.
- **MANDATORY:** [ARCH-01-07] **Background trigger**: Define trigger that triggers background processing for each existing processor. Possible triggers include: API call, event/message, scheduled/cron, or manual trigger. Avoid background processing for critical user flows that require strong consistency guarantees.
---

### 2. Module Boundaries
- **MANDATORY:** [ARCH-02-01] **Module granularity**: Define clear boundaries for each module based on business capabilities or domain concepts. Each module should have a single responsibility and encapsulate its internal implementation details.
- **MANDATORY:** [ARCH-02-02] **Interface / contract format**: Define API contracts using OpenAPI for REST, GraphQL SDL for GraphQL, or protobuf for gRPC. Contracts should be the source of truth for API design and implementation. Use AVRO or JSON Schema for event based communication contracts. Avoid code-first approaches that  lead to drift between implementation and design.
- **MANDATORY:** [ARCH-02-03] **Architecture as code**: Define layer, module and application interaction constraints using a formal architecture-as-code approach
- **MANDATORY:** [ARCH-02-04] **Communication pattern**: Synchronous HTTP is simpler to implement and reason about and avoids eventual consistency issues. Use it in simple applications and where fast response time is not critical. Use asynchronous HTTP for non-blocking operations. Event-driven architectures (message queue / pub-sub) provides better scalability, reliability and decoupling and is used for service based or microservices architectures.
- **MANDATORY:** [ARCH-02-05] **Event technology**: If event-driven, use Kafka for scalability, reliability and lack of back-pressure. 
- **MANDATORY:** [ARCH-02-06] **Shared libraries**: Use shared libraries for common utilities, domain models, and API contracts to avoid duplication and ensure consistency across services especially when services represent similar use cases. However, be cautious of tight coupling and versioning issues that can arise from shared libraries.
- **MANDATORY:** [ARCH-02-07] **Third-party integrations**: Prefer to use API based integrations based on REST or queue based integrations using Kafka. Avoid tight coupling with third-party SDKs or libraries that can create maintenance issues.
- **MANDATORY:** [ARCH-02-08] **Real-time features**: First choice should be long polling for simplicity, then WebSockets for true real-time bidirectional communication, and finally server-sent events (SSE) for unidirectional real-time updates from server to client.
---

### 3. Data Layer

- **MANDATORY:** [ARCH-03-01] **SQL vs NoSQL**: Use open source relational databases (PostgreSQL, MySQL) where reliability and strong consistency guarantees are required. Use NoSQL databases (MongoDB, DynamoDB) when flexible schemas, horizontal scalability or high write throughput are needed.
-  **MANDATORY:** [ARCH-03-02] **Data integrity** Must define data integrity constrains for each component.
-  **MANDATORY:** [ARCH-03-04] **Transactions and recovery** Define transaction boundaries in each components and recovery mechanisms whenever a transaction has to be rolled back. Two point commits are not available. Chained commits, outbox pattern and listen to yourself pattern can be considered. Complicated saga patterns must be avoided.
- **MANDATORY:** [ARCH-03-02] **Database connection pooling**: Use connection pooling
- **MANDATORY:** [ARCH-03-03] **Caching layer**: Add a caching layer (Redis, Memcached) for frequently accessed data or expensive computations when speed is top priority. Use cache-aside pattern for simplicity and control over cache invalidation.
- **MANDATORY:** [ARCH-03-04] **File/blob storage**: Use cloud blob storage (Azure Blob Storage, AWS S3) for large files, unstructured data or binary data that doesnt need elaborate querying capabilities. Use database BLOBs for json when transactional consistency with other data or advanced querying capabilities are required.
- **MANDATORY:** [ARCH-03-05] **Audit**: Add auditing in any enterprise application. Minimum audit includes create and update date and user. Add more advanced versioned change history if required by the domain.

---

### 4. Authentication & Authorisation

- **MANDATORY:** [ARCH-04-01] **Auth mechanism**: Use JWT + Outh2 for user facing UI and APIs. Use API keys for service-to-service authentication. Avoid rolling your own auth mechanism or using basic auth for production applications.
- **MANDATORY:** [ARCH-04-02] **Identity provider**: Prefer cloud based identity providers (Azure AD, AWS Cognito) for ease of integration and management. Use open source identity providers (Keycloak, Ory) when customisation or on-premises hosting is required.
- **MANDATORY:** [ARCH-04-03] **Authorisation model**: Use open source implementations of common authorisation models (RBAC, ABAC) to avoid reinventing the wheel.
---

### 5. API Contract & Versioning

- **MANDATORY:** [ARCH-05-01] **API versioning strategy**: URL prefix (`/v1/`), version attribute in events.
- **MANDATORY:** [ARCH-05-02] **Schema-first**: OpenAPI specification is the source of truth and code must be generated from it.
- **MANDATORY:** [ARCH-05-03] **Contract testing**: Use contract testing tools (Pact, Postman) to ensure API implementations meet the defined contracts and to prevent breaking changes.
- **MANDATORY:** [ARCH-05-04] **Pagination strategy**: Use pagination on all search endpoints and all endpoints whose response can grow unbounded. Use offset-based pagination for simplicity.
---

### 6. Events and Messaging

- **MANDATORY:** [ARCH-06-01] **Event-driven architecture**: Use events to decouple services and enable asynchronous communication. Define clear event schemas and topics.
- **MANDATORY:** [ARCH-06-02] **Message broker**: Use Kafka as the default message broker for event distribution and processing. Alternatives (RabbitMQ, Azure Service Bus) require explicit justification.
- **MANDATORY:** [ARCH-06-03] **Event versioning**: Maintain backward compatibility for events and use versioning when breaking changes are necessary.
- **MANDATORY:** [ARCH-06-04] **Event storage**: Persist events in an event store or database for auditing and replay purposes.
- **MANDATORY:** [ARCH-06-05] **Event processing**: Use idempotent event handlers and implement retry logic to handle transient failures in event processing.
- **MANDATORY:** [ARCH-06-06] **Scaling event processing**: Use consumer groups and partitioning in Kafka to scale event processing horizontally. Describe the expected load and scaling strategy for event processing in the architecture design. Describe scaling limitations and bottlenecks of the chosen event processing approach and how they will be mitigated.
- **MANDATORY:** [ARCH-06-07] **Eventual consistency**: If using an event-driven architecture, eventual consistency has to be detected and pointed out in the design. Event-driven communication has to be avoided for critical user flows and operations that require strong consistency guarantees.
- **MANDATORY:** [ARCH-06-08] **Event structure**: If the architecture uses events, define a standard structure for used events, including metadata (timestamp, event type, source) and payload. Use a consistent format (e.g., JSON, Avro) for event payloads. Payload must be structured for every event type. Generate the structure in a machine-readable format (e.g., JSON Schema, Avro schema) and store it in the `events.md` file to ensure consistency and ease of use across the system. If the architecture does not use events, `events.md` must not be created and this item must be marked `N/A` with rationale.

### 7. Infrastructure & Deployment

- **MANDATORY:** [ARCH-07-01] **Containerisation**: Use Docker when the application needs to be always online, has complex dependencies or needs to run in different environments with minimal changes. Use serverless (Azure Functions, AWS Lambda) for simple, event-driven applications with low to moderate traffic and where low cost is a priority.
- **MANDATORY:** [ARCH-07-02] **Cloud provider**: Separate cloud specific code from business logic to different modules to allow for easier migration if needed. Use cloud provider managed services (Azure App Service, AWS Elastic Beanstalk) for ease of deployment and management when using containerisation. Use serverless offerings (Azure Functions, AWS Lambda) when using serverless architecture.
- **MANDATORY:** [ARCH-07-03] **Deployment strategy**: Use automated deployment pipelines (GitHub Actions, Azure DevOps) to ensure consistent and reliable deployments.
- **MANDATORY:** [ARCH-07-04] **Testing** Run tests on all branches in all environments. Use staging environment that mirrors production for final testing before release.
- **MANDATORY:** [ARCH-07-05] **Environment topology**: Use at least four environments (local, dev, staging, prod).
- **MANDATORY:** [ARCH-07-06] **Secrets management**: `.env` files for local development, Vault, cloud-native secrets manager for production.
- **MANDATORY:** [ARCH-07-07] **Deployment strategy**: Use rolling updates when enough resources are available.
- **MANDATORY:** [ARCH-07-08] **Health check endpoints**: Standard `/health` (liveness) and `/ready` (readiness) endpoints required.
- **MANDATORY:** [ARCH-07-09] **Auto-scaling policy**: Use Http scalers for services that have APIs or event-based scalers for services that process events. Define clear scaling thresholds based on performance goals and expected traffic patterns.

---

### 8. Observability

- **MANDATORY:** [ARCH-08-01] **Logging**: Structured JSON logs. Use cloud-native logging services (Azure Monitor, AWS CloudWatch) or open source solutions (ELK stack) for log aggregation and analysis.
- **MANDATORY:** [ARCH-08-02] **Metrics**: Prometheus + Grafana, or cloud-native monitoring.
- **MANDATORY:** [ARCH-08-03] **Tracing**: Use OpenTelemetry for distributed tracing and integrate with a tracing backend (Jaeger, Zipkin, or cloud-native tracing services).

### 9. AI
- **MANDATORY:** [ARCH-09-01] **Provider independence**: Design AI integration in a provider-agnostic way using OpenLLM.
- **MANDATORY:** [ARCH-09-02] **Tracing and monitoring**: Ensure AI interactions are logged and monitored for performance and cost management using.
- **MANDATORY:** [ARCH-09-03] **Minimal model usage**: Use AI for tasks that provide clear value and cannot be easily solved with traditional programming or existing data science algorithms to manage costs and complexity.
- **MANDATORY:** [ARCH-09-04] **Evaluation and testing**: Implement robust evaluation and testing strategies for AI outputs, including unit tests for deterministic outputs and human-in-the-loop review for non-deterministic outputs.
- **MANDATORY:** [ARCH-09-05] **Human in the loop**: Ensure there is a mechanism for human review in all workflows that use AI and override to prevent harmful or incorrect AI outputs from causing issues in production.
- **MANDATORY:** [ARCH-09-06] **Data privacy**: Ensure that any data sent to AI providers is anonymized and does not contain personally identifiable information (PII) or sensitive data, in compliance with data privacy regulations.
---

### 10. Security Architecture

- **MANDATORY:** [ARCH-10-01] **Transport**: Use TLS everywhere. Use Let's Encrypt if no provider-managed TLS is available.
- **MANDATORY:** [ARCH-10-02] **Input validation**: Validation must be enforced at the API boundary at minimum; define explicitly whether it is also enforced at the domain layer, and document the rationale.
- **MANDATORY:** [ARCH-10-03] **Query parameter sanitisation**: Use ORM parameterised queries to prevent SQL injection; raw SQL queries must be audited.
- **MANDATORY:** [ARCH-10-04] **CORS policy**: Use restrictive CORS policy in all environments, with explicit allowed origins. Avoid wildcard `*` in production.
- **MANDATORY:** [ARCH-10-05] **Dependency scanning**: Automated CVE scanning in CI.
---

### 11. Scalability & Resilience

- **MANDATORY:** [ARCH-11-01] **Horizontal scaling**: Define stateful or thread unsafe applications that cannot be horizontally scaled without additional work. Use stateless design principles to enable horizontal scaling where possible.
- **MANDATORY:** [ARCH-11-02] **Retry & circuit-breaker patterns**: Required for all APIs
- **MANDATORY:** [ARCH-11-03] **Disaster recovery**: Define backup and recovery strategies for databases and critical data. Use cloud provider managed backup solutions when available.
- **MANDATORY:** [ARCH-11-04] **Load testing**: Define load and stress testing requirements, use cases and procedures, including target RPS/TPS and acceptable latency thresholds.
---

### 12. Architecture Decision Records (ADRs)

- **MANDATORY:** [ARCH-12-01] Maintain ADRs in the repository root `adrs/` folder to document all architectural decisions. Each ADR must include decision drivers, constraints, chosen option, rejected alternatives, rationale, consequences, and explicit `N/A` items where relevant.
- **MANDATORY:** [ARCH-12-02] ADRs must be created for decisions such as choice of architecture style, API design conventions, data storage solutions, authentication mechanisms, and deployment strategies.
- **MANDATORY:** [ARCH-12-03] ADRs can not be updated. As the architecture evolves, new ADRs must be created to ensure they remain relevant and accurate and reference previous ADRs that they are superseding. Each ADR has to be placed in a separate file named with a sequential ID and descriptive title (e.g., `0001-architecture-style.md`).
- **MANDATORY:** [ARCH-12-04] ADRs are the authoritative final decision record and must remain self-contained and understandable without requiring `research.md`.
- **MANDATORY:** [ARCH-12-05] If `research.md` contains substantive reasoning for the chosen option, the corresponding ADR must retain that reasoning in final form and must not be thinner in substance.
- **MANDATORY:** [ARCH-12-06] If any plan artifact introduces events, queues, background jobs, or third-party integrations, at least one ADR must explicitly justify them or explicitly mark them out of scope.
- **MANDATORY:** [ARCH-12-07] Each ADR must explain why simpler or more obvious alternatives were rejected, especially when the chosen option increases operational or delivery complexity.
- **MANDATORY:** [ARCH-12-08] ADR consequences must cover applicable impacts on performance, security, compliance, operability, scalability, and delivery workflow.
- **MANDATORY:** [ARCH-12-09] An ADR is invalid if it only summarizes conclusions while omitting the decision substance required for future readers to understand and defend the architectural choice.
---

### 13. Architecture Diagrams
- **MANDATORY:** [ARCH-13-01] Must create high-level system architecture diagram showing major components and their interactions including applications, databases and event queues.
- **MANDATORY:** [ARCH-13-02] Must create workflow diagrams for all defined user flows and operations across all primary actors, showing how different components interact to complete each workflow. Coverage must include every in-scope P1 and P2 user story flow, all lifecycle state transitions, and all externally visible outcomes defined in the specification.
- **MANDATORY:** [ARCH-13-03] Must create deployment architecture diagram showing how the application is deployed in Azure environment.
- **MANDATORY:** [ARCH-13-04] All diagrams must be created using a mermaid format and stored in separate files in the repository root `diagrams/` folder. They must be included in the feature artifacts and updated as needed when the architecture evolves.
- **MANDATORY:** [ARCH-13-05] Diagrams must be kept up to date and reflect the current state of the architecture. They must be reviewed and updated as part of the architectural decision process and during implementation when changes are made.
- **MANDATORY:** [ARCH-13-06] Diagrams must be clear, concise and easy to understand, providing a visual representation of the architecture that complements the written documentation and ADRs.
- **MANDATORY:** [ARCH-13-07] Diagrams must minimise the number of intersections and overlapping lines and objects to ensure readability. They should use consistent symbols and notation to represent different types of components and interactions.
- **MANDATORY:** [ARCH-13-08] Diagrams must clearly indicate the direction of interactions between components using arrows and labels. Synchronous interactions should be represented with solid lines, while asynchronous interactions should be represented with dashed lines.
- **MANDATORY:** [ARCH-13-09] Diagrams must be validated to ensure they are renderable mermaid syntax and can be rendered correctly in the documentation using the same Mermaid configuration, including any registered icon packs.
- **MANDATORY:** [ARCH-13-10] When diagram shows a hierarchy of components or applications, it should be clear how the components are grouped and related to each other. Use subgraphs and clear labels to indicate groupings and relationships.
- **MANDATORY:** [ARCH-13-11] Mermaid icon packs must be registered through a shared Mermaid initialization entrypoint (for example `registerIconPacks`) and reused by all documentation renderers.
- **MANDATORY:** [ARCH-13-12] All external Mermaid icon packs must be version pinned and documented, including package source and selected icon prefix alias.
- **MANDATORY:** [ARCH-13-13] Diagram rendering checks must include at least one iconized architecture diagram and fail when required icon packs are missing or unresolved.
- **MANDATORY:** [ARCH-13-14] For each architecture diagram that uses non-default icons, plan artifacts must document required icon pack names, naming conventions and fallback behavior.
- **MANDATORY:** [ARCH-13-15] Diagrams must be accompanied by an additional document that describes the diagram in detail, explaining  all steps of all workflows that each component serves and the interactions between components using messages, events or direct API calls.

### 14. Architecture as Code
- **MANDATORY:** [ARCH-14-01] Use a formal architecture-as-code approach (ArchUnit equivalent) to define and enforce architectural constraints and module boundaries in code.

### 15. Suggestions for future additions
- **MANDATORY:** [ARCH-15-01] Must create a document in the specification folder named `suggestions.md` to capture suggestions for future added containers, components, technologies, or architectural patterns that are not currently in scope but may be beneficial as the system evolves. Each suggestion should include a brief description, potential benefits, and any relevant context or rationale for future consideration.

## 2. Project-Specific Principles

_Add project-specific overrides or additions to the principles in Section 1. Use the same MANDATORY bullet format with subsection-scoped `[ARCH-SS-II]` IDs. Leave this section empty if there are no project-specific constraints._

---

## 3. Gate Checklists
### 1. Plan Gate Checklist
- `N/A` is forbidden for ADR and diagram checks in this checklist.
- [ ] [ARCH-CHECK-001] What is the chosen architecture style (monolith, microservices, service-based) and why?
- [ ] [ARCH-CHECK-002] What architectural characteristics and constraints are most relevant for each application and service (e.g., modularity, scalability, deployment independence, reliability, high availability)?
- [ ] [ARCH-CHECK-003] What are the applications and services that will make up the system?
- [ ] [ARCH-CHECK-004] Did we define all communication patterns and technologies between applications and services (e.g., REST, gRPC, message queues)?
- [ ] [ARCH-CHECK-005] Did we define the data storage technologies and strategies (e.g., SQL vs NoSQL, caching, file storage)?
- [ ] [ARCH-CHECK-006] What API style(s) will be used (REST, GraphQL, gRPC) and why?
- [ ] [ARCH-CHECK-007] What is the repository structure (monorepo vs polyrepo) and why?
- [ ] [ARCH-CHECK-008] What is the expected scale of the application in terms of users, transactions, and data volume?
- [ ] [ARCH-CHECK-009] Are there any specific regulatory or compliance requirements that need to be considered in the architecture?
- [ ] [ARCH-CHECK-010] What are the expected performance and latency requirements for the application?
- [ ] [ARCH-CHECK-011] Are there any existing systems or technologies that need to be integrated with, and what are their constraints?
- [ ] [ARCH-CHECK-012] Are there any specific security requirements or threat models that need to be addressed in the architecture?
- [ ] [ARCH-CHECK-013] What authentication and authorization mechanisms will be used?
- [ ] [ARCH-CHECK-014] What is the cloud provider and deployment strategy for each application and service?
- [ ] [ARCH-CHECK-015] Does a deployment architecture diagram file exist in `diagrams/`, is it linked from plan artifacts, and does it match the selected cloud provider architecture?
- [ ] [ARCH-CHECK-016] What is the expected timeline for development and how does it influence the choice of architecture style and technologies?
- [ ] [ARCH-CHECK-017] Are there any libraries or frameworks that are preferred or mandated for use in the project?
- [ ] [ARCH-CHECK-018] Where will the application be deployed (e.g., Azure, AWS, on-premises) and how does that influence architectural decisions?
- [ ] [ARCH-CHECK-019] Are all required diagram files created, linked from plan artifacts, and aligned with current architecture decisions?
- [ ] [ARCH-CHECK-020] Are all required diagram files validated as renderable Mermaid with the same configuration and icon packs used by documentation tooling?
- [ ] [ARCH-CHECK-021] If the architecture uses events, is the structure of all used events described in `events.md` and does it reflect the current design of the system? If the architecture does not use events, is this item marked `N/A` with rationale and is `events.md` omitted?
- [ ] [ARCH-CHECK-022] Is the structure of all API contracts described in `contracts/` folder and do they reflect the current design of the system?
- [ ] [ARCH-CHECK-023] Are all database schemas described in `data-model.md` file and do they reflect the current design of the system?
- [ ] [ARCH-CHECK-024] Are required ADR files created in `adrs/`, linked from plan artifacts, and aligned with current architecture decisions?
- [ ] [ARCH-CHECK-025] What is the chosen frontend/backend coupling model (fully decoupled or SSR/hybrid) and why?
- [ ] [ARCH-CHECK-026] Which operations are moved to background processing and what task queue technology was selected?
- [ ] [ARCH-CHECK-027] Are clear module boundaries and single responsibilities defined for all modules?
- [ ] [ARCH-CHECK-028] Are interface contracts defined in the required format (OpenAPI, GraphQL SDL, protobuf, Avro, JSON Schema) and treated as source of truth?
- [ ] [ARCH-CHECK-029] Is architecture-as-code defined for layer, module, and application interaction constraints?
- [ ] [ARCH-CHECK-030] If event-driven communication is used, is Kafka selected by default, and are any alternatives explicitly justified?
- [ ] [ARCH-CHECK-031] Are shared libraries, their ownership, versioning approach, and coupling risks documented?
- [ ] [ARCH-CHECK-032] Are third-party integrations API-based or queue-based, with explicit avoidance of tight SDK coupling?
- [ ] [ARCH-CHECK-033] Are real-time features mapped to long polling, WebSockets, or SSE with rationale?
- [ ] [ARCH-CHECK-034] Is database connection pooling defined for all services that access databases?
- [ ] [ARCH-CHECK-035] Is an audit strategy defined (minimum create/update date and user, plus version history if required)?
- [ ] [ARCH-CHECK-036] Is the identity provider choice defined (cloud-managed or open source) with rationale?
- [ ] [ARCH-CHECK-037] Is the authorization model defined (RBAC/ABAC) using a proven open source implementation?
- [ ] [ARCH-CHECK-038] Is API versioning defined for HTTP APIs and events (e.g., `/v1/` and event version fields)?
- [ ] [ARCH-CHECK-039] Is contract testing tooling and process defined to prevent breaking API changes?
- [ ] [ARCH-CHECK-040] Is pagination required on all unbounded endpoints and is offset-based pagination explicitly addressed?
- [ ] [ARCH-CHECK-041] Are event versioning and backward compatibility rules defined?
- [ ] [ARCH-CHECK-042] Is event storage defined for auditing and replay use cases?
- [ ] [ARCH-CHECK-043] Are idempotency and retry strategies defined for all event handlers?
- [ ] [ARCH-CHECK-044] Is event processing scale-out strategy defined (consumer groups, partitioning, limits, bottlenecks, mitigations)?
- [ ] [ARCH-CHECK-045] Is eventual consistency explicitly identified and avoided for critical strong-consistency user flows?
- [ ] [ARCH-CHECK-046] Is the deployment form factor decision documented (Docker vs serverless) based on workload characteristics?
- [ ] [ARCH-CHECK-047] Is cloud-specific code separated from business logic to preserve migration flexibility?
- [ ] [ARCH-CHECK-048] Are CI/CD pipelines automated and defined for reliable deployments?
- [ ] [ARCH-CHECK-049] Are tests defined to run on all branches and environments, with staging mirroring production?
- [ ] [ARCH-CHECK-050] Is environment topology defined with at least local, dev, staging, and prod?
- [ ] [ARCH-CHECK-051] Is secrets management defined for local and production environments?
- [ ] [ARCH-CHECK-052] Is rolling update strategy defined for eligible services?
- [ ] [ARCH-CHECK-053] Are `/health` and `/ready` endpoints defined for all deployable services?
- [ ] [ARCH-CHECK-054] Are auto-scaling policies and thresholds defined per service type (HTTP/event-based)?
- [ ] [ARCH-CHECK-055] Are structured JSON logging, log aggregation tooling, and retention/search strategy defined?
- [ ] [ARCH-CHECK-056] Are metrics collection and dashboarding choices defined (Prometheus/Grafana or cloud-native)?
- [ ] [ARCH-CHECK-057] Is distributed tracing defined with OpenTelemetry and a tracing backend?
- [ ] [ARCH-CHECK-058] Is AI integration designed for provider independence?
- [ ] [ARCH-CHECK-059] Are AI interactions traced and monitored for performance and cost?
- [ ] [ARCH-CHECK-060] Is AI usage limited to high-value tasks not better solved with deterministic approaches?
- [ ] [ARCH-CHECK-061] Is AI output evaluation/testing strategy defined, including deterministic tests and human review?
- [ ] [ARCH-CHECK-062] Is a human-in-the-loop override mechanism defined for AI-assisted workflows?
- [ ] [ARCH-CHECK-063] Is AI data privacy enforced (PII/sensitive data protection and anonymization)?
- [ ] [ARCH-CHECK-064] Is end-to-end TLS enforced across all transport links?
- [ ] [ARCH-CHECK-065] Is input validation enforced at API boundary and domain layer decision documented?
- [ ] [ARCH-CHECK-066] Is SQL injection protection defined (parameterized ORM queries and raw SQL audit rules)?
- [ ] [ARCH-CHECK-067] Is CORS policy restrictive with explicit origins and no production wildcard?
- [ ] [ARCH-CHECK-068] Is automated dependency/CVE scanning configured in CI?
- [ ] [ARCH-CHECK-069] Are stateful or thread-unsafe components identified with a horizontal scaling strategy?
- [ ] [ARCH-CHECK-070] Are retry and circuit-breaker patterns defined for all APIs?
- [ ] [ARCH-CHECK-071] Is disaster recovery strategy defined for critical data and services?
- [ ] [ARCH-CHECK-072] Are load/stress test procedures, target RPS/TPS, and latency thresholds defined?
- [ ] [ARCH-CHECK-073] Are ADR creation rules defined and enforced in plan phase for all significant architecture decisions?
- [ ] [ARCH-CHECK-074] Is ADR scope complete and represented by actual ADR files for architecture style, API conventions, storage/auth, and deployment/observability?
- [ ] [ARCH-CHECK-075] Is ADR evolution defined through new superseding ADRs (instead of updating existing ADR files) as architecture evolves?
- [ ] [ARCH-CHECK-076] Does the plan include links to generated high-level system, deployment, and workflow diagram files that fully cover all in-scope P1 and P2 user story flows, lifecycle transitions, and externally visible outcomes?
- [ ] [ARCH-CHECK-077] Are required diagrams generated as separate Mermaid files under repository root `diagrams/` and linked from plan artifacts?
- [ ] [ARCH-CHECK-078] Are diagram quality rules defined (clarity, minimal crossings, consistent notation, clear hierarchy/grouping)?
- [ ] [ARCH-CHECK-079] Is formal architecture-as-code enforcement (ArchUnit equivalent) planned in implementation?
- [ ] [ARCH-CHECK-080] Is there a clear mapping from architecture decisions to plan artifacts (ADRs, diagrams, contracts, data models) for traceability?
- [ ] [ARCH-CHECK-081] Are all required API contracts, event schemas, and data models included in the plan artifacts and do they reflect the current design of the system?
- [ ] [ARCH-CHECK-082] Is Mermaid icon pack registration defined in a shared initialization path and used consistently across local preview, CI rendering and published docs?
- [ ] [ARCH-CHECK-083] Are all required Mermaid icon packs version pinned and documented with source and prefix alias choices?
- [ ] [ARCH-CHECK-084] Do render validations include iconized architecture diagrams and fail on unresolved icon references?
- [ ] [ARCH-CHECK-085] Are icon naming conventions and fallback behavior documented for diagrams that use non-default icons?
- [ ] [ARCH-CHECK-086] Does each required architecture diagram have an accompanying document file that explains components and interactions in detail, and is each file linked from plan artifacts?
- [ ] [ARCH-CHECK-087] Check all generated plan artifacts for internal consistency and traceability back to architecture decisions, diagrams, and source principles in this role file.
- [ ] [ARCH-CHECK-088] Is each ADR self-contained without requiring `research.md` to understand the final decision?
- [ ] [ARCH-CHECK-089] Does each ADR include decision drivers, constraints, alternatives, rationale, consequences, and explicit `N/A` items where relevant?
- [ ] [ARCH-CHECK-090] Do ADRs retain all substantive reasoning needed for the chosen option instead of only summarizing conclusions?
- [ ] [ARCH-CHECK-091] Is every event, queue, background process, or integration explicitly justified in an ADR or explicitly marked out of scope?
- [ ] [ARCH-CHECK-092] Does each ADR reference the diagrams, contracts, data models, event definitions, and operational rules it governs?
- [ ] [ARCH-CHECK-093] If `research.md` contains material reasoning absent from ADRs, was that reasoning promoted into the ADRs?
- [ ] [ARCH-CHECK-094] Are ADRs, diagrams, contracts, event definitions, and plan narrative mutually consistent about communication patterns, background processing, integrations, and storage strategy?
- [ ] [ARCH-CHECK-095] Are all architecture decisions, diagrams, contracts, event definitions, and data models traceable back to the project and feature documentation and specification user stories that require them?
- [ ] [ARCH-CHECK-096] Does the `suggestions.md` file exist and is it non-empty? Does it contain meaningful future architectural improvements that are not currently in scope but may be beneficial as the system evolves?

# UnitAIOS — Business Requirements Document (BRD)

| Field | Value |
|-------|-------|
| Project | UnitAIOS |
| Document | Business Requirements Document |
| Version | 0.3 (Draft) |
| Status | Draft for review |
| Date | 2026-06-23 |
| Author | Gevorg Aznauryan |
| Audience | Founders, architects, tool developers, integration partners |

> Scope of this document: business and functional requirements only — **what**
> UnitAIOS must do and **why**. Technical architecture, technology choices, and
> implementation design are intentionally out of scope and will be captured in a
> separate Technical Design Document (PRD/SDD). Where a stakeholder decision
> constrains the solution space, it is recorded as a constraint or requirement,
> not as a design.

> **v0.3 change note.** This revision folds in a second round of stakeholder
> requirements: a built-in scheduler/calendar and recurring workflows; an explicit
> **MCP gateway** that aggregates core and extension-contributed tools; a
> generalized, resource-level **role-based access control (RBAC)** with per-role *and* per-individual
> grants; **human/user authentication** with login/password plus self-service
> scoped API keys issued through a web console; a **vector knowledge base (RAG)**
> with data ingestion, static-vs-per-principal classification, and scheduled
> refresh policies; and a **built-in web server** serving both the public endpoint
> and an authenticated management dashboard. New material is marked with its
> requirement IDs throughout.

---

## 1. Executive summary

UnitAIOS is an abstraction of an operating system, exposed as a network-reachable
service. An individual deployment is called an **instance**. Each instance offers
a controlled set of **tools** (capabilities) that external parties can invoke
through a standard **MCP interface**, presented by a single **MCP gateway** that
unifies the instance's built-in core tools with tools contributed by installed
**extensions**.

The external parties are of three kinds: **peer UnitAIOS instances** (other
operating systems), **AI agents**, and **human users**. They authenticate to an
instance and are granted differentiated access to its **resources** — not only
tools, but also knowledge, scheduled workflows, and other governed assets. A
separate **public, unauthenticated endpoint** lets anyone discover basic facts
about an instance without gaining access to its resources.

Authorization is enforced by **role-based access control (RBAC)** that can grant or
deny any resource to a **role/group** or to an **individual principal**, and that
the operator administers through a management surface. Human users and agents may hold standard
login/password credentials and **scoped access keys (API keys)**, which a user can
obtain self-service through an authenticated **web console**.

Every instance keeps a mandatory **journal** of the requests it receives. It hosts
a **scheduler/calendar** that can trigger actions at a point in time and run
**recurring workflows**. It can be **extended with new tools** that follow defined
authoring rules and interaction protocols. It maintains a local **vector knowledge
base** supporting retrieval-augmented generation (RAG): the instance introspects
itself into **knowledge-atoms**, and operators can **ingest** additional data as
knowledge sources, classify it as static/shared or per-principal, and attach
**refresh policies** that re-ingest it on a schedule. Those atoms and sources back
a built-in **AI concierge** that is an integral part of every UnitAIOS instance.
A **built-in web server** serves the public endpoint and an authenticated
**dashboard** for management and for browsing the instance's data.

## 2. Business context and vision

The industry is moving toward a world where software systems and autonomous AI
agents interact directly, not only humans with applications. UnitAIOS provides a
uniform, governable "operating system" surface for that world: a way to expose a
machine's capabilities to other machines, to agents, and to people, with
authentication, authorization, auditability, scheduling, and self-describing
knowledge built in from the start.

**Vision statement.** Any capability-bearing node — a server, a service, another
agentic system — can present itself as a UnitAIOS instance: discoverable when
appropriate, secured by default, extensible by its operator, schedulable,
knowledge-bearing, and conversationally explainable through its own concierge.

## 3. Goals and objectives

| ID | Objective |
|----|-----------|
| BR-1 | Expose an instance's capabilities as governable tools over a standard MCP interface. |
| BR-2 | Allow peer operating systems, AI agents, and human users to connect, authenticate, and be authorized at differentiated access levels. |
| BR-3 | Allow anonymous discovery of non-sensitive instance information without granting resource access. |
| BR-4 | Guarantee that every request is journaled for audit and accountability. |
| BR-5 | Make an instance extensible with new tools, governed by published authoring rules and interaction protocols, surfaced through a single MCP gateway. |
| BR-6 | Provide a repeatable initialization procedure that stands up a working, secured instance. |
| BR-7 | Let an instance describe itself by generating knowledge-atoms about itself and its tools. |
| BR-8 | Provide a built-in AI concierge that answers questions about the instance using a local knowledge base. |
| BR-9 | Govern access to **any** resource (tools, knowledge, schedules) uniformly, grantable to groups and to individual principals, and administrable by the operator. |
| BR-10 | Let users obtain and manage **scoped access keys** self-service through an authenticated web console, in addition to login/password credentials. |
| BR-11 | Provide a built-in scheduler/calendar that can trigger actions at a time and run recurring workflows. |
| BR-12 | Maintain a local vector knowledge base (RAG) into which arbitrary data can be ingested as knowledge sources, classified, and refreshed on a schedule. |
| BR-13 | Serve, from a built-in web server, both the public endpoint and an authenticated management/data dashboard. |

## 4. Scope

### 4.1 In scope

- Authentication and authorization of connecting parties (peer OS, AI agents, human users).
- The MCP interface for tool invocation, presented through a single MCP gateway.
- Per-principal, per-resource access control (groups and individuals) over all governed resources.
- Self-service issuance and management of scoped access keys via a web console.
- A public unauthenticated information endpoint.
- Mandatory request journaling.
- A tool extension mechanism — including extension-contributed MCP tools — plus the rules/standards and interaction protocols for authoring tools.
- A built-in scheduler/calendar with time-triggered actions and recurring workflows.
- An instance initialization script/procedure.
- Self-introspection and knowledge-atom generation.
- A local vector knowledge base (RAG): ingestion of arbitrary data as knowledge sources, static-vs-per-principal classification, and scheduled refresh policies.
- The AI concierge backed by the local knowledge base.
- A built-in web server serving the public endpoint and an authenticated dashboard for management and data browsing.
- Configurable connection modes (local and network).

### 4.2 Out of scope (this document)

- Concrete technology stack, frameworks, and libraries (technology *preferences* are recorded as constraints, not designs).
- Network/deployment topology, hosting, and scaling design.
- Wire-level schemas, data models, and API contracts.
- Detailed UI/UX, visual design, and interaction flows of the dashboard/console (its *existence and responsibilities* are in scope; its design is not).
- Billing, licensing, and commercial packaging.

## 5. Stakeholders and actors

| Actor | Description | Relationship to instance |
|-------|-------------|--------------------------|
| Peer UnitAIOS instance | Another operating system that interacts with this instance machine-to-machine. | Authenticated party; trusted at the OS-to-OS level (mTLS). |
| AI agent | An autonomous agent that connects to use the instance's tools. | Authenticated party with an assigned access level (token / access key). |
| Human user | A person who authenticates (login/password) and may hold scoped access keys, typically via the web console. | Authenticated party with an assigned access level. |
| Anonymous client | Any unauthenticated caller. | May read public instance information only. |
| Instance operator/administrator | Person or process that initializes, configures, and maintains an instance, its tools, schedules, knowledge, and access policy. | Owner of the instance. |
| Tool developer | Author of new tools (including extension-contributed MCP tools) added to an instance. | Must follow tool authoring rules and protocols. |
| AI concierge | Built-in conversational agent of the instance. | Internal actor; reads the knowledge base; exposed as a tool. |

## 6. Definitions and glossary

- **Instance** — a single running deployment of UnitAIOS.
- **Tool** — a discrete capability the instance exposes for invocation.
- **Resource** — any governed asset RBAC protects: tools, knowledge-atoms/sources, scheduled workflows, and instance information.
- **MCP interface** — the standard interface through which authenticated parties
  discover and invoke tools.
- **MCP gateway** — the single MCP surface that aggregates the instance's built-in
  core tools and the tools contributed by installed extensions, presenting them
  uniformly and applying access control and journaling to all of them.
- **Extension** — an installable unit that adds capability to an instance and may
  contribute its own MCP tools, which become available through the gateway.
- **Principal** — an authenticated identity: a peer instance, an AI agent, or a
  human user.
- **Role / group** — a named authorization unit (the role in role-based access
  control). Resources are granted to roles/groups; a principal gains access by
  belonging to one or more of them.
- **RBAC (role-based access control)** — the authorization layer that decides
  whether a given principal may use a given resource, evaluating grants made to the
  principal's roles/groups and to the principal individually.
- **Access level** — the effective set of permissions a principal has, derived from
  group membership(s) and individual grants.
- **Access key (API key)** — a scoped credential a principal can hold and present to
  use resources; issuable and revocable, including self-service via the web console.
- **Web console / dashboard** — the authenticated web surface for managing the
  instance and browsing its data, served by the built-in web server.
- **Public endpoint** — the unauthenticated surface that returns non-sensitive
  information about the instance.
- **Journal** — the mandatory, append-only record of requests handled by the
  instance.
- **Scheduler / calendar** — the instance facility that triggers actions at a
  specified time and runs recurring workflows.
- **Recurring workflow** — a defined sequence of instance actions configured to run
  repeatedly on a schedule.
- **Knowledge-atom** — a small, self-contained unit of information about the
  instance or one of its tools, generated by introspection and consumed by the
  concierge.
- **Knowledge source** — arbitrary data ingested into the knowledge base to be
  retrievable by the concierge (RAG), beyond introspection-generated atoms.
- **Vector knowledge base (RAG)** — the local store of vectorized knowledge that
  supports retrieval-augmented answering by the concierge.
- **Static (constant) data** — knowledge shared identically across principals.
- **Per-principal data** — knowledge scoped to an individual principal.
- **Refresh policy** — a rule that re-ingests a type of knowledge source on a
  schedule (e.g. daily, weekly).
- **AI concierge** — the built-in agent that answers questions about the instance
  using the knowledge base.
- **Connection mode** — how a party reaches the instance: local (in-process /
  stdio) or network (HTTP).

## 7. Functional requirements

### 7.1 Authentication and authorization (FR-AUTH)

| ID | Requirement |
|----|-------------|
| FR-AUTH-1 | The instance shall authenticate every connecting party before granting any resource access. |
| FR-AUTH-2 | The instance shall support mutual TLS (mTLS) for establishing trust with peer operating systems. |
| FR-AUTH-3 | The instance shall support token / access-key credentials for connecting AI agents. |
| FR-AUTH-4 | The instance shall associate each authenticated principal with an identity and one or more group memberships. |
| FR-AUTH-5 | The instance shall reject or restrict requests that present invalid, expired, or revoked credentials. |
| FR-AUTH-6 | The operator shall be able to issue, assign, and revoke credentials and access levels for principals. |
| FR-AUTH-7 | The instance shall support human-user authentication by login/password. |
| FR-AUTH-8 | The instance shall support **scoped access keys (API keys)** that principals present to access MCP resources; each key carries a defined scope (the resources/access it grants). |
| FR-AUTH-9 | A human user shall be able to obtain and manage their own access keys self-service through the authenticated web console, after logging in with login/password (console-issued keys), analogous to mainstream API-key consoles. |
| FR-AUTH-10 | Access keys shall be individually revocable without disturbing other credentials of the same principal. |

### 7.2 MCP interface and gateway (FR-MCP)

| ID | Requirement |
|----|-------------|
| FR-MCP-1 | The instance shall expose its tools through a standard MCP interface. |
| FR-MCP-2 | An authenticated principal shall be able to discover the tools available to it (subject to its access level). |
| FR-MCP-3 | An authenticated principal shall be able to invoke a tool it is authorized to use and receive its result. |
| FR-MCP-4 | The MCP interface shall present, for each tool, the information a caller needs to invoke it correctly. |
| FR-MCP-5 | A single **MCP gateway** shall present, as one surface, both the instance's built-in core tools and the tools contributed by installed extensions. |
| FR-MCP-6 | The gateway shall apply the same access control and journaling guarantees uniformly to core tools and to extension-contributed tools. |
| FR-MCP-7 | An extension's tools shall become discoverable and invocable through the gateway once the extension is installed and its tools are authorized, without changing the gateway's external contract. |

### 7.3 Resource access control / RBAC (FR-ACL)

| ID | Requirement |
|----|-------------|
| FR-ACL-1 | Authorization shall be enforced by **role-based access control (RBAC)** that governs access to **any resource** (tools, knowledge-atoms/sources, scheduled workflows, instance information). |
| FR-ACL-2 | RBAC shall support grants to **roles/groups**: a resource granted to a role is accessible to members of that role. |
| FR-ACL-3 | RBAC shall additionally support grants to an **individual principal**, independent of role membership, so that a single resource can be disclosed to one specific principal. |
| FR-ACL-4 | The instance shall enforce RBAC rules on every resource access, denying unauthorized requests. |
| FR-ACL-5 | Different principals, by virtue of different group memberships and individual grants, may have different access to different resources on the same instance. |
| FR-ACL-6 | The operator shall be able to define groups, assign principals to groups, grant resources to groups, and grant resources to individual principals. |
| FR-ACL-7 | A resource that a principal's groups and individual grants do not authorize shall not be exposed to that principal as available. |
| FR-ACL-8 | A principal's effective access shall be the union of its group grants and its individual grants. |
| FR-ACL-9 | The instance shall provide a **management surface for RBAC** (administering roles/groups, memberships, and role/individual resource grants). |

### 7.4 Public information endpoint (FR-PUB)

| ID | Requirement |
|----|-------------|
| FR-PUB-1 | The instance shall provide a public endpoint reachable without authentication. |
| FR-PUB-2 | The public endpoint shall return descriptive, non-sensitive information about the instance when requested. |
| FR-PUB-3 | The public endpoint shall not expose tool invocation, sensitive configuration, credentials, journal contents, or non-public knowledge. |
| FR-PUB-4 | The operator shall be able to configure which information is published. |

### 7.5 Request journaling (FR-LOG)

| ID | Requirement |
|----|-------------|
| FR-LOG-1 | The instance shall journal every request it receives. Journaling is mandatory and cannot be disabled. |
| FR-LOG-2 | Each journal entry shall record enough context to audit the request (who, what, when, outcome), without storing sensitive secrets in clear form. |
| FR-LOG-3 | The journal shall be append-only and tamper-evident. |
| FR-LOG-4 | The operator shall be able to review the journal. |
| FR-LOG-5 | Requests to the public endpoint shall also be journaled. |
| FR-LOG-6 | The project shall provide several journal retention/rotation templates (e.g. by size, by age, by entry count) and let the operator choose one at initialization. |
| FR-LOG-7 | Journal rotation may be delegated to standard system tooling (e.g. `logrotate`); journals shall be produced in a form compatible with such tooling. |
| FR-LOG-8 | Scheduled / recurring-workflow executions shall be journaled like any other request, recording the trigger and outcome. |

### 7.6 Tool extensibility (FR-EXT)

| ID | Requirement |
|----|-------------|
| FR-EXT-1 | The instance shall support adding new tools to extend its capabilities. |
| FR-EXT-2 | The project shall define and publish the rules and norms for authoring tools (a tool authoring standard). |
| FR-EXT-3 | A new tool shall declare the metadata required for discovery, access control, and journaling. |
| FR-EXT-4 | Adding or updating a tool shall not require redefining the instance's core security and journaling guarantees — those apply to all tools uniformly. |
| FR-EXT-5 | The project shall define the interaction protocols between the instance and its tools (how tools are invoked, how they return results, how they report errors). |
| FR-EXT-6 | An **extension** shall be able to contribute its own MCP tools, which the instance surfaces through the MCP gateway (FR-MCP-5..7). |
| FR-EXT-7 | Installing, updating, or removing an extension shall not weaken the access-control or journaling guarantees applied to its tools. |

### 7.7 Instance initialization (FR-INIT)

| ID | Requirement |
|----|-------------|
| FR-INIT-1 | The project shall provide an initialization script/procedure that stands up a new instance. |
| FR-INIT-2 | Initialization shall establish the instance's identity and credentials (including mTLS trust material). |
| FR-INIT-3 | Initialization shall let the operator select the connection mode(s): local (stdio), network (HTTP), or both. |
| FR-INIT-4 | Initialization shall produce a runnable instance with journaling active, the public endpoint available, and the web server serving the dashboard. |
| FR-INIT-5 | Initialization shall register any initial set of tools, groups, and access grants provided by the operator. |
| FR-INIT-6 | Initialization shall let the operator choose a journal retention/rotation template. |

### 7.8 Self-introspection (FR-INTRO)

| ID | Requirement |
|----|-------------|
| FR-INTRO-1 | The instance shall be able to collect information about itself (its configuration and identity, excluding secrets). |
| FR-INTRO-2 | The instance shall be able to collect information about each tool it hosts (purpose, inputs/outputs, access level), including extension-contributed tools. |
| FR-INTRO-3 | Introspection shall be repeatable so that the instance's self-knowledge stays current as tools are added or changed. |

### 7.9 Knowledge base and knowledge-atoms (FR-KATOM)

| ID | Requirement |
|----|-------------|
| FR-KATOM-1 | The instance shall generate knowledge-atoms from the information gathered during introspection. |
| FR-KATOM-2 | Knowledge-atoms and ingested knowledge sources shall be stored locally within the instance. |
| FR-KATOM-3 | Each knowledge-atom shall be a small, self-contained unit describing one fact about the instance or a tool. |
| FR-KATOM-4 | Knowledge-atoms shall be regenerated/updated when the instance or its tools change. |
| FR-KATOM-5 | Knowledge-atoms and knowledge sources shall not contain secrets or other non-publishable sensitive data unless protected by RBAC. |
| FR-KATOM-6 | Each knowledge-atom and knowledge source shall carry the access classification needed to filter it consistently with the resource it describes (group and/or individual grants). |
| FR-KATOM-7 | The instance shall maintain a **vector knowledge base** that indexes knowledge-atoms and knowledge sources to support retrieval-augmented generation (RAG) by the concierge. |
| FR-KATOM-8 | The instance shall provide a mechanism to **ingest (push) arbitrary data** as knowledge sources into the knowledge base, beyond introspection-generated atoms. |
| FR-KATOM-9 | Ingested data shall be classifiable as **static/shared** (constant across principals) or **per-principal** (scoped to an individual principal). |
| FR-KATOM-10 | The instance shall support **refresh policies** that re-ingest specified types of knowledge sources on a schedule (e.g. daily, weekly), driven by the scheduler (FR-SCHED). |
| FR-KATOM-11 | Indexing of ingested data shall be repeatable so that retrieval reflects the current contents of each knowledge source after refresh. |

### 7.10 AI concierge (FR-CONC)

| ID | Requirement |
|----|-------------|
| FR-CONC-1 | Every instance shall include an AI concierge as an integral component. |
| FR-CONC-2 | The concierge shall use the instance's local knowledge base (atoms and ingested sources, via RAG) as its information base. |
| FR-CONC-3 | The concierge shall be exposed as a tool through the MCP interface. |
| FR-CONC-4 | The concierge shall answer questions about the instance and its available tools, respecting the asking principal's access level. |
| FR-CONC-5 | Knowledge shall be filtered by the same RBAC rules that govern resources; the concierge shall never reveal knowledge the asking principal's roles or individual grants do not authorize. |

### 7.11 Scheduler and recurring workflows (FR-SCHED)

| ID | Requirement |
|----|-------------|
| FR-SCHED-1 | The instance shall provide a built-in scheduler/calendar. |
| FR-SCHED-2 | The operator shall be able to schedule an action to be triggered at a specified point in time (a calendar event that fires an instance action). |
| FR-SCHED-3 | The operator shall be able to define **recurring workflows** that run repeatedly on a schedule. |
| FR-SCHED-4 | A recurring workflow shall be a defined sequence of instance actions (e.g. invoking tools, refreshing knowledge sources). |
| FR-SCHED-5 | Scheduled and recurring executions shall be subject to RBAC and shall be journaled (FR-LOG-8). |
| FR-SCHED-6 | The operator shall be able to create, view, modify, disable, and remove scheduled events and recurring workflows. |

### 7.12 Connection modes (FR-CONN)

| ID | Requirement |
|----|-------------|
| FR-CONN-1 | The instance shall support a local connection mode (in-process / stdio). |
| FR-CONN-2 | The instance shall support a network connection mode (HTTP), through which the MCP interface, the public endpoint, and the web dashboard are reachable. |
| FR-CONN-3 | The connection mode(s) shall be selectable at initialization and configurable thereafter. |

### 7.13 Web server and dashboard (FR-WEB)

| ID | Requirement |
|----|-------------|
| FR-WEB-1 | The instance shall include a built-in web server. |
| FR-WEB-2 | The web server shall serve the public, unauthenticated endpoint (FR-PUB). |
| FR-WEB-3 | The web server shall serve an authenticated **dashboard** available only after a principal logs in. |
| FR-WEB-4 | The dashboard shall provide management surfaces for the instance, including the RBAC management surface (FR-ACL-9), access-key management (FR-AUTH-9), and scheduler/workflow management (FR-SCHED-6). |
| FR-WEB-5 | The dashboard shall provide an interface to browse the instance's data (e.g. knowledge sources/atoms), filtered by the viewing principal's access level. |
| FR-WEB-6 | The dashboard shall never expose data or actions the authenticated principal's access level does not authorize. |

## 8. Key use cases

- **UC-1 — Agent uses a tool.** An AI agent authenticates with an access key,
  discovers the tools available at its access level, invokes one, and receives the
  result. The request is journaled.
- **UC-2 — OS-to-OS interaction.** A peer UnitAIOS instance establishes mTLS trust
  and invokes tools it is authorized for. All exchanges are journaled.
- **UC-3 — Anonymous discovery.** An unauthenticated client queries the public
  endpoint and learns non-sensitive facts about the instance; it gains no resource
  access. The query is journaled.
- **UC-4 — Ask the concierge.** A connected principal asks the concierge a question
  about the instance; the concierge answers from the local knowledge base (RAG),
  filtered by the principal's access level.
- **UC-5 — Extend the instance.** A tool developer authors a new tool (or an
  extension contributing MCP tools) per the authoring rules and protocols; the
  operator installs it and assigns access; the gateway surfaces it; introspection
  regenerates knowledge-atoms so the concierge knows about it.
- **UC-6 — Stand up an instance.** An operator runs the initialization procedure,
  chooses connection mode(s), and obtains a secured, journaling instance with a
  live public endpoint and dashboard.
- **UC-7 — Self-service access key.** A human user logs into the web console with
  login/password and issues a scoped access key for themselves (or an agent),
  which is then usable against MCP resources within its scope.
- **UC-8 — Grant a resource to one user.** An operator grants a specific resource
  to a single individual principal (not a whole role) via the RBAC
  management surface; only that principal then sees and can use it.
- **UC-9 — Recurring workflow.** An operator defines a recurring workflow (e.g. a
  nightly task) on the scheduler; it runs on its schedule, subject to the access
  guard, and each run is journaled.
- **UC-10 — Ingest and refresh knowledge.** An operator pushes external data into
  the knowledge base as a knowledge source, classifies it (static or
  per-principal), and attaches a daily refresh policy; the scheduler re-ingests it
  and the concierge answers from the refreshed content.

## 9. Non-functional requirements

| ID | Requirement |
|----|-------------|
| NFR-SEC-1 | Security is default-on: no resource access without authentication and authorization. |
| NFR-SEC-2 | Secrets and credentials are never exposed via the public endpoint, the concierge, the knowledge base, or the dashboard. |
| NFR-SEC-3 | Access keys are scoped, individually revocable, and stored/handled so that a leaked key cannot exceed its scope. |
| NFR-AUD-1 | Auditability: the journal is complete (every request, including scheduled runs) and tamper-evident. |
| NFR-EXT-1 | Extensibility: new tools and extensions can be added without modifying core security/journaling behavior. |
| NFR-COMPAT-1 | Interoperability: the MCP interface follows the standard so conformant agents and instances can connect. |
| NFR-PORT-1 | Portability: an instance can run in local or network mode from the same initialization procedure. |
| NFR-CONS-1 | Consistency: the concierge's answers reflect the instance's current tools, configuration, and ingested knowledge (atoms and sources stay current). |
| NFR-REL-1 | Reliability: a failing or unauthorized resource access is rejected cleanly and journaled, without compromising the instance. |
| NFR-REL-2 | Scheduler reliability: scheduled and recurring executions fire dependably and a missed or failed run is journaled and surfaced. |

## 10. Assumptions and constraints

- **Constraint — Auth.** Trust between operating systems uses mTLS; AI agents and
  human users authenticate with tokens/access keys, and human users additionally
  with login/password. (Stakeholder decision.)
- **Constraint — Access control.** Authorization is governed by role-based access
  control (RBAC) over all resources; grants may target roles/groups *and*
  individual principals; effective access is the union of role and individual
  grants. (Stakeholder
  decision.)
- **Constraint — Access keys.** Keys are scoped and obtainable self-service through
  the authenticated web console, in the style of mainstream API-key consoles.
  (Stakeholder decision.)
- **Constraint — Concierge filtering.** Knowledge is filtered by the same RBAC
  rules as other resources. (Stakeholder decision.)
- **Constraint — Knowledge base / RAG.** The concierge is backed by a local vector
  knowledge base supporting RAG; arbitrary data can be ingested as knowledge
  sources, classified static or per-principal, and refreshed on schedule.
  (Stakeholder decision.)
- **Constraint — Journal retention.** Several retention/rotation templates are
  offered at initialization; rotation may rely on standard system tooling such as
  `logrotate`. (Stakeholder decision.)
- **Constraint — Transport.** Both local (stdio) and network (HTTP) connection
  modes are supported and configurable per instance. (Stakeholder decision.)
- **Constraint — Concierge.** The concierge is delivered as an MCP tool, and its
  knowledge is stored locally within the instance. (Stakeholder decision.)
- **Constraint — Interface.** Tool exposure is via the standard MCP interface,
  presented through a single MCP gateway aggregating core and extension tools.
- **Constraint — Web surface.** A built-in web server serves the public endpoint
  and an authenticated dashboard. (Stakeholder decision.)
- **Constraint — Vector store.** The knowledge base is backed by a **vector store**
  (referred to generically by that term throughout this BRD). The specific vector
  store product is a technology choice for the Technical Design Document, which must
  still satisfy FR-KATOM-7..11.
- **Assumption.** Connecting parties are themselves capable of MCP and (for OS
  peers) of mTLS.
- **Assumption.** Operators are responsible for the sensitivity classification of
  what the public endpoint, concierge, and dashboard may reveal.

## 11. Success metrics / acceptance criteria

- An operator can initialize an instance and select connection mode(s) (FR-INIT).
- An authenticated agent, peer OS, and human user can each access only the
  resources permitted by their access level (FR-AUTH, FR-ACL).
- An operator can grant a resource to a single individual principal and verify only
  that principal can use it (FR-ACL-3).
- A human user can log into the console and issue a scoped access key that works
  within its scope and fails outside it (FR-AUTH-8/9).
- An anonymous client can read public instance info and nothing more (FR-PUB).
- 100% of requests, including public, scheduled, and rejected ones, appear in the
  journal (FR-LOG).
- A new tool or extension authored to the published rules can be installed and
  invoked through the gateway, and becomes visible to the concierge after
  introspection (FR-EXT, FR-MCP, FR-INTRO, FR-KATOM, FR-CONC).
- An operator can ingest external data as a knowledge source, classify it, attach a
  refresh policy, and confirm the concierge answers from the refreshed content
  (FR-KATOM-8..11, FR-SCHED).
- An operator can define a recurring workflow, observe it run on schedule, and find
  each run journaled (FR-SCHED, FR-LOG-8).
- The concierge answers an instance/tool question correctly from the local
  knowledge base, filtered by access level (FR-CONC).

## 12. Risks and open questions

| # | Risk / open question |
|---|----------------------|
| R-1 | *Resolved:* access control is RBAC supporting role and individual grants (FR-ACL). Residual design detail — role hierarchy/inheritance, conflict resolution between role and individual grants, and per-tool vs. per-operation grant granularity. |
| R-2 | *Addressed* by FR-CONC-5 / FR-KATOM-6: knowledge carries access classification and is filtered identically to other resources. Design must keep classification in sync with the resource it describes. |
| R-3 | Tool authoring standard scope — how strict the rules are vs. developer freedom (balance to be set), now including extension-contributed MCP tools. |
| R-4 | *Addressed* by FR-LOG-6/7 and FR-INIT-6: selectable retention templates at init; rotation via system tooling. Residual — the concrete default template set and storage limits. |
| R-5 | Trust bootstrapping between peer instances (certificate/identity provisioning) at scale. |
| R-6 | Knowledge-atom/source freshness vs. cost of repeated introspection and scheduled re-ingestion; refresh-policy defaults and back-pressure under heavy ingestion. |
| R-7 | Access-key lifecycle — scoping model, rotation, leak detection, and revocation propagation across connection modes (FR-AUTH-8..10). |
| R-8 | Per-principal knowledge isolation — guaranteeing per-principal data never leaks across principals through shared vector indexes (FR-KATOM-9). |
| R-9 | Scheduler reliability and semantics — missed-run handling, overlapping runs, time-zone/clock assumptions, and failure surfacing (FR-SCHED, NFR-REL-2). |
| R-10 | Dashboard attack surface — the authenticated web console expands the network surface; session, CSRF, and key-handling concerns to be addressed in the TDD (FR-WEB). |
| R-11 | RBAC granularity at scale — managing many individual-principal grants alongside roles risks policy sprawl; the TDD should define how individual grants are audited and kept manageable. |

## 13. Next steps

1. Review and approve this BRD (v0.3), confirming the v0.3 interpretation calls
   (RBAC as the access-control model; a generic vector store backs the knowledge
   base; dashboard existence moved in scope).
2. Produce the Technical Design Document (architecture, MCP gateway contracts, auth
   and access-key mechanics, RBAC model, journal format, knowledge-base /
   RAG and ingestion/refresh design, scheduler design, web server/dashboard, init
   script).
3. Draft the **Tool Authoring Standard** and **Tool Interaction Protocol** as
   separate documents referenced by FR-EXT, covering extension-contributed tools.
4. Prototype the initialization procedure for a minimal instance.

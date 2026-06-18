# UnitAIOS — Business Requirements Document (BRD)

| Field | Value |
|-------|-------|
| Project | UnitAIOS |
| Document | Business Requirements Document |
| Version | 0.2 (Draft) |
| Status | Draft for review |
| Date | 2026-06-18 |
| Author | Gevorg Aznauryan |
| Audience | Founders, architects, tool developers, integration partners |

> Scope of this document: business and functional requirements only — **what**
> UnitAIOS must do and **why**. Technical architecture, technology choices, and
> implementation design are intentionally out of scope and will be captured in a
> separate Technical Design Document (PRD/SDD). Where a stakeholder decision
> constrains the solution space, it is recorded as a constraint or requirement,
> not as a design.

---

## 1. Executive summary

UnitAIOS is an abstraction of an operating system, exposed as a network-reachable
service. An individual deployment is called an **instance**. Each instance offers
a controlled set of **tools** (capabilities) that external parties can invoke
through a standard **MCP interface**.

The external parties are of two kinds: **peer UnitAIOS instances** (other
operating systems) and **AI agents**. Both authenticate to an instance and are
granted differentiated levels of access to the instance's tools. A separate
**public, unauthenticated endpoint** lets anyone discover basic facts about an
instance without gaining access to its tools.

Every instance keeps a mandatory **journal** of the requests it receives. It can
be **extended with new tools** that follow defined authoring rules and interaction
protocols. An instance can **introspect itself** — collecting information about the
instance and the tools it hosts — and distil that information into
**knowledge-atoms**. Those atoms form the knowledge base for a built-in
**AI concierge** that is an integral part of every UnitAIOS instance.

## 2. Business context and vision

The industry is moving toward a world where software systems and autonomous AI
agents interact directly, not only humans with applications. UnitAIOS provides a
uniform, governable "operating system" surface for that world: a way to expose a
machine's capabilities to other machines and to agents, with authentication,
authorization, auditability, and self-describing knowledge built in from the
start.

**Vision statement.** Any capability-bearing node — a server, a service, another
agentic system — can present itself as a UnitAIOS instance: discoverable when
appropriate, secured by default, extensible by its operator, and conversationally
explainable through its own concierge.

## 3. Goals and objectives

| ID | Objective |
|----|-----------|
| BR-1 | Expose an instance's capabilities as governable tools over a standard MCP interface. |
| BR-2 | Allow both peer operating systems and AI agents to connect, authenticate, and be authorized at differentiated access levels. |
| BR-3 | Allow anonymous discovery of non-sensitive instance information without granting tool access. |
| BR-4 | Guarantee that every request is journaled for audit and accountability. |
| BR-5 | Make an instance extensible with new tools, governed by published authoring rules and interaction protocols. |
| BR-6 | Provide a repeatable initialization procedure that stands up a working, secured instance. |
| BR-7 | Let an instance describe itself by generating knowledge-atoms about itself and its tools. |
| BR-8 | Provide a built-in AI concierge that answers questions about the instance using those knowledge-atoms. |

## 4. Scope

### 4.1 In scope

- Authentication and authorization of connecting parties.
- The MCP interface for tool invocation.
- Per-party, per-tool access control (access levels).
- A public unauthenticated information endpoint.
- Mandatory request journaling.
- A tool extension mechanism, plus the rules/standards and interaction protocols
  for authoring tools.
- An instance initialization script/procedure.
- Self-introspection and knowledge-atom generation.
- The AI concierge backed by local knowledge-atoms.
- Configurable connection modes (local and network).

### 4.2 Out of scope (this document)

- Concrete technology stack, frameworks, and libraries.
- Network/deployment topology, hosting, and scaling design.
- Wire-level schemas, data models, and API contracts.
- UI/UX of any management console.
- Billing, licensing, and commercial packaging.

## 5. Stakeholders and actors

| Actor | Description | Relationship to instance |
|-------|-------------|--------------------------|
| Peer UnitAIOS instance | Another operating system that interacts with this instance machine-to-machine. | Authenticated party; trusted at the OS-to-OS level. |
| AI agent | An autonomous agent that connects to use the instance's tools. | Authenticated party with an assigned access level. |
| Anonymous client | Any unauthenticated caller. | May read public instance information only. |
| Instance operator/administrator | Person or process that initializes, configures, and maintains an instance and its tools and access policy. | Owner of the instance. |
| Tool developer | Author of new tools added to an instance. | Must follow tool authoring rules and protocols. |
| AI concierge | Built-in conversational agent of the instance. | Internal actor; reads knowledge-atoms; exposed as a tool. |

## 6. Definitions and glossary

- **Instance** — a single running deployment of UnitAIOS.
- **Tool** — a discrete capability the instance exposes for invocation.
- **MCP interface** — the standard interface through which authenticated parties
  discover and invoke tools.
- **Group** — a named authorization unit. Tools and knowledge-atoms are granted to
  groups; a party gains access by belonging to one or more groups.
- **Access level** — the effective set of permissions a party has, derived from its
  group membership(s). Access control in UnitAIOS is group-based.
- **Public endpoint** — the unauthenticated surface that returns non-sensitive
  information about the instance.
- **Journal** — the mandatory, append-only record of requests handled by the
  instance.
- **Knowledge-atom** — a small, self-contained unit of information about the
  instance or one of its tools, generated by introspection and consumed by the
  concierge.
- **AI concierge** — the built-in agent that answers questions about the instance
  using knowledge-atoms.
- **Connection mode** — how a party reaches the instance: local (in-process /
  stdio) or network (HTTP).

## 7. Functional requirements

### 7.1 Authentication and authorization (FR-AUTH)

| ID | Requirement |
|----|-------------|
| FR-AUTH-1 | The instance shall authenticate every connecting party before granting any tool access. |
| FR-AUTH-2 | The instance shall support mutual TLS (mTLS) for establishing trust with peer operating systems. |
| FR-AUTH-3 | The instance shall support token-based credentials for connecting AI agents. |
| FR-AUTH-4 | The instance shall associate each authenticated party with an identity and one or more group memberships. |
| FR-AUTH-5 | The instance shall reject or restrict requests that present invalid, expired, or revoked credentials. |
| FR-AUTH-6 | The operator shall be able to issue, assign, and revoke credentials and access levels for parties. |

### 7.2 MCP interface (FR-MCP)

| ID | Requirement |
|----|-------------|
| FR-MCP-1 | The instance shall expose its tools through a standard MCP interface. |
| FR-MCP-2 | An authenticated party shall be able to discover the tools available to it (subject to its access level). |
| FR-MCP-3 | An authenticated party shall be able to invoke a tool it is authorized to use and receive its result. |
| FR-MCP-4 | The MCP interface shall present, for each tool, the information a caller needs to invoke it correctly. |

### 7.3 Tool access control (FR-ACL)

| ID | Requirement |
|----|-------------|
| FR-ACL-1 | Access control shall be **group-based**: every party belongs to one or more groups, and authorization is determined by group membership. |
| FR-ACL-2 | Each tool shall be granted to one or more groups; only members of a granted group may use the tool. |
| FR-ACL-3 | The instance shall enforce group-based access rules on every tool invocation, denying unauthorized calls. |
| FR-ACL-4 | Different parties, by virtue of different group memberships, may have different access to different tools on the same instance. |
| FR-ACL-5 | The operator shall be able to define groups, assign parties to groups, and map groups to tools. |
| FR-ACL-6 | A tool that a party's groups do not grant shall not be exposed to that party as available. |
| FR-ACL-7 | When a party belongs to multiple groups, its effective access is the union of those groups' grants. |

### 7.4 Public information endpoint (FR-PUB)

| ID | Requirement |
|----|-------------|
| FR-PUB-1 | The instance shall provide a public endpoint reachable without authentication. |
| FR-PUB-2 | The public endpoint shall return descriptive, non-sensitive information about the instance when requested. |
| FR-PUB-3 | The public endpoint shall not expose tool invocation, sensitive configuration, credentials, or journal contents. |
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

### 7.6 Tool extensibility (FR-EXT)

| ID | Requirement |
|----|-------------|
| FR-EXT-1 | The instance shall support adding new tools to extend its capabilities. |
| FR-EXT-2 | The project shall define and publish the rules and norms for authoring tools (a tool authoring standard). |
| FR-EXT-3 | A new tool shall declare the metadata required for discovery, access control, and journaling. |
| FR-EXT-4 | Adding or updating a tool shall not require redefining the instance's core security and journaling guarantees — those apply to all tools uniformly. |
| FR-EXT-5 | The project shall define the interaction protocols between the instance and its tools (how tools are invoked, how they return results, how they report errors). |

### 7.7 Instance initialization (FR-INIT)

| ID | Requirement |
|----|-------------|
| FR-INIT-1 | The project shall provide an initialization script/procedure that stands up a new instance. |
| FR-INIT-2 | Initialization shall establish the instance's identity and credentials (including mTLS trust material). |
| FR-INIT-3 | Initialization shall let the operator select the connection mode(s): local (stdio), network (HTTP), or both. |
| FR-INIT-4 | Initialization shall produce a runnable instance with journaling active and the public endpoint available. |
| FR-INIT-5 | Initialization shall register any initial set of tools and groups provided by the operator. |
| FR-INIT-6 | Initialization shall let the operator choose a journal retention/rotation template. |

### 7.8 Self-introspection (FR-INTRO)

| ID | Requirement |
|----|-------------|
| FR-INTRO-1 | The instance shall be able to collect information about itself (its configuration and identity, excluding secrets). |
| FR-INTRO-2 | The instance shall be able to collect information about each tool it hosts (purpose, inputs/outputs, access level). |
| FR-INTRO-3 | Introspection shall be repeatable so that the instance's self-knowledge stays current as tools are added or changed. |

### 7.9 Knowledge-atoms (FR-KATOM)

| ID | Requirement |
|----|-------------|
| FR-KATOM-1 | The instance shall generate knowledge-atoms from the information gathered during introspection. |
| FR-KATOM-2 | Knowledge-atoms shall be stored locally within the instance. |
| FR-KATOM-3 | Each knowledge-atom shall be a small, self-contained unit describing one fact about the instance or a tool. |
| FR-KATOM-4 | Knowledge-atoms shall be regenerated/updated when the instance or its tools change. |
| FR-KATOM-5 | Knowledge-atoms shall not contain secrets or other non-publishable sensitive data. |
| FR-KATOM-6 | Each knowledge-atom shall carry the group classification needed to filter it consistently with the tool or instance information it describes. |

### 7.10 AI concierge (FR-CONC)

| ID | Requirement |
|----|-------------|
| FR-CONC-1 | Every instance shall include an AI concierge as an integral component. |
| FR-CONC-2 | The concierge shall use the instance's local knowledge-atoms as its information base. |
| FR-CONC-3 | The concierge shall be exposed as a tool through the MCP interface. |
| FR-CONC-4 | The concierge shall answer questions about the instance and its available tools, respecting the asking party's group memberships. |
| FR-CONC-5 | Knowledge-atoms shall be filtered by the same group-based access rules that govern tools; the concierge shall never reveal an atom the asking party's groups do not grant. |

### 7.11 Connection modes (FR-CONN)

| ID | Requirement |
|----|-------------|
| FR-CONN-1 | The instance shall support a local connection mode (in-process / stdio). |
| FR-CONN-2 | The instance shall support a network connection mode (HTTP), through which the MCP interface and the public endpoint are reachable. |
| FR-CONN-3 | The connection mode(s) shall be selectable at initialization and configurable thereafter. |

## 8. Key use cases

- **UC-1 — Agent uses a tool.** An AI agent authenticates with a token, discovers
  the tools available at its access level, invokes one, and receives the result.
  The request is journaled.
- **UC-2 — OS-to-OS interaction.** A peer UnitAIOS instance establishes mTLS trust
  and invokes tools it is authorized for. All exchanges are journaled.
- **UC-3 — Anonymous discovery.** An unauthenticated client queries the public
  endpoint and learns non-sensitive facts about the instance; it gains no tool
  access. The query is journaled.
- **UC-4 — Ask the concierge.** A connected party asks the concierge a question
  about the instance; the concierge answers from local knowledge-atoms, filtered
  by the party's access level.
- **UC-5 — Extend the instance.** A tool developer authors a new tool per the
  authoring rules and protocols; the operator registers it and assigns access
  levels; introspection regenerates knowledge-atoms so the concierge knows about
  it.
- **UC-6 — Stand up an instance.** An operator runs the initialization procedure,
  chooses connection mode(s), and obtains a secured, journaling instance with a
  live public endpoint.

## 9. Non-functional requirements

| ID | Requirement |
|----|-------------|
| NFR-SEC-1 | Security is default-on: no tool access without authentication and authorization. |
| NFR-SEC-2 | Secrets and credentials are never exposed via the public endpoint, the concierge, or knowledge-atoms. |
| NFR-AUD-1 | Auditability: the journal is complete (every request) and tamper-evident. |
| NFR-EXT-1 | Extensibility: new tools can be added without modifying core security/journaling behavior. |
| NFR-COMPAT-1 | Interoperability: the MCP interface follows the standard so conformant agents and instances can connect. |
| NFR-PORT-1 | Portability: an instance can run in local or network mode from the same initialization procedure. |
| NFR-CONS-1 | Consistency: the concierge's answers reflect the instance's current tools and configuration (atoms stay current). |
| NFR-REL-1 | Reliability: a failing or unauthorized tool call is rejected cleanly and journaled, without compromising the instance. |

## 10. Assumptions and constraints

- **Constraint — Auth.** Trust between operating systems uses mTLS; AI agents
  authenticate with tokens. (Stakeholder decision.)
- **Constraint — Access control.** Authorization is group-based: parties belong to
  groups; tools and knowledge-atoms are granted to groups; multiple memberships
  combine as a union. (Stakeholder decision.)
- **Constraint — Concierge filtering.** Knowledge-atoms are filtered by the same
  group-based access rules as tools. (Stakeholder decision.)
- **Constraint — Journal retention.** Several retention/rotation templates are
  offered at initialization; rotation may rely on standard system tooling such as
  `logrotate`. (Stakeholder decision.)
- **Constraint — Transport.** Both local (stdio) and network (HTTP) connection
  modes are supported and configurable per instance. (Stakeholder decision.)
- **Constraint — Concierge.** The concierge is delivered as an MCP tool, and its
  knowledge-atoms are stored locally within the instance. (Stakeholder decision.)
- **Constraint — Interface.** Tool exposure is via the standard MCP interface.
- **Assumption.** Connecting parties are themselves capable of MCP and (for OS
  peers) of mTLS.
- **Assumption.** Operators are responsible for the sensitivity classification of
  what the public endpoint and concierge may reveal.

## 11. Success metrics / acceptance criteria

- An operator can initialize an instance and select connection mode(s) (FR-INIT).
- An authenticated agent and an authenticated peer OS can each invoke only the
  tools permitted by their access level (FR-AUTH, FR-ACL).
- An anonymous client can read public instance info and nothing more (FR-PUB).
- 100% of requests, including public and rejected ones, appear in the journal
  (FR-LOG).
- A new tool authored to the published rules can be registered and invoked, and
  becomes visible to the concierge after introspection (FR-EXT, FR-INTRO,
  FR-KATOM, FR-CONC).
- The concierge answers an instance/tool question correctly from local atoms,
  filtered by access level (FR-CONC).

## 12. Risks and open questions

| # | Risk / open question |
|---|----------------------|
| R-1 | *Resolved:* access control is group-based (FR-ACL). Residual design detail — group nesting/inheritance, and exact per-tool vs. per-operation grant granularity within a group. |
| R-2 | *Addressed* by FR-CONC-5 / FR-KATOM-6: atoms carry group classification and are filtered identically to tools. Design must keep an atom's classification in sync with the tool/info it describes. |
| R-3 | Tool authoring standard scope — how strict the rules are vs. developer freedom (balance to be set). |
| R-4 | *Addressed* by FR-LOG-6/7 and FR-INIT-6: selectable retention templates at init; rotation via system tooling. Residual — the concrete default template set and storage limits. |
| R-5 | Trust bootstrapping between peer instances (certificate/identity provisioning) at scale. |
| R-6 | Knowledge-atom freshness vs. cost of repeated introspection. |

## 13. Next steps

1. Review and approve this BRD.
2. Produce the Technical Design Document (architecture, MCP contracts, auth
   mechanics, journal format, atom schema, init script).
3. Draft the **Tool Authoring Standard** and **Tool Interaction Protocol** as
   separate documents referenced by FR-EXT.
4. Prototype the initialization procedure for a minimal instance.

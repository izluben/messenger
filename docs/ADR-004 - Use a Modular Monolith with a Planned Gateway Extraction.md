# ADR-004: Use a Modular Monolith with a Planned Gateway Extraction

## Status

Proposed

## Date

2026-08-04

## Context

We must choose between a monolithic and a distributed deployment architecture for the backend.

The system must handle production-like load and be designed to scale to millions of users. In a messenger, load concentrates in two places: many long-lived client connections and message fan-out.

We compared three options:

**Modular monolith.** It is one deployable unit. Inside the code, modules (WebSocket handling, chat logic, and so on) are separated by strict interfaces. All calls between modules are in-process.

**Monolith plus a separate WebSocket gateway.** It is two deployable units: a thin gateway that only holds client connections, and an application service with all business logic. This is the standard first split for messengers at very large scale. Its benefits are that a deploy of business logic does not disconnect clients, and that connection nodes can scale separately from processing nodes. Its costs are a second deployable, a second pipeline, an internal API between the gateway and the application service, and one extra network hop for client events going to the backend.

**Microservices.** It splits the backend into many services (chat, presence, notifications, and so on). It gives the highest operational cost and the weakest performance (network hops everywhere). Its benefits are independent scaling and independent deploys per service. In our system, the only part with a clearly different scaling profile is connection handling, and the gateway option already covers it with one split. Also, good service boundaries are hard to draw before the domain is stable, and a wrong early split is expensive to undo.

## Considered options

* Modular monolith
* Monolith plus a separate WebSocket gateway
* Microservices

## Decision outcome

We will build a modular monolith.

So that we can still extract the gateway later, we add two rules:

* **Module boundaries are enforced by the build, not only by agreement.** The WebSocket module talks to the rest of the application only through defined interfaces. We enforce this with separate build modules and architecture tests (for example, ArchUnit).
* **The gateway extraction is pre-planned.** If connection counts or deploy problems ever require it, the WebSocket module becomes the separate gateway. Its interfaces are designed so that every call across the boundary can become a network call later.

### Consequences

#### Positive

* There is one codebase, one pipeline, and one thing to deploy, observe, and debug.
* Performance is the best of the three options: all calls between modules are in-process, without network hops.
* Testing is simple: integration tests run the whole system in one process.
* There are no internal APIs to version, secure, and keep compatible during deploys.

#### Negative

* Fault isolation is weak: a fatal bug in any module can crash a node together with all WebSocket connections it holds. (Clients can recover missed events afterwards, per ADR-002.)
* Scaling is coarse: we cannot scale connection handling separately from message processing; we must add whole nodes.
* Every deploy restarts nodes and disconnects their clients. (Rolling deploys reduce the impact.)
* There is one technology stack for the whole application. One runtime must serve both the REST API and many long-lived WebSocket connections, so the stack choice is a compromise between two very different workloads.
* Module discipline needs constant enforcement, or the modular monolith becomes a big ball of mud.

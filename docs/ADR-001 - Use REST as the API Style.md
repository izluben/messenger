# ADR-001: Use REST as the API Style

## Status

Proposed

## Date

2026-06-11

## Context

The Messenger application is a web client talking to a backend, so we need an API between them. We need to choose the API style.

The API covers operations on users, chats, and messages, for example creating a chat, sending a message, and loading message history. Most of them are simple CRUD operations.

How new messages and other events reach clients in real time is not decided in this ADR.

## Considered options

* **REST.** Simple, well known, and a natural fit for CRUD operations. Standard HTTP tooling works out of the box.
* **GraphQL.** Useful for complex queries, but we do not need them now. It also needs more setup and adds extra complexity.
* **gRPC.** The application is web-only, and browsers cannot use native gRPC. It would need an extra proxy layer (gRPC-Web). This adds complexity and gives no clear benefit for simple CRUD operations.

## Decision outcome

We will use REST as the API style, because most operations are simple CRUD, and REST is the simplest option that fits them.

### Consequences

#### Positive

* The API will be simple to implement and test with standard HTTP tooling.

#### Negative

* Unlike GraphQL, REST has no built-in schema. We must maintain an API specification ourselves, for example with OpenAPI.
* Some screens will need several API calls to load all data. GraphQL would load them in one query.

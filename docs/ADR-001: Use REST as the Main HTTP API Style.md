# ADR-001: Use REST as the Main HTTP API Style

## Status

Proposed

## Date

2026-06-11

## Context

We need to choose the main HTTP API style for the Messenger application.

The application needs API for users, chats, and messages. Most operations are simple CRUD operations.

GraphQL was considered, but we do not need complex queries now. GraphQL also needs more setup and adds extra complexity.

Real-time communication is not decided in this ADR

## Decision

We will use REST as the main HTTP API style

## Consequences

REST is simple and good enough for current requirements.

The API will be easier to implement and test for our current CRUD use cases.

Developers can use common HTTP tools and standard HTTP methods.

REST API documentation will need to be maintained separately, for example with OpenAPI.

Some screens may need more than one API call to load all data

# ADR-002: Use Server-Push Real-Time Delivery for Chat Events

## Status

Proposed

## Date

2026-07-05

## Context

We need to choose how the Messenger application will deliver new chat events to active web clients.

REST is already chosen as the main API style. It is still good for normal operations, for example login, loading message history, creating chats, and managing chat members.

But chat is event-driven. A new message, read receipt, message edit, or typing indicator can happen at any time. The client does not know when it should ask for new data.

Polling is simple, but it has a bad trade-off:

* If the client polls rarely, messages can be delayed.
* If the client polls often, the backend gets many empty requests.

For a messenger, active chats should feel real-time. We also want to avoid constant polling because the application should be designed with scalability in mind.

## Considered options

* Polling
* Long polling
* Server-push real-time delivery

## Decision outcome

We will use server-push real-time delivery for chat events.

The client should not constantly ask the backend if something changed.

### Consequences

#### Positive

* Active chats can feel more real-time.
* The backend will avoid many empty polling requests.
* This can support messages, typing indicators, read receipts, edits, and deletes.

#### Negative

* The backend becomes more complex than REST-only backend.
* The system must handle connected clients, disconnects, reconnects, etc.
* Deployment setup must support long-lived client connections.

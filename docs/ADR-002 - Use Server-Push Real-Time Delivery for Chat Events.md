# ADR-002: Use Server-Push Real-Time Delivery for Chat Events

## Status

Proposed

## Date

2026-07-05

## Context

We need to choose how the Messenger application will deliver new chat events to active web clients.

REST is already chosen as the API style (ADR-001). It works well for normal operations, for example sending a message, loading message history, creating chats, and managing chat members.

But chat is event-driven. A new message, message edit, message delete, read receipt, or typing indicator can happen at any time. The client does not know when it should ask for new data.

For a messenger, active chats should feel real-time. We also want to avoid constant polling because the application should be designed with scalability in mind.

## Considered options

* **Polling.** Simple, but a bad trade-off: if the client polls rarely, messages can be delayed; if the client polls often, the backend gets too many requests.
* **Long polling.** Better, because the server holds the request open until new data appears. But it still needs a new HTTP request for every event, and it is hard to share one connection between many event types (messages, typing, read receipts).
* **Server-push real-time delivery.** The server sends events to the client over a persistent channel. It gives the same result as long polling with less overhead, and one channel can carry all event types.

## Decision outcome

We will use server-push real-time delivery for chat events, because the client should not constantly ask the backend if something changed.

This is an exception from ADR-001: chat event delivery will not use REST.

Server-push delivery is "best effort". A connection can drop at any time, and events sent during that time are lost on that connection. Because of this, the push channel is only a fast delivery path. The source of truth for correctness is the backend: every chat event must have a stable order (for example, a sequence number per chat), and the client must load missed events over REST after it reconnects.

### Consequences

#### Positive

* Active chats can feel more real-time.
* The backend will avoid many empty polling requests.
* This can support delivering new messages, typing indicators, read receipts, edits, and deletes.

#### Negative

* The backend becomes more complex than REST-only backend.
* The system must handle connected clients, disconnects, reconnects, etc.
* Deployment setup must support long-lived client connections.
* Server-push alone does not guarantee delivery. The system needs per-chat event ordering and a catch-up mechanism over REST.
* With more than one backend node, the node that receives a new message is usually not the node that holds the reader's connection. The system will need a fan-out mechanism (publish/subscribe) to route events to the right connections. The concrete technology is a separate decision. Group chats multiply this work, because one message must reach hundreds of clients.

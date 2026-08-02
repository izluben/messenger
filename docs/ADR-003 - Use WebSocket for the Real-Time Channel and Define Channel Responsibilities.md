# ADR-003: Use WebSocket for the Real-Time Channel and Define Channel Responsibilities

## Status

Proposed

## Date

2026-08-01

## Context

ADR-002 chose server-push for chat events. This ADR picks the concrete transport and defines which traffic uses REST and which uses the real-time channel, because these two questions depend on each other.

The application is web-only, so the transport must work in browsers and pass through common proxies and load balancers.

There is also client-to-server traffic to think about. Typing indicators and read position updates are small, frequent client events; sending each one as a REST request would recreate the polling problem from ADR-002, in the other direction. So a channel that can carry events in both directions is a big plus.

## Considered options

* **WebSocket.** One persistent connection in both directions, supported by all modern browsers. But it is a raw channel: reconnect, resume, and message format are our job to design.
* **Server-Sent Events (SSE), with REST for client-to-server events.** A simple HTTP stream from server to client. Reconnect and resume (`Last-Event-ID`) are built in. But it is one-directional: every client event must still go over REST.
* **WebTransport.** A newer technology on top of HTTP/3. Browser and infrastructure support is not complete yet. Too early for this project.

## Decision outcome

We will use WebSocket for the real-time channel, with this traffic split:

* **Send a message: REST.** A message is durable data. REST gives retries, timeouts, and idempotency keys, so a retry does not create a duplicate.
* **Receive chat events: WebSocket.** All server-push events from ADR-002: new messages, edits, deletes, read receipts, typing.
* **Send light signals: WebSocket.** These events are safe to lose or to repeat. A lost typing indicator breaks nothing. A read position update ("read up to message N") can simply be sent again after reconnect. Read position updates from one user become read receipts for the other users.
* **Catch-up after reconnect: REST.** The client loads missed events over REST, as decided in ADR-002. The WebSocket is only the fast path.

The rule: **if losing the request would lose data, use REST. If the event is frequent and safe to lose or repeat, use WebSocket.**

### Consequences

#### Positive

* One connection carries all real-time traffic in both directions, so frequent small events create no HTTP request noise.
* Message sending keeps simple and reliable HTTP semantics.

#### Negative

* We must design the reconnect logic ourselves. SSE gives this for free.
* Load balancers and deployment must support the WebSocket upgrade and long-lived connections.
* WebSocket traffic is harder to observe and debug than plain HTTP.

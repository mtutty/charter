# Real-time Transport

## Generation Instructions

Real-time Transport provides a single shared WebSocket or SSE server and client utilities for all capabilities that require real-time push — primarily Presence and Messaging. Generating this as a singleton ensures both capabilities share one transport rather than each creating their own server, which would require separate ports and separate authentication handshakes.

This capability should only be declared if at least one capability that requires real-time push (Presence, Messaging) is included. If neither is present, do not generate a real-time transport.

Resolve the protocol from `config.protocol`:

- `websocket`: Generate a WebSocket server using `ws` (not socket.io — keep it minimal unless the application specifically needs socket.io's room/namespace abstractions). Attach the WebSocket server to the existing HTTP server so both run on the same port.
- `sse`: Generate Server-Sent Events endpoints using the standard HTTP response streaming approach. SSE is server-to-client only — if bidirectional communication is needed, use WebSocket. SSE works through HTTP/2 proxies and load balancers that may not support WebSocket upgrades.

Authentication: all connections must be authenticated before any data is sent. On WebSocket handshake (or SSE connection), extract the session token from the request (cookie or `Authorization` header per the Auth capability's session mechanism). Validate the token against the Auth session store. Reject unauthenticated connections with HTTP 401 before the connection is established. Never allow anonymous real-time connections — if a public channel is needed, it must be an explicit design decision recorded in `decisions.md`.

Generate a channel/room abstraction: `realtime.publish(channel, event, payload)` on the server side and a client-side subscription model. Channels are named strings that capabilities define (e.g., `presence:document:123`, `messaging:conversation:456`). The transport does not know what channels mean — it routes messages to subscribers.

Multi-instance scaling: when `infrastructure.scalability.horizontal` is `true`, a WebSocket connection is sticky to one instance. To broadcast to all clients regardless of which instance they are connected to, generate Redis pub/sub as the coordination layer. Each instance subscribes to a Redis channel and forwards published messages to its own connected clients. Use the Redis connection from the Caching capability if present. When `horizontal` is `false`, skip Redis pub/sub — instances do not need to coordinate.

Generate a client-side utility at `lib/realtime-client` that manages the connection lifecycle: connect on app load, reconnect with exponential backoff on disconnect, expose `subscribe(channel, handler)` and `unsubscribe(channel)`. The client utility is used by frontend capability code — Presence avatars, Messaging thread updates — without knowing the underlying transport details.

## Configuration

```yaml
capability: real-time-transport
type: infrastructure
version: 1.0

exposes:
  - realtime-server: >
      lib/realtime — publish(channel, event, payload) for server-side broadcasting.
      Imported by Presence, Messaging, and any capability that pushes to clients.
  - realtime-client-config: >
      lib/realtime-client — connect(), subscribe(channel, handler), unsubscribe(channel).
      Used by frontend capability code. Manages connection lifecycle and reconnection.

config:
  protocol: websocket          # websocket | sse

  auth:
    require: true              # always true — unauthenticated connections are rejected
    mechanism: inherit         # inherits session mechanism from Auth capability (jwt | server-side)

  # multi-instance Redis pub/sub is generated automatically when
  # infrastructure.scalability.horizontal is true and Caching uses Redis
```

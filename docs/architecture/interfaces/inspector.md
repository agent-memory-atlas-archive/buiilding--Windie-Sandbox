# Inspector

The Inspector is Windie's browser-based client. It runs from
`vendor/windie-inspector/frontend/` and gives users a visual way to inspect
conversation state and request work from the local Windie API.

## Purpose

The Inspector is a thin client for the API. It lets users select
conversations and message heads, view the conversation tree, configure models
and tools, start or continue sessions, respond to approvals, and inspect live
runtime progress.

The Inspector runs from a local loopback development or packaged origin. The
browser communicates with the local API rather than reading Windie's SQLite
state or calling Bifrost directly.

## Owns

The Inspector owns the browser-facing presentation boundary:

- browser routes for the workspace (`/`) and addressable sessions
  (`/sessions/:sessionId`);
- conversation-tree rendering, selected-path presentation, panels, controls,
  and browser interaction state;
- loading authoritative conversation, session, model, provider, and plugin
  snapshots from the API;
- sending explicit user actions such as message queries, session control,
  tool approvals, wakeups, and component configuration; and
- consuming replayed and live Server-Sent Events (SSE), keeping event cursors,
  and rendering short-lived streaming previews before durable snapshots arrive.

The Inspector is a local client of the loopback API. It does not own runtime
credentials, account sessions, or pairing state.

## Does not own

The Inspector does not own conversation storage, session ownership, message
head resolution, model context, approval policy, tool execution, provider
inference, or process lifecycle. Those decisions belong to the local API and
the shared runtime, operation, and storage layers.

The Inspector does not infer a session's conversation or branch from its
cached browser state. It asks the API for the authoritative session record and
selected-head resolution, then renders the result.

## Main flow

1. `WindieProvider` loads authoritative snapshots from the local API for
   conversations, sessions, models, tools, providers, and plugins.
2. A user action sends an explicit API request containing the conversation,
   selected message head, or session ID. The API resolves the durable target
   and performs the shared operation.
3. For live session work, the Inspector subscribes to the session SSE route
   with its last accepted event ID. It rejects duplicate or older events,
   reduces accepted events into browser projections, and updates the selected
   conversation and transient stream preview.
4. The Inspector renders the resulting state. Replayed events recover missed
   activity after a disconnect, while the API remains responsible for the
   durable session even if the browser tab closes.

## Important invariants

- The Inspector never infers durable session ownership or conversation
  ownership from cached browser state. The API is authoritative.
- Closing, reloading, or losing the Inspector's SSE connection does not stop
  API-owned session execution.
- The browser never reads SQLite, calls Bifrost, executes tools, or decides
  what the model sees.
- The browser communicates only with the local loopback API; it does not read
  SQLite or attach cloud account credentials to runtime requests.
- SSE event IDs are replay cursors. The Inspector advances a cursor only for
  accepted events so reconnects do not apply the same durable event twice.
- Streaming previews are transient presentation state. Persisted messages and
  session snapshots from the API remain the source of truth.

## Related code

- [`vendor/windie-inspector/frontend/src/App.js`](../../../vendor/windie-inspector/frontend/src/App.js) — mounts the local runtime provider and browser routes.
- [`vendor/windie-inspector/frontend/src/context/WindieContext.jsx`](../../../vendor/windie-inspector/frontend/src/context/WindieContext.jsx) — composes the Inspector's conversation, session, model, tool, plugin, and provider state.
- [`vendor/windie-inspector/frontend/src/hooks/useSessionRuntime.js`](../../../vendor/windie-inspector/frontend/src/hooks/useSessionRuntime.js) — coordinates session selection, API actions, and event projections.
- [`vendor/windie-inspector/frontend/src/hooks/useSessionTransport.js`](../../../vendor/windie-inspector/frontend/src/hooks/useSessionTransport.js) — manages SSE subscriptions and replay cursors.
- [`vendor/windie-inspector/frontend/src/lib/windieApi.js`](../../../vendor/windie-inspector/frontend/src/lib/windieApi.js) — sends HTTP requests to the local API.
- [`vendor/windie-inspector/frontend/src/lib/sessionStream.js`](../../../vendor/windie-inspector/frontend/src/lib/sessionStream.js) — parses session SSE responses.

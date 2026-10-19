---
title: mcl-beam-corpus — Glossary
layer: glossary
audience: [agent, human]
stage: stable
---

# Glossary

Canonical vocabulary for BEAM, OTP, and Event Sourcing / CQRS. One term,
one meaning. When two domains use a word differently, both meanings are
listed.

---

## BEAM & OTP

| Term | Meaning |
|------|---------|
| **BEAM** | The Erlang virtual machine (Bogdan/Björn's Erlang Abstract Machine). Runs all Erlang and Elixir code. |
| **OTP** | Open Telecom Platform: the standard library — behaviours, applications, supervision — plus the middleware (Mnesia, SASL, etc.). "OTP" in context usually means the library, not the telecom middlewares. |
| **Process** | A BEAM unit of concurrency: lightweight, isolated, pre-emptively scheduled. ~2-3 KB initial memory; millions can run. |
| **Message passing** | The only inter-process communication. `send` is async and never fails; the receiver's mailbox is FIFO. |
| **Behaviour** | A module interface: a set of callbacks a module implements plus functions the runtime calls on it (`gen_server`, `supervisor`, `application`, `gen_statem`). |
| **Application** | A component of an OTP system: a supervision tree plus metadata (start type, dependencies). The unit that starts and stops together. |
| **Supervisor** | A process whose only job is starting, watching, and restarting its children. Supervisors never do domain work. |
| **Supervision tree** | The nested hierarchy of supervisors and workers in an application. |
| **Worker** | A process that does domain work (a `gen_server`, a `gen_statem`, a `Task`). Leaves of the tree. |
| **Let it crash** | The OTP failure philosophy: handle the expected errors, let everything else crash, and let supervision restart from a known state. |
| **Restart strategy** | `one_for_one`, `one_for_all`, `rest_for_one`: how a supervisor restarts siblings when one child dies. |
| **Child spec** | The `{id, start, restart, shutdown, type, modules}` tuple describing one supervised child. |
| **Restart intensity / period** | The limit that turns crash loops into supervisor shutdowns: e.g. 3 restarts in 5 seconds is allowed, the 4th terminates the supervisor. |
| **DynamicSupervisor** | A supervisor that starts children on demand (`start_child/2`), not from a static spec list. Replaced `:simple_one_for_one`. |
| **ETS** | Erlang Term Storage: an in-memory key-value store owned by a process (or `:public`), used for caches, counters, lookups. Data dies with the owner unless `:heir` is set. |
| **Mnesia** | A distributed DBMS built into OTP: ETS tables plus transactions, replication, and persistence. |
| **Node** | One BEAM instance with a name, able to message processes on other nodes. The unit of distribution. |
| **Distribution** | Transparent cross-node messaging over TCP/TLS. Processes send to `{Name, Node}`; location is transparent. |
| **Hot code upgrade** | Loading a new version of a module into a running node, then transitioning processes to it (an appup/relup release upgrade). |
| **Release** | A self-contained OTP application bundle produced by `relx`/`mix release`: the ERTS, all dependencies, no source needed to run. |

## Elixir / Erlang language

| Term | Meaning |
|------|---------|
| **Macro** | Elixir: a function that runs at compile time and returns AST. Powers `if`, `defmodule`, `use`. |
| **Protocol** | Elixir: polymorphism by data type, dispatch by the first argument's type. |
| **Struct** | Elixir: a named, key-checked map with defaults, defined by `defstruct`. |
| **Record** | Erlang: a fixed-arity tuple with named, compile-time-checked fields. |
| **Parse transform** | Erlang: an AST-to-AST transform run at compile time, the Erlang equivalent of metaprogramming. |
| **mix** | Elixir's build tool: tasks, deps, test runner, release assembler. |
| **rebar3** | Erlang's build tool: the same role. |

## Event Sourcing & CQRS

| Term | Meaning |
|------|---------|
| **Event** | An immutable fact about something that happened, in the past tense. The source of truth. Never edited, only appended. |
| **Command** | A request that something happen. May be rejected; produces zero or more events. |
| **Aggregate** | The consistency boundary: the unit that decides whether a command produces an event, by loading its own event history and deciding. |
| **Event store** | The append-only log of all events. The single source of truth; every read model derives from it. |
| **Event log** | One stream of events; an append-only log per aggregate, or per stream. |
| **Projection** | A function from events to a read model. The "write" side of the query service: it consumes events and upserts query tables. |
| **Read model** | A query-optimized representation built by projections (a table, a report, a file). Thrown away and rebuilt, never edited by hand. |
| **Checkpoint** | The last-consumed event position of a projection, so it resumes where it stopped. May live in memory, a file, or the database. |
| **Saga** | A long-running, cross-aggregate workflow that coordinates several steps and compensates for failures. |
| **Process manager** | A message-driven state machine, one instance per business process (correlated by id), that drives the process to a known end state. Defined by that job, not by spanning aggregates or divisions. |
| **Causation id** | Identifies the event that caused another event: the causal chain, for tracing and deduplication. |
| **Correlation id** | Identifies one workflow or user action across all the events and commands it produced. |
| **Conversation id** | Identifies one interaction between two services (one request-response exchange). |
| **Idempotency** | Reprocessing the same message produces the same result, by checking message ids against those already seen. |
| **CQRS** | Command Query Responsibility Segregation: separate write (command → aggregate → events) and read (events → projection → query) sides. |
| **Stream fork** | One event stream fanned out to several consumers, each with its own checkpoint. |
| **Stream join** | Two streams merged for a consumer that needs both (e.g. a report). |
| **Optimistic concurrency** | Writes carry an expected version; a mismatch means a conflict to retry, not silent overwrite. |

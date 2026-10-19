---
title: mcl-beam-corpus — Index
layer: index
audience: [agent, human]
stage: stable
---

# mcl-beam-corpus

*Knowledge corpus for BEAM languages (Erlang, Elixir), OTP, and Event Sourcing / CQRS.*

This index maps every note in the corpus. An agent answering a question
recalls from this repo through mcl-rag; this file is the human-readable
map of what is here and where a fuller answer lives.

---

## Domain map

### BEAM & OTP — `beam/`

The virtual machine and the fault-tolerance model every service on the
mesh is built on.

| Note | Covers |
|------|--------|
| [README](beam/README.md) | The BEAM in one page: processes, scheduling, memory |
| [SUPERVISION_TREES](beam/SUPERVISION_TREES.md) | Failure model, supervisor strategies, child specs, DynamicSupervisor |
| [APPLICATIONS](beam/APPLICATIONS.md) | The composition unit: resource files, start types, dependencies |
| [GENSERVER](beam/GENSERVER.md) | The stateful process: callbacks, the boundary shape, call vs cast |
| [TASKS_AND_AGENTS](beam/TASKS_AND_AGENTS.md) | One-shot async work, simple state, the hierarchy of choosing |
| [REGISTRY](beam/REGISTRY.md) | Naming processes: unique/duplicate keys, the via pattern |
| [ETS](beam/ETS.md) | In-memory tables: types, ownership, what it is and is not for |
| [SCHEDULER](beam/SCHEDULER.md) | The m:n model, reductions, concurrency vs parallelism |
| [DISTRIBUTION](beam/DISTRIBUTION.md) | Location transparency, nodes, EPMD, the cookie |
| [HOT_CODE_UPGRADES](beam/HOT_CODE_UPGRADES.md) | appup/relup, release_handler, current vs permanent |
| [CONCURRENCY_MODELS](beam/CONCURRENCY_MODELS.md) | The seven models; the actor model's place among them |
| [DESIGN_PATTERNS](beam/DESIGN_PATTERNS.md) | The GoF 23 mapped onto BEAM built-ins |
| [NERVES](beam/NERVES.md) | The BEAM on bare metal: firmware as an OTP release |
| [GENETIC_ALGORITHMS](beam/GENETIC_ALGORITHMS.md) | Populations on the BEAM: selection, crossover, the GenServer loop |

### Elixir — `elixir/`

Language-specific knowledge.

| Note | Covers |
|------|--------|
| [README](elixir/README.md) | The language in one page |
| [MACROS](elixir/MACROS.md) | Quote/unquote, `use`, hygiene, macros are for libraries |
| [PROTOCOLS](elixir/PROTOCOLS.md) | Dispatch by first argument's type, `@derive`, consolidation |
| [STRUCTS](elixir/STRUCTS.md) | Fixed keys, enforced keys, structs vs maps |
| [MIX](elixir/MIX.md) | Project layout, tasks, releases |

### Erlang — `erlang/`

Language-specific knowledge.

| Note | Covers |
|------|--------|
| [README](erlang/README.md) | The language in one page |
| [MODULES_AND_RECORDS](erlang/MODULES_AND_RECORDS.md) | Exports, attributes, records as tagged tuples |
| [PARSE_TRANSFORMS](erlang/PARSE_TRANSFORMS.md) | Compile-time AST rewrites |

### Event Sourcing & CQRS — `event-sourcing/`

The write model, the read models, and everything between.

| Note | Covers |
|------|--------|
| [README](event-sourcing/README.md) | The pattern landscape at a glance |
| [EVENT_SOURCING](event-sourcing/EVENT_SOURCING.md) | State derived from events; replay; reinterpretation; trade-offs |
| [EVENTS](event-sourcing/EVENTS.md) | Past tense, atomicity, immutability |
| [COMMANDS_AND_QUERIES](event-sourcing/COMMANDS_AND_QUERIES.md) | Commands return status only; queries never mutate |
| [CQRS](event-sourcing/CQRS.md) | The one-line split; what it is not |
| [EVENT_LOGS](event-sourcing/EVENT_LOGS.md) | Appending, segmented, distributed; streams; log vs sourcing; one file per process |
| [MULTI_EVENT_LOG](event-sourcing/MULTI_EVENT_LOG.md) | More than one log: order, position and exactly-once across them |
| [PROJECTIONS](event-sourcing/PROJECTIONS.md) | Simple, updating, inserting, batched; replay vs live |
| [CHECKPOINTS](event-sourcing/CHECKPOINTS.md) | Memory, file, database, stream, shared, consensus |
| [SAGAS_AND_PROCESS_MANAGERS](event-sourcing/SAGAS_AND_PROCESS_MANAGERS.md) | The distinction; versioning long processes |
| [AGGREGATES](event-sourcing/AGGREGATES.md) | The consistency boundary: hydration, aggregate root, invariants |
| [DOMAIN_MODELING](event-sourcing/DOMAIN_MODELING.md) | Bounded contexts, entities vs value objects, ubiquitous language |
| [IDS_AND_CORRELATION](event-sourcing/IDS_AND_CORRELATION.md) | Message, causation, correlation, conversation ids |
| [INTEGRATION_EVENTS](event-sourcing/INTEGRATION_EVENTS.md) | Denormalized contracts; aggregators; direct integration |
| [DIRECT_EVENT_INTEGRATIONS](event-sourcing/DIRECT_EVENT_INTEGRATIONS.md) | Services subscribing to each other's events; when it holds and how it fails |
| [NEGOTIATING_AGGREGATOR](event-sourcing/NEGOTIATING_AGGREGATOR.md) | One document plus a link; consumers fetch the version they understand |
| [STREAM_FORK_AND_JOIN](event-sourcing/STREAM_FORK_AND_JOIN.md) | Reindexing with link events; join ordering |

### Testing — `testing/`

How to hold all of the above to a standard.

| Note | Covers |
|------|--------|
| [README](testing/README.md) | The three highest-value test shapes |
| [TESTING_EVENT_SOURCING](testing/TESTING_EVENT_SOURCING.md) | Given/when/then, replay tests, tests as documentation |
| [PROPERTY_BASED_TESTING](testing/PROPERTY_BASED_TESTING.md) | Stateless vs stateful, shrinking, models, state machines |
| [FAULT_INJECTION](testing/FAULT_INJECTION.md) | Killing processes, cleanup assertions, wait_for_passing |
| [EXUNIT](testing/EXUNIT.md) | The four phases, describe, setup blocks, async |
| [LEGACY_CODE](testing/LEGACY_CODE.md) | Seams, characterization tests, dependency breaking |

### Architecture — `architecture/`

The system-level patterns above the BEAM.

| Note | Covers |
|------|--------|
| [README](architecture/README.md) | The pattern landscape above the process model |
| [EVENT_DRIVEN_ARCHITECTURE](architecture/EVENT_DRIVEN_ARCHITECTURE.md) | Mediator vs broker topology |
| [ARCHITECTURE_PATTERNS](architecture/ARCHITECTURE_PATTERNS.md) | Layered, microkernel, microservices, space-based |
| [EXUNIT](testing/EXUNIT.md) | The four phases, describe, setup blocks, async |

---

## Reading path — agent

1. [`GLOSSARY.md`](GLOSSARY.md) for the vocabulary.
2. The domain README closest to the question.
3. The specific note, if the README points at one.

## Reading path — human

1. [`README.md`](README.md) → this index → the domain that interests you.
2. Notes are short and standalone; there is no required order.

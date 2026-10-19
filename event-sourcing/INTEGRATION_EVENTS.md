---
title: "Event Sourcing: Integration Events"
layer: guide
audience: [agent, human]
stage: stable
---

# Event Sourcing: Integration Events

*Domain events are an internal detail of one bounded context. Integration events are the deliberately designed, versioned contract that context offers everyone else.*

---

## Two kinds of event

Inside a context, events are shaped for the context's own aggregates and
projections. They refer to other things by id and change whenever the
model changes:

```elixir
%SeatReserved{show_id: "sh-77", seat_id: "B-12", customer_id: "cust-981"}
```

A consumer in another context usually cannot do anything with bare ids
without calling back. The integration event for the same fact carries
what an outsider needs, in terms the outsider understands:

```elixir
%{
  type: "box_office.seat_reserved", version: 2,
  show: %{id: "sh-77", title: "The Tempest", starts_at: "2026-10-03T19:30:00Z"},
  seat: %{id: "B-12", row: "B", number: 12, zone: "stalls"},
  customer_id: "cust-981"
}
```

The usual mechanism is an **adapter** (an event handler or process
manager) that subscribes to internal events, enriches them
from its own read models, and publishes the integration event. Several
adapters publishing several versions of the same fact at once is normal.

## Two rules

1. **Keep internal events private.** Publishing them directly couples
   every consumer to your internal model, and every refactor becomes a
   cross-team negotiation.
2. **Design for a long life.** A popular integration event may have many
   subscribers on release cycles you do not control. Do not change a
   published schema in a breaking way; publish a new version alongside
   the old one and retire the old one on an announced schedule.

## Coarser contracts: the aggregator

Some consumers do not care about every fine-grained step; they want
"the current state of this order". An **event aggregator** listens to
the detailed events and publishes one summary event, carrying the latest
state, whenever anything relevant changes. The consumer integrates with
one schema instead of a dozen.

A variation publishes only a small notification with a link, and lets
the consumer fetch the state:

```elixir
%{type: "box_office.order_changed", order_id: "ord-4410",
  href: "https://box-office.example/orders/ord-4410"}
```

The consumer then asks for the representation and version it
understands (HTTP content negotiation), and the provider converts. That
decouples version upgrades: each consumer moves when it is ready. The
trade-off is an extra round trip and a provider that must stay available
to serve reads. This is the notification-versus-state-transfer choice
Martin Fowler describes. Worked through in full, including how to learn
who still uses an old version, in
[NEGOTIATING_AGGREGATOR](NEGOTIATING_AGGREGATOR.md).

## Direct subscription and its limits

The simplest integration is for context A to subscribe straight to
context B's events. With a handful of services and low volume that is
fine. It breaks down organisationally rather than technically: once
dozens of services subscribe to each other, nobody can say who depends
on what, chains of reactions are invisible, and every change needs many
teams.

One situation where broad direct subscription stays sensible: a single
dataset that almost everyone needs (a price feed, a product catalogue).
Then a single well-documented feed is simpler than many adapters. The
pattern, its failure modes and the ways out are in
[DIRECT_EVENT_INTEGRATIONS](DIRECT_EVENT_INTEGRATIONS.md).

## Choosing

In the wider industry the choice runs:

- Few services, low volume: direct subscription.
- Many consumers or outside consumers: explicit integration events from
  an adapter, versioned from day one.
- Consumers that only need current state, on independent release
  cycles: an aggregator, possibly notification-plus-fetch.

Macula codebases take the second option always. Domain events stay in
their context's store; a process manager (`on_{event}_{action}`)
decides what to publish as an integration fact on the mesh, with its own
public schema. No context subscribes to another context's domain
events, however small the system.

See also [DOMAIN_MODELING](DOMAIN_MODELING.md) for bounded contexts and
[IDS_AND_CORRELATION](IDS_AND_CORRELATION.md) for the metadata every
integration event should carry.

## Sources

- Greg Young, *Patterns of Event Sourced Systems*, Leanpub (in progress, last updated 2025). https://leanpub.com/patternsofeventsourcedsystems
- Martin Fowler, "What do you mean by 'Event-Driven'?", martinfowler.com, 2017 (free). https://martinfowler.com/articles/201701-event-driven.html
- Microsoft patterns & practices, *Exploring CQRS and Event Sourcing* (Reference 5, "Communicating between Bounded Contexts"), 2012 (free online). https://learn.microsoft.com/en-us/previous-versions/msp-n-p/jj554200(v=pandp.10)
- Microsoft, "Event Sourcing pattern" (note on integration events), Azure Architecture Center (free). https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

---
title: "Event Sourcing: Direct Event Integration"
layer: guide
audience: [agent, human]
stage: stable
---

# Event Sourcing: Direct Event Integration

*Service A subscribes to service B's events and acts on them, with no adapter and no designed contract. The simplest integration there is, sound while the web of subscriptions is small enough to draw, and a liability once it is not.*

---

## The shape

Each service listens to the events of the others and does something
because of them. Not only building a read model: taking action. With four
services:

```
A listens to C, D
B listens to A, C
C listens to A, B
D listens to A, B
```

Every arrow is a subscription that somebody added, usually in a small
pull request, usually for a good local reason. Nothing in the topology is
designed; it is the sum of those decisions.

## Why it works at first

- Nothing to design: an existing event is already published, so a new
  consumer costs one handler.
- With three or four services and a handful of event types, one person
  can hold the whole graph in their head and can answer "who reacts to
  this?" from memory.
- Reactions are immediate and easy to follow in a log.

This is a real engineering choice, not a mistake, and the book says so
plainly: there is nothing inherently wrong with direct event
integration, and there are situations where it is the normal thing to do.

## Why it stops working

The failure is not volume. It is the number of links.

| Symptom | What it means |
|---------|---------------|
| Nobody can list the consumers of an event | The graph is now folklore |
| Chains of reaction appear (A changes, B reacts, C reacts to B) | No one owns the chain, and no one can see it end to end |
| A change needs a coordinated multi-team release | The internal event has quietly become a public contract |
| A new event needs a survey before it can ship | The cost of a change is now organisational |
| A slow subscriber slows the producer | The coupling is operational as well as logical |

At thirty services the same four questions become unanswerable: who
listens to what, which chains exist, which services take part in a chain,
and how do the teams running all of them learn about a change. The
pattern does not collapse with a bang; it becomes expensive to change
anything, which is the same thing over a longer horizon.

There is a second, sharper cost. The consumed events are internal to the
producing context: shaped for its aggregates, renamed when its model
changes, and never designed for outsiders. Direct integration makes that
internal detail the integration contract by accident
([INTEGRATION_EVENTS](INTEGRATION_EVENTS.md) is the deliberate
alternative).

## Where it nonetheless stays right

One dataset that much of the organisation genuinely needs. A market data
feed is the standard example: nearly every system in a trading
organisation consumes prices, at different service levels, and pricing a
single contract can reach back into that feed. The direct fan-out is not
the problem there, and replacing it with dozens of adapters would be
worse.

The lesson is not "remove the fan-out". It is that when everything
depends on one feed, that feed has stopped being an internal event type
and has become a product: it deserves its own contract, its own
versioning, its own availability target, and its own owner. Direct
integration is acceptable when the thing being consumed is stable enough
to carry that weight with no formalities. The moment it is not stable,
the formalities are how you survive.

## What to do instead

In rough order of cost:

1. **Make the subscription explicit.** Write down who consumes what, put
   the consumer list next to the event's definition, and version the event.
   The topology stays; the folklore goes. Often enough.
2. **Insert an adapter.** One component subscribes to the internal events
   and publishes an integration event with its own schema
   ([INTEGRATION_EVENTS](INTEGRATION_EVENTS.md)). Consumers now depend on
   a contract that can change independently of the producer's model.
3. **Publish state, not steps.** If consumers rebuild the same picture
   from the same twenty events, an aggregator that publishes the picture
   is cheaper for everyone
   ([NEGOTIATING_AGGREGATOR](NEGOTIATING_AGGREGATOR.md)).
4. **Split the feed.** If everything consumes one dataset, give it an
   owner; publish it like a product, not like a side effect.

## Signs you have already gone too far

- You cannot answer "who consumes this event?" without grepping.
- You cannot draw the dependency graph on one page.
- Two services subscribe to each other.
- An internal event, defined for one aggregate, has consumers in other
  repositories.

## In this corpus

Macula codebases answer this with a rule rather than a judgement call: a
context does not subscribe to another context's domain events, however
small the system. A process manager in the producing context decides what
to publish, as a versioned integration fact with its own schema
([SAGAS_AND_PROCESS_MANAGERS](SAGAS_AND_PROCESS_MANAGERS.md)).
Subscription inside a context, between its own aggregates, is ordinary
and stays.

## Sources

- Greg Young, *Patterns of Event Sourced Systems*, Leanpub (in progress, last updated 2025): the chapters "Direct Event Integration" and "Integration Event". https://leanpub.com/patternsofeventsourcedsystems
- Martin Fowler, "What do you mean by 'Event-Driven'?", martinfowler.com, 2017 (free). https://martinfowler.com/articles/201701-event-driven.html
- Microsoft patterns & practices, *Exploring CQRS and Event Sourcing* (Reference 5, "Communicating between Bounded Contexts"), 2012 (free online). https://learn.microsoft.com/en-us/previous-versions/msp-n-p/jj554200(v=pandp.10)

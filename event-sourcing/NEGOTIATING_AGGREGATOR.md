---
title: "Event Sourcing: The Negotiating Aggregator"
layer: guide
audience: [agent, human]
stage: stable
---

# Event Sourcing: The Negotiating Aggregator

*An event aggregator that publishes a link alongside the current state and converts on request, so each consumer moves to a new schema version when it is ready instead of when the producer ships.*

---

## The problem it solves

An [event aggregator](INTEGRATION_EVENTS.md) republishes the current
state of something as one document, so consumers stop tracking your
fine-grained events. The document's schema then changes, as schemas do.
With two consumers you agree a release date. With dozens, each on its own
cycle, you cannot, and a change nobody agreed to breaks somebody.

Two ways out:

1. **Publish every version side by side.** Build the old document and the
   new one and put each out on its own channel; retire the old one when
   its consumers have moved. Simple, and it multiplies the work with
   every version you keep alive.
2. **Publish one document and let the consumer ask for the shape it
   understands.** The message carries the latest representation plus a
   URI; a consumer that cannot read the latest fetches the URI with an
   `Accept` header naming the version it wants, and the provider converts.

The second is the negotiating aggregator, and Greg Young's book leans
toward it wherever several consumers upgrade on their own schedules.

## The shape

```elixir
%{type: "box_office.order_changed", order_id: "ord-4410", version: 3,
  doc: %{..., status: :paid},
  href: "https://box-office.example/orders/ord-4410"}
```

A consumer that knows version 3 reads `doc` and stops. A consumer still
on version 2 ignores `doc` and asks:

```
GET /orders/ord-4410
Accept: application/vnd.boxoffice.order.v2+json
```

The provider answers with the version 2 document. It already knows how to
convert, because internal code that reads old events has needed that
conversion for years. The pattern turns an existing capability into an
external one.

## Content type is the mechanism, not the point

Most people meeting this pattern file it under "XML versus JSON", or
"let the client choose a format". That is the plumbing. The purpose is
versioning: many consumers, different release cadences, no coordinated
upgrade, and a producer that must keep serving all of them at once.
Format negotiation is how the version request travels; a project that
negotiates formats but not versions has missed it.

## Operating it

- **Log every negotiation**: which version, which caller, when. That log
  is the only trustworthy answer to "who still needs version 2", and it
  turns a deprecation argument into a fact.
- **Give each version an end date** and say so on the response when a
  caller uses a version inside its final window. A warning before the
  change beats an outage after it.
- **Keep the documents addressable for as long as events that point at
  them are replayable.** A link that 404s a year later is worse than no
  link: replaying history now depends on an endpoint that no longer
  answers.
- **Treat the endpoint as public surface**: authentication, rate limits,
  and an id scheme stable enough to be cited in an event.

## What it costs

| Cost | Consequence |
|------|-------------|
| An extra round trip per consumer that is behind | Latency on the first read |
| The provider must serve reads | Its availability becomes part of the integration contract |
| Two dependencies per consumer | The feed, and the endpoint behind the link |
| A query surface to secure and keep | Work that a plain fan-out does not have |

The costs are worth paying when the alternative is many channels of
near-duplicate documents, or a rollout that every consuming team must
join. With one consumer, or a set of consumers that release together,
publishing a single version and changing it is cheaper.

Related: [INTEGRATION_EVENTS](INTEGRATION_EVENTS.md) for the aggregator
and the notification-versus-state-transfer choice,
[DIRECT_EVENT_INTEGRATIONS](DIRECT_EVENT_INTEGRATIONS.md) for the
integration that has no contract at all.

## Sources

- Greg Young, *Patterns of Event Sourced Systems*, Leanpub (in progress, last updated 2025): the chapters "Event Aggregator", "Integration Event" and "Negotiating Event Aggregator". https://leanpub.com/patternsofeventsourcedsystems
- IETF, "HTTP Semantics", RFC 9110, section 12 (proactive content negotiation). https://www.rfc-editor.org/rfc/rfc9110#name-content-negotiation
- Martin Fowler, "What do you mean by 'Event-Driven'?" (event notification versus event-carried state transfer), martinfowler.com, 2017 (free). https://martinfowler.com/articles/201701-event-driven.html
- Microsoft, "Event Sourcing pattern" (versioning integration events), Azure Architecture Center (free). https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

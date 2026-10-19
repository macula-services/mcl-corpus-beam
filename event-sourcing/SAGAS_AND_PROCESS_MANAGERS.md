---
title: "Event Sourcing: Sagas and Process Managers"
layer: guide
audience: [agent, human]
stage: stable
---

# Event Sourcing: Sagas and Process Managers

*A saga moves forward step by step and undoes completed steps when it cannot finish. A process manager is a message-driven state machine that runs one business process to a known end state. What defines the second is the job, not how many aggregates or services it touches.*

---

## What a process manager is

A process manager is a message-driven state machine with one instance per
business process, keyed by a correlation id. It receives the events that
belong to its process, decides which command to send next, and is
responsible for making sure the process reaches a known end state. Its
lifetime matches the process: it starts when the process starts and is
done when the process is done.

The pattern is older than event sourcing and goes by several names:
orchestration, workflow, business process (Hohpe and Woolf called it
Process Manager). Event sourcing changes only where its state lives.

The defining property is its **job**. A process manager is not defined by
spanning two aggregates, or two divisions, or two services. Those are
things it may do when the process requires it. A rule that reacts to one
event inside a single aggregate and sends one more command is written the
same way; it is the degenerate case, one transition and nothing to
remember.

## The terms, and why they get confused

"Saga" has two meanings in circulation:

1. **The original** (Garcia-Molina and Salem, 1987): a long-running
   transaction split into a sequence of local transactions, each paired
   with a *compensating* transaction. If step 4 fails, the compensations
   for steps 3, 2 and 1 run. No distributed lock is held throughout.
2. **The CQRS-community usage**, where "saga" often names any piece of
   code that coordinates messages, which is really a process manager.

Microsoft's CQRS Journey adopted "process manager" for the second
meaning to avoid the clash; this corpus does the same.

| | Saga | Process manager |
|---|---|---|
| Shape | A planned sequence of steps; the plan can travel with the message (a *routing slip*) | A state machine instance per process, correlated by id |
| Where state lives | In the message / slip, or in each step's own records | In the process manager (as events, if event-sourced) |
| Good at | Mostly linear flows with a clear undo for each step | Branches, waiting, timeouts, retries, loops |
| Failure handling | Run compensations for completed steps | Decide: retry, compensate, escalate to a human |

Start with the simpler shape. A forward-only flow with compensations is
easier to reason about than a state machine. Move to a process manager
when the flow needs to wait, branch or revisit earlier steps; a small
change in business rules is often enough to tip the balance.

## Where a policy lives

Every rule that says "when this happened, maybe do that" is written the
same way in an event-sourced system, as an `on_*` slice. It becomes a
process manager when it carries state from one message to the next, since
that state is what lets it drive a process; a rule that reacts to a single
event is the degenerate case, one transition and no state to keep. What
varies beyond that is its reach:

| Reach | Example |
|-------|---------|
| Inside one aggregate | A full table starts the game; a flagged clock ends it |
| Across aggregates in one division | Order paid starts the reservation; reservation expired releases the stock |
| Across divisions | Order placed asks billing to charge, as an integration fact |

All three are `on_*` slices with the same shape: subscribe to the event,
decide, send the next command. Crossing a division boundary only adds
the integration rules ([INTEGRATION_EVENTS](INTEGRATION_EVENTS.md)); it
does not make the slice a different kind of thing.

**Do not fold a policy into a command handler to make two events atomic.**
It is tempting when the reaction is immediate: emit both events in one
append and the crash window between them disappears. The cost is that the
policy becomes invisible at the filesystem level, hidden inside a handler
that now does two jobs, and nobody can find it when its rule changes.

Close a crash window from inside the process manager instead: **catch up
on start**. If the process manager starts and finds an event it has seen
that has no follow-up command, it sends the missing command then. That
makes the reaction at-least-once and recoverable, at the price of an
idempotent command handler, which was needed anyway.

## A process manager, concretely

A seat purchase touching reservation, payment and ticketing:

```
SeatReserved      ─▶ PM: state :reserved   ─▶ command ChargeCard
PaymentCaptured   ─▶ PM: state :paid       ─▶ command IssueTicket
PaymentDeclined   ─▶ PM: state :declined   ─▶ command ReleaseSeat
(15 min, no payment) timeout ─▶ PM         ─▶ command ReleaseSeat
TicketIssued      ─▶ PM: state :done (process ends)
```

The process manager holds no business rules of its own; those stay in
the aggregates. It routes, waits, and makes sure the process ends in a
known state.

## Event-sourcing the process manager

An event-sourced process manager stores the events it has received (or
its own decisions) and derives its state by folding them, exactly like
an aggregate. The benefit shows up over long lifetimes. A process that
runs for weeks or months will outlive several deployments; if its state is
a stored snapshot, every new version must understand every old snapshot
format. If its state is derived from events, a new version can simply
fold the same events with new logic.

That one move, storing the decisions as events rather than the state as a
snapshot, is what makes long processes survivable. Court documents, loans
and shipping claims really do take months; the version that starts a
process is rarely the version that finishes it.

## Changing a process while instances are running

A new version must not take over instances that are already running. The
reason is not packaging, it is that the change is usually to the **process
itself**. If version 1 values the goods, releases them, moves them and
then sends the invoice, and version 2 sends the invoice before the move,
then an instance that was at "send the invoice" at the switchover has
already moved the goods. Continue it under version 2 and it moves them
again.

So versions are tied to the version the instance was running when it
started. New instances start on the new version; old instances finish on
the old one; both run until the old ones drain. That is the cheap answer,
and usually enough.

When it is not acceptable, the hand-over has to be explicit. The old
version emits an event recording exactly where the instance stands (for
example `PurchaseInterrupted{purchase_id, reserved: true, paid: false}`),
carrying the state the new version needs; the new version subscribes to
that event and takes over from there.

The hand-over has to be planned in advance: the old version must know
which state to put on the interruption event, and the new version must
know how to translate it. It is also simpler than most of the
alternatives, because everything that matters travels in one event.

## On the BEAM

A process manager instance maps well onto a supervised process
(GenServer or `gen_statem`) registered by correlation id, with timeouts
from `Process.send_after/3` or `gen_statem` state timeouts. Timeouts
must themselves be recorded or recomputed from events on restart, or a
crash will lose them. The catch-up-on-start rule above is the other half
of the same idea: whatever the process was waiting for, it can work out
again from its own events.

## Sources

- Hector Garcia-Molina and Kenneth Salem, "Sagas", *Proceedings of ACM SIGMOD 1987* (free PDF). https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf
- Microsoft patterns & practices, "Reference 6: A Saga on Sagas", *Exploring CQRS and Event Sourcing*, 2012 (free). https://learn.microsoft.com/en-us/previous-versions/msp-n-p/jj591569(v=pandp.10)
- Gregor Hohpe and Bobby Woolf, "Process Manager" and "Routing Slip", Enterprise Integration Patterns (free pattern pages). https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html and https://www.enterpriseintegrationpatterns.com/patterns/messaging/RoutingTable.html
- Greg Young, *Patterns of Event Sourced Systems*, Leanpub (in progress, last updated 2025): the chapters "Event Sourced Process Manager" and "Event Sourced Saga". https://leanpub.com/patternsofeventsourcedsystems
- Greg Young, *Versioning in an Event Sourced System*, Leanpub (last updated 2017; free to read online). https://leanpub.com/esversioning/read

---
title: "Event Sourcing: Multi Event Log"
layer: guide
audience: [agent, human]
stage: stable
---

# Event Sourcing: Multi Event Log

*One system, more than one log. The moment the log stops being a single ordered thing, total order, position and exactly-once answers stop being free.*

---

> **Note on the source.** In the public draft of Greg Young's book the
> chapter "Multi Event Log" carries a title and no body. What follows is
> our account, grounded in its sibling chapters ("Event Log", "Event
> Stream", "Distributed Event Log") and in the practice of stores that
> implement several logs side by side. It is marked as ours, not as his
> position.

## How a system ends up with several logs

Nobody sets out to build a multi event log. It arrives one decision at a
time:

- **By partition.** A log is partitioned into streams, one per aggregate
  or per process ([EVENT_LOGS](EVENT_LOGS.md)); there may be millions of
  them.
- **By ownership.** Each service or bounded context owns its store. Two
  contexts mean two logs, in two technologies, with two backup policies.
- **By copy.** A read-side or archive store, a second region, or a bridge
  that subscribes to one store and appends into another.
- **By design of the small thing.** A file per process instance
  ([EVENT_LOGS](EVENT_LOGS.md)) is several logs by construction, one per
  instance, and so is any log written by an embedded device that syncs
  when it can.

## What is lost, and what replaces it

| Question | One log | Several logs |
|----------|---------|--------------|
| Where is the total order? | The log's position | Nowhere. Order exists inside each log only |
| How do I tie one business process together? | A position range | A correlation id carried on every event ([IDS_AND_CORRELATION](IDS_AND_CORRELATION.md)) |
| How do I know I already applied an event? | The position | The event's own id, plus your own checkpoint ([CHECKPOINTS](CHECKPOINTS.md)) |
| How does a new read model see all history? | Read from position 0 | One reader per log, merged with a rule for ties |
| Who is allowed to write this stream? | The owner | The owner, in exactly one of the logs |

The last row is the rule that matters most: **a stream has one writer and
lives in exactly one log**. Two writers appending to the same stream in
two stores is a merge conflict nobody can resolve later, because both
sides have plausible history.

## Replication and bridges

Copying events between logs is a consumer like any other: it needs a
checkpoint, it delivers at least once, and it can stop. That gives a
short, useful list of obligations:

- Order is preserved only from the bridge's own start point, and only
  within what it copies. Consumers downstream must not assume that
  anything they see arrived in the order it was produced.
- Duplicates are normal. The bridge may re-send after a restart; the
  target deduplicates on the event id.
- The bridge is a dependency. If it stops, the target is stale while
  looking healthy; lag against the source is the metric to watch.
- An event copied across a context boundary is an integration event
  ([INTEGRATION_EVENTS](INTEGRATION_EVENTS.md)), whether or not anyone
  designed it as one. The producing context owns its content; the copying
  side must not reshape it.

## Positions are local

A position means something only inside the log that issued it: a byte
offset, a stream version, a global sequence. Two logs happen to both have
a position 41,283 and they have nothing to do with each other. Code that
compares positions across stores, or stores one store's position as a
checkpoint into another, produces a reader that silently skips or repeats
events. The only portable coordinates across logs are the identifiers:
message id, correlation id, and time (with the usual caveat that clocks
disagree).

## When several logs need one story

Audit, a search index, or an analyst's question sometimes wants one
chronological view across several logs. Build it as a projection into one
place, with an explicit merge rule, and accept that it is eventually
consistent. Do not promote that merged view to the write path: a log
assembled from other logs has no single writer to enforce its order.

## Choosing

- Prefer one log per owned context, and explicit integration between
  contexts, over a shared log that several services write into.
- Prefer one store per context over the same stream copied into two
  stores with two writers.
- Copy events between logs when you need availability or locality, and
  treat the copy as a consumer with a checkpoint, not as a second source
  of truth.
- If you cannot say which log a stream lives in, the design has a hole.

Related: [EVENT_LOGS](EVENT_LOGS.md) for the kinds of log and their
limits, [STREAM_FORK_AND_JOIN](STREAM_FORK_AND_JOIN.md) for producing new
streams from existing ones without leaving the log.

## Sources

- Greg Young, *Patterns of Event Sourced Systems*, Leanpub (in progress, last updated 2025): the chapter "Multi Event Log" (title only in the current draft), with "Event Log", "Event Stream" and "Distributed Event Log". https://leanpub.com/patternsofeventsourcedsystems
- KurrentDB documentation, "Streams" and "$all" (one global log, many streams, a global position that is local to that store), free. https://docs.kurrent.io/
- Apache Kafka documentation, "Log compaction" and consumer offsets (positions meaningful only within a partition), free. https://kafka.apache.org/documentation/
- Microsoft, "Event Sourcing pattern", Azure Architecture Center (free). https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

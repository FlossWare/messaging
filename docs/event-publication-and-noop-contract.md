# Event Publication and No-Op Publisher Contract

## Purpose

FlossWare services need a stable way to announce domain changes without requiring a broker, network service, or consumer to exist at first. This contract establishes the call boundary now; transports and durable delivery are separate capabilities.

## Core contract

A publisher exposes a fire-and-forget operation that accepts a message envelope and does not wait for consumers or downstream acknowledgements. The contract is language-neutral; implementations may be interfaces, protocols, or equivalent abstractions. The no-op implementation returns immediately. A transport-backed implementation SHOULD enqueue or hand off work promptly and return without waiting for consumer processing. Its acceptance result means only what that implementation explicitly documents.

Conceptually:

```text
Publisher.publish(message) -> PublishResult
```

The message envelope SHOULD include:

- `messageId`: unique identifier for this event/message
- `type`: stable domain event name, such as `memory.record.created`
- `envelopeVersion`: version of the common envelope, distinct from the domain event type
- `occurredAt`: event occurrence time
- `producer`: service identity
- `subject`: affected resource/aggregate identity
- `revision`: resource revision when the domain supports it
- `payload`: schema-defined event data

Events SHOULD communicate facts that have already happened. Payloads SHOULD be compact and identify the changed resource so consumers can fetch the full resource through its owning service. Large source content SHOULD NOT be copied into every event unless a documented event contract requires it.

## No-op publisher

A no-op publisher SHALL implement the same publisher contract as transport-backed implementations.

- Calling `publish` SHALL be valid, return immediately with a result that explicitly identifies no-op behavior, and MUST NOT claim that a message was delivered, retained, or made available to consumers.
- The no-op publisher SHALL require no configuration, broker, network service, background thread, or external resource.
- It SHALL NOT persist, queue, replay, or forward messages.
- It SHALL NOT change the success/failure outcome of the domain operation merely because it did not deliver a message.
- Services SHALL call the publisher through the shared contract regardless of whether the configured implementation is no-op. Domain logic SHALL NOT branch on transport availability or wait for consumers.
- The no-op publisher provides architectural substitutability, not a delivery guarantee.

## Transport and delivery semantics

Transport-backed implementations MUST document their handoff, buffering, and failure guarantees. Fire-and-forget means the producer does not wait for consumer processing; it does not by itself require asynchronous threads or queues if a transport can perform a prompt handoff. A successful publish result means only what that implementation's contract explicitly states. At-least-once delivery requires consumers to tolerate duplicates. Ordering is scoped only where the implementation explicitly guarantees it. Exactly-once delivery MUST NOT be claimed without a precise, testable definition.

A publish performed after a database write is not atomic with that write. If losing notifications is unacceptable, the producer SHOULD use an outbox or equivalent durable handoff. The core contract does not mandate an outbox or broker.

## Event publication versus querying

Publication is fire-and-forget: producers announce a change and continue without waiting for consumers. Events let consumers react to changes without repeatedly scanning every resource. Events are not the source of truth and do not replace direct retrieval, enumeration, export, or reconciliation. Consumers must account for no-op deployments, missed notifications, duplicate delivery, and restart/recovery.

## Conformance tests

Implementations SHOULD share contract tests verifying:

- A valid message can be submitted through the common interface.
- No-op publication returns an explicit no-op result and has no external side effects.
- The no-op implementation requires no transport configuration.
- Transport-backed implementations accurately report their documented acceptance/failure semantics.
- Envelope identity and event type remain stable across implementations.

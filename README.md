# messaging

**Reusable transient messaging between FlossWare services and agents.**

This repository is intended to define message envelopes and delivery interfaces for service-to-service or agent collaboration. Transient messaging is not automatically durable event storage.

## Scope

- Language-neutral message envelope and correlation metadata
- Publish/subscribe and point-to-point interfaces where required
- Delivery, acknowledgement, timeout, and error semantics
- Pluggable transports
- Diagnostics and contract tests

## Event publication

The shared publication and no-op behavior is specified in [Event Publication and No-Op Publisher Contract](docs/event-publication-and-noop-contract.md). A no-op publisher implements the same contract as transport-backed publishers but does not claim delivery, retention, or replay.

## Architectural boundaries

- `loom` defines execution semantics; messaging may transport messages but does not define Loom orchestration.
- `storage` owns persistence mechanisms.
- Durable queues, replay, and event history must be explicit capabilities, not accidental side effects of a messaging adapter.
- Security policy and authorization remain authoritative.

Avoid making a specific broker or transport part of the core contract. Do not require a standalone network service when an in-process implementation satisfies the same contract.

## Status

Repository scaffold. Specify delivery guarantees and failure semantics before publishing a stable API.

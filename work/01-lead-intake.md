# Case study 1 — A lead intake pipeline that stopped sending messages at 22:00

**Type:** case study, client-shaped work · **Status:** delivered, documentation in the repository ·
**Disclosure:** built by Dejan Milošević; AI agents wrote part of the code under his direction and
review.

## The problem

A webhook form forwarded leads into a notification pipeline. Two things went wrong in production:
a browser retry or a duplicate webhook sent the same customer **two SMS**, and messages went out
**at 22:00** to recipients whose local time nobody checked. Both are the kind of failure that
costs a company customers quietly.

## The solution

The intake was rebuilt around a guarantee: one lead is processed once, and a message never leaves
outside the recipient's quiet hours.

- **Idempotency** — a duplicate webhook returns the first result instead of sending again.
- **Outbox with retry** — every message is written before it is sent; retries at 1s, 5s, 25s,
  125s with backoff.
- **Dead-letter queue with replay** — a message that fails four times is kept with its reason and
  can be replayed, never dropped silently.
- **Quiet hours per recipient timezone**, with opt-out respected before anything else.
- **Trace and cost per stage**, so a failure can be located instead of guessed.

## Architecture

```
webhook → validate → idempotency key → queue (outbox) → worker → provider (SMS/email)
                                            ↓ after 4 attempts
                                       dead-letter + replay endpoint
```

State lives in PostgreSQL. The outbox is written in the same transaction as the lead, which is why
a crash cannot produce a lead without a queued message or a message without a lead.

## Technologies

Node.js, TypeScript, PostgreSQL, Docker, Redis.

## The role

Product creator and system architect: defined the guarantee, designed the outbox and the
dead-letter model, wrote the test matrix and reviewed every change the agents produced. The agent
contributed a large part of the implementation (9 commits, all visible in the history).

## Demonstration

The test suite is the demonstration. It runs against a fresh, empty database, so nothing passes
because of leftover state.

## Verifiable results

Documented in the repository's `VERIFICATION.md`, recorded on 17.11.2026:

- **26 / 26 tests** pass on a fresh database;
- **200 requests per second, p95 16 ms** on a single node for the intake path;
- duplicate webhook produces **one** lead and returns `duplicate: true`;
- an opted-out recipient is **suppressed**, nothing queued;
- a failing send reaches the dead-letter queue after 4 attempts and **replays** to `sent`.

## Limits

- The source is not public (client-shaped work), so a stranger cannot reproduce the numbers
  without the code.
- The tests prove behaviour under the scenarios written; they do not prove every provider failure
  mode.
- No claim is made about deliverability rates with a real carrier.

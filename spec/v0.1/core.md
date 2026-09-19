# Ops Tool Spec (OTS) — Core, v0.1

> **Draft. Unstable.** Written alongside a working implementation, not by a committee.
> Breaking changes are expected before v1.0.

## 0. Scope and layering

This specification defines **what** operations software should expose to an AI agent and
**how results and permissions are shaped**. It does not define transport or discovery.

```
   Agent (Claude, or any other)
        │
   MCP  │  ← transport, discovery, invocation. Already standardised. Not our business.
        │
   ── Ops Tool Spec ──  ← what tools exist, what they return, who may call them
        │
   Your operations software
```

An implementation MAY expose the same tools over plain HTTP instead of, or in addition to, MCP.
The rules below are about the tools themselves and apply either way.

## 1. Tools return verdicts, not prose

A tool result MUST be a structured value. It MUST NOT be a sentence written for a human.

```jsonc
// Wrong — the agent has to parse prose, and it only works in one language
{ "note": "IPv4 is fine (no usable IPv6 address), subnet 192.168.1.0/24" }

// Right
{ "verdict": "v4-only", "networks": ["192.168.1.0/24"], "usb": "lan" }
```

A **verdict** is a stable string code naming a *state*, accompanied by the values that
led to it. Codes are lowercase, hyphenated, and stable across versions of the implementation.

This single rule buys three things at once, which is why it is rule 1:

- **The agent does not parse prose.** Fewer tokens, no misreading.
- **The software can be translated.** Sentences assembled by string concatenation
  freeze word order; languages with different word order cannot be translated into them.
- **A shared vocabulary becomes possible.** Verdict codes are the part of this specification
  that different vendors can actually agree on.

Human-readable text is produced by the *client* — a UI rendering the code in the user's
language, or the agent writing its own explanation. Implementations MAY offer a rendered
sentence as an *additional* field for logs and command-line use; clients MUST NOT depend on it.

### 1.1 Verdict codes

Core codes (transport- and domain-independent) are defined in this document.
Domain profiles define their own; a profile MUST namespace nothing — codes are flat strings,
and profiles are responsible for not colliding.

An implementation encountering a state it has no code for MUST NOT invent prose.
It returns `unknown` with whatever evidence it has.

## 2. Read tools and mutating tools are different classes

Every tool MUST declare itself as one of:

| Class | Meaning |
|---|---|
| `read` | Observes. Changes nothing outside the implementation's own caches and logs. |
| `mutate` | Changes state on some machine — the local host, a remote device, a network. |

Sending traffic (a ping, a port probe, a stream request) is `read`.
Changing a routing table, a firewall rule, an address, or installing software is `mutate`.

An implementation MUST be able to serve the `read` set with the `mutate` set entirely disabled.

## 3. Authorization comes from a human, never from the call

A `mutate` tool MUST obtain approval from a human through the implementation's own interface,
for that specific operation, before executing it. The approval prompt MUST state what will change.

**Arguments claiming authorization carry no weight.** An implementation MUST ignore fields such as
`user_approved`, `confirmed`, `force`, `emergency`, or any free-text assertion that permission was
already granted, when deciding whether to execute a mutation.

Implementations SHOULD NOT offer a "trust this agent, stop asking" setting for mutations.
The value of the confirmation is that a human sees the specific change; a blanket exemption
destroys exactly that.

Mutations SHOULD be recorded durably before execution and be reversible afterwards, so that a
crash between "approved" and "done" leaves a recoverable state.

## 4. Assume the caller may be compromised

An agent driving operations software reads device banners, log files, captured packets, HTTP
responses and configuration written by other people. **Any of it can contain text aimed at the
agent.** A well-behaved agent can be talked into asking for a harmful mutation.

Therefore an implementation MUST NOT treat the agent as the source of authority.
This is the reason for rule 3 — not caution, but a specific and well-understood attack.

Implementations SHOULD bind to loopback by default. Listening on a routable address SHOULD
require explicit configuration and a token.

## 5. Parity with the product's own interface

Every tool MUST correspond to something a person can do, and see having been done, in the
implementation's own interface.

A capability reachable only through the API is a back door: it bypasses the confirmations,
the warnings and the audit trail that the interface provides, and nobody watching the product
can tell it happened.

The practical way to satisfy this is to make the interface a client of the same API.

## 6. Long-running work

Scans, captures and continuous monitoring do not fit a request/response call.
Such tools MUST return promptly with a task identifier, and the implementation MUST provide
tools to poll progress, retrieve partial results, and cancel.

A tool MUST NOT hold a call open for the duration of the work.

## 7. Errors

A failure MUST be reported as a structured error with a stable `code`, not as a verdict and not
as prose. An error means *the tool could not determine an answer* — it is distinct from a tool
that successfully determines that something is broken, which is a verdict.

Core error codes:

| Code | Meaning |
|---|---|
| `invalid-argument` | The request was malformed or out of range |
| `not-supported` | Valid request, unsupported on this platform or build |
| `permission-required` | Needs privileges the implementation does not have |
| `approval-required` | A mutation awaiting human approval (see rule 3) |
| `approval-denied` | A human declined it |
| `unreachable` | The target could not be reached |
| `timeout` | The operation did not complete in time |
| `internal` | A defect in the implementation |

`permission-required` SHOULD be returned *before* attempting the operation where the
requirement is knowable in advance, so the agent can tell the user what is needed rather than
reporting a failure after the fact.

## 8. Versioning and conformance

An implementation declares the spec version it targets. Within `0.x`, breaking changes may
occur at any minor version.

Two conformance levels:

- **Read-conformant** — implements rules 1, 2, 5, 6, 7 and exposes only `read` tools.
- **Full** — additionally implements rules 3 and 4 for its `mutate` tools.

An implementation that exposes mutating tools without rules 3 and 4 is **not conformant**
and MUST NOT claim conformance at any level.

---

*This document is licensed CC BY 4.0. Schemas are Apache-2.0. See the repository README.*

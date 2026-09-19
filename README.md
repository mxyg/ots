# Ops Tool Spec

**A vocabulary for operations software that AI agents can drive safely.**

> Status: **v0.1 — draft, unstable.** Expect breaking changes. Please try it and tell us where it is wrong.

## What problem this solves

[MCP](https://modelcontextprotocol.io) settled *how* software exposes tools to an AI agent:
transport, discovery, invocation. That part is done, and this spec builds on it rather than competing with it.

What MCP deliberately leaves open is *what* a given kind of software should expose:

- Which tools a network diagnostic tool ought to offer, and what they are called
- What a tool should return so an agent can act on it without parsing prose
- Which operations an agent may perform on its own, and which ones a human must approve
- How to stay safe when the data the agent reads may itself contain instructions

Every vendor answers these differently today. The result is that an agent which can drive one
vendor's tool has to be taught again for the next one. Other domains solved this with a
semantic layer on top of a generic transport — ONVIF over HTTP/SOAP for cameras,
FHIR over HTTP for health records. **This layer does not exist yet for AI-callable operations software.**

## What is in here

| | |
|---|---|
| `spec/v0.1/core.md` | The core rules: return shape, read/mutate split, human approval, safety |
| `schema/` | Machine-readable tool and result schemas |
| Domain profiles | Concrete tool vocabularies per domain. First one: network diagnostics |

## The five rules, in short

1. **Tools return structured verdicts, not sentences.** A verdict is a stable code plus
   parameters. Prose is for the UI to render, in whatever language the user reads.
2. **Read and mutate are separate classes.** Read tools are freely callable.
   Mutating tools change someone's machine and are governed by rule 3.
3. **Authorization comes from a human action, never from a call argument.**
   `"user_approved": true` in a request means nothing.
4. **Assume the caller may be compromised.** Device banners, logs, captured packets and
   web pages the agent reads can contain instructions aimed at the agent. This is the actual
   reason mutations need a human, not mere caution.
5. **No capability exists only in the API.** Anything an agent can do, a person can do and see
   in the product's own interface.

Read `spec/v0.1/core.md` for the detail.

## Why we wrote it

We build operations software — video surveillance, ticketing, broadcast control, and a network
toolkit — and we kept writing the same answers to the questions above, slightly differently each time.
This is those answers, written down once, in the open, so that anyone can implement them,
including our competitors. That is what makes it a specification and not a product feature.

The reference implementation is **NetKit**, a field network toolkit. It exists because on
2026-09-19 an engineer spent a day diagnosing a site by hand — changing NIC addresses, scanning
ports, probing RTSP streams, reading GPU load over SSH — and every one of those steps should
have been a tool an agent could call.

## Licensing — please read this part

| What | License | Why |
|---|---|---|
| Specification text | **CC BY 4.0** (`LICENSE`) | A standard has to be implementable by anyone, **including competitors**. That is what a standard is. |
| Schemas in `schema/` | **Apache-2.0** (`schema/LICENSE`) | So you can copy them straight into your codebase. |

The reference implementation is licensed separately and more restrictively. **That is deliberate
and it does not apply to this specification.** Implement this spec commercially, freely, with no
obligation to us beyond attribution.

## Status and how to help

v0.1 is a draft written alongside a working implementation, not a committee document.
It will change. The most useful thing you can do is implement part of it and tell us what did
not survive contact with your product.

---

Maintained by 辽宁昱弘智能科技有限公司 (Liaoning Yuhox Intelligent Technology Co., Ltd.) ·
[中文版](README.zh-CN.md)

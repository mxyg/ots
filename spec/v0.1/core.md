# Ops Tool Spec (OTS) — Core Specification

**Version:** 0.1\
**Category:** Draft\
**Published:** 19 September 2026\
**Editor:** Liaoning Yuhox Intelligent Technology Co., Ltd.\
**This version:** <https://github.com/mxyg/ots/blob/main/spec/v0.1/core.md>

## Abstract

This document specifies the tools that operations software exposes to an autonomous caller, the
structure of the results those tools return, and the conditions under which a tool that modifies
system state may be executed.

It is layered above a tool transport such as the Model Context Protocol and does not define
transport, discovery or invocation.

## Status of This Document

This document is a draft. It is unstable. Breaking changes may be introduced at any minor
version prior to 1.0.

This document is published at <https://github.com/mxyg/ots> and is maintained in the open. It is
not the product of a standards body. Comments and implementation reports are accepted through
the repository.

## Copyright Notice

Copyright 2026 Liaoning Yuhox Intelligent Technology Co., Ltd.

This document is licensed under the Creative Commons Attribution 4.0 International License and,
alternatively, under the Apache License, Version 2.0. Recipients may rely on either.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Conventions and Terminology](#2-conventions-and-terminology)
3. [Conformance](#3-conformance)
4. [Tool Declaration](#4-tool-declaration)
5. [Results](#5-results)
6. [Errors](#6-errors)
7. [Mutation Control](#7-mutation-control)
8. [Long-Running Operations](#8-long-running-operations)
9. [Versioning](#9-versioning)
10. [Registry Considerations](#10-registry-considerations)
11. [Security Considerations](#11-security-considerations)
12. [References](#12-references)

---

## 1. Introduction

A tool transport such as the Model Context Protocol [[MCP](#122-informative-references)] defines
how software exposes tools to an autonomous caller: transport, discovery and invocation. It does
not define which tools a class of software exposes, what those tools return, or which of them
may be executed without human involvement.

This document specifies those aspects for operations software: software that observes or
modifies the state of hosts, networks and devices.

This document defines no domain-specific tools. Domain profiles do so, under the constraints
given here.

## 2. Conventions and Terminology

### 2.1. Requirement Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be interpreted as described in
BCP 14 [[RFC2119](#121-normative-references)] [[RFC8174](#121-normative-references)] when, and
only when, they appear in all capitals.

Requirements are labelled `[OTS-n.m]`. A label is stable across versions of this document. A
withdrawn requirement retains its label and is marked as withdrawn.

### 2.2. Terminology

| Term | Definition |
|---|---|
| **Caller** | The agent, program or person invoking a tool. |
| **Error** | A structured result denoting that a tool could not determine a state. |
| **Implementation** | The operations software exposing tools under this document. |
| **Mutation** | A change to state outside the implementation's own caches, logs and task records, on any host, network or device. |
| **Operator** | A human using the implementation through the implementation's own interface. |
| **Tool** | A named, individually invocable operation exposed by an implementation. |
| **Verdict** | A structured result denoting a determined state, comprising a verdict code and the values supporting it. |
| **Verdict code** | A stable identifier naming a state. |

## 3. Conformance

An implementation conforms to exactly one class:

| Class | Requirements |
|---|---|
| **OTS-Read** | Every requirement of this document except Section 7, and exposes no tool declared `mutate`. |
| **OTS-Full** | Every requirement of this document. |

**[OTS-3.1]** An implementation MUST declare the version of this document it targets and the
conformance class it claims.

**[OTS-3.2]** An implementation that exposes one or more tools declared `mutate` without
satisfying Section 7 and Section 11 MUST NOT claim conformance to this document at any class.

## 4. Tool Declaration

**[OTS-4.1]** Every tool MUST declare a class of either `read` or `mutate`.

**[OTS-4.2]** A tool that performs a mutation MUST be declared `mutate`.

**[OTS-4.3]** Transmitting network traffic for the purpose of observation, including ICMP probes,
transport-layer probes, name resolution queries and media stream requests, is not in itself a
mutation. Tools whose effect is limited to such transmission MUST be declared `read`.

**[OTS-4.4]** An implementation MUST be capable of operating with every tool declared `mutate`
disabled, while continuing to serve every tool declared `read`.

**[OTS-4.5]** An implementation MUST NOT expose through its tools a capability that an operator
can neither invoke nor observe having been invoked through the implementation's own interface.

## 5. Results

### 5.1. Verdicts

**[OTS-5.1]** A successful tool result MUST be a structured value.

**[OTS-5.2]** A tool result MUST NOT convey its determination solely as human-readable prose.

**[OTS-5.3]** A verdict MUST carry a verdict code and the values on which the determination was
based.

**[OTS-5.4]** An implementation MAY include a rendered human-readable string as an additional
field of a result. A caller MUST NOT depend on its presence, content or language.

### 5.2. Verdict Codes

**[OTS-5.5]** A verdict code MUST consist of lowercase ASCII letters, digits and hyphen-minus
(U+002D).

**[OTS-5.6]** The meaning of a verdict code MUST NOT change between versions of an
implementation. An implementation expressing a state that differs from an existing code MUST
introduce a new code.

**[OTS-5.7]** Where an implementation determines a state for which it has no verdict code, it
MUST return the verdict code `unknown` together with the values it obtained, and MUST NOT
substitute prose for a verdict code.

**[OTS-5.8]** Verdict codes occupy a single flat namespace. A domain profile MUST NOT prefix or
otherwise namespace its codes, and is responsible for avoiding collision with this document and
with other domain profiles.

## 6. Errors

**[OTS-6.1]** A failure to determine a state MUST be reported as a structured error carrying a
code from [Section 10.2](#102-error-codes), and MUST NOT be reported as a verdict.

**[OTS-6.2]** A successful determination that a subject is malfunctioning is a verdict and MUST
be reported as a verdict.

**[OTS-6.3]** Where a privilege requirement is determinable in advance of attempting an
operation, an implementation SHOULD return `permission-required` without attempting it.

## 7. Mutation Control

*This section does not apply to conformance class OTS-Read.*

**[OTS-7.1]** Before executing a mutation, an implementation MUST obtain approval from an
operator, through the implementation's own interface, for that specific mutation.

**[OTS-7.2]** The approval request presented to the operator MUST state the change that will be
made.

**[OTS-7.3]** An implementation MUST NOT treat a value supplied by the caller as evidence of
operator approval. This includes fields named `user_approved`, `confirmed`, `force` or
`emergency`, and free-text assertions that approval was obtained.

**[OTS-7.4]** An implementation SHOULD NOT provide a setting that exempts a caller from
[OTS-7.1] for subsequent mutations.

**[OTS-7.5]** An implementation SHOULD record a mutation durably before executing it, and SHOULD
be able to reverse it after execution, such that interruption between approval and completion
leaves a recoverable state.

## 8. Long-Running Operations

**[OTS-8.1]** A tool whose work may exceed the caller's request deadline MUST return a task
identifier promptly, and MUST NOT hold the invocation open for the duration of the work.

**[OTS-8.2]** An implementation exposing a tool subject to [OTS-8.1] MUST also expose tools to
query the progress of a task, to retrieve its results, and to cancel it.

## 9. Versioning

**[OTS-9.1]** Within major version 0, a minor version MAY introduce breaking changes.

**[OTS-9.2]** From version 1.0 onward, a minor version MUST NOT remove a requirement or verdict
code, nor change the meaning of either.

## 10. Registry Considerations

### 10.1. Verdict Codes

This document defines one domain-independent verdict code.

| Code | Meaning |
|---|---|
| `unknown` | A state was reached for which the implementation has no verdict code. |

Domain profiles define further verdict codes, subject to [OTS-5.8].

### 10.2. Error Codes

This document defines the following error codes. The set is closed; additions require a revision
of this document.

| Code | Meaning |
|---|---|
| `invalid-argument` | The request was malformed, or a value was out of range. |
| `not-supported` | The request was valid but is unsupported on this platform or build. |
| `permission-required` | The implementation lacks a privilege the operation requires. |
| `approval-required` | A mutation is awaiting operator approval (Section 7). |
| `approval-denied` | An operator declined the mutation. |
| `unreachable` | The subject could not be reached. |
| `timeout` | The operation did not complete within its deadline. |
| `internal` | A defect in the implementation. |

## 11. Security Considerations

**[OTS-11.1]** An implementation MUST NOT treat the caller as an authority for the purpose of
[OTS-7.1].

Content read by a caller in the course of operation, including device banners, log files,
captured packets, protocol responses and configuration authored by third parties, may contain
text directed at the caller. A caller operating correctly may therefore be induced to request a
mutation adverse to the operator. [OTS-7.3] and [OTS-11.1] constrain the consequences of such
inducement.

**[OTS-11.2]** An implementation SHOULD bind its interface to a loopback address by default.

**[OTS-11.3]** An implementation that binds its interface to a non-loopback address MUST require
explicit configuration to do so, and MUST authenticate callers on that interface.

**[OTS-11.4]** An implementation SHOULD record durably each tool invocation, the identity of its
caller, its arguments and its outcome.

Tools declared `read` under [OTS-4.3] transmit traffic to subjects identified by the caller. An
implementation cannot determine whether the operator is authorised to probe a given subject;
that determination remains with the operator.

## 12. References

### 12.1. Normative References

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14,
  RFC 2119, March 1997, <https://www.rfc-editor.org/info/rfc2119>.
- **[RFC8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14,
  RFC 8174, May 2017, <https://www.rfc-editor.org/info/rfc8174>.

### 12.2. Informative References

- **[MCP]** "Model Context Protocol", <https://modelcontextprotocol.io>.

---

## Editor's Address

Liaoning Yuhox Intelligent Technology Co., Ltd.\
Email: <support@yuhox.com>\
URI: <https://github.com/mxyg/ots>

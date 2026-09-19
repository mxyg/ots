# Ops Tool Spec (OTS)

A specification for the tools that operations software exposes to an autonomous caller: what
those tools return, and the conditions under which a tool that modifies system state may be
executed.

Layered above a tool transport such as the [Model Context Protocol](https://modelcontextprotocol.io).
It does not define transport, discovery or invocation.

**Version 0.1 — Draft. Unstable.** Breaking changes may be introduced at any minor version
prior to 1.0.

## Contents

| Path | Contents |
|---|---|
| `spec/v0.1/core.md` | Core specification |
| `spec/v0.1/core.zh-CN.md` | Chinese translation (informative) |
| `schema/` | Machine-readable definitions |

English is the normative text. Where a translation differs, the English text governs.

## Scope

The core specification defines:

- Declaration of tools as read or mutating
- The structure of results: verdicts, verdict codes and errors
- The conditions under which a mutation may be executed
- Handling of long-running operations
- Conformance classes and versioning

It defines no domain-specific tools. Domain profiles do so.

## Licence

| Part | Licence |
|---|---|
| Specification text | [CC BY 4.0](LICENSE) and [Apache-2.0](LICENSE-APACHE) — recipients may rely on either |
| `schema/` | [Apache-2.0](schema/LICENSE) |

Both licences permit commercial implementation. Apache-2.0 is offered alongside CC BY 4.0
because it includes an express patent grant and patent-retaliation clause.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Contributions require a Developer Certificate of Origin
sign-off.

This specification may be used, implemented, forked and maintained by anyone, including
commercially. The obligations are those stated in the licences above.

## Provenance

First published 19 September 2026 by 辽宁昱弘智能科技有限公司
(Liaoning Yuhox Intelligent Technology Co., Ltd.). The commit history of this repository is the
record of publication.

Contact: support@yuhox.com

[中文](README.zh-CN.md)

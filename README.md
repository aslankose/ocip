# OCIP — Open Conversation Interchange Protocol

**An open standard for distributed AI inference, peer governance, and conversation portability.**

OCIP is a five-spec family defining the full stack for decentralized AI collaboration — from compute infrastructure at the base to conversation portability at the top. It is vendor-neutral, transport-agnostic, and designed to be implementable by any team in any language.

> **Reference implementation:** [Inferix](https://github.com/aslankose/inferix) — a decentralized AI compute cooperative built on the OCIP standard.

---

## The Spec Family

Each spec depends only on lower-numbered specs. Read from bottom to top to follow the dependency order.

| Spec | Title | Layer | Status |
|---|---|---|---|
| [OCIP-0001](spec/OCIP-0001.md) | Distributed Inference Protocol | Compute foundation | Draft 1 |
| [OCIP-0002](spec/OCIP-0002.md) | Peer Governance and Capability Handshake | Trust and discovery | Draft 1 |
| [OCIP-0003](spec/OCIP-0003.md) | Group Session Layer | Collaborative work | Draft 1 |
| [OCIP-0004](spec/OCIP-0004.md) | Project Layer | Multi-conversation organization | Draft 1 |
| [OCIP-0005](spec/OCIP-0005.md) | Open Conversation Interchange Protocol | Conversation portability | Draft 1 |

---

## What Each Spec Covers

### [OCIP-0001 — Distributed Inference Protocol](spec/OCIP-0001.md)
The compute foundation. Defines how contributor nodes register hardware, declare transformer layer shards, form inference pipelines, pass activations between nodes, respond to reliability challenges, and earn GigaFLOP-Tokens (GFT). The coordination API is standardized HTTP/JSON. Activation passing between shard nodes is transport-agnostic — implementations may use HTTP, gRPC, WebSocket, or any other transport.

### [OCIP-0002 — Peer Governance and Capability Handshake](spec/OCIP-0002.md)
The trust layer. Defines how nodes discover each other via a structured capability taxonomy, how two parties run an optional mutual challenge-and-evaluation handshake, and how a formalized `CollaborationAgreement` is produced when both parties independently accept. Challenges are quota-controlled and sovereign — each party evaluates responses against their own expectations with no central judge. Supports three governance modes: anonymous contribution, voluntary handshake, and institution-governed handshake.

### [OCIP-0003 — Group Session Layer](spec/OCIP-0003.md)
The collaboration layer. Defines multi-participant, multi-platform conversation sessions where each participant uses their own AI model. Supports async and live sync modes. Turns are attributed to participants, globally sequenced, and optionally carry settled inference cost records (`InferenceCost`) linked to the OCIP-0001 token ledger via `ledger_ref`. Sessions may reference an OCIP-0002 `CollaborationAgreement` to declare their governance basis.

### [OCIP-0004 — Project Layer](spec/OCIP-0004.md)
The organization layer. Defines a portable container grouping related conversations, system instructions, and file attachments into a single transferable unit. Instructions support both single-block and multi-block forms. Attachments use a tiered inline/reference model with mandatory checksums. Projects may reference an OCIP-0002 `CollaborationAgreement` via `project.metadata`.

### [OCIP-0005 — Open Conversation Interchange Protocol](spec/OCIP-0005.md)
The portability layer. Defines a single portable conversation object that can be moved between AI platforms mid-conversation, shared between users on different platforms, or handed off between models. The `Handoff` object carries not just what was said but why the switch happened and what the receiving model needs next. The `ContextSnapshot` orients the receiving model without requiring it to reprocess the full history.

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│  OCIP-0005: Conversation Portability             │  ← Single conversation, model handoff
├─────────────────────────────────────────────────┤
│  OCIP-0004: Project Layer                        │  ← Multi-conversation, attachments
├─────────────────────────────────────────────────┤
│  OCIP-0003: Group Session Layer                  │  ← Multi-participant, inference cost
├─────────────────────────────────────────────────┤
│  OCIP-0002: Peer Governance & Handshake          │  ← Discovery, challenges, agreements
├─────────────────────────────────────────────────┤
│  OCIP-0001: Distributed Inference Protocol       │  ← Compute, tokens, pipelines
└─────────────────────────────────────────────────┘
```

---

## Key Design Decisions

**Transport agnosticism.** OCIP-0001 defines the message contract for activation passing between shard nodes but leaves transport to the implementation. HTTP/JSON is the mandatory baseline. gRPC, WebSocket, and other transports are declared and negotiated per pipeline.

**Sovereign evaluation.** OCIP-0002 capability challenges have no central judge. Each party evaluates responses against their own expectations and signals accept or reject independently. The protocol does not define what "good enough" means — that is each party's decision.

**Settled costs only.** OCIP-0003 `InferenceCost` records are only written after the OCIP-0001 token ledger confirms a transaction. Pending or estimated costs never appear in session records.

**Optional governance.** Anonymous compute contribution (OCIP-0001) requires no identity, no declaration, and no agreement. OCIP-0002 governance is a layer on top — opt-in for individuals, enforceable at institutional boundaries.

**Taxonomy extensibility.** The OCIP-0002 capability taxonomy is structured (two-level domain/subdomain) but open. Unknown domain and subdomain values MUST be accepted by conforming parsers. The taxonomy grows through community contribution.

---

## Relationship to Inferix

[Inferix](https://github.com/aslankose/inferix) is the reference implementation of the OCIP standard. It implements:

- **OCIP-0001** — the coordination API, shard registry, pipeline scheduler, reliability challenge system, and GFT token ledger
- **OCIP-0002** — the discovery index integrated into the Inferix coordination layer
- **OCIP-0003** — group session support with Inferix-settled `InferenceCost` records
- **OCIP-0004** — project portability for Inferix-hosted collaborative work
- **OCIP-0005** — conversation export and import for Inferix session history

OCIP is an independent open standard. Inferix is one implementation. Any team may implement OCIP independently.

---

## Relationship to Existing Standards

| Standard | Relationship |
|---|---|
| **MCP** (Anthropic) | Complementary. MCP connects AI to external tools. OCIP moves conversations and compute context between humans and platforms. |
| **A2A** (Google) | Complementary. A2A is agent-to-agent communication. OCIP is human-to-human and human-to-platform. |
| **MBOX** | Inspirational. OCIP-0005 aims to be for AI conversations what MBOX is for email archives. |
| **iCalendar (RFC 5545)** | Structural inspiration. A single portable object, transport-agnostic, widely adopted via community standardization. |
| **SETI@home / BOINC** | Conceptual inspiration for OCIP-0001 volunteer compute model. |

---

## File Format

All OCIP documents are UTF-8 encoded JSON (RFC 8259). Each document includes an `ocip_spec` field identifying which spec it conforms to.

| ocip_spec | Document type | Extension |
|---|---|---|
| `"OCIP-0001"` | Not applicable (API protocol, not a document format) | — |
| `"OCIP-0002"` | Discovery record or collaboration agreement | `.ocip2-discovery.json`, `.ocip2-agreement.json` |
| `"OCIP-0003"` | Group session | `.ocip3.json` |
| `"OCIP-0004"` | Project | `.ocip4.json` |
| `"OCIP-0005"` | Conversation | `.ocip5.json` or `.ocip.json` |

---

## Conformance Levels

Implementations may conform to any subset of the spec family. Conformance to a higher-numbered spec does not require conformance to all lower-numbered specs — only to those the higher spec explicitly depends on.

| Implementation type | Required specs |
|---|---|
| Compute node only | OCIP-0001 |
| Governed compute node | OCIP-0001, OCIP-0002 |
| Collaborative session platform | OCIP-0001, OCIP-0002, OCIP-0003 |
| Full project platform | OCIP-0001 through OCIP-0004 |
| Full stack | OCIP-0001 through OCIP-0005 |

---

## Contributing

OCIP is an open standard. Contributions are welcome via GitHub Issues and Pull Requests.

**Read the specs first:**
- [`spec/OCIP-0001.md`](spec/OCIP-0001.md) — Distributed Inference Protocol
- [`spec/OCIP-0002.md`](spec/OCIP-0002.md) — Peer Governance and Capability Handshake
- [`spec/OCIP-0003.md`](spec/OCIP-0003.md) — Group Session Layer
- [`spec/OCIP-0004.md`](spec/OCIP-0004.md) — Project Layer
- [`spec/OCIP-0005.md`](spec/OCIP-0005.md) — Open Conversation Interchange Protocol

**Open areas for contribution:**

- JSON Schema files for all five specs (`schemas/`)
- Reference node implementation (Python, Go, Rust) for OCIP-0001
- Platform converters (ChatGPT → OCIP-0005, Gemini → OCIP-0005, Claude → OCIP-0005)
- Reference WebSocket host for OCIP-0003 live sync
- OCIP-0002 capability taxonomy extension proposals (new domains and sub-domains)
- Inferix discovery index integration for OCIP-0002
- Cryptographic signing reference implementation for OCIP-0002
- FLOPs measurement reference implementation for OCIP-0001
- Federated coordinator protocol (multi-coordinator federation) for OCIP-0001
- Governance policy templates for institutions implementing OCIP-0002 Mode 3

---

## License

Apache 2.0. See [LICENSE](LICENSE).

---

## Author

**Aslan Kose**  
[github.com/aslankose](https://github.com/aslankose)

---

*OCIP is an independent open standard. It is not affiliated with Anthropic, OpenAI, Google, or any AI platform vendor.*

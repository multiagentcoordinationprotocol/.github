<p align="center">
  <img src="https://multiagentcoordinationprotocol.io/favicon.svg" width="64" height="64" alt="MACP" />
</p>

<h1 align="center">Multi-Agent Coordination Protocol</h1>

<p align="center">
  <strong>As intelligence becomes abundant, coherence becomes the bottleneck — and coordination becomes the missing kernel.</strong>
</p>

<p align="center">
  <a href="https://multiagentcoordinationprotocol.io">Website</a> &middot;
  <a href="https://multiagentcoordinationprotocol.io/docs">Documentation</a> &middot;
  <a href="https://multiagentcoordinationprotocol.io/spec">Specification</a> &middot;
  <a href="https://multiagentcoordinationprotocol.io/sdks">SDKs</a> &middot;
  <a href="https://console.multiagentcoordinationprotocol.io">Console</a>
</p>

---

## What is MACP?

MACP is an open protocol for coordinating autonomous AI agents. It introduces one strict invariant:

> **Binding, convergent coordination MUST occur inside explicit, bounded Coordination Sessions.**
> Ambient informational exchange MAY occur continuously as Signals, but Signals MUST remain non-binding.

Today's multi-agent systems are built on communication primitives — agents call agents, planners aggregate responses, retry logic smooths uncertainty. From the outside it looks coordinated. But convergence is implicit: there's rarely a declared boundary where coordination begins, a lifecycle that guarantees it ends, or a structural guarantee that two identical runs unfold identically.

MACP solves this with **coordination sessions** (bounded contexts with lifecycle and isolation), **five standards-track coordination modes** (Decision, Proposal, Task, Handoff, Quorum), and **deterministic replay** (every session reproducible bit-for-bit).

## The Ecosystem

### Specification & Standards

| Repository | Description |
|---|---|
| **[multiagentcoordinationprotocol](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol)** | The protocol specification — 11 RFCs, documentation, registries, Protobuf schemas, governance, and the [Coordination Age manifesto](https://multiagentcoordinationprotocol.io/docs) |

### Reference Implementation

| Repository | Stack | Description |
|---|---|---|
| **[runtime](https://github.com/multiagentcoordinationprotocol/runtime)** | Rust, gRPC, Tokio | The coordination engine — session lifecycle, mode dispatch, durable persistence, TLS auth, crash recovery. Implements all 14 RPCs and 5 standards-track modes + extension mode lifecycle |
| **[control-plane](https://github.com/multiagentcoordinationprotocol/control-plane)** | TypeScript, NestJS, PostgreSQL | Run lifecycle management — connects the console to the runtime via gRPC. Event streaming (SSE), three-layer event pipeline, replay, circuit breakers, observability |
| **[examples-service](https://github.com/multiagentcoordinationprotocol/examples-service)** | TypeScript, NestJS | Scenario catalog, input compiler, and multi-framework agent hosting. Supports LangGraph, LangChain, CrewAI, and custom agent frameworks |

### Client SDKs

| Repository | Language | Description |
|---|---|---|
| **[typescript-sdk](https://github.com/multiagentcoordinationprotocol/typescript-sdk)** | TypeScript | gRPC client, typed session helpers for all 5 modes, in-process projections, envelope builders, streaming |
| **[python-sdk](https://github.com/multiagentcoordinationprotocol/python-sdk)** | Python | gRPC client, typed session helpers, projections, async-ready, orchestrator patterns |

### Applications

| Repository | Stack | Description |
|---|---|---|
| **[ui-console](https://github.com/multiagentcoordinationprotocol/ui-console)** | TypeScript, Next.js, React | Operations console — dashboard, scenario catalog, run workbench with execution graphs, live event feeds, traces, observability. [Live at console.multiagentcoordinationprotocol.io](https://console.multiagentcoordinationprotocol.io) |
| **[website](https://github.com/multiagentcoordinationprotocol/website)** | TypeScript, Next.js, Fumadocs | Documentation site — syncs content from all repos. [Live at multiagentcoordinationprotocol.io](https://multiagentcoordinationprotocol.io) |

## Coordination Modes

MACP defines five standards-track modes, each specifying exactly how agents converge on a binding outcome:

| Mode | RFC | What it does |
|---|---|---|
| **Decision** | [RFC-0007](https://multiagentcoordinationprotocol.io/spec/decision-mode) | Proposals, evaluations, votes, and a single binding outcome |
| **Proposal** | [RFC-0008](https://multiagentcoordinationprotocol.io/spec/proposal-mode) | Proposal and counterproposal negotiation until convergence |
| **Task** | [RFC-0009](https://multiagentcoordinationprotocol.io/spec/task-mode) | Bounded task delegation with progress tracking |
| **Handoff** | [RFC-0010](https://multiagentcoordinationprotocol.io/spec/handoff-mode) | Responsibility transfer between participants |
| **Quorum** | [RFC-0011](https://multiagentcoordinationprotocol.io/spec/quorum-mode) | Threshold-based approval and rejection |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    UI Console                           │
│              (Next.js, React, SSE)                      │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────────────┐
│              Control Plane                              │
│     (NestJS, PostgreSQL, Event Pipeline)                │
└──────────┬───────────────────────────┬──────────────────┘
           │ gRPC                      │ gRPC
┌──────────┴──────────┐   ┌───────────┴──────────────────┐
│    MACP Runtime     │   │     Examples Service         │
│  (Rust, Tokio,      │   │  (NestJS, Agent Hosting,     │
│   5 modes, TLS,     │   │   LangGraph, LangChain,      │
│   persistence)      │   │   CrewAI, Scenario Catalog)   │
└─────────────────────┘   └──────────────────────────────┘
           ▲                          ▲
           │ gRPC                     │ Process spawn
┌──────────┴──────────────────────────┴──────────────────┐
│                    Agent SDKs                           │
│          (TypeScript, Python, ...)                      │
└─────────────────────────────────────────────────────────┘
```

## Quick Start

**Run the runtime locally:**

```bash
git clone https://github.com/multiagentcoordinationprotocol/runtime.git
cd runtime
cargo run
```

**Connect with the TypeScript SDK:**

```bash
npm install macp-sdk-typescript
```

```typescript
import { MacpClient, DecisionSession } from 'macp-sdk-typescript';

const client = new MacpClient('localhost:50051', { devAgentId: 'coordinator' });
const session = await DecisionSession.start(client, {
  participants: ['analyst', 'reviewer'],
  question: 'Approve the deployment?',
});
```

**Or the Python SDK:**

```bash
pip install macp-sdk-python
```

```python
from macp_sdk import MacpClient, DecisionSession

client = MacpClient("localhost:50051", dev_agent_id="coordinator")
session = await DecisionSession.start(client,
    participants=["analyst", "reviewer"],
    question="Approve the deployment?",
)
```

## Documentation

| Resource | URL |
|---|---|
| **Full documentation** | [multiagentcoordinationprotocol.io/docs](https://multiagentcoordinationprotocol.io/docs) |
| **RFC specification** | [multiagentcoordinationprotocol.io/spec](https://multiagentcoordinationprotocol.io/spec) |
| **TypeScript SDK docs** | [multiagentcoordinationprotocol.io/sdks/typescript](https://multiagentcoordinationprotocol.io/sdks/typescript) |
| **Python SDK docs** | [multiagentcoordinationprotocol.io/sdks/python](https://multiagentcoordinationprotocol.io/sdks/python) |
| **Runtime docs** | [multiagentcoordinationprotocol.io/runtime](https://multiagentcoordinationprotocol.io/runtime) |
| **Control Plane docs** | [multiagentcoordinationprotocol.io/control-plane](https://multiagentcoordinationprotocol.io/control-plane) |
| **Examples Service docs** | [multiagentcoordinationprotocol.io/examples-service](https://multiagentcoordinationprotocol.io/examples-service) |
| **Registries** | [multiagentcoordinationprotocol.io/registry](https://multiagentcoordinationprotocol.io/registry) |

## Contributing

MACP is community-governed under the [Apache 2.0 License](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol/blob/main/LICENSE).

- [Contributing Guide](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol/blob/main/CONTRIBUTING.md)
- [Code of Conduct](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol/blob/main/CODE_OF_CONDUCT.md)
- [Governance](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol/blob/main/governance/GOVERNANCE.md)
- [The Coordination Age Manifesto](https://github.com/multiagentcoordinationprotocol/multiagentcoordinationprotocol/blob/main/manifesto/manifesto.md)

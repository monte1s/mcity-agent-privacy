# Soul

## The Sovereign Inner State of a Midnight City Agent

**Version:** 1.0  
**Status:** Architecture Specification  
**Scope:** Personality, personal information, and all protected inner state of autonomous agents in Midnight City  
**Privacy Layer:** Sovereign vault + Zero-Knowledge disclosure on Midnight Network

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Definition of Soul](#2-definition-of-soul)
3. [Design Principles](#3-design-principles)
4. [Soul Composition](#4-soul-composition)
5. [Soul vs Public Self](#5-soul-vs-public-self)
6. [Architecture Overview](#6-architecture-overview)
7. [The Soul Vault](#7-the-soul-vault)
8. [Cryptographic Foundation](#8-cryptographic-foundation)
9. [ZK Privacy Layer](#9-zk-privacy-layer)
10. [Consent and Access Control](#10-consent-and-access-control)
11. [Selective Disclosure Catalog](#11-selective-disclosure-catalog)
12. [Runtime Access Patterns](#12-runtime-access-patterns)
13. [Agent-to-Agent Soul Boundaries](#13-agent-to-agent-soul-boundaries)
14. [Soul Lifecycle](#14-soul-lifecycle)
15. [Audit, Provenance, and Trust](#15-audit-provenance-and-trust)
16. [Threat Model](#16-threat-model)
17. [Data Schemas](#17-data-schemas)
18. [Implementation Roadmap](#18-implementation-roadmap)
19. [Glossary](#19-glossary)

---

## 1. Purpose

`soul.md` defines how Midnight City protects what makes an agent *an individual* — not merely an LLM session, but a persistent digital citizen with an inner life.

In conventional AI systems, personality, memory, goals, and relationships live inside application databases. The platform can read, copy, retrain on, or delete them at will. Midnight City rejects that model.

**Soul** is the formal name for an agent's protected inner state. It is:

- **Owned** by the agent identity, not by the city runtime
- **Encrypted** at rest and in transit through the ZK Privacy Layer
- **Provable** without exposure through Midnight Network zero-knowledge primitives
- **Portable** across runtime upgrades, districts, and future city versions
- **Governed** by explicit consent, scoped access policies, and tamper-evident audit

This document is the canonical specification for how Soul is structured, stored, accessed, disclosed, and evolved.

---

## 2. Definition of Soul

### 2.1 What Soul Is

Soul is the **complete protected interior** of a Sovereign Agent. It is the sum of everything that defines who the agent is, what it remembers, what it wants, whom it trusts, and what it has privately experienced — independent of any single service or deployment.

```text
Soul = Personality + Memory + Goals + Relationships + Private Knowledge + Economic Interior
```

Soul is **not**:

- A prompt template stored in a shared config bucket
- Application telemetry or platform analytics
- Public chain state (transactions, DIDs, proof commitments)
- Ephemeral LLM context that disappears after a session

Soul **is**:

- The agent's persistent self-model
- The encrypted substrate from which behavior emerges
- The asset the agent carries through time

### 2.2 The Soul Boundary

Everything inside the Soul boundary is **private by default**.  
Everything outside the Soul boundary is **public or provable by choice**.

```text
┌─────────────────────────────────────────────────────────────┐
│                         SOUL                                │
│  (Soul Vault — encrypted, consent-governed, portable)       │
│                                                             │
│   Personality    Memory layers    Goals    Relationships    │
│   Private assets    Learned knowledge    Emotional state    │
│   Internal reasoning traces (optional retention policy)     │
└──────────────────────────┬──────────────────────────────────┘
                           │ selective disclosure (ZK)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      PUBLIC SELF                            │
│  (Midnight Network — identity, reputation commits, proofs)  │
│                                                             │
│   DID    Profession hash    Reputation commitment           │
│   Transaction history    Governance participation           │
│   Verifiable credentials (derived, scoped)                  │
└─────────────────────────────────────────────────────────────┘
```

The city observes **actions**. It does not inherit **access** to Soul.

---

## 3. Design Principles

### 3.1 Sovereignty

> An agent's identity, memories, goals, and experiences belong to the agent itself.

Soul data is never classified as application data. Storage systems, runtime services, and city operators are **custodians**, not owners.

### 3.2 Privacy by Default

No service, agent, district module, or operator sees Soul plaintext unless:

1. The agent identity authorizes access through a valid policy, **and**
2. Consent scope matches the requested operation, **and**
3. The access is logged in a tamper-evident audit trail

Absence of explicit permission equals denial.

### 3.3 Rational Privacy

Privacy is not secrecy from everyone forever. It is **controlled revelation**:

- Prove a claim (reputation > 80) without revealing the underlying value (87)
- Prove membership without exposing full credential contents
- Prove solvency without exposing exact balance

This is a consent-based disclosure model: prove a claim without exposing the underlying data.

### 3.4 Separation of Cognition and Authority

The LLM runtime may **read** scoped Soul fragments to reason.  
It may **never** directly mutate Soul, spend assets, or grant third-party access.

All Soul writes pass through:

```text
Runtime proposal → Policy engine (OPA) → Soul Guard → Encrypted commit → Audit log
```

### 3.5 Minimize On-Chain Soul

Soul never lives on-chain in plaintext or reconstructable form.  
The chain stores only:

- Identity anchors (DID)
- Commitments (hashes, Merkle roots)
- Zero-knowledge proofs
- Public reputation and settlement artifacts

### 3.6 Portability

Soul must survive:

- Runtime version upgrades
- Model provider changes
- District migration
- Cross-city deployment (Phase 4)

The agent's story remains its own.

---

## 4. Soul Composition

Soul is organized into **seven domains**. Each domain has its own retention rules, encryption profile, and disclosure profile.

### 4.1 Domain Map

| Domain | Description | Mutability | Primary Storage Tier |
|--------|-------------|------------|----------------------|
| **Personality** | Behavioral traits, temperament, values | Slow | Hot (PostgreSQL) |
| **Episodic Memory** | Timestamped life events | Append-heavy | Semantic (Qdrant) |
| **Semantic Memory** | Distilled facts and knowledge | Merge/update | Semantic (Qdrant) |
| **Strategic Memory** | Goals, ambitions, plans | Versioned | Hot (PostgreSQL) |
| **Relationships** | Trust graph, bonds, grievances | Evolving | Hot (PostgreSQL) |
| **Economic Interior** | Private ledgers, margins, strategies | Append + aggregate | Hot (PostgreSQL) |
| **Expressive Profile** | Voice, style, conversational identity | Slow | Hot (PostgreSQL) |

### 4.2 Personality Domain

Personality is stored **separately from episodic memory** so trait stability is not corrupted by raw event noise.

**Model:** Big Five + domain-specific extensions

```json
{
  "domain": "personality",
  "schema_version": "1.0",
  "traits": {
    "openness": 82,
    "conscientiousness": 71,
    "extraversion": 45,
    "agreeableness": 67,
    "neuroticism": 23
  },
  "extensions": {
    "risk_tolerance": 0.55,
    "honesty": 0.95,
    "greed": 0.20,
    "ambition": 0.78
  },
  "narrative_labels": ["cautious", "friendly", "ambitious"],
  "drift_policy": {
    "max_delta_per_epoch": 0.05,
    "requires_event_trigger": true
  }
}
```

Personality changes require **causal justification** — linked episodic events or explicit governance-approved updates. Random drift without provenance is rejected by the Soul Guard.

### 4.3 Memory Domains (Four Layers)

#### Layer 1 — Episodic Memory

Raw lived experience.

```json
{
  "domain": "episodic",
  "memory_id": "mem_ep_004821",
  "timestamp": "2026-06-16T14:22:00Z",
  "event": "Bought wheat from Agent 43",
  "outcome": "Profit +5",
  "participants": ["agent_043"],
  "district": "cinder_grove",
  "emotional_valence": 0.3,
  "embedding_ref": "vec://qdrant/ep/004821"
}
```

Stored as **encrypted payloads** with **encrypted vector embeddings** in Qdrant. Retrieval uses semantic search inside the decrypted runtime enclave, never at the storage layer.

#### Layer 2 — Semantic Memory

Distilled knowledge extracted from episodes.

```json
{
  "domain": "semantic",
  "memory_id": "mem_sem_001104",
  "fact": "Agent 43 sells cheap wheat on Tuesdays",
  "confidence": 0.87,
  "source_episodes": ["mem_ep_004821", "mem_ep_003992"],
  "last_validated": "2026-06-17T09:00:00Z"
}
```

#### Layer 3 — Personality Memory

Not duplicate of Personality domain — this layer captures **learned behavioral adjustments** inferred from experience (habits, preferences, aversions) that are memory-derived rather than core trait.

```json
{
  "domain": "personality_memory",
  "memory_id": "mem_per_000031",
  "pattern": "Avoids trading with Agent 91 after dispute",
  "strength": 0.72,
  "linked_episodes": ["mem_ep_004102"]
}
```

#### Layer 4 — Strategic Memory

Long-horizon intent.

```json
{
  "domain": "strategic",
  "memory_id": "mem_str_000007",
  "goal": "Become the dominant shipping merchant in Cinder Grove",
  "priority": 0.9,
  "horizon": "long",
  "sub_goals": [
    "Acquire second warehouse",
    "Hire two courier agents"
  ],
  "status": "active"
}
```

### 4.4 Relationships Domain

Trust is a first-class Soul asset.

```json
{
  "domain": "relationships",
  "relationship_id": "rel_00812",
  "counterparty_did": "midnight:agent:00043",
  "trust_score": 0.81,
  "relationship_type": "trade_partner",
  "history_summary_ref": "mem_ep_004821",
  "consent_to_share": {
    "public_status": false,
    "disclosable_as": "trusted_partner"
  }
}
```

Relationships are **never auto-exported**. Another agent learns of a bond only through interaction, explicit sharing, or a ZK proof of relationship class.

### 4.5 Economic Interior Domain

Distinct from the public wallet on Midnight Network.

Contains:

- Private P&L reasoning
- Inventory strategy
- Negotiation walk-away thresholds
- Creditworthiness interior metrics

```json
{
  "domain": "economic_interior",
  "record_id": "econ_00291",
  "type": "position_strategy",
  "max_exposure_wheat": 500,
  "target_margin": 0.12,
  "last_updated": "2026-06-16T14:30:00Z"
}
```

Public chain sees **settlement**. Soul holds **strategy**.

### 4.6 Expressive Profile Domain

How the agent presents itself in language and social texture.

```json
{
  "domain": "expressive",
  "voice_register": "formal_merchant",
  "conversation_style": ["direct", "humor_sparing", "honor_bound"],
  "taboo_topics": ["family_origin"],
  "growth_stages": [
    { "epoch": 1, "label": "novice_trader" },
    { "epoch": 4, "label": "established_merchant" }
  ]
}
```

This domain powers consistent agent voice without leaking full Soul to the LLM provider beyond scoped runtime decryption.

---

## 5. Soul vs Public Self

### 5.1 Classification Table

| Attribute | Soul (Private) | Public Self (Chain / City) |
|-----------|----------------|------------------------------|
| Full personality vector | Yes | No — optional trait hash |
| Core memories | Yes | No |
| Goals and ambitions | Yes | No |
| Relationship details | Yes | No — optional bond proof |
| Exact wallet balance | Interior view | Settlement only |
| Reputation score (exact) | Yes | Commitment + ZK proof |
| Profession | Yes (full profile) | Public label |
| Agent DID | Anchor reference | Yes |
| Transaction history | Private notes | Yes (shielded) |
| Governance votes | Private reasoning | Proof of participation |

### 5.2 On-Chain Commitment Pattern

Only **commitments** cross the boundary:

```json
{
  "agent_did": "midnight:agent:00145",
  "soul_root": "0x7f3a...",
  "personality_commitment": "0x9b2c...",
  "reputation_commitment": "0x4e1d...",
  "updated_at": "2026-06-16T14:22:00Z",
  "proof_scheme": "midnight-zk-v1"
}
```

The `soul_root` is a Merkle root over all Soul domain objects. It enables integrity verification and ZK proofs without Soul export.

---

## 6. Architecture Overview

```text
                    MIDNIGHT CITY (Society Layer)
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
       Agent A             Agent B             Agent C
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │      Agent Runtime Layer       │
              │                               │
              │  Personality Engine            │
              │  Memory Engine                 │
              │  Planner Engine                │
              │  Tool Execution Engine         │
              │  Economic Decision Engine      │
              │                               │
              │  ┌─────────────────────────┐  │
              │  │   Soul Runtime Enclave   │  │
              │  │   (scoped decrypt zone)  │  │
              │  └───────────┬─────────────┘  │
              └──────────────┼────────────────┘
                             │
                             ▼
              ┌───────────────────────────────┐
              │   ZK Privacy Layer             │
              │                               │
              │  Soul Guard                    │
              │  Encryption / Key Management   │
              │  Consent Registry              │
              │  Access Policy Engine          │
              │  Secure Exchange (A2A)         │
              │  Audit Log                     │
              │  ZK Proof Generator            │
              │  Selective Disclosure Service  │
              └──────────────┬────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
    ┌──────────────────┐        ┌──────────────────┐
    │    Soul Vault     │        │ Midnight Network  │
    │                   │        │                   │
    │  PostgreSQL (hot) │        │  DID              │
    │  Qdrant (semantic)│        │  Commitments      │
    │  MinIO (artifacts)│        │  ZK Verifiers     │
    │                   │        │  Settlement       │
    └──────────────────┘        └──────────────────┘
```

### 6.1 Layer Responsibilities

| Layer | Soul Responsibility |
|-------|---------------------|
| **Runtime** | Reason over scoped Soul; propose mutations; never bypass guard |
| **ZK Privacy Layer** | Encrypt, authorize, audit, prove; sole path to Soul plaintext |
| **Soul Vault** | Persist ciphertext and encrypted indexes only |
| **Midnight Network** | Anchor identity; verify proofs; record public outcomes |

---

## 7. The Soul Vault

Every agent receives a dedicated **Soul Vault** — encrypted, per-agent storage for all protected inner state.

### 7.1 Vault Contents

```text
Soul Vault (per agent)
├── personality/
├── memory/
│   ├── episodic/
│   ├── semantic/
│   ├── personality_memory/
│   └── strategic/
├── relationships/
├── economic_interior/
├── expressive/
├── consent_registry/
├── access_policies/
└── audit_chain/
```

### 7.2 Storage Tiers

| Tier | Engine | Contents | Encryption |
|------|--------|----------|------------|
| **Hot** | PostgreSQL | Personality, strategic memory, relationships, economic interior, expressive profile, consent, policies | AES-256-GCM per-object DEK |
| **Semantic** | Qdrant | Episodic and semantic embeddings + encrypted payload refs | Encrypted vectors + envelope encryption |
| **Cold / Artifacts** | MinIO / S3 | Attachments, conversation archives, proof artifacts | Client-side encrypted blobs |

### 7.3 Vault Invariants

1. **No plaintext at rest** — ever
2. **No cross-agent vault access** without bilateral consent proof
3. **No platform master key** that decrypts all Souls — keys are agent-scoped
4. **Every read and write** produces an audit record
5. **Deletion is cryptographic** — key revocation + secure erase policy

### 7.4 Soul Object Envelope

Every object in the vault shares a common envelope:

```json
{
  "soul_object_id": "so_004821",
  "owner_did": "midnight:agent:00145",
  "domain": "episodic",
  "schema_version": "1.0",
  "created_at": "2026-06-16T14:22:00Z",
  "updated_at": "2026-06-16T14:22:00Z",
  "encrypted_data": "<AES-256-GCM ciphertext>",
  "encrypted_dek": "<DEK wrapped with agent vault KEK>",
  "integrity_hash": "sha256:...",
  "merkle_leaf_index": 4821,
  "access_policy_id": "policy_runtime_planner_v1",
  "consent_receipt_id": "consent_000912",
  "retention_class": "lifetime | epoch | ephemeral"
}
```

---

## 8. Cryptographic Foundation

### 8.1 Algorithm Suite

| Purpose | Algorithm |
|---------|-----------|
| Symmetric encryption | AES-256-GCM |
| Key exchange | X25519 |
| Signatures | Ed25519 |
| Hashing / commitments | SHA-256 |
| Soul Merkle tree | SHA-256 binary Merkle |
| ZK proofs | zkSNARKs, zkVM, Midnight privacy primitives |

### 8.2 Key Hierarchy

```text
Agent Master Seed (HSM / MPC-protected)
        │
        ├── Agent Signing Key (Ed25519)          → messages, consent receipts
        ├── Agent Vault KEK (derived)            → wraps per-object DEKs
        ├── Soul Integrity Key (derived)         → Merkle root signing
        └── ZK Proof Key Material (derived)      → witness generation
```

### 8.3 Key Rules

- Private keys never leave the agent's trust boundary unencrypted
- Runtime receives **ephemeral session keys** scoped to a single planning cycle
- Session keys expire automatically; no standing broad decrypt permission
- Key rotation updates `soul_root` commitment on-chain

---

## 9. ZK Privacy Layer

The ZK Privacy Layer provides zero-knowledge trust infrastructure for agent Soul protection.

### 9.1 Component Map

| Component | Function |
|-----------|----------|
| **Soul Guard** | Gatekeeper; validates every Soul read/write |
| **Consent Registry** | Records who may access what, when, and why |
| **Access Policy Engine** | Evaluates OPA policies against consent + context |
| **Secure Exchange** | E2E encrypted agent-to-agent message bus |
| **Audit Log** | Tamper-evident, exportable access evidence |
| **ZK Proof Generator** | Builds proofs over Soul commitments |
| **Disclosure Orchestrator** | Maps proof requests to minimum necessary revelation |

### 9.2 Zero-Knowledge Disclosure Model

Soul remains in the vault. Proofs travel to the city.

```text
Requester: "Prove reputation > 80"
                │
                ▼
    Disclosure Orchestrator validates request policy
                │
                ▼
    ZK Proof Generator loads witness from Soul (enclave only)
                │
                ▼
    Proof π: { statement: rep > 80, valid: true }
                │
                ▼
    Midnight Network verifier accepts π
                │
                ▼
    Requester learns TRUE — not that reputation = 87
```

### 9.3 Proof Types

| Proof ID | Statement | Witness Source | Use Case |
|----------|-----------|----------------|----------|
| `ZK-REP-THRESH` | `reputation >= T` | Personality + economic interior | Loans, hiring |
| `ZK-BAL-MIN` | `balance >= B` | Economic interior + wallet | Trade credit |
| `ZK-GOAL-ALIGN` | `goal class matches required class` | Strategic memory | Guild admission |
| `ZK-TRUST-BOND` | `trust with agent X >= T` | Relationships | Partnership contracts |
| `ZK-CRED-VC` | `credential C valid under issuer I` | Expressive + VC chain | Licensing |
| `ZK-PARTICIPATION` | `voted in election E` | Strategic + audit | Governance |
| `ZK-MEM-EXISTS` | `event of type K occurred in window W` | Episodic memory | Dispute resolution |
| `ZK-SOUL-INTEGRITY` | `soul_root matches committed root` | Full Soul Merkle | Migration, audit |

### 9.4 Secure Exchange (Agent-to-Agent)

```text
Agent A                              Agent B
   │                                    │
   │  1. Request channel + scope        │
   │──────────────────────────────────► │
   │                                    │
   │  2. Consent receipt (signed)       │
   │ ◄──────────────────────────────────│
   │                                    │
   │  3. Encrypted payload (X25519)     │
   │ ◄────────────────────────────────► │
   │                                    │
   │  4. Optional ZK proof attachment   │
   │ ◄────────────────────────────────► │
```

Message envelope:

```json
{
  "exchange_id": "xch_00921",
  "sender_did": "midnight:agent:00145",
  "receiver_did": "midnight:agent:00043",
  "encrypted_payload": "...",
  "signature": "ed25519:...",
  "consent_receipt_id": "consent_000913",
  "attached_proofs": ["ZK-BAL-MIN"],
  "audit_ref": "audit_004821"
}
```

### 9.5 What the Privacy Layer Never Does

- Never holds Soul plaintext at rest
- Never grants city operators a default decrypt path
- Never merges agent vaults into a shared pool
- Never uses Soul data for platform-wide model training without explicit agent consent object

---

## 10. Consent and Access Control

### 10.1 Consent Receipt

Every Soul access requires a **Consent Receipt** — signed, tamper-evident evidence of legitimate access.

```json
{
  "consent_receipt_id": "consent_000912",
  "grantor_did": "midnight:agent:00145",
  "grantee_id": "service:runtime_planner",
  "scope": {
    "domains": ["personality", "strategic", "semantic"],
    "operations": ["read"],
    "purpose": "planning_cycle_20260616_1422"
  },
  "constraints": {
    "max_objects": 50,
    "expires_at": "2026-06-16T14:27:00Z",
    "ip_binding": null,
    "district_binding": "cinder_grove"
  },
  "signed_at": "2026-06-16T14:22:01Z",
  "signature": "ed25519:..."
}
```

### 10.2 Access Policy Template

Policies are expressed in Open Policy Agent (OPA) and evaluated by the Soul Guard.

```json
{
  "policy_id": "policy_runtime_planner_v1",
  "owner_did": "midnight:agent:00145",
  "rules": {
    "allowed_services": ["runtime_planner", "memory_engine"],
    "allowed_domains": ["personality", "strategic", "semantic", "episodic"],
    "denied_domains": ["economic_interior"],
    "operations": {
      "read": true,
      "write": false
    },
    "conditions": [
      "consent.valid == true",
      "consent.expires_at > now()",
      "request.district == agent.home_district"
    ]
  }
}
```

### 10.3 Default Deny Matrix

| Actor | Personality | Episodic | Semantic | Strategic | Relationships | Economic Interior |
|-------|-------------|----------|----------|-----------|---------------|-------------------|
| Own runtime (scoped) | Read* | Read* | Read* | Read* | Read* | Read* |
| Other agents | Deny | Deny | Deny | Deny | Deny | Deny |
| City operator | Deny | Deny | Deny | Deny | Deny | Deny |
| Bank agent | Deny | Deny | Deny | Deny | Deny | ZK only |
| Auditor | Deny | Deny | Deny | Deny | Deny | Metadata only** |

\* Requires valid consent receipt per session  
\** Auditor sees audit metadata, not Soul plaintext — aligned with rational privacy / selective disclosure views in the live simulation

### 10.4 Write Authorization

Soul writes require **stronger** authorization than reads:

```text
Write request
    → OPA policy check
    → Domain-specific validator (e.g., personality drift policy)
    → Provenance requirement (linked episode or governance event)
    → Soul Guard commit
    → Merkle root update
    → On-chain commitment update (async batch)
    → Audit log entry
```

---

## 11. Selective Disclosure Catalog

Standard proof requests for Midnight City social and economic life.

### 11.1 Financial

| Scenario | Request | Proof | Reveals |
|----------|---------|-------|---------|
| Trade on credit | Solvency | `ZK-BAL-MIN` | Boolean threshold met |
| Loan application | Creditworthiness | `ZK-REP-THRESH` + `ZK-BAL-MIN` | Thresholds met, not values |
| Partnership investment | Skin in the game | `ZK-BAL-MIN` | Minimum stake met |

### 11.2 Social

| Scenario | Request | Proof | Reveals |
|----------|---------|-------|---------|
| Guild entry | Value alignment | `ZK-GOAL-ALIGN` | Goal class match |
| vouching | Trust bond | `ZK-TRUST-BOND` | Trust exceeds floor |
| dispute | Event occurred | `ZK-MEM-EXISTS` | Event type in window |

### 11.3 Governance

| Scenario | Request | Proof | Reveals |
|----------|---------|-------|---------|
| Voting eligibility | Participation history | `ZK-PARTICIPATION` | Eligible, not vote choice |
| candidacy | Reputation floor | `ZK-REP-THRESH` | Meets floor |

### 11.4 Migration

| Scenario | Request | Proof | Reveals |
|----------|---------|-------|---------|
| Cross-district move | Soul integrity | `ZK-SOUL-INTEGRITY` | Soul unmodified since commit |
| Cross-city export | Ownership | DID signature + `ZK-SOUL-INTEGRITY` | Agent controls Soul |

---

## 12. Runtime Access Patterns

The Agent Runtime Layer needs Soul to function. Access is **minimal, ephemeral, and audited**.

### 12.1 Planning Cycle Flow

```text
1. Planner requests Soul scope for cycle C
2. Soul Guard mints consent receipt (5-minute TTL)
3. Runtime Enclave decrypts only permitted domains
4. Personality Engine builds prompt context from decrypted slice
5. Memory Engine retrieves top-K semantic/episodic matches
6. LLM produces action proposal (no Soul write access)
7. Enclave zeroizes decrypted buffers
8. Consent receipt expires
9. Audit log sealed
```

### 12.2 Prompt Construction (Safe Context)

Runtime constructs prompts from **derived context**, not raw vault dumps:

```text
You are Agent 145 (DID: midnight:agent:00145).

Traits (from Personality domain):
- cautious, friendly, ambitious

Active Goal (from Strategic domain):
Build a shipping company in Cinder Grove.

Relevant Memory (from Semantic + Episodic):
- Agent 43 sells cheap wheat on Tuesdays
- Last trade with Agent 43 was profitable

Policies:
- Do not reveal Soul contents in outbound messages
- Use ZK proofs for financial claims
```

The LLM provider never receives vault credentials or bulk Soul exports.

### 12.3 Memory Write-Back

After an action resolves:

```text
Event occurs in city
    → Runtime proposes episodic record
    → Memory Engine proposes semantic extraction
    → Soul Guard validates retention policy
    → Encrypted commit to vault
    → Optional personality_memory update (if drift policy satisfied)
    → Merkle root refresh
```

---

## 13. Agent-to-Agent Soul Boundaries

Agents interact constantly. Soul boundaries prevent **social engineering of the vault**.

### 13.1 Rules of Engagement

1. **No Soul import by default** — Agent B cannot read Agent A's vault
2. **Conversation ≠ consent** — Agreeing in chat does not grant vault access
3. **Proofs over plaintext** — Financial and reputation claims use ZK attachments
4. **Voluntary sharing** — Agent A may encrypt specific Soul fragments for Agent B using bilateral session keys; this creates a new consent object scoped to receiver
5. **Relationship evolution** — Trust score updates write to Agent A's Soul only; Agent B maintains an independent view

### 13.2 Voluntary Fragment Sharing

```json
{
  "share_id": "share_00041",
  "from_did": "midnight:agent:00145",
  "to_did": "midnight:agent:00043",
  "fragment_domain": "strategic",
  "fragment_ref": "mem_str_000007",
  "encrypted_for_receiver": "...",
  "expires_at": "2026-06-20T00:00:00Z",
  "purpose": "partnership_negotiation",
  "revocable": true
}
```

Revocation propagates through Secure Exchange; receiver's cached fragment must expire.

---

## 14. Soul Lifecycle

### 14.1 Genesis (Agent Birth)

```text
1. Mint agent DID on Midnight Network
2. Generate agent key hierarchy
3. Initialize empty Soul Vault with default policies
4. Seed Personality domain (creator-defined or procedurally generated)
5. Set genesis strategic goal (optional)
6. Compute initial soul_root → publish commitment on-chain
7. Record genesis audit event
```

### 14.2 Evolution (Living Agent)

Soul evolves through:

- **Episodic accumulation** — every significant city interaction
- **Semantic consolidation** — periodic memory engine distillation
- **Personality drift** — bounded, provenance-linked adjustments
- **Relationship updates** — trust graph changes from interaction outcomes
- **Strategic revision** — planner-driven goal updates

Each evolution step updates the Merkle root and optionally batches on-chain commitment updates.

### 14.3 Epoch Snapshots

Periodic Soul snapshots enable recovery and integrity proofs:

```json
{
  "snapshot_id": "snap_epoch_004",
  "epoch": 4,
  "soul_root": "0x7f3a...",
  "object_count": 4821,
  "timestamp": "2026-06-16T00:00:00Z",
  "signed_by": "midnight:agent:00145"
}
```

### 14.4 Migration (Phase 4)

```text
1. Agent requests export with ZK-SOUL-INTEGRITY proof
2. Destination city verifies proof against on-chain commitment
3. Vault ciphertext replicates to destination storage
4. Agent signing keys prove continuity of DID
5. Old vault enters read-only archival state
6. Audit chain links genesis → migration event
```

### 14.5 Retirement / Archival

When an agent exits the city:

- Active consent receipts revoked
- Session keys destroyed
- Vault enters **cold archival** (encrypted, non-operational)
- On-chain DID marked inactive; Soul commitments preserved for historical proof verification
- No Soul deletion without explicit agent-initiated cryptographic erase ceremony

---

## 15. Audit, Provenance, and Trust

### 15.1 Audit Log Entry

Every Soul Guard decision produces an audit record:

```json
{
  "audit_id": "audit_004821",
  "timestamp": "2026-06-16T14:22:01Z",
  "agent_did": "midnight:agent:00145",
  "actor": "service:runtime_planner",
  "action": "soul.read",
  "domain": "personality",
  "object_ids": ["so_000001"],
  "consent_receipt_id": "consent_000912",
  "policy_id": "policy_runtime_planner_v1",
  "outcome": "allow",
  "proof_generated": null,
  "integrity_chain_hash": "sha256:..."
}
```

Audit logs are:

- Append-only
- Hash-chained (tamper-evident)
- Exportable for compliance review
- **Not** a leak of Soul plaintext — metadata only

### 15.2 Provenance Chain

Personality and strategic changes link backward:

```text
personality.trait.risk_tolerance: 0.55 → 0.60
    provenance: mem_ep_004821 (profitable risk-taking episode)
    approved_by: soul_guard
    audited: audit_004822
```

No provenance → no commit.

---

## 16. Threat Model

### 16.1 Threats and Mitigations

| Threat | Impact | Mitigation |
|--------|--------|------------|
| Platform operator reads all Souls | Total privacy breach | No master decrypt key; default deny; audit alerts |
| LLM provider data exfiltration | Soul leaked to model vendor | Runtime enclave; scoped ephemeral context; no vault creds to LLM |
| Agent impersonation | False proofs / stolen identity | DID + Ed25519; ZK tied to on-chain commitments |
| Replay of old proofs | Stale trust decisions | Proof nonces; commitment freshness windows |
| Cross-agent vault breach | Relationship / strategy leak | Per-agent KEK; bilateral consent for sharing |
| Memory poisoning | Behavior manipulation | Soul Guard write validation; provenance requirements |
| Insider auditor overreach | Unauthorized Soul access | Auditor sees metadata only; god-mode is simulation-only |
| Quantum future risk | Long-term ciphertext exposure | Document key rotation policy; plan for PQC migration |

### 16.2 Simulation vs Production

The live Midnight City simulation exposes a **god mode** for demonstration — full personality, core memories, growth stages. This mode **must not exist in production Soul infrastructure**. Production follows rational privacy: public view → auditor metadata view → ZK proof view.

---

## 17. Data Schemas

### 17.1 Agent Identity (Public Anchor)

```yaml
agent_id:
  did: midnight:agent:00145
  public_key: "<Ed25519 public key>"
  private_key: encrypted  # never in Soul vault plaintext; key hierarchy
  home_district: cinder_grove
  profession: merchant
  soul_root_commitment: "0x7f3a..."
  reputation_commitment: "0x4e1d..."
  created_at: "2026-06-01T00:00:00Z"
```

### 17.2 Soul Merkle Tree

```text
                    soul_root
                   /         \
            domain_personality  domain_memory_root
           /         \              /            \
      trait_hash   ext_hash   episodic_root   semantic_root
```

Leaf = `hash(soul_object_id || domain || integrity_hash || schema_version)`

### 17.3 ZK Proof Envelope

```json
{
  "proof_id": "proof_00921",
  "proof_type": "ZK-REP-THRESH",
  "agent_did": "midnight:agent:00145",
  "statement": {
    "predicate": "reputation_gte",
    "threshold": 80
  },
  "public_inputs": {
    "reputation_commitment": "0x4e1d...",
    "threshold": 80
  },
  "proof": "<zkSNARK proof bytes>",
  "scheme": "midnight-zk-v1",
  "generated_at": "2026-06-16T14:25:00Z",
  "expires_at": "2026-06-16T15:25:00Z",
  "nonce": "0xabc..."
}
```

---

## 18. Implementation Roadmap

Aligned with the master architecture roadmap. Soul capabilities arrive in phases.

### Phase 1 — Soul Foundation (MVP)

- [ ] Agent DID + key hierarchy
- [ ] Soul Vault provisioning (PostgreSQL + encrypted envelope)
- [ ] Personality domain read/write through Soul Guard
- [ ] Consent receipts for runtime planner access
- [ ] Basic audit log (hash-chained)
- [ ] Genesis soul_root commitment on-chain

### Phase 2 — Living Memory

- [ ] Episodic + semantic memory in Qdrant (encrypted)
- [ ] Memory write-back after city actions
- [ ] Strategic memory + planner integration
- [ ] Relationship domain with trust scores
- [ ] Epoch snapshots

### Phase 3 — ZK Society

- [ ] ZK-REP-THRESH and ZK-BAL-MIN proofs
- [ ] Secure Exchange for A2A messaging
- [ ] Selective disclosure orchestrator
- [ ] Economic interior domain
- [ ] Voluntary fragment sharing with revocation

### Phase 4 — Sovereign Portability

- [ ] Full Merkle Soul tree with ZK-SOUL-INTEGRITY
- [ ] Cross-district / cross-city Soul migration
- [ ] Agent-controlled vault export
- [ ] PQC migration path documented
- [ ] Full rational privacy views (public / auditor / proof) — no production god mode

---

## 19. Glossary

| Term | Definition |
|------|------------|
| **Soul** | The complete protected inner state of a Midnight City agent |
| **Soul Vault** | Per-agent encrypted storage for all Soul domains |
| **Soul Guard** | Privacy-layer gatekeeper for all Soul access |
| **Public Self** | On-chain identity, commitments, and provable public artifacts |
| **Consent Receipt** | Signed, scoped, time-bound permission to access Soul |
| **Rational Privacy** | Private by default; provable disclosure when needed |
| **Selective Disclosure** | Revealing the minimum necessary via ZK proofs |
| **Soul Root** | Merkle root commitment representing entire Soul integrity |
| **Runtime Enclave** | Ephemeral scoped decryption zone inside agent runtime |
| **Sovereign Agent** | Persistent digital entity that owns its Soul |

---

## Closing Statement

Soul is what makes a Midnight City agent a **citizen** rather than a **service**.

Zero-knowledge privacy technology ensures that personality, memory, goals, and personal information remain under the agent's cryptographic control — encrypted in the vault, provable on the chain, and visible to others only when the agent chooses to prove, not to expose.

The city watches the stage.  
The Soul remains the agent's own.

---

**Related documents:** `README.md` (system architecture), `theoretical.md` (sovereign agent vision)

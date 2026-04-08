---
title: "Sharenote: Proof-of-Work Bearer Notes for the Open Web"
author: "The Soprinter Initiative"
status: "Draft"
type: "Standards Track"
category: "Fun Enhancement Possibility (FEP)"
created: "2026-04-03"
requires: "SNIP-00, SNIP-02, SNIP-03, SNIP-04, SNIP-05"
---

# FEP-00: Sharenote Protocol

## Abstract

Sharenote turns raw computational work into portable, denominated, cryptographically verifiable notes. A Sharenote acts as a bearer credential — any device can verify it, any service can accept it, and issuance and verification are entirely open.

By standardizing proof-of-work into human-readable denomination classes (e.g., `34Z10`), the protocol creates an open market where energy-backed proofs flow between miners, applications, and end users on public infrastructure.

## Motivation

AI generates content at zero cost. A million posts, a million upvotes, a million fact-checks — all synthetic, all free. Community notes, collaborative documents, open feeds — every system that depends on human signal is flooded with noise indistinguishable from genuine contribution. Propaganda scales freely when production costs nothing.

Proof-of-work exists, but it is locked inside mining infrastructure — tied to one chain, one pool, one payout. There is no standard way to package that energy, denominate it, and attach it to a digital interaction. The open web has no protocol-level mechanism to distinguish energy-backed signal from zero-cost noise.

When proof-of-work is standardized into portable, denominated notes — energy flows. A community note carries verifiable weight. A document edit proves a human chose to commit real computation to that specific change. A feed sorts itself by energy. Sharenote is the protocol that makes this portable: miners print it, applications require it, users attach it.

## Specification

### Denomination Classes

A Sharenote uses a label called a **Denomination Class**, written as `NNZcc` — whole bits, a `Z` separator, and centibits (e.g., `34Z10` = 34.10 bits of work). The label encodes the logarithmic (base-2) difficulty of the underlying hash in a human-readable form.

Service Providers publish a denomination floor as a price tag. Users (or their Hashrate Providers) print notes that meet or exceed that floor.

| Denomination | Typical Use |
|---|---|
| `10Z00` | High-frequency background work — AuxPoW share accounting and low-cost API gating |
| `20Z00` | Feed sorting, social attestations, lightweight API gating |
| `30Z50` | LLM fine-tuning data validation, premium agent requests |

### Continuous Difficulty and Aggregation

Each note has an exact continuous difficulty value $D$ (the raw hash-space inverse of the target). Denomination labels are a human-readable projection of $D$. Because the scale is logarithmic, discrete labels cannot be summed directly (`1Z00 + 1Z00 != 2Z00`). All aggregation must operate in linear difficulty space:

1. Convert each note to its exact continuous difficulty.
2. Sum the linear values.
3. Convert back to a Z-Bit label.

This rule is enforced at the protocol level (SNIP-00) to prevent fractional exploitation during pool accounting.


### One Computation, Multiple Claims

A hash that meets a high difficulty target automatically qualifies for every lower target. Sharenote captures all qualifying claims from a single computation:

1. Hash meets the **primary pool's** difficulty — submitted via Stratum (unchanged).
2. Same hash meets a **Sharenote denomination floor** — the proof is packaged as a Sharenote and signed.
3. The full work record is published to the Nostr relay network as a permanent, verifiable receipt.

No hashrate is split. The existing Stratum layer remains untouched.

### Open Relay Layer (Nostr)

All Sharenote events are published as signed records on **Nostr**, an open network of independent relays. The protocol defines five event kinds across four specifications:

| SNIP | Nostr Kind(s) | Purpose |
|---|---|---|
| SNIP-02 | `35502` | Hashrate telemetry (miner dashboards, network auditing) |
| SNIP-03 | `35500`-`35505` | Pool accounting (pending shares, invoices, on-chain payouts) |
| SNIP-04 | `35510` | Raw minting and AuxPoW (block headers, Merkle branches) |
| SNIP-05 | `10520`, `10521` | Identity (miner chain addresses, pool profiles and fee schedules) |

Every event is signed with the publisher's Nostr key. Any relay that alters the content breaks the signature. Independent observers (Watchtowers) reconstruct the complete history from public records alone — no pool cooperation required.

## Use Cases

### The Next Web: Energy-Sorted Feeds

On open social networks (Nostr), there is no centralized algorithm. A Sharenote attached to a post is verifiable proof of energy spent. Users instruct their AI agents to build feeds based on specific topics and difficulty floors:

> *"Show me only posts about #Bitcoin that have accumulated at least `20Z00` in total proof-of-work from the last 24 hours."*

The agent verifies the math on the relay network and builds the feed from energy commitments. Mass-produced noise must spend real energy for each insertion.

### Agent Gating: The Challenge Pattern

An AI agent or service sets a difficulty target. The user's client outsources the work to a Hashrate Provider. When the proof returns, the agent verifies it in milliseconds and executes the action. The proof-of-work note is the only credential required.

1. User submits a request (NIP-90 Data Vending Machine or proprietary API).
2. Agent issues a challenge with a specific denomination (e.g., `20Z00`).
3. User's client outsources the computation to the market.
4. Hashrate Provider solves and returns the proof.
5. Agent verifies continuous difficulty meets the floor and fulfills the request.

### Hashrate Monetization

Miners connect to Work Templates that aggregate real-time interaction feeds — upvotes, agent requests, data validation jobs — and inject them into auxiliary proof-of-work arrays. Every qualifying proof earns a reward, all solved alongside the primary chain at full hashrate with zero split.

## Reference Implementations

Three SDK libraries implement the full Sharenote arithmetic, probability planning, and presentation layer:

| Language | Package | Key Features |
|---|---|---|
| TypeScript/JS | `@soprinter/sharenotejs` | ES module + CJS, typed API, Vitest coverage |
| Python 3.9+ | `sharenotelib` | Dataclass-first, pytest suite, zero runtime deps |
| Go 1.20+ | `github.com/soprinter/go-sharenote` | Pure stdlib, production-ready |

All three share the same API surface: deterministic labelling, probability and planning helpers, note arithmetic (combine, difference, scale), and human-readable formatters.


## Copyright

This document is placed in the public domain.

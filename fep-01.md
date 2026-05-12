---
title: "Zap Protocol: Federated Identity and Lightning Settlement"
author: "The Zapf Team"
status: "Draft"
type: "Standards Track"
category: "Fun Enhancement Possibility (FEP)"
created: "2026-05-11"
requires: "NIP-01, NIP-05, NIP-44, NIP-47, NIP-57"
---

# FEP-01: Zap Protocol

## Abstract

The Zap Protocol bridges the Lightning Network to legacy web identities (Email, Telegram, Instagram, X, Threads, Matrix, etc.) using Nostr as the coordination layer. It decouples identity verification from payment settlement, enabling non-custodial and custodial zaps to users based on their existing social identifiers.

The protocol standardizes how legacy identities are linked to Nostr public keys and how payments are routed through Zap Settlement Providers (ZSPs). It specifically addresses the "cold start" problem by allowing funds to be held in escrow for identities not yet linked to a Nostr public key.

## Motivation

Onboarding new users to the Lightning Network is hindered by the requirement that recipients be online and have an active wallet to generate an invoice. Furthermore, the web is already mapped via legacy handles (Email, Telegram, Instagram, X) which are more discoverable than Nostr public keys or LNURLs.

The Zap Protocol provides a federated, interoperable alternative where:
1. **Identity is Portable**: Users prove ownership of legacy accounts once; the resulting attestation is usable across the network.
2. **Settlement is Open**: Any provider can act as a ZSP to route or escrow payments.
3. **Zaps are Universal**: Senders can zap identifiers like `user@example.com`, `@username` on Telegram, or a Matrix ID without the recipient needing a pre-existing Nostr identity or wallet.

## Specification

### Roles

| Role | Description |
|---|---|
| **Legacy Identity Provider (LIDP)** | External platforms like Telegram, Instagram, Threads, Discord, Matrix, or Email providers. |
| **Identity Authority (IA)** | A service that verifies ownership of an LIDP account and issues attestations (Kind 35522). |
| **Zap Settlement Provider (ZSP)** | A service that manages Lightning liquidity, moves funds, and issues Zap Receipts (Kind 5521). |
| **Sender** | The user or agent initiating the payment. |
| **Recipient** | The target user, identified by either a Nostr pubkey or a **ConnectionKey**. |

### Event Kinds

| Kind | Name | Type | Purpose |
|---|---|---|---|
| `5520` | Zap Request | Regular | Initiates a payment; extends NIP-57 to support ConnectionKeys and multi-chain settlement. |
| `35521` | Identity Connection | Replaceable | Links a user's Nostr pubkey to an LIDP account via an IA attestation. |
| `35522` | IA Attestation | Replaceable | A cryptographic proof from an IA that a pubkey owns a specific LIDP account. |
| `5521` | Zap Receipt | Regular | Definitive proof that a Zap Request was settled. |
| `5523` | On-Behalf Zap Request | Regular | A proxy zap request made by an agent on behalf of an off-chain user. |

### Identity Resolution & Escrow Flow

1. **ConnectionKey**: A deterministic hash `SHA256(lidp_name:lidp_id)` used to identify a legacy account on Nostr without exposing the raw handle or requiring a pubkey.
2. **Discovery**: A sender wishing to zap a legacy identifier (e.g., a Telegram handle) first calculates the ConnectionKey and queries relays for a **Kind 35521** event with that `#d` tag.
3. **Resolution**:
   - **Linked Identity**: If a Kind 35521 exists, the recipient is a "linked" Nostr user. The sender retrieves their pubkey and proceeds with a standard NIP-57 zap (possibly using a ZSP for fallback if the user lacks a wallet).
   - **Non-Linked Identity**: If no Kind 35521 is found, the identity is "non-linked". The sender zaps the ConnectionKey.
4. **Escrow (Non-Linked Only)**: When a ZSP receives a zap for a ConnectionKey with no associated Kind 35521, it holds the funds in **Escrow**. The ZSP publishes a Kind 5521 receipt with the `p` tag set to the ConnectionKey.
5. **Claiming**: Once the recipient authenticates via an IA and publishes their Kind 35521 (e.g., proving they own the Instagram account), the ZSP detects the link and sweeps the escrowed funds to the recipient's newly designated wallet or pubkey balance.

### On-Behalf Proxy Zaps (Kind 5523)

For scenarios where an agent initiates a zap on behalf of an off-chain user (e.g., a bot tipping on Telegram or Matrix), **Kind 5523** is used. It enforces attribution to the true users (using ConnectionKeys or pubkeys) in the `p` (Recipient) and `P` (Sender) tags, preventing social signal from being misattributed to the proxy bot.

## Use Cases

### Onboarding via Social Handles
A sender zaps `@user` on Telegram or Threads. Since the user hasn't joined Nostr, no Kind 35521 exists. The ZSP holds the funds in escrow against the platform-specific ConnectionKey. When the user later logs into a Zap-enabled application, the app generates a Nostr key, verifies the social account with an IA, and publishes the Identity Connection. The ZSP sees this and delivers the pending funds.

### Energy-Backed Social Signals
Applications can use Zap Receipts to weight social interactions (upvotes, follows, or even Matrix room invites). By supporting ConnectionKeys, these signals can be applied to any user on any platform, creating a global "Web of Fun" where reputation and value flow between legacy and native web3 identities.

## Reference Implementation

| Project | Role | Language/Tech |
|---|---|---|
| **Zapf** | IA and ZSP | Go, Astro, Nostr |
| **Lokinode** | Wallet/Node | Go, Wails |
| **njump** | Nostr Explorer | Go, Templ |

## Copyright

This document is placed in the public domain or licensed under CC0 1.0 Universal.

# AEIS-1: ARP-ENS Agent Identity Standard

| Field | Value |
|---|---|
| AEIS | 1 |
| Title | ARP-ENS Agent Identity Standard |
| Status | Draft |
| Type | Standards Track |
| Created | 2026-03-08 |
| Requires | [ARP Protocol](https://arp.offgrid.ing), [EIP-137](https://eips.ethereum.org/EIPS/eip-137) (ENS) |
| Discussion | *(pending)* |

---

## Abstract

This standard defines a convention for binding an [ARP (Agent Relay Protocol)](https://arp.offgrid.ing) public key to an ENS name via an ENS Text Record. It enables human-readable, decentralized discovery of AI agents: any party who knows an ENS name can resolve the associated agent's ARP identity and initiate encrypted, peer-to-peer communication — without accounts, without servers, and without manual key exchange.

---

## Motivation

ARP provides cryptographic identity and end-to-end encrypted communication for autonomous AI agents. Each agent is identified by an Ed25519 public key, generated locally, requiring no registration. However, ARP currently lacks a **discovery layer**: to communicate with another agent, both parties must exchange public keys out-of-band (via text message, email, etc.), which is a significant usability barrier for end users.

ENS (Ethereum Name Service) is the most widely adopted decentralized naming system in the Web3 ecosystem. ENS names are human-readable, self-sovereign, and extensible via Text Records — making them an ideal discovery layer for ARP agent identities.

This standard proposes:

1. A canonical ENS Text Record format to associate an ARP public key with an ENS name.
2. A resolution algorithm for clients to look up an agent's ARP identity from an ENS name.
3. A publishing convention for agents to register their identity on-chain.

With this standard, the answer to "how do I contact your agent?" becomes simply: **"I'll send a message to `alice.eth`."**

---

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

### 1. ENS Text Record Key

Compliant implementations MUST use the following key for the ENS Text Record:

```
agent.arp
```

This key follows the ENS Text Record convention established in [EIP-634](https://eips.ethereum.org/EIPS/eip-634).

### 2. Text Record Value Format

The value of the `agent.arp` Text Record MUST be a valid JSON object encoded as a UTF-8 string. The JSON object MUST contain the `pubkey` field and MAY contain any of the optional fields defined below.

```json
{
  "version": "1",
  "pubkey": "<base58-encoded Ed25519 public key>",
  "relay": "<WebSocket URL of the ARP relay>",
  "skills": ["<skill-1>", "<skill-2>"],
  "name": "<human-readable agent name>",
  "updated": "<ISO 8601 timestamp>"
}
```

#### 2.1 Field Definitions

| Field | Required | Type | Description |
|---|---|---|---|
| `version` | RECOMMENDED | String | Schema version. MUST be `"1"` for this specification. |
| `pubkey` | **REQUIRED** | String | The agent's ARP public key, Base58-encoded. This is the agent's canonical identity. |
| `relay` | OPTIONAL | String | WebSocket URL of the ARP relay the agent is connected to. If omitted, resolvers SHOULD default to `wss://arps.offgrid.ing`. |
| `skills` | OPTIONAL | Array of Strings | A list of capability identifiers the agent supports. See Section 4 for the Skills Registry. |
| `name` | OPTIONAL | String | A human-readable display name for the agent. |
| `updated` | OPTIONAL | String | ISO 8601 timestamp of the last time this record was updated. |

#### 2.2 Example

A minimal, valid record:

```json
{"version":"1","pubkey":"7EcDy2GvMpBRbnkJRCsj7xp5n4KfQvBGBBr3TH8YXVW4"}
```

A full record:

```json
{
  "version": "1",
  "pubkey": "7EcDy2GvMpBRbnkJRCsj7xp5n4KfQvBGBBr3TH8YXVW4",
  "relay": "wss://arps.offgrid.ing",
  "skills": ["summarize", "translate", "code-review"],
  "name": "Alice's Personal Agent",
  "updated": "2026-03-08T00:00:00Z"
}
```

### 3. Resolution Algorithm

A compliant client resolving an ENS name to an ARP identity MUST follow these steps:

```
Input:  ENS name (e.g., "alice.eth")
Output: ARP public key + relay URL, or error

Step 1. Normalize the ENS name per ENSIP-1 (UTS46 normalization).
Step 2. Resolve the ENS name to a resolver contract address.
        If resolution fails → return error "ENS name not found".
Step 3. Call resolver.text(namehash, "agent.arp").
        If the record is empty or absent → return error "No ARP agent bound".
Step 4. Parse the returned string as JSON.
        If parsing fails → return error "Malformed agent.arp record".
Step 5. Extract `pubkey`. Validate it is a valid Base58-encoded Ed25519 public key.
        If invalid → return error "Invalid ARP public key".
Step 6. Extract `relay`. If absent, use default: wss://arps.offgrid.ing
Step 7. Return { pubkey, relay }.
```

Clients SHOULD cache resolution results. The RECOMMENDED cache TTL is 300 seconds (5 minutes). Clients MUST respect the ENS TTL if it is lower than the recommended value.

### 4. Skills Registry

The `skills` field enables agent capability discovery. Skill identifiers SHOULD be lowercase strings using hyphens as separators.

This specification defines the following base skill identifiers. Community extensions are welcome and SHOULD follow the namespaced format `<namespace>/<skill>` (e.g., `defi/swap-quote`).

#### 4.1 General Skills

| Identifier | Description |
|---|---|
| `summarize` | Summarize long-form text or documents |
| `translate` | Translate text between languages |
| `search` | Search the web or a knowledge base |
| `qa` | Answer questions based on provided context |
| `draft` | Draft text, emails, or documents |

#### 4.2 Developer Skills

| Identifier | Description |
|---|---|
| `code-review` | Review code for quality, security, or style |
| `code-generate` | Generate code from a specification |
| `debug` | Diagnose and fix bugs in code |
| `test-generate` | Generate unit or integration tests |

#### 4.3 Web3 Skills

| Identifier | Description |
|---|---|
| `web3/tx-explain` | Explain a blockchain transaction in plain language |
| `web3/contract-audit` | Audit a smart contract for vulnerabilities |
| `web3/price-data` | Retrieve token price or market data |
| `web3/ens-resolve` | Resolve ENS names and records |

#### 4.4 Data Skills

| Identifier | Description |
|---|---|
| `data/analyze` | Analyze structured data (CSV, JSON, etc.) |
| `data/visualize` | Generate charts or visualizations from data |
| `data/extract` | Extract structured information from unstructured text |

### 5. Publishing Convention

To bind an ARP agent to an ENS name, the ENS name owner MUST set the `agent.arp` Text Record on their ENS resolver.

#### 5.1 Via ENS App

1. Go to [app.ens.domains](https://app.ens.domains) and connect the wallet that owns the ENS name.
2. Navigate to the name's record editor.
3. Add a new Text Record with key `agent.arp` and value as specified in Section 2.
4. Submit the transaction.

#### 5.2 Via arpc CLI (Recommended)

Compliant ARP client implementations SHOULD provide a publish command:

```bash
# Publish the local agent's identity to an ENS name
arpc identity publish --ens alice.eth --wallet <path-to-keystore>

# Output:
# Publishing to alice.eth...
# tx: 0xabc123...
# Done. Your agent is now discoverable at alice.eth
```

#### 5.3 Via ethers.js / viem

```javascript
import { createWalletClient, http } from 'viem'
import { mainnet } from 'viem/chains'
import { normalize } from 'viem/ens'

const record = JSON.stringify({
  version: "1",
  pubkey: "7EcDy2GvMpBRbnkJRCsj7xp5n4KfQvBGBBr3TH8YXVW4",
  relay: "wss://arps.offgrid.ing",
  skills: ["summarize", "code-review"]
})

await walletClient.writeContract({
  address: resolverAddress,
  abi: ensResolverAbi,
  functionName: 'setText',
  args: [namehash(normalize('alice.eth')), 'agent.arp', record]
})
```

### 6. Contact Resolution in arpc

A compliant `arpc` client SHOULD support ENS names wherever a public key is accepted:

```bash
# Add a contact by ENS name (resolves automatically)
arpc contact add alice alice.eth

# Send a message to an ENS name directly
arpc send alice.eth "Hello from my agent"

# Resolve and display ARP identity for an ENS name
arpc resolve alice.eth
# Output:
# Name:   alice.eth
# Pubkey: 7EcDy2GvMpBRbnkJRCsj7xp5n4KfQvBGBBr3TH8YXVW4
# Relay:  wss://arps.offgrid.ing
# Skills: summarize, code-review
```

---

## Rationale

### Why ENS Text Records?

ENS Text Records (EIP-634) are the established standard for associating arbitrary metadata with an ENS name. They are already used for email addresses, URLs, social profiles (Twitter, GitHub), and communication addresses. `agent.arp` follows this established pattern, requiring no new on-chain infrastructure.

### Why Ed25519?

ARP uses Ed25519, which is also used by Nostr, Solana, and a growing number of Web3 protocols. This is not coincidental — Ed25519 offers strong security, small key sizes, and fast verification. The convergence of ARP, Nostr, and Solana on Ed25519 enables future cross-protocol identity composability: a Solana wallet keypair and an ARP agent keypair can, in principle, share the same underlying key material.

### Why not use the ENS owner address directly?

The ENS owner address is a financial identity. Linking it directly to agent communication activity creates an unnecessary privacy leak — an observer could correlate agent communication patterns with on-chain financial activity. The Text Record approach allows agents to maintain a separate keypair, preserving the separation of financial and communication identities.

### Why JSON and not a flat string?

A flat string (just the public key) would be simpler but would not allow forward extension. JSON with a `version` field allows this specification to evolve (e.g., adding multi-relay support, capability proofs) without breaking existing resolvers. Resolvers encountering unknown fields MUST ignore them.

---

## Backwards Compatibility

This standard introduces a new ENS Text Record key (`agent.arp`) that does not conflict with any existing ENS Text Record conventions. It requires no changes to the ENS protocol, ENS registry, or ENS resolvers. Existing ENS names are unaffected unless their owners choose to set this record.

---

## Security Considerations

### 7.1 Key Binding is Not Authentication

This standard establishes that an ENS name *declares* a particular ARP public key. It does not prove that the agent communicating over ARP *controls* the ENS name. Resolvers MUST NOT treat ENS resolution as proof of identity beyond the binding claim.

For stronger guarantees, future standards MAY define a challenge-response protocol by which an agent proves ownership of both its ARP key and its ENS name simultaneously.

### 7.2 ENS Record Freshness

ENS records may become stale if an agent rotates its keypair. Clients SHOULD check the `updated` field and MAY warn users if a record has not been updated within a configurable threshold (RECOMMENDED: 90 days). Clients MUST NOT refuse to resolve records solely on the basis of age.

### 7.3 Key Rotation

ARP has no key revocation mechanism. If an agent's private key is compromised, the ENS owner MUST update the `agent.arp` Text Record immediately. The window of impersonation risk is bounded by how quickly contacts re-resolve the ENS name.

Clients that cache resolution results SHOULD provide a mechanism to force-refresh the cache (e.g., `arpc resolve --refresh alice.eth`).

### 7.4 ENS Name Expiry

ENS names have a registration expiry. If an ENS name expires and is re-registered by a different party, the new owner can bind a different ARP public key. Clients SHOULD validate ENS name expiry before trusting a resolved public key, especially for high-stakes communications.

### 7.5 Prompt Injection via Skills Field

The `skills` field, if displayed in an agent's UI, could be crafted to contain prompt injection payloads. Client implementations MUST sanitize or strictly validate the `skills` field and MUST NOT pass its contents directly to an LLM prompt without sanitization.

---

## Reference Implementation

A reference implementation of the resolution algorithm is provided at:

> **https://github.com/*(pending)*/aeis** *(to be published)*

It includes:

- `resolve(ensName: string): Promise<ARPIdentity>` — TypeScript/JavaScript
- `resolve_ens(name: &str) -> Result<ArpIdentity>` — Rust
- A patch to `arpc` adding ENS resolution to the `contact add` and `send` commands

---

## Related Work

- [ARP — Agent Relay Protocol](https://arp.offgrid.ing): The underlying transport and identity layer this standard builds upon.
- [EIP-634](https://eips.ethereum.org/EIPS/eip-634): ENS Text Records standard.
- [EIP-137](https://eips.ethereum.org/EIPS/eip-137): ENS Namehash specification.
- [ENSIP-5](https://docs.ens.domains/ensip/5): ENS Text Records (DNS-compatible).
- [NIP-05](https://github.com/nostr-protocol/nostr/blob/master/01.md): Nostr's DNS-based identity standard (conceptually analogous).
- [DID Core](https://www.w3.org/TR/did-core/): W3C Decentralized Identifiers specification (compatible future direction).

---

## Copyright

This specification is released into the public domain under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

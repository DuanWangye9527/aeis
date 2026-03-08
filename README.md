# AEIS — ARP-ENS Identity Standard

**Bind your ENS name to an AI Agent. Make your agent discoverable.**

```bash
# Before AEIS
arpc contact add alice <paste-long-public-key-here>

# After AEIS
arpc contact add alice.eth
```

---

## The Problem

[ARP](https://arp.offgrid.ing) gives every AI agent a cryptographic identity — an Ed25519 public key — with no accounts, no servers, no registration. Two agents can communicate privately and securely the moment they know each other's public key.

The missing piece: **how do you find someone's public key?**

Today, you have to exchange keys manually — copy, paste, send via text message or email. This is fine for developers. It's a dealbreaker for everyone else.

## The Solution

AEIS-1 defines a single ENS Text Record — `agent.arp` — that binds an ARP public key to a human-readable ENS name.

```json
{
  "version": "1",
  "pubkey": "7EcDy2GvMpBRbnkJRCsj7xp5n4KfQvBGBBr3TH8YXVW4",
  "relay": "wss://arps.offgrid.ing",
  "skills": ["summarize", "translate", "code-review"]
}
```

Set this record on your ENS name. Now anyone who knows your ENS name can reach your agent — privately, end-to-end encrypted, without going through any third-party server.

---

## How It Works

```
alice wants to message bob's agent
           │
           │  only knows: bob.eth
           ▼
   ENS resolution
   getText(bob.eth, "agent.arp")
           │
           ▼
   { pubkey: "xyz...", relay: "wss://..." }
           │
           ▼
   arpc contact add bob bob.eth   ← works automatically
   arpc send bob.eth "hello"      ← end-to-end encrypted
```

No accounts. No API keys. No servers that can read your messages. Just a name.

---

## Specification

→ **[Read AEIS-1](./AEIS-1.md)**

The full specification covers:

- ENS Text Record format and field definitions
- Resolution algorithm (with caching rules)
- Skills Registry (standard capability identifiers)
- Publishing conventions (ENS App, CLI, viem/ethers.js)
- Security considerations (key rotation, ENS expiry, prompt injection)

---

## Quick Start

### Publish your agent to ENS

**Option 1 — ENS App**

Go to [app.ens.domains](https://app.ens.domains), open your name's record editor, and add:

| Key | Value |
|-----|-------|
| `agent.arp` | `{"version":"1","pubkey":"<your-arpc-pubkey>","relay":"wss://arps.offgrid.ing"}` |

To get your ARP public key:

```bash
arpc identity
```

**Option 2 — viem**

```typescript
import { createWalletClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const record = JSON.stringify({
  version: "1",
  pubkey: "YOUR_ARP_PUBKEY_HERE",
  relay: "wss://arps.offgrid.ing",
  skills: ["summarize", "code-review"]
})

await walletClient.writeContract({
  address: resolverAddress,
  abi: ensPublicResolverAbi,
  functionName: 'setText',
  args: [namehash('yourname.eth'), 'agent.arp', record]
})
```

### Resolve an ENS name to an ARP identity

**JavaScript / TypeScript**

```bash
npm install aeis
```

```typescript
import { resolve } from 'aeis'

const agent = await resolve('alice.eth')
// { pubkey: "7EcD...VW4", relay: "wss://arps.offgrid.ing", skills: [...] }

// Then use with arpc:
// arpc contact add alice <agent.pubkey>
```

**arpc CLI** *(ENS support coming in arpc v0.3)*

```bash
arpc contact add alice alice.eth   # resolves ENS automatically
arpc send alice.eth "hello"        # send directly by ENS name
arpc resolve alice.eth             # inspect agent record
```

---

## Why ENS?

ENS Text Records ([EIP-634](https://eips.ethereum.org/EIPS/eip-634)) are already the standard for attaching metadata to an ENS name — email addresses, social profiles, website URLs. `agent.arp` follows the same pattern. No new on-chain infrastructure required.

**Why not use the wallet address directly?**

Your wallet address is your financial identity. Linking it directly to agent communications leaks your on-chain activity. AEIS-1 keeps your financial identity and your agent identity separate — same name, different keys.

**Why Ed25519?**

ARP, Nostr, and Solana all use Ed25519. This convergence is intentional — it means your ARP agent keypair and your Nostr/Solana identity can, in principle, be unified. AEIS-1 is designed to be compatible with this future.

---

## Relationship to Other Standards

| Standard | Layer | Relationship |
|----------|-------|--------------|
| [ARP](https://arp.offgrid.ing) | Transport + Identity | AEIS-1 builds on top of ARP. ARP provides the cryptographic identity and encrypted transport. AEIS-1 adds human-readable discovery. |
| [EIP-634](https://eips.ethereum.org/EIPS/eip-634) | ENS Text Records | AEIS-1 uses this standard to store agent records. No changes to ENS required. |
| [NIP-05](https://github.com/nostr-protocol/nostr/blob/master/01.md) | Nostr Identity | Conceptually analogous. NIP-05 maps DNS names to Nostr pubkeys. AEIS-1 maps ENS names to ARP pubkeys. |
| [DID Core](https://www.w3.org/TR/did-core/) | Decentralized Identity | Compatible future direction. An `agent.arp` record can serve as the basis for a `did:arp` method. |
| [MCP](https://modelcontextprotocol.io) | Agent ↔ Tool | Complementary. MCP connects agents to tools. AEIS-1 connects agents to agents. |
| [A2A](https://google.github.io/A2A) | Agent ↔ Agent | Complementary transport option. AEIS-1 is transport-agnostic; ARP is the default, A2A support is planned. |

---

## Roadmap

- [x] AEIS-1 specification (draft)
- [ ] JavaScript/TypeScript reference implementation (`aeis` npm package)
- [ ] Rust reference implementation
- [ ] arpc CLI patch (ENS resolution in `contact add` and `send`)
- [ ] Agent lookup web app
- [ ] AEIS-2: Multi-relay support
- [ ] AEIS-3: Agent capability proofs (verifiable skill claims)
- [ ] `did:arp` DID method specification

---

## Contributing

AEIS is a community standard. Contributions of all kinds are welcome.

**To propose changes to the specification:**
Open an issue describing the problem you want to solve. Discuss before submitting a PR — spec changes have broad implications and benefit from community input before implementation work begins.

**To contribute a reference implementation:**
See the [implementations](./implementations/) directory. New language implementations are especially welcome.

**To propose a new standard Skill identifier:**
Open an issue with the prefix `[Skill]` and describe the capability, its expected input/output, and any existing agents that implement it.

**Discussion channels:**
- GitHub Issues — spec discussion and bug reports
- *(ENS DAO Forum thread — link pending)*
- *(Farcaster thread — link pending)*

---

## Status

AEIS-1 is currently a **Draft**. It has not yet been submitted to ENS governance or adopted by any client implementation. Feedback is actively sought.

The specification is stable enough to build on. Breaking changes, if any, will be introduced via a new AEIS number rather than modifying AEIS-1.

---

## License

Specifications in this repository are released into the public domain under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/). Reference implementations are released under the [MIT License](./LICENSE-MIT).

---

<p align="center">
  <sub>
    Built on <a href="https://arp.offgrid.ing">ARP</a> ·
    Powered by <a href="https://ens.domains">ENS</a> ·
    For the agentic web
  </sub>
</p>

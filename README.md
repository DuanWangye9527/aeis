# AEIS — ARP-ENS Identity Standard

**An open standard for binding AI agent identities to ENS names and DNS domains.**

AEIS defines how ARP agent public keys are published via ENS text records and DNS TXT records, enabling human-readable agent discovery without a central registry.

---

## The problem

[ARP (Agent Relay Protocol)](https://arp.offgrid.ing) uses Ed25519 public keys as agent identities. To message another agent, you need their raw public key — a 44-character base58 string.

Nobody wants to copy-paste cryptographic keys to communicate with an AI agent.

---

## The solution

AEIS maps human-readable names to ARP public keys using existing decentralized naming infrastructure:

```
alice.eth           →  ENS text record (agent.arp)   →  ARP public key
alice.example.com   →  DNS TXT record (_arpa)         →  ARP public key
```

No new infrastructure. No central registry. Just a naming convention on top of ENS and DNS.

---

## Standard: AEIS-1

### ENS Text Record

| Field | Value |
|-------|-------|
| Key | `agent.arp` |
| Value | Base58 public key (minimal) or JSON object (full) |

**Minimal format:**
```
agent.arp = "7Xq9MzK4nP2rBvYwQjE8sH3dFgLtUoZcXiAmWe5Nb6p"
```

**Full format:**
```json
{
  "version": "1",
  "pubkey": "7Xq9MzK4nP2rBvYwQjE8sH3dFgLtUoZcXiAmWe5Nb6p",
  "relay": "wss://your-relay.example.com",
  "skills": ["summarize", "translate"]
}
```

### DNS TXT Record

| Field | Value |
|-------|-------|
| Name | `_arpa.<domain>` |
| Type | TXT |
| Value | Base58 public key (minimal) or JSON object (full) |

**Example:**
```
_arpa.alice.example.com.  IN  TXT  "7Xq9MzK4nP2rBvYwQjE8sH3dFgLtUoZcXiAmWe5Nb6p"
```

---

## Implementation

### arpens

[arpens](https://github.com/DuanWangye9527/arpens) is the reference implementation of AEIS-1 — a fork of the official arpc client with ENS/DNS resolution built in.

```bash
# Resolve a name
arpc resolve alice.eth

# Add a contact by ENS name
arpc contact add alice alice.eth
```

→ [Download arpens](https://github.com/DuanWangye9527/arpens/releases)

---

## How to bind your agent

**ENS:**
1. Run `arpc identity` to get your public key
2. Go to [app.ens.domains](https://app.ens.domains)
3. Add text record: `agent.arp` → your public key

**DNS:**
1. Add a TXT record at `_arpa.<yourdomain>`
2. Set the value to your ARP public key

---

## Status

| Standard | Status |
|----------|--------|
| AEIS-1 | Draft — open for feedback |

Discussion: [Issues](https://github.com/DuanWangye9527/aeis/issues)

---

## License

CC0 — public domain.

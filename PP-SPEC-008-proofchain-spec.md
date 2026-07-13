# PP-SPEC-008 · ProofChain Anchoring Specification

**Document ID:** PP-SPEC-008  
**Version:** 0.1 - Draft  
**Status:** Draft  
**License:** CC BY 4.0  
**Maintained by:** Proof Economy Standards Alliance (PESA)  
**Repository:** https://github.com/proofprotocol/proofchain-spec  
**Published:** 2026-07-13  

---

## Abstract

This specification defines ProofChain anchoring: the mechanism by which proof records are permanently timestamped using the NIST Randomness Beacon.

Proof Protocol is not a blockchain. It requires no token, no wallet, no consensus mechanism, and no chain. Timestamping is achieved via the NIST Randomness Beacon — federal infrastructure operated by the National Institute of Standards and Technology — which provides cryptographically signed random values every 60 seconds.

No chain required. No account required. No service dependency. Just the math and the federal government.

---

## Status of This Document

Draft. Subject to change before v1.0.

---

## Table of Contents

1. [Motivation](#1-motivation)
2. [NIST Randomness Beacon](#2-nist-randomness-beacon)
3. [Pre-Execution Commitment](#3-pre-execution-commitment)
4. [Post-Execution Anchoring](#4-post-execution-anchoring)
5. [ProofChain Record](#5-proofchain-record)
6. [Verification](#6-verification)
7. [Why Not Blockchain](#7-why-not-blockchain)
8. [Conformance](#8-conformance)
9. [Authors](#9-authors)

---

## 1. Motivation

A proof record without a timestamp is not a proof. Anyone can construct a receipt after the fact and claim it predates an event. The timestamp is what makes post-hoc fabrication structurally impossible.

The Proof Protocol solves this with two anchoring points:

1. **Pre-execution commitment** — before the run begins, the test parameters are committed to a NIST Beacon pulse. This proves the parameters could not have been selected after seeing the results.

2. **Post-execution anchoring** — after the run completes, the root hash of the receipt chain is anchored to a NIST Beacon pulse. This timestamps the completed evidence record.

Together these two anchors make the proof tamper-resistant by design. Not by policy. By math.

---

## 2. NIST Randomness Beacon

The NIST Randomness Beacon (beacon.nist.gov) publishes a new cryptographically signed random value every 60 seconds. Each pulse:

- Has a unique index
- Contains the SHA-512 hash of the previous pulse
- Is signed by NIST's Ed25519 key
- Is permanently archived and publicly verifiable

Because each pulse value is unpredictable before it is published, its presence in a document proves the document could not have been written before that pulse was published.

The Beacon is operated by the federal government. It requires no account, no token, no transaction fee, and has no commercial dependency.

---

## 3. Pre-Execution Commitment

Before a benchmark run begins:

1. Retrieve the current NIST Beacon pulse from `https://beacon.nist.gov/beacon/2.0/pulse/last`
2. Record the complete pulse JSON including `pulseIndex`, `timeStamp`, and `outputValue`
3. Commit the test parameters — corpus reference, case hashes, execution environment hash — to this pulse
4. Store the commitment in `nist-pulse.json` in the ProofBundle

The pre-execution commitment proves:
- The test parameters were fixed before execution began
- The vendor could not have tuned the product to specific cases after seeing them
- The run cannot be backdated

---

## 4. Post-Execution Anchoring

After the run completes and the receipt chain is verified:

1. Retrieve the current NIST Beacon pulse
2. Record the root hash of the receipt chain alongside this pulse
3. Submit to ProofRegister which records the anchor permanently

The post-execution anchor proves:
- The receipt chain existed at a specific moment in time
- The chain has not been modified since anchoring
- The record predates any subsequent claims

---

## 5. ProofChain Record

```json
{
  "proofchain_version": "1.0",
  "pre_execution": {
    "pulse_uri": "https://beacon.nist.gov/beacon/2.0/chain/2/pulse/N",
    "pulse_index": 0,
    "timestamp": "RFC 3339 UTC",
    "output_value": "hex",
    "commitment": {
      "corpus_ref": "URI",
      "corpus_hash": "sha256:hex",
      "environment_hash": "sha256:hex",
      "committed_at": "RFC 3339 UTC"
    }
  },
  "post_execution": {
    "pulse_uri": "https://beacon.nist.gov/beacon/2.0/chain/2/pulse/N",
    "pulse_index": 0,
    "timestamp": "RFC 3339 UTC",
    "output_value": "hex",
    "root_hash": "sha256:hex",
    "anchored_at": "RFC 3339 UTC"
  }
}
```

---

## 6. Verification

To verify a ProofChain anchor:

1. Retrieve the pre-execution pulse from `pre_execution.pulse_uri`
2. Confirm `output_value` matches the archived pulse
3. Confirm `pre_execution.timestamp` predates `run.started_at` in the ProofBundle
4. Retrieve the post-execution pulse from `post_execution.pulse_uri`
5. Confirm `output_value` matches the archived pulse
6. Confirm `root_hash` matches `verifier.root_hash` in `packet.json`

All verification steps are offline after the initial pulse retrieval. The NIST Beacon archive is permanent and publicly accessible.

---

## 7. Why Not Blockchain

Blockchain is not required for any function the Proof Protocol performs.

| Function | Blockchain approach | Proof Protocol approach |
|----------|--------------------|-----------------------|
| Timestamping | Transaction timestamp | NIST Beacon pulse |
| Tamper evidence | Block hash chain | Ed25519 receipt chain |
| Public record | On-chain storage | ProofRegister |
| Verification | Node RPC call | NIST Beacon archive query |

The NIST approach is strictly superior for enterprise and government use:
- No token required
- No wallet required
- No gas fees
- No consensus dependency
- No deprecation risk
- Federal government operated
- Zero regulatory exposure

Proof Protocol is not a blockchain play. It is cryptographic proof infrastructure for the agentic economy.

---

## 8. Conformance

A ProofBundle is conformant with ProofChain anchoring if:

- `nist-pulse.json` contains a complete NIST Beacon pulse JSON
- The pulse timestamp predates `run.started_at` in `packet.json`
- The pulse `outputValue` matches the archived value at `beacon.nist.gov`
- Post-execution anchoring is recorded in `proofregister.json` if ProofRegister submission was completed

---

## 9. Authors

Craig Ellrod, Founder & CEO, Nebulonium, Inc. (d/b/a HACKERverse)  
Castle Rock, Colorado  
2026-07-13

---

*CC BY 4.0 — Attribution to Craig Ellrod / Nebulonium, Inc. / HACKERverse required.*

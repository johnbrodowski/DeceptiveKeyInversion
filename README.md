# Deceptive Key Derivation

**A Single-Secret Encryption Protocol Using Layered Key Derivation**
John Brodowski
March 11, 2026

---

## Abstract

Standard symmetric encryption assumes that a correct key produces plaintext and an incorrect key produces garbage. This protocol introduces a **second hidden layer** of security by deriving a secondary key from a single shared secret using a deterministic, personal derivation function. Even if an attacker obtains the shared secret and attempts direct decryption, the ciphertext remains unintelligible. The protocol relies solely on **standard cryptographic primitives** (AES‑GCM) and a derivation function (KDF), and requires no additional secret transmission. Illustrative examples show how deterministic transformations can be used to derive unique per-implementation keys.

---

## 1. Motivation

In conventional symmetric encryption, the system fails catastrophically if the key is exposed: an attacker immediately recovers plaintext. Many applications—agents, long-running processes, and autonomous systems—store secrets in memory, creating **single points of failure**.

The goal of this protocol is simple: **make key exposure non-catastrophic** without introducing new cryptographic primitives or additional shared secrets.

---

## 2. Core Concept

The protocol introduces a **derived key KeyA**:

1. Start with the shared secret **KeyB**.
2. Apply a **deterministic derivation function** to compute KeyA.
3. Encrypt plaintext with AES‑GCM using KeyA.

KeyA is never stored or transmitted. An attacker with only KeyB cannot recover the plaintext without also knowing the derivation parameters or personal transformations.

---

## 3. Illustrative Derivation Approaches

While the underlying derivation should be implemented with a **cryptographically secure KDF**, examples illustrate the concept:

1. **Bitwise NOT of KeyB bytes**

   * Each byte of KeyB is flipped. Deterministic and reversible.

2. **Reversal**

   * Characters or bytes are reversed. Simple, deterministic, and self-inverting.

3. **Rotation derived from key sum**

   * Sum the ASCII values of KeyB, modulo N, then rotate the key string by that amount.

4. **XOR against a fixed constant**

   * Every byte is XORed with a private constant.

These steps can be used **individually or in sequence**, then formalized with a **KDF**, e.g.:

```
KeyA = HKDF(KeyB || personal_salt, info="Derived Key")
Ciphertext = AES-GCM(KeyA, plaintext)
```

---

## 4. Protocol Definition

### Roles

* **KeyB** — the shared secret.
* **KeyA** — derived encryption key.

### Sender

1. Compute KeyA = KDF(KeyB, personal parameters)
2. Encrypt plaintext using AES‑GCM(KeyA)
3. Transmit ciphertext

### Receiver

1. Compute KeyA = KDF(KeyB, same parameters)
2. Decrypt ciphertext using AES‑GCM(KeyA)
3. Recover plaintext

### Attacker Scenario

* Intercepts ciphertext and obtains KeyB.
* Attempts AES decryption with KeyB → output is garbled.
* Without knowledge of derivation function or parameters, attacker cannot efficiently guess KeyA.

---

## 5. Security Properties

1. **Layered secrecy:** KeyB alone is insufficient.
2. **Standard primitives:** Uses AES‑GCM; no custom ciphers.
3. **Decoupled from ciphertext:** Derived key is never stored or transmitted.
4. **Personalization:** Each implementation can use unique KDF parameters or transformation sequence.

---

## 6. Application: Memory‑Resident Secrets

* Useful for passwords, API keys, or session secrets in long-running agents.
* Only ciphertext and KeyB exist in memory; KeyA is derived on-demand.
* Reduces exposure during process inspection or memory dumps.

---

## 7. Limitations

* Security depends on **keeping KeyB and derivation parameters secret**.
* If the derivation is discovered, protection vanishes.
* Does not replace secure key exchange.
* Adds minimal computational overhead; modern KDFs are fast.

---

## 8. Summary

This protocol converts single-key exposure into a **layered security problem** using a derived key. It:

* Requires **only one shared secret**
* Uses **standard cryptography (AES‑GCM + KDF)**
* Introduces a **personalized second layer**, optionally using sequential derivation steps

The approach is **safe, practical, and demonstrably stronger than using the shared key directly**, while remaining suitable for open-source publication and peer review.

---

**John Brodowski | March 11, 2026 | github.com/johnbrodowski**
 

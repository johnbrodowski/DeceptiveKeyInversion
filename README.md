Deceptive Key Inversion
A Single-Secret Encryption Protocol Using Jumbled Key Derivation
John Brodowski
March 6, 2026


Abstract
Standard encryption assumes that a correct key produces plaintext and an incorrect key produces garbage. This paper describes a protocol that deliberately exploits that assumption as a security layer. By encrypting data with a derived, jumbled variant of the real key, the system ensures that an attacker who obtains the real key and performs a correct decryption still receives what appears to be garbled output. The protocol requires only one shared secret, uses standard unmodified encryption algorithms, and introduces a personal jumble derivation step that functions as a second independent secret without ever needing to be transmitted.

1.  The Problem With Conventional Key Security
In conventional symmetric encryption, the security model is straightforward: keep the key secret and the data is safe. This creates a single point of failure. If the key is exposed through a memory dump, a process inspection, a leak, or social engineering, the attacker has everything they need. There is no secondary barrier.
The assumption that makes this dangerous is universal and deeply held: correct key plus ciphertext equals plaintext. Every attacker operates from this assumption. Every decryption tool is built around it. It is so fundamental that it is rarely questioned.
This protocol questions it.

2.  Core Insight
The core idea is simple: encrypt data not with the key the receiver knows, but with a deterministically jumbled version of that key. The receiver derives the same jumbled version independently and decrypts correctly. An attacker with the original key applies it, gets a successful-looking decryption, and receives garbage. They have no reason to suspect the garbage is the intended output of a correct operation.
The jumble is the hidden layer. It does not need to be transmitted. It does not need to be stored. It lives in the implementation logic, invisible to anyone who does not know it exists.

3.  Protocol Definition
3.1  Roles
KeyB — the real key. This is the only value that needs to be shared between sender and receiver.
KeyA — the jumbled key. Derived from KeyB using a deterministic, private jumble function. Never transmitted.
Jumble(x) — a deterministic transformation of a key string. Personal to each implementer. Not part of the shared secret.

3.2  Sender Process
1. Derive KeyA = Jumble(KeyB)
2. Encrypt plaintext using KeyA
3. Transmit the ciphertext

3.3  Receiver Process
4. Derive KeyA = Jumble(KeyB)  [same derivation, independent computation]
5. Decrypt ciphertext using KeyA
6. Recover plaintext

3.4  Attacker Scenario
An attacker intercepts the ciphertext and obtains KeyB through any means — memory inspection, key theft, or forced disclosure. They apply KeyB to the ciphertext using standard decryption. Because the data was encrypted with KeyA, not KeyB, the output is indistinguishable from random noise. The attacker concludes they have the wrong key and moves on. They are correct that KeyB alone is insufficient. They have no indication that a jumble step exists.

4.  The Jumble Function
The jumble function is a deterministic transformation applied to KeyB to produce KeyA. It must satisfy three properties:
Deterministic — given the same input, it always produces the same output.
Non-obvious — the relationship between KeyB and KeyA should not be visually or computationally apparent.
Personal — each implementer defines their own jumble. There is no standard jumble to attack.

Example jumble approaches for a string key such as "N*)B*Vob9pnM)9N*OBIV^CRVUIBO*N(P":

Bitwise NOT of each character byte:
Each byte of the key string is flipped at the bit level. This is self-reversing: NOT(NOT(x)) = x. The resulting key is entirely different in appearance with no visual relationship to the original.

Reverse the string:
The character order of the key is reversed. Simple and fast. Self-reversing: Reverse(Reverse(x)) = x.

Key-derived rotation:
Sum the ASCII values of KeyB, take modulo N, and rotate the key string by that amount. The rotation amount is derived from the key itself, requiring no additional parameters.

XOR each byte against a constant:
XOR every byte of the key against a fixed constant (e.g. 0x5A). Self-reversing with the same constant. Requires the constant to be part of the private implementation.

Any of these, or any combination, constitutes a valid jumble. The implementer chooses. The choice is never recorded, transmitted, or documented outside the implementation itself.

5.  Security Properties
5.1  Key Exposure Does Not Break the System
Conventional encryption fails completely on key exposure. This protocol degrades gracefully. An attacker with KeyB can decrypt the ciphertext — and receive the output of decrypting data that was encrypted with a different key. The output is computationally indistinguishable from a wrong-key decrypt. The attacker has no signal that they are one derivation step away from the plaintext.

5.2  The Jumble Is a Second Secret That Does Not Look Like One
Traditional two-key systems require distributing two keys. This protocol achieves two-key security with one distributed key because the second key is never a key — it is a transformation rule embedded in code. An attacker who fully understands the concept of the protocol but does not know the specific jumble implementation still cannot recover the plaintext.

5.3  No New Cryptographic Primitives
The underlying encryption algorithm is entirely standard and unmodified. AES, ChaCha20, or any symmetric cipher can be used without alteration. The security properties of the chosen cipher are fully preserved. This protocol adds a layer above the cipher, not inside it.

5.4  Personalization Eliminates Universal Attacks
Because every implementer defines their own jumble, there is no universal jumble dictionary, no rainbow table, and no standard attack surface. Breaking one implementation reveals nothing about any other implementation using the same protocol.

6.  Application: Protecting Secrets in Memory
The immediate practical motivation for this protocol is the storage of passwords and API keys in running application memory. In long-running agent systems and autonomous software, secrets must persist in memory for the duration of operation. A process memory dump is a realistic attack vector.
Under this protocol, what lives in memory is ciphertext encrypted with KeyA. KeyA itself is never stored — it is derived on demand from KeyB via the jumble function. KeyB may be stored, provided to the application at startup, or itself held in a secure enclave. A memory dump yields ciphertext and possibly KeyB. The attacker applies KeyB, gets noise, and does not know a jumble derivation step exists.
This can be further hardened by implementing the jumble derivation and decryption in a separate process communicating over a named pipe, such that the derived KeyA and the plaintext exist in memory only for the duration of a single decryption call and in a separate memory space from the main application.

7.  Limitations and Honest Assessment
This protocol provides defense in depth, not a replacement for strong key management. The following limitations apply:
If an attacker has access to the source code or binary and can identify the jumble function, the second layer is defeated. The jumble must be treated as a secret implementation detail, not just a pattern.
The security of the protocol ultimately depends on the quality of the underlying cipher and the secrecy of KeyB. A weak cipher or a leaked KeyB still represents a serious exposure, though the jumble layer delays and complicates exploitation.
This protocol does not address key exchange. KeyB must be shared through a secure channel before communication begins.
The jumble function adds a small computational step to every encryption and decryption operation. For simple transformations like bitwise NOT or string reversal, this overhead is negligible.

8.  Summary
Deceptive Key Inversion is a simple, practical protocol that exploits the universal assumption that correct key plus ciphertext equals plaintext. By encrypting with a jumbled derivative of the shared key, the system ensures that key exposure produces a plausible and convincing decryption failure rather than plaintext recovery.
The protocol has three defining properties:
One shared secret. KeyB is the only value that needs to be distributed.
Standard primitives. The underlying cipher is unmodified and fully trusted.
Personal second layer. The jumble function is private to each implementer and constitutes an undistributed second secret.

It does not invent new mathematics. It does not replace existing security infrastructure. It takes something everyone already uses and makes it behave in a way that is foreign to every assumption an attacker brings to the problem — without changing what it is.



John Brodowski  |  March 6, 2026  |  github.com/johnbrodowski

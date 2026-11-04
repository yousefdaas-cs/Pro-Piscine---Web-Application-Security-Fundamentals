# Padding Oracle Attack Exploit Laboratory

An educational guide and reference repository explaining the mechanics of the **Padding Oracle Attack**. 

This document clarifies common industry misconceptions regarding the term "Oracle," details how cryptographic block padding works under the hood, and provides concrete scenarios demonstrating how this vulnerability can be exploited to achieve full decryption and privilege escalation.

---

## Clarifying the Misconception: Database Padding vs. Cryptographic Oracles

Before diving into the cryptographic exploit, it is vital to clear up a major terminology pitfall. 

### What This Is NOT (Database Padding)
In database administration (such as Oracle Database systems), **padding** refers to string alignment functions used to format exact character lengths or mask sensitive data:
* **String Alignment:** Transforming sequence numbers (`1`, `42`) into fixed lengths (`000001`, `000042`) using `LPAD` or `RPAD`.
* **Data Masking:** Hiding Credit Cards or Social Security Numbers via query manipulation:
  ```sql
  -- Masks a credit card to output: ************1234
  LPAD(SUBSTR(credit_card, -4), 16, '*')
  ```

### What This IS (Cryptographic Oracles)
A **Padding Oracle Attack** has **nothing to do with Oracle Database software**. 
* **The Oracle:** In cryptography, an "oracle" is any system or server that responds to inquiries by leaking a hint. 
* **The Attack:** It is a side-channel exploit targeting block ciphers. It allows an attacker to **completely decrypt data without knowing the key**, simply by observing how a server responds to padding errors.

---

## 1. The Core Concept: Block Padding (PKCS#7)

When data is encrypted using algorithms like AES or DES in **CBC (Cipher Block Chaining) mode**, the plaintext must be divided into fixed-size blocks (typically 16 bytes for AES). 

Because real-world data rarely fits perfectly into these block boundaries, the system applies a padding standard known as **PKCS#7**:
* If a block lacks $N$ bytes, the value of the byte $N$ is repeated $N$ times to fill the block.
* **Example:** If a 16-byte block has 13 bytes of data, it requires 3 bytes of padding. The block is padded with three `03` bytes:

When the server receives an encrypted message, the very first operation it performs post-decryption is validating this trailing padding structure.

---

## ⚙️ 2. How the Attack Works (Step-by-Step)

An attacker intercepts an encrypted payload (such as a session cookie or auth token) and exploits the decryption routine as a guessing game:

[Attacker] -- Sends Modified Ciphertext --> [Server]
|
[Attacker] <-- Evaluates Leaked Response ------+ (Leaks Padding Validity)

1. **The Tampering:** The attacker isolates a target ciphertext block and modifies its bytes to manipulate the decrypted output of the next block.
2. * **The Server Response (The Oracle Flaw):** The server decrypts the altered block and checks the PKCS#7 validation. It leaks its evaluation status based on its behavior:
   **Invalid Padding:** The server returns a specific error (`Invalid Padding`), a distinct HTTP 500 status code, or responds faster/slower than usual.
   * **Valid Padding:** The server successfully strips the padding but fails at the application layer, returning a generic error (`Invalid Username/Password` or HTTP 200/403).
3. **Blind Decryption:** Since an attacker knows that a successful padding check means the block ended in a valid PKCS#7 sequence (e.g., `01`), they can systematically deduce the plaintext of the entire block—one byte at a time.

---

## Important Distinction: Padding Oracle vs. CBC Bit-Flipping

These two attacks are closely related but rely on different execution patterns:
* **Padding Oracle Attack:** Requires sending **hundreds of automated requests** to an online server, evaluating error responses (`ERROR!` vs standard application responses) to dynamically calculate plaintext bytes blindly.
* **CBC Bit-Flipping Attack:** A purely mathematical, offline calculation. If you **already know** the plaintext of a block (e.g., `user=guest`), you can apply a direct XOR mask to the preceding Initialization Vector (IV) or ciphertext block to systematically force the decrypted plaintext to alter to a value of your choice (e.g., `user=admin`).

---

## Exploitation Scenarios

### Scenario A: Session Token Decryption (Information Disclosure)
A web application tracks user sessions by storing an encrypted cookie named `AUTH_SESSION`. The developer uses AES-CBC encryption but forgets to implement an integrity check (like an HMAC).

* **The Setup:** An attacker captures their own session cookie: `4a7b9c...`
* **The Vulnerability:** When the web server receives an invalid cookie, its error logging is overly descriptive. If the padding is incorrect, it answers with `HTTP 500 Internal Server Error (Padding Exception)`. If the padding is correct but the session data is garbage, it returns `HTTP 403 Forbidden`.
* **The Exploit Execution:** 
  1. The attacker deploys an automated script to systematically cycle the last byte of the first block through values `0x00` to `0xFF`.
  2. The script filters for the single request that triggers an `HTTP 403 Forbidden` response instead of an `HTTP 500`.
  3. Using the equation $\text{Plaintext} = \text{Decrypted Byte} \oplus \text{Ciphertext Byte}$, the script extracts the character string byte-by-byte.
* **The Impact:** The attacker extracts confidential parameters inside the session cookie, exposing secret internal IDs, user email addresses, and structural API keys.

### Scenario B: Privilege Escalation via Core Configuration Spoofing
An enterprise application accepts encrypted configuration parameters passed back and forth via an unauthenticated URL query parameter string: `?cfg=6f2d91...`. 

* **The Setup:** The application expects a serialized plaintext block structure containing configuration logic: `uid=1002&role=user&exp=2028`.
* **The Vulnerability:** The server's API gateway throws a structural timestamp validation fault if the padding structure decodes properly, but returns a generic decryption runtime fault if the PKCS#7 validation fails.
* **The Exploit Execution:**
  1. The attacker utilizes the padding oracle attack framework to completely recover the structure of the `?cfg=` block without knowing the master system key.
  2. Once the baseline mapping is recovered, the attacker manipulates the preceding blocks to execute a controlled bit-flip over the decrypted intermediate state.
  3. The attacker craftily recalculates valid padding bytes for the new payload length.
* **The Impact:** The attacker crafts a validly padded ciphertext payload that reads `uid=0001&role=root&exp=2035`, gaining unauthorized administrative management control across the targeted platform infrastructure.

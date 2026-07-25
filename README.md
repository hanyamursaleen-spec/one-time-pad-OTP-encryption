# One-Time Pad (OTP) Encryption System

A working implementation of Shannon's **perfectly secret** cipher: the one-time pad.
Includes a Python module/CLI and a self-contained browser interface, both implementing
the exact same algorithm so ciphertext and keys are interchangeable between them.

```
C = (M + K) mod 256      encryption
M = (C - K) mod 256      decryption
```

Where `M` is the message as bytes (0–255 each, extended ASCII / latin-1), and `K` is a
truly random key at least as long as `M`.

## Files

| File          | What it is                                                              |
|---------------|--------------------------------------------------------------------------|
| `otp.py`      | Core encrypt/decrypt logic + command-line interface. No dependencies beyond the Python standard library. |
| `index.html`  | Browser-based interface (blue/black theme) for encrypting/decrypting interactively. |
| `style.css`   | Stylesheet for `index.html`. Must stay in the same folder as the HTML file. |

## Why this is secure (and what "secure" actually means here)

This isn't security through computational difficulty, like AES or RSA — it's
**information-theoretic** security. Proven by Claude Shannon in 1949: if the key is
truly random, at least as long as the message, used only once, and kept secret, then
a given ciphertext is equally consistent with *every possible* plaintext of that
length. There is no mathematical foothold for any attacker — human, classical
computer, or AI — to exploit, because the ciphertext simply doesn't encode enough
information to distinguish the real plaintext from any other.

That guarantee depends entirely on three rules. Break any one of them and the "one-time
pad" becomes an ordinary, breakable stream cipher:

1. **The key must be truly random.** Both `otp.py` (via Python's `secrets` module) and
   `index.html` (via `crypto.getRandomValues`) use their platform's cryptographically
   secure random source — never a plain pseudo-random generator.
2. **The key must never be reused**, in whole or part, for a second message. Key reuse
   is exactly how the real-world VENONA project broke Soviet OTP traffic in the 1940s —
   not through breaking the math, but because keys were reused.
3. **The key must be kept as secret as the message**, exchanged securely, and destroyed
   after use.

This tool enforces rule 1 in code and validates key length against rule requirements,
but rules 2 and 3 are operational — no software can guarantee them once a key leaves
the program. Treat this as a demonstration/learning tool for the algorithm rather than
a channel for real secrets on a machine you don't fully trust.

## Requirements

- **Web interface:** any modern browser. No installation, no internet connection, no
  server needed — just open the file.
- **Python CLI:** Python 3.6 or later. No external packages required.

## Using the web interface

1. Open `index.html` in a browser (double-click it, or drag it into a browser window).
2. Keep `index.html` and `style.css` in the same folder — the page loads the stylesheet
   by relative path.
3. **To encrypt:** type your message, click "Generate Random Key" (or paste an existing
   hex key), then click "Encrypt." Save both the ciphertext and the key — you'll need
   the key to decrypt later.
4. **To decrypt:** paste the ciphertext hex and the matching key hex into the Decrypt
   panel and click "Decrypt."

## Using the Python CLI

Open a terminal, `cd` into the project folder, then:

**Generate a standalone key:**
```bash
python3 otp.py genkey --length 16
```

**Encrypt a message** (auto-generates a matching-length key if you don't supply one):
```bash
python3 otp.py encrypt --message "Hello World"
```
This prints the ciphertext (hex) and the key (hex). Save both.

**Encrypt with a specific key:**
```bash
python3 otp.py encrypt --message "Hello World" --key-hex <your key hex>
```

**Decrypt:**
```bash
python3 otp.py decrypt --cipher-hex <ciphertext hex> --key-hex <key hex>
```

On Windows, use `python` instead of `python3` if that's what your system has on PATH.

## Using the Python module in your own code

```python
import otp

key = otp.generate_key(20)                      # truly random, CSPRNG-backed
ciphertext = otp.encrypt_text("Hello World!!!!", key)
plaintext = otp.decrypt_to_text(ciphertext, key)
assert plaintext == "Hello World!!!!"
```

## Compatibility between the two implementations

The Python and JavaScript code use identical logic: bytes 0–255, elementwise mod-256
addition/subtraction, Latin-1 text encoding. A key and ciphertext generated in the
Python CLI will decrypt correctly in the web interface, and vice versa — just make
sure the message text only uses characters in the 0–255 range (plain ASCII text always
qualifies).

## Limitations/things this is NOT

- **Not a replacement for TLS/HTTPS, PGP, Signal, etc.** for everyday secure
  communication — the practical burden of generating, exchanging, and never reusing
  keys equal in length to every message makes OTP impractical outside specific
  high-assurance use cases.
- **Not protected against key mismanagement.** If you reuse a key, store it insecurely,
  or lose it, the security guarantee is gone or the ciphertext becomes unrecoverable.
- **Not a file-encryption tool** out of the box — the CLI currently handles text
  messages. (Ask if you'd like a `--file` mode added.)

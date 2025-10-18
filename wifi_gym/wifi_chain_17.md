# Chain #17: Cisco Password Cracking

## Goal & Scope

Understand Cisco password types, their weaknesses, and how to crack them:

- Identify password types in configs.
- Recognize which are plaintext, weakly hashed, or strongly hashed.
- Apply cracking tools (John the Ripper, Hashcat, scripts).
- Know best practices and recommendations.

---

## Cisco Password Types Overview

| Type | Storage Method | John format / notes                          | Hashcat mode | Recommendation                               |
| ---- | -------------- | -------------------------------------------- | ------------ | -------------------------------------------- |
| 0    | Plaintext      | — (plaintext)                                | —            | Do not use                                   |
| 4    | Weak SHA‑256   | Raw‑SHA256 (`Raw-SHA256`)                    | 5700         | Do not use                                   |
| 5    | MD5‑crypt      | `md5crypt`                                   | 500          | Avoid; only if 8/9 not possible              |
| 6    | AES reversible | — (AES reversible; requires master key)      | —            | Use only when reversible encryption required |
| 7    | Vigenère (obf) | — (obfuscation; decryptor scripts available) | —            | Do not use                                   |
| 8    | PBKDF2‑SHA256  | `pbkdf2-hmac-sha256`                         | 9200         | Recommended                                  |
| 9    | Scrypt         | `scrypt`                                     | 9300         | Recommended                                  |

---

## Type 0 — Plaintext

Example:

```config
username tom password 0 P@ssw0rd
```

- No hashing/encryption.
- Anyone can read directly.
- **Critical risk**.

---

## Type 4 — Weak SHA-256

Example:

```config
username bob secret 4 g1rTD89b38NIXbGJ...
```

- Flawed implementation: 1 iteration of SHA-256, no salt.
- Deprecated since 2013.

Cracking:

```bash
# John
john --format=Raw-SHA256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 5700 -O -a 0 hash /usr/share/wordlists/rockyou.txt
```

---

## Type 5 — MD5-crypt

Example:

```config
username bob secret 5 $1$w1Jm$bCt7eJNv.CjWPwyfWcobP0
```

- Uses MD5 with 1000 iterations, 32-bit salt.
- Weak by modern standards.

Cracking:

```bash
# John (parallel)
john --format=md5crypt --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash

# Hashcat
hashcat -m 500 -O -a 0 hash /usr/share/wordlists/rockyou.txt
```

---

## Type 6 — AES reversible

Configuration:

```console
configure terminal
(config)# password encryption aes
(config)# key config-key password-encrypt MyS3cretkey
(config)# username alice password cisco123
end
```

Stored as:

```config
username alice password 6 fZbe^WdXO...
```

- Encrypted with AES, reversible with master key.
- Stronger than Type 7, but reversible = caution.

---

## Type 7 — Vigenère cipher

Example:

```config
username bob password 7 08116C5D1A0E550516
```

- Obfuscation only, reversible instantly.

Cracking:

```bash
wget https://raw.githubusercontent.com/theevilbit/ciscot7/master/ciscot7.py
python ciscot7.py -d -p 08116C5D1A0E550516
```

- Online tools exist but unsafe for client data.

---

## Type 8 — PBKDF2-SHA256

Example:

```config
username tom secret 8 $8$kMehFGHe4ew.chRm...
```

- Salted, 20,000 iterations.
- Strong, NSA-recommended.

Cracking:

```bash
# John
john --format=pbkdf2-hmac-sha256 --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash

# Hashcat
hashcat -m 9200 -a 0 hash /usr/share/wordlists/rockyou.txt
```

---

## Type 9 — Scrypt

Example:

```config
username tom secret 9 $9$ApsgnGtdkTswkfjucj...
```

- Salted, 16,384 iterations, memory-hard.
- Strongest, industry recommended.

Cracking:

```bash
# John
john --format=scrypt --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash

# Hashcat
hashcat -m 9300 -a 0 --force hash /usr/share/wordlists/rockyou.txt
```

---

## Security Recommendations

- **Avoid:** Types 0, 4, 5, 7.
- **Use cautiously:** Type 6 only if reversible encryption is required.
- **Prefer:** Types 8 and 9.
- Follow NSA Cisco Password Best Practices.

---

## Checklist

1. Identify Cisco password type from config.
2. Immediately flag Types 0/7 (plaintext/obfuscated).
3. Apply John/Hashcat for Types 4/5/8/9.
4. Use decryptor for Type 7 if testing.
5. Recommend migration to Types 8/9.
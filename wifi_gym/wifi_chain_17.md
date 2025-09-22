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

---

## Anki Deck

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

A17aa11BB	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	What are Cisco <code>Type 0</code> passwords?	Plaintext passwords stored in configs, immediately readable. Critical risk.	wifi
B17bb22CC	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Show a config example of a Cisco <code>Type 0</code> password.	<code>username tom password 0 P@ssw0rd</code>	wifi
C17cc33DD	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password type was historically flawed by using unsalted single SHA-256 iteration?	<code>Type 4</code>	wifi
D17dd44EE	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Hashcat mode flag corresponds to Cisco <code>Type 4</code> (unsalted SHA-256)?	<code>-m 5700</code>	wifi
E17ee55FF	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Provide an example Hashcat command for cracking Cisco <code>Type 4</code> (mode <code>-m 5700</code>) using <code>rockyou.txt</code>.	<code>hashcat -m 5700 -O -a 0 hash /usr/share/wordlists/rockyou.txt</code>	wifi
F17ff66GG	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password type uses MD5-crypt with ~1000 iterations and a salt?	<code>Type 5</code>	wifi
G17gg77HH	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Hashcat mode flag corresponds to Cisco <code>Type 5</code> (MD5-crypt)?	<code>-m 500</code>	wifi
H17hh88II	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Give a config example of a Cisco <code>Type 5</code> password.	<code>username bob secret 5 $1$w1Jm$bCt7eJNv.CjWPwyfWcobP0</code>	wifi
I17ii99JJ	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password type uses reversible AES encryption?	<code>Type 6</code>	wifi
J17jj00KK	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Why is <code>Type 6</code> safer than <code>Type 7</code> but still a risk?	It uses AES instead of simple obfuscation, but the encryption is reversible if the AES key or method is known.	wifi
K17kk11LL	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	What weak cipher underlies Cisco <code>Type 7</code> passwords?	A Vigenère-like obfuscation (simple reversible cipher with known key), easily reversible.	wifi
L17ll22MM	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which script/tool is commonly used to decrypt Cisco <code>Type 7</code> passwords offline?	<code>ciscot7.py</code>	wifi
M17mm33NN	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Hashcat mode flag corresponds to Cisco <code>Type 8</code> (PBKDF2-SHA256)?	<code>-m 9200</code>	wifi
N17nn44OO	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which John the Ripper format name corresponds to cracking Cisco <code>Type 8</code>?	<code>pbkdf2-hmac-sha256</code>	wifi
O17oo55PP	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Hashcat mode flag corresponds to Cisco <code>Type 9</code> (scrypt)?	<code>-m 9300</code>	wifi
P17pp66QQ	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which John the Ripper format corresponds to Cisco <code>Type 9</code>?	<code>scrypt</code>	wifi
Q17qq77RR	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password types should always be treated as insecure and avoided?	Types <code>0</code>, <code>4</code>, <code>5</code>, and <code>7</code>.	wifi
R17rr88SS	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password types are recommended (NSA / modern guidance)?	Types <code>8</code> and <code>9</code>.	wifi
S17ss99TT	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Why is <code>Type 4</code> considered weaker than <code>Type 5</code> despite using SHA-256?	Because <code>Type 4</code> used no salt and only a single iteration, while <code>Type 5</code> (MD5crypt) uses a salt and many iterations.	wifi
T17tt00UU	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which Cisco password type is essentially plaintext with simple obfuscation?	<code>Type 7</code>	wifi
U17uu11VV	Basic (type in the answer)	Ops::WiFi Gym::Chain 17: Cisco Passwords	Which <code>john</code> <code>--format</code> value should you use to crack Cisco <code>Type 5</code> (MD5crypt)?	<code>--format=md5crypt</code>	wifi
V17vv22WW	Basic	Ops::WiFi Gym::Chain 17: Cisco Passwords	What is the recommended remediation if you find Cisco <code>Type 7</code> passwords in configs?	Immediately rotate those secrets and migrate to stronger types (e.g., <code>Type 8</code> or <code>Type 9</code>), and remove plaintext/obfuscated secrets from configs.	wifi
```
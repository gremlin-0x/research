# Chain #11: Advanced Password Cracking Techniques (Rules, Masks, Combinator & Hybrid)

## Goal & Scope

Master advanced Hashcat attack modes that extend beyond simple dictionary cracking:

- **Rules** → runtime transformations of dictionary entries.
- **Masks** → brute force with structure.
- **Combinator** → join two wordlists.
- **Hybrid** → combine dictionary + mask.

---

## Part 1 — Hashcat Rules

### Step 1 — What Are Rules?

Rules modify dictionary words in real time, covering human password patterns like appending numbers or substituting letters.

### Step 2 — Common Rule Operators

- `c` → Capitalize first letter, lowercase rest. `password` → `Password`    
- `C` → Lowercase first, uppercase rest. `password` → `pASSWORD`
- `t` → Toggle case of all characters. `password` → `PASSWORD`
- `T2` → Toggle case at position 2. `password` → `paSsword`
- `$1` → Append 1. `password` → `password1`
- `^1` → Prepend 1. `password` → `1password`
- `r` → Reverse word. `password` → `drowssap`
- `sa@` → Replace `a` with `@`. `password` → `p@ssword`
- `d` → Duplicate. `password` → `passwordpassword`
- `z5` → Duplicate first char 5 times. `password` → `pppppassword`

### Step 3 — Custom Rule Example

```bash
printf "winteriscoming\n" > wordlist
printf "T0si1se3so0$!\n" > rule
```

- `T0`: Capitalize first letter.
- `si1`: Replace i→1.
- `se3`: Replace e→3.
- `so0`: Replace o→0.
- `$!`: Append `!`.

Test with:

```bash
hashcat -r rule wordlist --stdout
```

Output: `W1nt3r1sc0m1ng!`

### Step 4 — Built‑In Rule Files

Hashcat ships rule files in `/usr/share/hashcat/rules/`.  
Examples: `best64.rule`, `T0XlC.rule`, `dive.rule`.

Apply during cracking:

```bash
hashcat -m 22000 hash wordlist.txt -r /usr/share/hashcat/rules/best64.rule
```

### Step 5 — Word Length Filtering

Use reject rules to enforce WPA length limits:

- `>8` → Reject shorter than 8.    
- `<63` → Reject longer than 63.

---

## Part 2 — Mask Attacks

### Step 1 — What Is a Mask?

Masks (`-a 3`) brute force passwords following a pattern.

### Step 2 — Mask Symbols

- `?l`: lowercase letters.    
- `?u`: uppercase letters.
- `?d`: digits.
- `?s`: specials.
- `?a`: all printable.
- `?b`: all bytes.

### Step 3 — Fixed Length Mask Example

Suppose password is 12 chars: first uppercase, 7 lowercase, last 4 digits.

```bash
hashcat -m 22000 -a 3 hash '?u?l?l?l?l?l?l?l?d?d?d?d'
```

### Step 4 — Variable Length Mask

If uncertain, use increments:

```bash
hashcat -m 22000 -a 3 hash --increment --increment-min=8 --increment-max=12 '?u?l?l?l?d?d?d'
```

### Step 5 — Custom Character Sets

Limit positions to specific sets:

```bash
hashcat -m 22000 -a 3 hash -1 ?d?s '?u?l?l?l?1?1?1'
```

---

## Part 3 — Combinator Attacks

### Step 1 — What Is a Combinator Attack?

Concatenates words from two dictionaries.

### Step 2 — Example

**wordlist1:** summer, secret  
**wordlist2:** 2023, password

### Step 3 — Command

```bash
hashcat -m 22000 -a 1 wpa_hash wordlist1 wordlist2
```

Generates: `summer2023`, `secretpassword`.

---

## Part 4 — Hybrid Attacks

### Step 1 — What Is a Hybrid Attack?

Mixes dictionary with mask.

- `-a 6`: Dict + Mask (append).    
- `-a 7`: Mask + Dict (prepend).

### Step 2 — Mode 6 (Dict + Mask)

Append 3 digits to each word:

```bash
hashcat -m 22000 -a 6 wpa_hash wordlist.txt ?d?d?d
```

Generates: `password123`, `welcome456`.

### Step 3 — Mode 7 (Mask + Dict)

Prepend 3 digits:

```bash
hashcat -m 22000 -a 7 wpa_hash ?d?d?d wordlist.txt
```

Generates: `123password`, `456welcome`.

---

## Checklist

1. Choose dictionary.    
2. Use rules with `-r` to expand candidates.
3. Use masks (`-a 3`) for structural brute force.
4. Use combinator (`-a 1`) to join words.
5. Use hybrid (`-a 6/7`) for words + digits/patterns.
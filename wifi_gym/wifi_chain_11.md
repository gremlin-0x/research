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

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

X8r2K7m1P5	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is the purpose of Hashcat rules?	To modify dictionary words in real time to cover human password patterns.	wifi
J9t4M2q8L6	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule appends the digit <code>1</code> to a word?	<code>$1</code>	wifi
Q1b7N6y3V9	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule prepends the digit <code>1</code> to a word?	<code>^1</code>	wifi
M6z2H3a9C1	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule substitutes every letter <code>a</code> with <code>@</code>?	<code>sa@</code>	wifi
R5k8D2f4B7	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule operator reverses the word?	<code>r</code>	wifi
G3u9P4x1S0	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule operator duplicates the word?	<code>d</code>	wifi
N7c1W5e3F2	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat rule operator repeats the first character 5 times?	<code>z5</code>	wifi
H2v6L4k9T8	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Apply the Hashcat rule sequence <code>T0si1se3so0$!</code> to the word <code>winteriscoming</code>. What single transformed output is produced?	<code>W1nt3r1sc0m1ng!</code>	wifi
B1y3Q9m7Z4	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat command tests a custom rule file named <code>rule</code> against the wordlist file <code>wordlist.txt</code> and prints transformed words to stdout?	<code>hashcat -r rule wordlist.txt --stdout</code>	wifi
P4s8O2d6K3	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Where are built-in Hashcat rule files located on a typical Linux installation?	<code>/usr/share/hashcat/rules/</code>	wifi
U9a5E7r3J2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Name two default Hashcat rule files.	<code>best64.rule</code>, <code>T0XlC.rule</code>	wifi
L8n2C6f9M1	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Why filter word length when cracking WPA passphrases?	Because WPA requires passphrases of length 8–63 characters; filtering avoids testing invalid lengths.	wifi
O5q7H3b1V6	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which reject rules enforce WPA length limits (minimum 8, maximum 63) in Hashcat rule syntax?	<code>>8</code> and <code><63</code>	wifi
Z3m9J2g5T4	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a mask attack and which Hashcat attack mode number implements it?	A structured brute force using masks; mode <code>-a 3</code>.	wifi
D7f1R6p3X2	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat mask enforces 1 uppercase, 7 lowercase, and 4 digits?	<code>?u?l?l?l?l?l?l?l?d?d?d?d</code>	wifi
A6e4N1x8W5	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat flags enable variable mask length between 8 and 12?	<code>--increment --increment-min=8 --increment-max=12</code>	wifi
C2j5K8o9Y7	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	How do you define a custom character set of digits+specials in Hashcat and reference it as <code>?1</code>?	Use <code>-1 ?d?s</code> and reference it as <code>?1</code>.	wifi
F4p6L7u2R8	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat command uses custom set <code>-1 ?d?s</code> with mask <code>'?u?l?l?l?1?1?1'</code> to crack hash file <code>hash</code> (mode <code>-m 22000</code>)?	<code>hashcat -m 22000 -a 3 hash -1 ?d?s '?u?l?l?l?1?1?1'</code>	wifi
K9o1G3t7E5	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a Combinator attack and which Hashcat mode uses it?	Concatenating words from two dictionaries; mode <code>-a 1</code>.	wifi
S8h2B4v9Q1	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat command runs a Combinator attack on WPA hash file <code>wpa_hash</code> using <code>wordlist1.txt</code> and <code>wordlist2.txt</code>?	<code>hashcat -m 22000 -a 1 wpa_hash wordlist1.txt wordlist2.txt</code>	wifi
E1g3N7c4H6	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Given <code>wordlist1.txt</code> = [summer, secret] and <code>wordlist2.txt</code> = [2023, password], give two example concatenated passwords produced by a Combinator attack.	<code>summer2023</code>, <code>secretpassword</code>	wifi
T5m7J1d8U2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a Hybrid attack and what are its two Hashcat modes?	Combines dictionary + mask; append mode <code>-a 6</code>, prepend mode <code>-a 7</code>.	wifi
V2c9L5w3A8	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat command appends 3 digits (mask <code>?d?d?d</code>) to each word in <code>wordlist.txt</code> for WPA hash file <code>wpa_hash</code> (mode <code>-m 22000</code>)?	<code>hashcat -m 22000 -a 6 wpa_hash wordlist.txt ?d?d?d</code>	wifi
Y6n4P8k2I7	Basic (type in the answer)	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which Hashcat command prepends 3 digits (mask <code>?d?d?d</code>) to each word in <code>wordlist.txt</code> for WPA hash file <code>wpa_hash</code> (mode <code>-m 22000</code>)?	<code>hashcat -m 22000 -a 7 wpa_hash ?d?d?d wordlist.txt</code>	wifi
W0q2R9b5Z3	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Give two example passwords generated by Hybrid append mode <code>-a 6</code> with mask <code>?d?d?d</code> on word <code>admin</code>.	<code>admin123</code>, <code>admin456</code>	wifi
```
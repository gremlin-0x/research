# Wi‑Fi Gym — Chain 11: Advanced Password Cracking Techniques (Rules, Masks, Combinator & Hybrid)

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
J9t4M2q8L6	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which rule appends the digit 1 to a word?	<code>$1</code>	wifi
Q1b7N6y3V9	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which rule prepends the digit 1 to a word?	<code>^1</code>	wifi
M6z2H3a9C1	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Which rule substitutes all <code>a</code> with <code>@</code>?	<code>sa@</code>	wifi
R5k8D2f4B7	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What does the rule operator <code>r</code> do?	Reverses the word.	wifi
G3u9P4x1S0	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What does the rule operator <code>d</code> do?	Duplicates the word.	wifi
N7c1W5e3F2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What does the rule operator <code>z5</code> do?	Repeats the first character 5 times.	wifi
H2v6L4k9T8	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What rule transforms winteriscoming into W1nt3r1sc0m1ng!? 	<code>T0si1se3so0$!</code>	wifi
B1y3Q9m7Z4	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What command tests a custom rule file called rule on wordlist.txt?	<code>hashcat -r rule wordlist.txt --stdout</code>	wifi
P4s8O2d6K3	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Where are built‑in rule files located?	<code>/usr/share/hashcat/rules/</code>	wifi
U9a5E7r3J2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Name two default Hashcat rule files.	<code>best64.rule</code>, <code>T0XlC.rule</code>	wifi
L8n2C6f9M1	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Why filter word length in WPA cracking?	Because WPA requires 8–63 character passphrases.	wifi
O5q7H3b1V6	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What reject rules enforce WPA length limits?	<code>>8</code> and <code><63</code>	wifi
Z3m9J2g5T4	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a mask attack and which mode uses it?	A structured brute force using <code>-a 3</code>.	wifi
D7f1R6p3X2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What mask enforces 1 uppercase, 7 lowercase, and 4 digits?	<code>?u?l?l?l?l?l?l?l?d?d?d?d</code>	wifi
A6e4N1x8W5	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What flags enable variable mask length between 8–12?	<code>--increment --increment-min=8 --increment-max=12</code>	wifi
C2j5K8o9Y7	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	How do you define a custom set of digits+specials in Hashcat?	Use <code>-1 ?d?s</code> and reference it as <code>?1</code>.	wifi
F4p6L7u2R8	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What command uses custom set -1 ?d?s with mask '?u?l?l?l?1?1?1'?	<code>hashcat -m 22000 -a 3 hash -1 ?d?s '?u?l?l?l?1?1?1'</code>	wifi
K9o1G3t7E5	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a Combinator attack and which mode uses it?	Concatenating words from two dictionaries; mode <code>-a 1</code>.	wifi
S8h2B4v9Q1	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What command runs a Combinator attack on WPA hash with wordlist1 and wordlist2?	<code>hashcat -m 22000 -a 1 wpa_hash wordlist1 wordlist2</code>	wifi
E1g3N7c4H6	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Give two example passwords from wordlist1=[summer, secret], wordlist2=[2023, password].	<code>summer2023</code>, <code>secretpassword</code>	wifi
T5m7J1d8U2	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What is a Hybrid attack and what are its two modes?	Combines dictionary + mask; <code>-a 6</code> (append), <code>-a 7</code> (prepend).	wifi
V2c9L5w3A8	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What command appends 3 digits to each word in wordlist.txt?	<code>hashcat -m 22000 -a 6 wpa_hash wordlist.txt ?d?d?d</code>	wifi
Y6n4P8k2I7	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	What command prepends 3 digits to each word in wordlist.txt?	<code>hashcat -m 22000 -a 7 wpa_hash ?d?d?d wordlist.txt</code>	wifi
W0q2R9b5Z3	Basic	Ops::WiFi Gym::Chain 11: Advanced Cracking	Give examples generated by Hybrid mode 6 with mask ?d?d?d on word 'admin'.	<code>admin123</code>, <code>admin456</code>	wifi
```
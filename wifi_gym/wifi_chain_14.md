# Chain #14: Generating Custom Wordlists

## Goal & Scope

Learn to generate targeted wordlists using:

- CUPP (personalized wordlists from OSINT/social data).
    
- CeWL (scraping words from websites).
    
- Crunch (systematic wordlist generator).
    

These techniques improve cracking efficiency by focusing on context‑relevant candidates instead of generic lists.

---

## Step 1 — Why Generate Custom Wordlists?

- Default lists (e.g., rockyou.txt) are large but generic.
    
- Custom lists incorporate target‑specific data.
    
- Efficiency improves by aligning guesses with real patterns.
    

---

## Step 2 — CUPP (Common User Passwords Profiler)

Tool for OSINT‑based, personal data wordlists.

```bash
cupp -i
```

- Launches interactive mode.
    
- Prompts for name, birthday, partner/pet names, hobbies, etc.
    
- Outputs tailored list.
    

Other flags:

- `-w FILENAME` → improve existing list.
    
- `-l` → download large common names.
    
- `-a` → add default creds from Alecto DB.
    
- `-q` → quiet mode.
    

Example interactive session builds `roger.txt` with 2419 candidates.

---

## Step 3 — Why CUPP Matters

- Many people reuse family/pet names and important dates.
    
- CUPP automates combining and mutating them.
    
- Enables faster cracks against personal Wi‑Fi and accounts.
    

---

## Step 4 — CeWL (Custom Word List Generator)

Web scraper that builds wordlists from site content.

```bash
cewl -d 4 -m 8 -w inlane.wordlist http://logistics.local
```

- `-d` depth of crawl.
    
- `-m` minimum word length.
    
- `-w` output file.
    
- Target URL = source of words.
    

Result: context‑specific list (company names, slogans, jargon).

---

## Step 5 — Why CeWL Matters

- Organizations often use brand slogans or jargon in passwords.
    
- Employees/bloggers reuse words from daily environment.
    
- Site scraping yields a focused wordlist aligned with target.
    

---

## Step 6 — Crunch Basics

Tool for brute force wordlist generation.

```bash
crunch <min> <max> <charset> -t <pattern> -o <file>
```

- `<min>` `<max>`: length range.
    
- `<charset>`: allowed characters.
    
- `-t` pattern with placeholders.
    
- `-o` output.
    

Pattern placeholders:

- `@` → lowercase
    
- `,` → uppercase
    
- `%` → digits
    
- `^` → symbols
    

---

## Step 7 — Crunch Example

Generate 8‑char passwords starting with `Ab` followed by 6 digits:

```bash
crunch 8 8 0123456789 -t Ab%%%%%% -o number_passwords.txt
```

- Generates 1,000,000 lines.
    
- Includes candidates like `Ab123456`.
    

---

## Step 8 — When to Use Crunch

- Password structure is partially known.
    
- Combine known prefix/suffix with brute‑force middle.
    
- Avoids wasting resources on impossible candidates.
    

---

## Step 9 — Integration Strategy

1. Start simple (rockyou).
    
2. Try CUPP for personal targets.
    
3. Try CeWL for org/brand‑related terms.
    
4. Use Crunch for structured brute‑forcing.
    

---

## Checklist

-  Gather OSINT details (names, dates, pets).
    
-  Run CUPP to generate personalized wordlist.
    
-  Crawl target site with CeWL.
    
-  Use Crunch for structured patterns.
    
-  Merge lists and run with Hashcat/Aircrack.
    

---

## Anki Deck

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

AA1bb2CC3	Basic	Ops::WiFi Gym::Chain 14: Wordlists	What does CUPP stand for and what is its purpose?	Common User Passwords Profiler, used to create personalized wordlists from OSINT and social data.	wifi
BB2cc3DD4	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	Which <code>CUPP</code> flag launches interactive guided profiling?	<code>-i</code>	wifi
CC3dd4EE5	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	Which <code>CUPP</code> flag downloads huge wordlists from the repository?	<code>-l</code>	wifi
DD4ee5FF6	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Give three types of personal info CUPP combines into passwords.	Names, birthdays, pet names, hobbies, company names.	wifi
EE5ff6GG7	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Why are CUPP wordlists effective?	They reflect real target’s life data, increasing chances of matching personal passwords.	wifi
FF6gg7HH8	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Which tool scrapes a target website to generate wordlists?	CeWL (Custom Word List Generator).	wifi
GG7hh8II9	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>CeWL</code>, what does the <code>-d</code> option control?	Crawl depth (how many link levels to follow).	wifi
HH8ii9JJ0	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>CeWL</code>, what does the <code>-m</code> option specify?	Minimum word length for the output list.	wifi
II9jj0KK1	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	Which <code>CeWL</code> command crawls <code>http://logistics.local</code> at depth 4 with min word length 8, saving to <code>inlane.wordlist</code>?	<code>cewl -d 4 -m 8 -w inlane.wordlist http://logistics.local</code>	wifi
JJ0kk1LL2	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Why does CeWL align with Wi-Fi cracking needs?	It extracts org-specific jargon that admins/users may reuse as passwords.	wifi
KK1ll2MM3	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Which tool systematically generates wordlists from patterns and charsets?	Crunch.	wifi
LL2mm3NN4	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>Crunch</code>, what does <code>@</code> represent in patterns?	Lowercase letters.	wifi
MM3nn4OO5	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>Crunch</code>, what does <code>%</code> represent in patterns?	Digits 0–9.	wifi
NN4oo5PP6	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>Crunch</code>, what does <code>^</code> represent in patterns?	Symbols (!, @, #, etc.).	wifi
OO5pp6QQ7	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	Which <code>Crunch</code> command generates 8-character passwords starting with <code>Ab</code> followed by 6 digits, saving to <code>number_passwords.txt</code>?	<code>crunch 8 8 0123456789 -t Ab%%%%%% -o number_passwords.txt</code>	wifi
PP6qq7RR8	Basic	Ops::WiFi Gym::Chain 14: Wordlists	How many lines does the Crunch example generate?	1,000,000 lines (8 MB).	wifi
QQ7rr8SS9	Basic	Ops::WiFi Gym::Chain 14: Wordlists	When is Crunch most useful compared to CUPP/CeWL?	When partial structure of password is known (e.g., prefix/suffix, length).	wifi
RR8ss9TT0	Basic	Ops::WiFi Gym::Chain 14: Wordlists	List the four recommended wordlist generation steps in order.	1. Generic list (rockyou), 2. CUPP, 3. CeWL, 4. Crunch.	wifi
SS9tt0UU1	Basic	Ops::WiFi Gym::Chain 14: Wordlists	What is the risk of using Crunch without constraints?	It produces massive lists, wasting time and resources.	wifi
TT0uu1VV2	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>CUPP</code>, what does the <code>-w</code> flag do?	Improves an existing dictionary by adding mutations.	wifi
UU1vv2WW3	Basic	Ops::WiFi Gym::Chain 14: Wordlists	What type of attacks benefit most from CeWL wordlists?	Attacks against organizations using brand terms, slogans, or product names.	wifi
VV2ww3XX4	Basic	Ops::WiFi Gym::Chain 14: Wordlists	Why is combining multiple generation methods useful?	Covers personal data (CUPP), org data (CeWL), and structural brute force (Crunch) for maximum coverage.	wifi
WW3xx4YY5	Basic	Ops::WiFi Gym::Chain 14: Wordlists	What output file did CUPP create in the Roger Penrose example?	<code>roger.txt</code> containing 2419 words.	wifi
XX4yy5ZZ6	Basic (type in the answer)	Ops::WiFi Gym::Chain 14: Wordlists	In <code>Crunch</code> patterns, what placeholder represents uppercase letters?	<code>,</code>	wifi
```
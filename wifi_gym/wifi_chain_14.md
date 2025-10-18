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
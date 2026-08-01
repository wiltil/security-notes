# Authentication — Revision Notes

---

## 🧠 Core Concepts (Read First)

### What is Authentication?
The process of **verifying identity** — confirming a user/client is who they claim to be. Since websites are exposed to anyone on the internet, robust authentication is critical to web security.

### The 3 Authentication Factors
| Factor | Also Called | Example |
|--------|-------------|---------|
| Something you **know** | Knowledge factor | Password, security question answer |
| Something you **have** | Possession factor | Phone, security token (OTP device) |
| Something you **are/do** | Inherence factor | Biometrics (fingerprint, face), behavior patterns |

> Multi-Factor Authentication (MFA) = combining 2+ of these factors.

### Authentication vs. Authorization
| | Authentication | Authorization |
|---|----------------|----------------|
| **Question answered** | "Are you who you say you are?" | "Are you allowed to do this?" |
| **Example** | Confirming `Carlos123` is really Carlos | Deciding if Carlos123 can delete another user's account |
| **Order** | Happens first | Happens after, based on identity |

**Key insight:** Authentication happens *before* authorization — you can't decide permissions for someone whose identity you haven't verified.

---

## ⚠️ How Authentication Vulnerabilities Arise

Two main root causes:

1. **Weak mechanisms** — fail to protect against brute-force attacks (no rate limiting, no lockout, weak passwords allowed).
2. **Broken authentication (logic flaws)** — poor implementation lets attackers bypass auth entirely, not just guess credentials faster.

> Because authentication is so security-critical, even small logic bugs (that might be harmless elsewhere in the app) tend to become serious vulnerabilities here.

---

## 💥 Impact of Vulnerable Authentication

| Scenario | Consequence |
|----------|-------------|
| Attacker compromises a **high-privilege account** (e.g. admin) | Full control of the application, possibly internal infrastructure access |
| Attacker compromises a **low-privilege account** | Access to sensitive data the account holds (e.g. commercially sensitive info) |
| Account has **no sensitive data at all** | Still grants access to more pages → expands attack surface for further exploits (attacks often only work from internal/logged-in pages, not public ones) |

**Takeaway:** Even a "worthless" account being compromised is a real risk — it's a foothold, not just a dead end.

---

## 🔍 Username Enumeration

**Definition:** Attacker observes differences in the website's behavior to figure out whether a given username **exists**, without needing the password.

**Where it typically happens:**
- **Login page** — valid username + wrong password behaves differently than invalid username + wrong password.
- **Registration form** — "username already taken" reveals an account exists.

**Why it matters:** Once an attacker has a confirmed list of valid usernames, brute-forcing passwords becomes far more efficient — they only need to guess *one* of two unknowns instead of both.

### Signals to Watch For (the 3 tells)

| Signal | What to look for | Best practice (often not followed) |
|--------|-------------------|--------------------------------------|
| **Status codes** | A guess returns a *different* HTTP status than the rest → likely correct username | Always return identical status code regardless of outcome |
| **Error messages** | Message differs for "bad username" vs. "bad password" — even a single stray character/typo counts, even if invisible on the rendered page | Use identical, generic error message for both cases |
| **Response times** | A slower response than usual suggests extra backend processing occurred (e.g., password check only runs if username is valid) — attacker can amplify this by submitting a very long password to exaggerate the delay | Ensure consistent processing time regardless of validity |

---

## ✅ Master Revision Table

| Concept | One-Line Definition |
|---------|----------------------|
| Authentication | Verifying *who* you are |
| Authorization | Verifying *what* you're allowed to do |
| Knowledge factor | Something you know (password) |
| Possession factor | Something you have (phone/token) |
| Inherence factor | Something you are/do (biometrics/behavior) |
| Weak authentication | No protection against brute-force |
| Broken authentication | Logic flaw lets attacker bypass auth entirely |
| Username enumeration | Detecting valid usernames via behavioral differences |
| Status code tell | Different code = strong signal of valid username |
| Error message tell | Even one differing character reveals validity |
| Response time tell | Slower response = extra backend check ran (valid username) |

---

# Brute-Force Attacks — Revision Notes

Quick-recall notes on brute-forcing, username enumeration, flawed protections, and the associated labs.

---

## 🧠 Core Concepts (Read First)

### What is a Brute-Force Attack?
Trial-and-error guessing of valid credentials, usually **automated** with wordlists of usernames/passwords. Not always random — attackers combine wordlists with **logic + public info** to make smarter guesses, massively increasing efficiency.

### Brute-Forcing Usernames — Where to Look
| Weakness | Why it helps the attacker |
|----------|----------------------------|
| Predictable pattern (`firstname.lastname@company.com`) | Usernames can be generated/guessed systematically |
| Predictable admin names (`admin`, `administrator`) | High-privilege accounts targeted directly |
| Public user profiles (viewable without login) | Display name may match the actual login username |
| HTTP responses leaking emails | Sometimes exposes admin/IT support addresses |

### Brute-Forcing Passwords — Human Behavior Beats Password Policy
Password policies (min length, mixed case, special chars) look strong on paper, but users respond predictably:
- `mypassword` → policy blocks it → user tries `Mypassword1!` or `Myp4$$w0rd`
- Forced periodic password changes → user just increments: `Mypassword1!` → `Mypassword2!`

**Takeaway:** Attackers don't need to brute-force *truly* random space — they brute-force the *predictable human patterns* people fall back on to satisfy a policy.

---

## 🔍 Username Enumeration (Recap)

**Definition:** Detecting valid usernames via behavioral differences in the app's response — massively narrows brute-force scope (attacker only has to guess passwords for known-valid usernames).

### The 3 Signals to Diff
| Signal | What reveals a valid username |
|--------|-------------------------------|
| **Status code** | Differs from the "wrong username" baseline (e.g. 200 vs 302) |
| **Error message** | Even a single stray character/typo differs between "bad username" and "bad password" messages |
| **Response time** | Slower response = extra backend step ran (e.g. password check only happens if username is valid) — amplify with a very long password |

---

## 🧪 Lab 1: Username Enumeration via Different Responses (Apprentice)

**Vulnerability:** Error message literally differs — `Invalid username` vs. `Incorrect password`.

**Method:**
1. Send login POST request to **Burp Intruder**, mark `username` as payload position.
2. **Sniper attack** + Simple list of candidate usernames → run.
3. Sort by **response length** — one entry stands out (says "Incorrect password" instead of "Invalid username") → valid username found.
4. Fix username, mark `password` as payload position, load password wordlist → run.
5. Sort by **status code** — the one **302** (redirect = success) response gives the valid password.

**Key trick:** Look for *any* differing text/length in the response — direct message difference is the giveaway here.

---

## 🧪 Lab 2: Username Enumeration via Subtly Different Responses (Practitioner)

**Vulnerability:** Same idea as Lab 1, but the error message difference is **subtle** — a trailing space instead of a period in `Invalid username or password.`

**Method:**
1. Send login POST to Intruder, mark `username` as payload.
2. Load candidate usernames.
3. Use **Grep - Extract** (Settings tab) to pull out the exact error message text as its own column.
4. Sort/compare that column — one entry has a typo (trailing space vs. full stop) → valid username.
5. Fix username, brute-force `password` list as before → look for **302** response.

**Key trick:** When messages *look* identical, use **Grep-Extract** to isolate and diff the exact text byte-for-byte — reveals typos invisible to the eye.

---

## 🧪 Lab 3: Username Enumeration via Response Timing (Practitioner)

**Vulnerability:** Backend only checks the password (a slow step) if the username is valid — creating a timing side-channel. Also has **IP-based brute-force protection**.

**Bypass for IP block:** Spoof IP using the `X-Forwarded-For` header (attacker controls the header value per-request).

**Method:**
1. Confirm `X-Forwarded-For` is honored by the server (test in Repeater).
2. Set password to a **very long string** (~100 chars) — amplifies the timing gap for valid usernames (since the backend takes measurably longer processing a long password *only if username check already passed*).
3. Send to Intruder → **Pitchfork attack** (pairs positions 1:1, not all combinations).
4. Payload position 1: `X-Forwarded-For` → **Numbers 1–100** (fresh IP per request, dodges IP block).
5. Payload position 2: `username` → candidate usernames.
6. Run → enable **Response received / Response completed** columns → find the username with a consistently longer response time.
7. New Intruder attack: fix username, pitchfork `X-Forwarded-For` (numbers again) + `password` (wordlist) → find the **302** response.

**Key trick:** Response **timing itself** is the oracle when messages/codes are identical. Long password + isolate the backend's extra processing step = visible delay.

---

## 🛡️ Flawed Brute-Force Protection (Concepts)

Two common defenses — both breakable if logic is flawed:

| Defense | How it normally works | Common Flaw |
|---------|------------------------|-------------|
| **Account locking** | Lock account after N failed attempts | Doesn't stop attacker trying **many different accounts** (1 guess each) — only protects a *specific* targeted account |
| **IP blocking** | Block IP after N failed attempts in quick succession | Counter sometimes **resets on a successful login** — attacker just logs into their *own* account periodically to reset the counter |

### Credential Stuffing (related concept)
Using massive **username:password pairs** leaked from real data breaches (not guessed). Exploits password reuse across sites. **Account locking does NOT stop this** — each username is only tried *once*, so no lockout threshold is ever crossed.

### Workaround for Account Locking (when limit is small, e.g. 3 attempts)
1. Build a list of likely-valid usernames.
2. Pick a **shortlist of passwords ≤ the lockout threshold** (e.g. max 3 guesses if limit = 3).
3. Try each password against **every** username (not each username 3x) — spreads attempts thin enough to never trigger any single account's lock, while still covering the "one user has this password" possibility.

---

## 🧪 Lab 4: Broken Brute-Force Protection, IP Block (Practitioner)

**Vulnerability:** IP gets blocked after 3 failed logins in a row — but the counter **resets whenever your own IP logs in successfully**.

**Method:**
1. Confirm: 3 wrong logins → temp IP block; logging into your own valid account resets the counter.
2. Send login POST to Intruder → **Pitchfork attack**, positions on `username` and `password`.
3. **Resource pool** → set max concurrent requests to **1** (ensures requests hit the server in the exact order you define — critical for the reset trick to work).
4. Payload 1 (`username`): alternate between **your own username** and the victim's (`carlos`), with your username appearing first, `carlos` repeated ~100 times.
5. Payload 2 (`password`): your own correct password paired with your username entries, and the candidate password list paired with each `carlos` entry.
6. Run → filter out 200s, sort by username → the single **302** for `carlos` reveals the correct password.

**Key trick:** Interleave a **known-good login** (yours) between every attempt against the victim — this resets the IP block counter before it can trigger, letting you brute-force indefinitely.

---

## 🧪 Lab 5: Username Enumeration via Account Lock (Practitioner)

**Vulnerability:** Account locking itself leaks whether a username is valid — the lockout error message is longer/different (`You have made too many incorrect login attempts.`).

**Method:**
1. Send login POST to Intruder → **Cluster bomb attack** (all combinations of multiple positions).
2. Payload position 1: `username` → candidate usernames.
3. Add a **blank/dummy second payload position** at the end of the request body, set to **Null payloads** generating 5 copies → effectively sends each username **5 times in a row** (enough to trigger account lock if the account is real).
4. Run → look for the username whose responses are **longer/different** — contains the lockout message → valid username found (its repeated bad logins triggered the lock).
5. New attack: **Sniper**, fix username, payload on `password`, add Grep-Extract for the error message column.
6. Run → find the **one response with no error message at all** → correct password.
7. **Wait ~1 minute** for the account lock to expire, then log in for real.

**Key trick:** Deliberately trigger the lockout for each candidate username (5 attempts each) — the lockout *message itself* becomes the username-validity oracle.

---

## ✅ Master Revision Table

| Lab | Vulnerability | Core Technique | Attack Type (Intruder) |
|-----|----------------|-----------------|--------------------------|
| 1 | Different error messages (`Invalid username` vs `Incorrect password`) | Compare response length/text directly | Sniper |
| 2 | Subtly different error message (typo: trailing space) | Grep-Extract to isolate exact text | Sniper |
| 3 | Timing side-channel + IP block | Long password to amplify delay; spoof IP via `X-Forwarded-For` | Pitchfork |
| 4 | IP block resets on your own successful login | Interleave your valid login between victim attempts; Resource pool = 1 | Pitchfork |
| 5 | Account lock message reveals valid username | Force lockout (5x) per username via Null payloads; check for lockout message | Cluster bomb → Sniper |

---

## 🎯 One-Line Cheat Summary
- Users mangle passwords to *just barely* satisfy policy — predictable patterns beat raw entropy.
- Enumerate the username **first** — always more efficient than blind cluster-bombing both fields.
- Diff everything: **status code, exact error text (use Grep-Extract), and response timing**.
- Amplify subtle timing gaps with an **abnormally long password**.
- Spoof `X-Forwarded-For` to dodge IP-based blocking.
- If IP block resets on success, **interleave your own good login** to keep the counter at zero forever.
- Account locking stops attacks on *one* account but not **credential stuffing** or **low-and-slow multi-account** attacks — cap password guesses to the lockout threshold and spread across many usernames instead.

---


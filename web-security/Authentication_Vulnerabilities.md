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

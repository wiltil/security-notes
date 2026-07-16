# Pre-requisite questions 
1. What is cross site scripting ? 
Cross site scripting allows an attacker to compromise the interactions the user has with vulnerable application.
Usually it works by returning malicious javascript to users.

---
## 🛡️ XSS Proof of Concept (PoC)

- **Classic PoC:** Use `alert()` to confirm execution of injected JavaScript.  
- **Chrome version >= 92 change (July 2021):** Cross‑origin iframes can no longer call `alert()`.  
- **Impact:** Some advanced XSS labs relying on iframes break with `alert()`.  
- **Recommended alternative:** Use `print()` instead — short, harmless, and reliable in Chrome.  
- **Labs update:** Affected labs now accept both `alert()` and `print()`, with instructions noting this change.  

---

## Terms to understand  : 

- **source:** It means the entry by which attacker controlled input enters.
- **Sink:** : It basically means the area where the input lands.

---
## ⚔️ Types of XSS Attacks

- **Reflected XSS** → Payload comes from current HTTP request and is reflected in the response.  
- **Stored XSS** → Payload is saved in the server/database and served to users later.  
- **DOM‑Based XSS** → Vulnerability exists in client‑side JS; unsafe handling of input in the DOM triggers execution.

---

## 🎯 What XSS Can Be Used For
- Impersonate victim user  
- Perform user actions  
- Read accessible data  
- Steal login credentials  
- Deface website  
- Inject trojan functionality  

## ⚠️ Impact of XSS
- **Low:** Brochureware apps (anonymous/public info)  
- **High:** Apps with sensitive data (banking, email, healthcare)  
- **Critical:** Compromised privileged users → full app takeover  

## 🔍 Finding & Testing XSS
- Use scanners (e.g., Burp Suite) for quick detection  
- Manual testing: inject unique input → check reflections → craft payloads  
- DOM‑based: test URL params, search DOM, review JS code for unsafe sinks  

---

## 🛡️ Content Security Policy (CSP)
- Browser mechanism to mitigate XSS and related attacks.  
- Restricts sources of scripts, styles, and other resources.  
- Can hinder exploitation of XSS, but often possible to **circumvent** CSP to exploit underlying flaws.  

### 📖 Example CSP Header
```http
Content-Security-Policy: default-src 'self';
                        script-src 'self' https://cdn.example.com;
                        style-src 'self' 'unsafe-inline';
                        img-src 'self' data: https://images.example.com;
                        object-src 'none';
                        frame-ancestors 'none';
```
## 🧩 Dangling Markup Injection
- Technique for cross‑domain data capture when full XSS isn’t possible.  
- Works by injecting **incomplete HTML tags** so the browser misinterprets page structure.  
- Can leak sensitive info (e.g., CSRF tokens) → attacker can perform unauthorized actions.  
- Useful fallback when input filters or defenses block script execution.  
---
## 1. EVERYTHING ABOUT REFLECTED XSS:
## ⚠️ Impact of Reflected XSS
- Attacker‑controlled script runs in victim’s browser → full compromise possible.  
- Can:
  - Perform any user actions  
  - View/modify user data  
  - Interact with other users (appearing as victim)  

### 📤 Delivery
- Requires external mechanism (malicious link in email, tweet, attacker’s site, etc.).  
- Can target specific users or be broad/indiscriminate.  

### 🔎 Severity
- Generally **less severe** than Stored XSS (since it needs external delivery).  
- Still dangerous: session hijacking, credential theft, user impersonation.  
---
## 🔍 Finding & Testing Reflected XSS

- **Automated:** Use Burp Suite’s scanner for quick detection.  
- **Manual Steps:**
  1. **Test all entry points** → URL params, body, path, headers.  
  2. **Inject random value** (8‑char alphanumeric) → check if reflected.  
  3. **Identify reflection context** → HTML text, attribute, JS string, etc.  
  4. **Insert candidate payload** → e.g., `<script>alert(1)</script>` in Burp Repeater.  
  5. **Try alternative payloads** if filtered/modified.  
  6. **Confirm in browser** → e.g., `alert(document.domain)` popup shows execution.

### EXAMPLE :
Web application shows this for query search: 
```https
https://insecure-website.com/search?term=gift
```
It flows into the body in the following way :
```https
<p>You searched for: gift</p>
```
Then it can be exploited given no encoding is present : 
```https
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>
```
Reflects in the page like : 

```https
<p>You searched for: <script>/* Bad stuff here... */</script></p>
```
---

## 💾 Stored XSS (Persistent)
- Occurs when attacker input is **saved** (e.g., DB, comments, profiles) and later rendered unsafely in responses.  
- Example: Malicious `<script>` in a blog comment → executes for every visitor.  

### ⚠️ Impact
- Same as reflected XSS (user actions, data theft, defacement, trojans).  
- **Key difference:** Self‑contained in the app → no external delivery needed.  
- Especially dangerous for logged‑in users → guaranteed compromise.  

### 🔍 Finding & Testing
- Use Burp Suite scanner for detection.  
- Manually:  
  - Identify **entry points** (params, body, path, headers, external feeds).  
  - Track to **exit points** (responses, logs, feeds, UI).  
  - Insert unique values → confirm persistence.  
  - Test payloads based on context (HTML, attributes, JS).  
- Stored data may be overwritten, so test systematically.  

### 💾 Stored XSS Example
- User submits comment → stored in DB → shown in responses.  
- Normal input:  
  ```http
  POST /post/comment
  comment=This+post+was+extremely+helpful

  ```

- Normal output:  
  ```http
   Output: <p>This post was extremely helpful.</p>
  ```

- Malicious input:
  ```http
  comment=<script>/* Bad stuff here... */</script>
  ```
- Malicious Output:
  ```http
  comment=<script>/* Bad stuff here... */</script>
  ```

---


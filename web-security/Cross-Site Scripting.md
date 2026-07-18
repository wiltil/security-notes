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

# ⚔️ Exploiting DOM XSS with Different Sources and Sinks

## 📌 Overview
- DOM‑based XSS occurs when **user‑controlled data (source)** flows into a **dangerous DOM method (sink)** without proper sanitization.
- Exploitability depends on:
  - Which source is used (`location.search`, `location.hash`, etc.)
  - Which sink is used (`document.write`, `innerHTML`, jQuery functions, etc.)
  - Any surrounding context (inside tags, attributes, or elements).
- Some sinks execute `<script>` directly, others require creative payloads (event handlers, image tags, iframes).

---

## 🧩 Common Sources (summarised view)
- `location.search` → query string of the URL  
- `location.hash` → fragment identifier after `#`  
- `document.cookie` → cookies  
- `document.referrer` → referring URL  
- `window.name` → window name property

## 🧩 Common Sources

These are typical **user‑controlled sources** in the DOM that can lead to XSS if passed into unsafe sinks.

- `location.search` → query string of the URL  
  ```javascript
  // URL: https://site.com/page?q=hello
  console.log(location.search); // "?q=hello"
  ```
- `location.hash` → fragment identifier after #
  ```javascript
  console.log(location.hash); // "#section1"
  ```
- `document.cookie` → cookies for the current domain
  ```javascript
  // Cookies: sessionid=abc123; theme=dark
  console.log(document.cookie); // "sessionid=abc123; theme=dark"
  ```
- `document.referrer` → URL of the previous page
 ```javascript
  // If user came from https://google.com
  console.log(document.referrer); // "https://google.com/"
  ```
- `window.name` → name of the current window (can be set via links/iframes)
  ```javascript
  window.name = "Avinash";
  console.log(window.name); // "Avinash"
  ```

- `location.href` → full URL of the current page
  ```javascript
  // URL: https://site.com/page?q=123#test
  console.log(location.href); // "https://site.com/page?q=123#test"

  ```

- `location.pathname` → path part of the URL
  ```javascript
  // URL: https://site.com/books/list
  console.log(location.pathname); // "/books/list"
  ```

- `location.origin` → scheme + host + port
  ```javascript
  // URL: https://site.com:8080/page
  console.log(location.origin); // "https://site.com:8080"
  ```

- `localStorage` → persistent key/value storage in browser
  ```javascript
  localStorage.setItem("user", "Avinash");
  console.log(localStorage.getItem("user")); // "Avinash"
  ```

- `sessionStorage` → temporary key/value storage (per tab)
  ```javascript
  sessionStorage.setItem("token", "xyz");
  console.log(sessionStorage.getItem("token")); // "xyz"
  ```
- `document.URL` → full URL of the document
  ```javascript
  console.log(document.URL); // "https://site.com/page?q=123"
  ```

- `document.domain` → domain of the current document
  ```javascript
  console.log(document.domain); // "site.com"
  ```
Any of these sources can be attacker‑controlled if the application reads from them and passes the value into a sink like document.write, innerHTML, or jQuery functions. Always trace source → sink paths when testing for DOM XSS.
---

## 🧩 Common Sinks(summarised)
- `document.write()` → writes raw HTML into the page  
- `element.innerHTML` → injects HTML into an element  
- `element.outerHTML` → replaces entire element  
- `element.insertAdjacentHTML()` → inserts HTML at a position  
- jQuery functions: `.html()`, `.attr()`, `$()` selector  

## 🧩 Common Sinks

These are DOM methods and properties that can interpret attacker‑controlled input as HTML or JavaScript. If user data flows into them, XSS may occur.



1. `document.write()` :  Writes raw HTML directly into the page.
-  Risk: Executes <script> immediately.
- **Example**:
  ```javascript
  document.write("<script>alert(1)</script>");
  ```
2. `element.innerHTML` : Injects HTML inside an element.
- Risk : <script> tags won’t run, but event handlers (onerror, onload, <img src=1 onerror=alert('asd')>) will.
- **Example**
```javascript
div.innerHTML = "<img src=x onerror=alert(1)>";
```
3. `element.outerHTML` : Replaces the entire element with new HTML.
- Risk : Can replace safe elements with malicious ones.
- **Example**
```javascript
div.innerHTML = "<img src=x onerror=alert(1)>";
```

4. `element.insertAdjacentHTML()` : HTML relative to an element.
- Risk: Same as innerHTML, but more flexible placement.
- **Example**
```javascript
div.insertAdjacentHTML("beforeend", "<img src=x onerror=alert(1)>");
```

5. `eval()` : Executes a string as JavaScript.
- Risk: Direct code execution — highest severity.
- **Example**
  ```javascript
  eval("alert(document.cookie)");
  ```

6. `setTimeout() / setInterval() with string` : Interprets string argument as code.
- Risk: Same danger as eval.
- ```javascript
  setTimeout("alert(1)", 1000);
  ```
7. Function() Constructor :  Creates a new function from a string.
- Risk: Executes arbitrary code.
- ```javascript
  new Function("alert(1)")();
  ```

8. jQuery .html() -  Sets HTML content of an element.
- Risk : Same as innerHTML.
- ```javascript
  $("#target").html("<img src=x onerror=alert(1)>");
  ```

9. jQuery .attr() : Sets attributes of elements.
- Risk : Dangerous if attacker controls attribute values.
- ```javascript
  $("#link").attr("href", "javascript:alert(1)");
  ```
10.  jQuery $() Selector : Interprets input as a selector or HTML.
- Risk : Can inject malicious HTML if input is not sanitized.
- ```javascript
  var el = $(location.hash); // attacker controls hash
  ```

11. location.href / element.src / element.action : Assigns URLs to navigation or resource attributes.
- Risk : Open redirects or JS execution via javascript: URLs.
- ```javascript
  window.location.href = "javascript:alert(1)";
  ```
12. document.location / window.location : Redirects browser to attacker‑controlled URL.
- Risk : Open redirects or JS execution via javascript: URLs.
- ```javascript
  document.location = "javascript:alert(1)";
  ```  
---

📝 Key Takeaway
- **HTML sinks** (innerHTML, outerHTML, insertAdjacentHTML, jQuery .html()) → require event handlers or special tags.
- **Execution sinks** (eval, setTimeout with string, Function) → directly run attacker code.
- **Attribute sinks** (.attr(), href, src, action) → can trigger redirects or load malicious resources.

* Always trace source → sink paths when analyzing DOM XSS.
---

## 🧩 Sources and Sinks in Third‑Party Dependencies

Modern apps often use libraries like **jQuery**, which introduce extra sources/sinks for DOM XSS.

### jQuery `.attr()` Sink
- **Code**:
  ```javascript
  // backend processing : $('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
  payload : ?returnUrl=javascript:alert(document.domain)
  Back link becomes javascript: → executes on click.
  ```

jQuery $() Selector Sink : 

code backend processing: 
```javascript
 $(window).on('hashchange', function() {
  var element = $(location.hash);
  element[0].scrollIntoView();
});
```
source : location.hash (fragment).
Exploit: Inject HTML into selector via hash.
classic payload : 
```html
<iframe src="https://vulnerable-website.com#" 
        onload="this.src+='<img src=1 onerror=alert(1)>'">
</iframe>
```
effect : Hashchange event fires → malicious element injected.

---

# ⚔️ DOM XSS in AngularJS and Reflected/Stored Data

## 📌 AngularJS DOM XSS
- AngularJS processes expressions inside **double curly braces `{{ ... }}`** when `ng-app` is present.
- This allows execution of JavaScript‑like code without `<script>` tags or event handlers.
- Example payload:
  ```html
  <div ng-app>
    {{constructor.constructor('alert(document.domain)')()}}
  </div>
  ```
Even if angle brackets < > and quotes " " are HTML‑encoded, AngularJS still evaluates expressions inside {{ ... }}.

---

## Reflected DOM XSS :
Occurs when the server reflects user input into the page, and client‑side JavaScript processes it unsafely.

Example:

```javascript
eval('var data = "reflected string"');
```
Flow:
-User input → echoed by server into HTML/JS.
-Client script reads reflected data.
-Data flows into a sink (eval, innerHTML, etc.).
-Vulnerability is partly server‑side (reflection) and partly client‑side (unsafe sink).
--- 
## Stored DOM XSS : 
Occurs when the server stores attacker input and later reflects it into a page where client‑side JS processes it unsafely.

Example:
```javascript
element.innerHTML = comment.author;
```

Flow:
- Attacker submits malicious input (e.g., comment).
- Server stores it.
- Later page loads comment and inserts it into DOM via unsafe sink.
- Payload executes in victim’s browser.

---

📝 Key Takeaways
- AngularJS: Exploitable via {{ ... }} expressions, even without <script> tags.
- Reflected DOM XSS: Input is echoed immediately and processed by client‑side JS.
- Stored DOM XSS: Input is saved on server, then later processed unsafely in the DOM.
- Defense: Encode output, sanitize input, avoid unsafe sinks (eval, innerHTML, etc.), and use frameworks securely.

---

## 🧩 Common Sinks Leading to DOM‑XSS

DOM‑XSS occurs when attacker‑controlled data flows into unsafe sinks. Below are key sinks to watch for:

### Native DOM Sinks
- `document.write()` / `document.writeln()` → write raw HTML directly.
- `document.domain` → can be manipulated for cookie scoping.
- `element.innerHTML` → injects HTML inside element.
- `element.outerHTML` → replaces entire element.
- `element.insertAdjacentHTML()` → inserts HTML relative to element.
- `element.onevent` → event handler properties (e.g., `element.onclick = userInput`).

### jQuery Sinks
- DOM manipulation functions:
  - `add()`, `after()`, `append()`, `before()`, `prepend()`
  - `insertAfter()`, `insertBefore()`
  - `replaceAll()`, `replaceWith()`
  - `wrap()`, `wrapInner()`, `wrapAll()`
- Content/attribute functions:
  - `html()` → sets HTML content.
  - `animate()` → can inject styles with attacker data.
  - `attr()` (covered earlier) → sets attributes.
- Internal functions:
  - `has()`, `constructor()`, `init()`, `index()` → can process attacker input.
  - `jQuery.parseHTML()` / `$.parseHTML()` → parse strings into DOM nodes.

---

### 📝 Key Notes
- **HTML sinks** (`innerHTML`, `outerHTML`, `insertAdjacentHTML`, jQuery `.html()`) → require event handlers or special tags for execution.
- **Direct execution sinks** (`document.write`, `onevent`) → can run `<script>` immediately.
- **jQuery sinks** expand attack surface since they wrap native DOM methods.
- Always trace **source → sink** paths when analyzing DOM XSS.

---

## 🔗 Sources → Sinks → Exploit Strategies

| **Source**            | **Sink**                  | **Exploit Strategy / Example Payload** |
|------------------------|---------------------------|----------------------------------------|
| `location.search`      | `document.write()`        | `?x=<script>alert(1)</script>` → executes immediately |
| `location.search`      | `innerHTML`               | `?q=<img src=x onerror=alert(1)>` → event handler fires |
| `location.hash`        | jQuery `$()` selector     | `#<img src=x onerror=alert(1)>` → injected via hashchange |
| `location.search`      | jQuery `.attr("href")`    | `?returnUrl=javascript:alert(1)` → link becomes JS URL |
| `document.cookie`      | `eval()`                  | `eval(document.cookie)` → attacker‑controlled cookie executes |
| `document.referrer`    | `document.write()`        | If referrer is injected: `<script>alert(document.referrer)</script>` |
| `window.name`          | `innerHTML`               | `window.name="<img src=x onerror=alert(1)>"; div.innerHTML=window.name;` |
| `localStorage`         | `insertAdjacentHTML()`    | `localStorage.setItem("x","<img src=x onerror=alert(1)>"); div.insertAdjacentHTML("beforeend", localStorage.getItem("x"));` |
| `sessionStorage`       | `outerHTML`               | Replace element with malicious HTML stored in sessionStorage |
| `location.href`        | `element.src` / `href`    | `window.location.href="javascript:alert(1)"` → executes on navigation |
| `document.URL`         | `Function()`              | `new Function("alert(document.URL)")();` |
| Any user input         | `setTimeout()` / `setInterval()` | `setTimeout("alert(1)",1000)` → string executes as code |

---

### 📝 Quick Notes
- **Text sinks** (`innerHTML`, `outerHTML`, `insertAdjacentHTML`, jQuery `.html()`) → need event handlers (`onerror`, `onload`) or special tags (`iframe`, `svg`).  
- **Execution sinks** (`eval`, `Function`, `setTimeout` with string, `document.write`) → run attacker code directly.  
- **Attribute sinks** (`.attr()`, `href`, `src`, `action`) → can trigger redirects or load malicious resources.  
- **Third‑party sinks** (jQuery functions like `.append()`, `.replaceWith()`, `.wrap()`) → wrap native DOM methods, so same risks apply.  

---

### ⚠️ Defense
- Always sanitize sources before use.  
- Encode output (`<` → `&lt;`, `>` → `&gt;`).  
- Avoid unsafe sinks (`eval`, `document.write`, `innerHTML`).  
- Upgrade libraries (e.g., jQuery) to patched versions.

---

# 🌐 Web Input → Server → Browser Flow

## 1. User Input
- User types data into a form or sends it via HTTP request.
- Example: `<script>alert(1)</script>`

---

## 2. Server Handling
- Server receives the raw input.
- It may **encode** or **sanitize** before storing/returning.
  - `<` → `&lt;`
  - `>` → `&gt;`
- **Safe server**: encodes input → prevents execution.
- **Unsafe server**: returns raw input → risk of injection.

---

## 3. Response Back to Browser
- Encoded input is sent as HTML entities.
- Browser receives:
  - Safe: `&lt;script&gt;alert(1)&lt;/script&gt;`
  - Unsafe: `<script>alert(1)</script>`

---

## 4. HTML Parsing
- Browser parses HTML into the **DOM tree**.
- Encoded entities are decoded into characters but treated as **text**, not tags.
- Raw tags (`<script>`) become actual DOM elements.

---

## 5. JavaScript Engine
- If parser encounters `<script>`, it pauses and sends code to the **JS engine**.
- Engine executes JavaScript:
  - Can manipulate DOM
  - Can send requests
  - Can run logic
- If input was encoded, engine never sees executable code.

---

## 🔒 Security Implications
- **Encoding**: Defense against XSS (cross‑site scripting).
- **Parsing**: Decides whether input is text or code.
- **JS Engine**: Attackers aim to reach this layer to execute payloads.

---

## ⚡ Example Flow

### Unsafe (XSS):
1. Input: `<script>alert(1)</script>`
2. Server: returns raw
3. Browser: parses as `<script>`
4. JS Engine: executes → popup

### Safe (Encoded):
1. Input: `<script>alert(1)</script>`
2. Server: encodes → `&lt;script&gt;alert(1)&lt;/script&gt;`
3. Browser: parses → shows literal text
4. JS Engine: nothing runs

---

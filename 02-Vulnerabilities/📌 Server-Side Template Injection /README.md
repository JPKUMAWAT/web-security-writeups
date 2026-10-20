# SSTI — Server-Side Template Injection

> Detection-focused notes aur reference material. Exploitation sirf authorized targets, bug bounty scope, ya legal practice labs (PortSwigger Academy, TryHackMe, etc.) par karo.

---

## Table of Contents

1. [What Is SSTI?](#1-what-is-ssti)
2. [SSTI vs CSTI — Confuse Mat Karo](#2-ssti-vs-csti--confuse-mat-karo)
3. [Types of SSTI](#3-types-of-ssti)
4. [Common Template Engines](#4-common-template-engines)
5. [Detection Methodology](#5-detection-methodology)
6. [Detection Flow (Diagram)](#6-detection-flow-diagram)
7. [Practical Example — Request & Response](#7-practical-example--request--response)
8. [Possible Impact](#8-possible-impact)
9. [Prevention](#9-prevention)
10. [Quick Testing Checklist](#10-quick-testing-checklist)
11. [Payload Reference (this repo)](#11-payload-reference-this-repo)
12. [Real-World Reference](#12-real-world-reference)
13. [References](#13-references)

---

## 1. What Is SSTI?

**Simple Definition:** SSTI tab hoti hai jab web application user ke input ko plain text ke bajaye **template syntax** ki tarah process kar leti hai — yani input directly template engine ke parser tak pahunch jata hai, data ki tarah nahi, code ki tarah.

**Root Cause:** User-controlled input ko template source string ke andar unsafely concatenate karna, phir usi string ko engine se render karwana.

**Flow:**

```text
User Input → Web Application → Template Source (string built with input) → Template Engine → Rendered Output
```

### Vulnerable vs Safe Pattern

```python
# Vulnerable — input directly template source me concatenate ho raha hai
template_str = "Hello " + user_input
return render_template_string(template_str)

# Safe — input data ke roop me pass ho raha hai, template source me nahi
return render_template_string("Hello {{ name }}", name=user_input)
```

Jab input data ki tarah pass hota hai, engine use literal value ki tarah treat karta hai. Jab input source string ka hissa ban jata hai, engine use *evaluate karne layak code* samajh leta hai — yehi root cause hai.

---

## 2. SSTI vs CSTI — Confuse Mat Karo

|  | SSTI | CSTI |
| --- | --- | --- |
| Execution | Server par | Browser (client) par |
| Engine | Jinja2, Twig, Freemarker, etc. | AngularJS, Vue (older versions) |
| Typical impact | Info disclosure → RCE | XSS tak |
| Example framework | Flask, Django, Spring | Legacy AngularJS apps |

Dono ka detection probe similar lag sakta hai (`{{...}}`), lekin impact aur exploitation path bilkul alag hote hain.

---

## 3. Types of SSTI

### A. Context ke hisaab se

**Plaintext context** — input seedha template body me render hota hai:

```text
Hello {{name}}
```

Yaha attacker apna poora expression likh sakta hai.

**Code context** — input kisi existing expression/variable ke andar jata hai:

```text
{{ greeting.<input> }}
```

Yaha pehle current expression se "escape" karna padta hai (jaise closing braces se), tabhi naya expression inject ho payega. Ye detect karna harder hota hai kyunki syntax errors jaldi aate hain.

### B. Output visibility ke hisaab se

**In-band (reflected) SSTI** — result response me directly dikhta hai. `{{7*7}}` bhejo, response me `49` wapas aata hai.

**Blind SSTI** — result response me dikhta nahi (e.g., email body, PDF generator, background job me render hota hai). Detect karne ke tariqe:

- **Time-based**: aisa expression inject karo jo engine ko delay kare (sleep/loop equivalent), response time measure karo.
- **Out-of-band (OOB)**: engine se ek external request trigger karwane ki koshish karo (DNS/HTTP callback service jaise Burp Collaborator), aur apne listener par hit confirm karo.

Blind SSTI real-world bug bounty targets me kaafi common hai, isliye sirf in-band probes par depend mat karo.

---

## 4. Common Template Engines

| Engine | Language / Framework | Exploitability Note |
| --- | --- | --- |
| Jinja2 | Python / Flask | Full Python object graph tak access possible — high risk |
| Django Templates | Python / Django | By design restricted sandbox — direct RCE mushkil |
| Twig | PHP / Symfony | Sandbox mode hota hai, bypass history exist karti hai |
| FreeMarker | Java | Powerful utility classes expose karta hai — high risk |
| Thymeleaf | Java / Spring | SpEL expressions ke through risky ho sakta hai |
| Handlebars | JavaScript | Logic-less by design — exploitability kam |
| Velocity | Java | Reflection-based access possible |
| ERB | Ruby | System calls tak direct access — high risk |
| Smarty | PHP | Sandbox mode available, config par depend karta hai |
| EJS | JavaScript / Node | `require` tak access ho sakta hai agar unsanitized |

**Key point:** sirf engine ka naam se risk level predict mat karo — configuration (sandboxed ya nahi) final exploitability decide karti hai.

---

## 5. Detection Methodology

Sirf authorized targets ya practice labs mein harmless probes use karo.

### Step 1 — Baseline record karo

Pehle normal, non-special input bhejo aur response note karo:

```text
Input:  ssti-test-123
Output: ssti-test-123
```

### Step 2 — Arithmetic probe

```text
Input:  {{7*7}}
Output (vulnerable): 49
Output (not vulnerable): {{7*7}}   (literal text, unchanged)
```

Alternative syntax (engine ke hisaab se try karo):

```text
${7*7}
#{7*7}
<%= 7*7 %>
```

### Step 3 — Engine fingerprinting (decision tree)

Ek simple probe se multiple engines ka fark pata chal sakta hai:

```text
Input: {{7*'7'}}

Jinja2   → 7777777   (Python string repetition)
Twig     → 49         (type coercion, string ko number treat karta hai)
Others   → error ya literal text
```

```text
Input: ${7*7}

Freemarker/Velocity-style engines → 49 evaluate ho sakta hai
Jinja2 (curly-brace only)          → literal text, unchanged
```

| Observation | Likely Direction |
| --- | --- |
| `{{7*7}}` → `49` | Curly-brace syntax engine (Jinja2, Twig family) |
| `{{7*'7'}}` → `7777777` | Jinja2 (Python semantics confirm) |
| `{{7*'7'}}` → `49` | Twig (PHP type coercion confirm) |
| `${7*7}` → `49` | Dollar-brace syntax engine (Freemarker, Velocity family) |
| `<%= 7*7 %>` → `49` | ERB (Ruby) |
| Template-related error text | Error message aksar engine/version ka naam leak karta hai — logs/error page dhyan se padho |
| Expression literal dikhta hai | Engine unsupported syntax hai, ya input escape/sanitize ho raha hai |

**Important:** Ek single observation kabhi confirm nahi karta. `49` ya ek error message milna sirf ek *signal* hai — isse pehle baseline, phir multiple probes, phir cross-verify karo.

### Step 4 — Context check

Pata karo input plaintext context me gir raha hai ya code context me (section 3A dekho) — isse pata chalega ki escape characters chahiye ya nahi.

### Step 5 — Blind indicators

Agar direct output nahi dikh raha, time-based ya OOB technique try karo (section 3B dekho) authorized scope ke andar hi.

---

## 6. Detection Flow (Diagram)

```mermaid
flowchart TD
    A[Send baseline input] --> B{Output reflects
    input unchanged?}
    B -->|No, looks odd already| C[Note anomaly, proceed carefully]
    B -->|Yes, normal| D[Send arithmetic probe e.g. {{7*7}}]
    D --> E{Does output show
    evaluated result?}
    E -->|No change / literal text| F{Try alternate syntax
    ${7*7}, #{7*7}, <%= 7*7 %>}
    F -->|Still no evaluation| G[Likely not SSTI
    in this input point]
    F -->|Evaluates| H[Engine syntax family identified]
    E -->|Evaluates e.g. 49| H
    H --> I[Send fingerprint probe {{7*'7'}}]
    I --> J{7777777 or 49?}
    J -->|7777777| K[Jinja2 - Python semantics]
    J -->|49| L[Twig - PHP type coercion]
    J -->|Error / other| M[Check docs for that
    syntax family's engines]
    K --> N[Confirm context:
    plaintext or code]
    L --> N
    M --> N
    N --> O{Output visible
    directly in response?}
    O -->|Yes| P[In-band SSTI confirmed]
    O -->|No| Q[Try time-based / OOB probes
    for blind SSTI]
    Q --> R{Delay or callback
    observed?}
    R -->|Yes| S[Blind SSTI confirmed]
    R -->|No| T[Re-check context,
    escaping, input point]
```

---

## 7. Practical Example — Request & Response

Ye ek illustrative example hai jo dikhata hai ki detection probe real HTTP traffic me kaisa dikhta hai. Values generic/educational hain.

### Baseline request

```http
GET /greeting?name=ssti-test-123 HTTP/1.1
Host: example-lab.local
```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<p>Hello ssti-test-123</p>
```

Observation: input as-is reflect ho raha hai — koi transformation nahi.

### Arithmetic probe

```http
GET /greeting?name={{7*7}} HTTP/1.1
Host: example-lab.local
```

**Response (vulnerable):**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<p>Hello 49</p>
```

**Response (not vulnerable):**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<p>Hello {{7*7}}</p>
```

Observation: `49` dikhna batata hai ki expression ko engine ne evaluate kiya — input data nahi, code ki tarah treat ho raha hai.

### Fingerprint probe

```http
GET /greeting?name={{7*'7'}} HTTP/1.1
Host: example-lab.local
```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<p>Hello 7777777</p>
```

Observation: `7777777` Python-style string repetition hai → Jinja2 family ka strong indicator (section 5, Step 3 ka decision tree dekho).

> Ye sirf detection/fingerprinting ke liye hai. Is point ke baad exploitation sirf scoped, authorized environment me, engine-specific documentation follow karte hue karo.

---

## 8. Possible Impact

Severity confirmed impact par depend karti hai, sirf detection par nahi:

- **Information disclosure** — internal variables, config values, environment details leak ho sakte hain.
- **Sensitive data exposure** — secrets, tokens, credentials jo template context me accidentally expose ho jate hain.
- **Server-side file read** — kuch engines/configs me file system access possible hota hai.
- **Remote Code Execution (RCE)** — sabse severe outcome, lekin engine, sandboxing, aur app configuration par depend karta hai. **Har SSTI RCE tak nahi jaata.**

Severity report karte waqt hamesha specify karo ki impact *verified* hai ya *theoretical* — ye professional reporting ka core part hai.

---

## 9. Prevention

- User input ko kabhi bhi template **source string** me concatenate mat karo.
- Predefined, static templates use karo — input sirf **data/parameters** ke through pass karo.
- Agar user-supplied templates zaroori hain, **sandboxed environment** use karo (jaise Jinja2 ka `SandboxedEnvironment`, Twig sandbox extension).
- Jahan possible ho, **logic-less template engines** (jaise Mustache) consider karo — inme code-execution surface bahut kam hota hai.
- App ko **least-privilege** user ke saath run karo, taaki worst-case impact limit ho.
- Dependencies aur template engine versions **updated** rakho — known sandbox-escape CVEs patch hote rehte hain.
- Sirf **blacklisting/WAF rules** par depend mat karo — ye bypass-able hote hain; root cause fix (data vs code separation) hi reliable solution hai.

---

## 10. Quick Testing Checklist

- [ ] Target authorized hai (scope document / lab environment confirmed)
- [ ] Baseline response record kiya
- [ ] Harmless arithmetic probes test kiye (multiple syntax variants)
- [ ] Response differences baseline se compare kiye
- [ ] Context identify kiya (plaintext vs code)
- [ ] Blind indicators check kiye (agar direct output nahi mila)
- [ ] Engine fingerprint decision-tree se narrow kiya
- [ ] False positives rule out kiye (single observation par confirm nahi kiya)
- [ ] Actual impact verify kiya (sirf claim nahi)
- [ ] Evidence (request/response, screenshots) properly documented

---

## 11. Payload Reference (this repo)

| File | Engine |
| --- | --- |
| `jinja2-flask.txt` | Jinja2 (Python / Flask) |
| `twig.txt` | Twig (PHP) |
| `freemarker.txt` | FreeMarker (Java) |
| `thymeleaf.txt` | Thymeleaf (Java / Spring) |
| `erb-ruby.txt` | ERB (Ruby) |
| `smarty.txt` | Smarty (PHP) |
| `ejs.txt` | EJS (Node.js) |
| `polyglot.txt` | Multi-engine detection probes |

> Note: Ye files sirf **detection/fingerprinting** probes ke liye reference hain. Exploitation-grade payloads is repo me documented nahi hain — authorized testing ke liye PortSwigger Web Security Academy ya HackTricks jaise maintained resources use karo.

---

## 12. Real-World Reference

- **CVE-2024-32651** — changedetection.io me Jinja2-based SSTI jo RCE tak le gaya. [GitHub Advisory](https://github.com/dgtlmoon/changedetection.io/security/advisories/GHSA-4r7v-whpg-8rx3)

---

## 13. References

- [PortSwigger — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)
- [CWE-1336 — Improper Neutralization of Special Elements Used in a Template Engine](https://cwe.mitre.org/data/definitions/1336.html)
- [Jinja2 Documentation](https://jinja.palletsprojects.com/)
- James Kettle — "Server-Side Template Injection: RCE for the Modern Web App" (original research, PortSwigger)
- HackTricks — SSTI methodology and per-engine notes

---

**Key Takeaway:** SSTI tab confirm hoti hai jab evidence dikhaye ki untrusted input server-side template syntax ki tarah **evaluate** ho raha hai — sirf ek unusual response se nahi. Detection ek process hai: baseline → probe → compare → context → verify.

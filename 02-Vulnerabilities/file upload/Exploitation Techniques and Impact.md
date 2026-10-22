# File Upload Vulnerabilities — Practical Notes


## Table of Contents

1. [What Is a File Upload Vulnerability?](#1-what-is-a-file-upload-vulnerability)
2. [Extension Validation](#2-extension-validation)
3. [MIME-Type Validation](#3-mime-type-validation)
4. [Magic Bytes / File Signatures](#4-magic-bytes--file-signatures)
5. [SVG Upload & Stored XSS](#5-svg-upload--stored-xss)
6. [HTML Injection via Uploaded Files](#6-html-injection-via-uploaded-files)
7. [Path Traversal / Unsafe Storage Path](#7-path-traversal--unsafe-storage-path)
8. [IDOR / Broken Access Control on Files](#8-idor--broken-access-control-on-files)
9. [ZIP Slip & Decompression Bombs](#9-zip-slip--decompression-bombs)
10. [XXE in XML Uploads](#10-xxe-in-xml-uploads)
11. [HTTP PUT Method](#11-http-put-method)
12. [Bug Bounty Report Template](#12-bug-bounty-report-template)



---

## 1. What Is a File Upload Vulnerability?

**Simple Definition:** Jab website user ko file upload karne deti hai, lekin file type, content, size, storage location ya access permissions ko properly validate nahi karti.

> **Analogy:** Ek dawa ki bottle par koi "Candy" likh de — bottle ka asli content dawa hi rahega, sirf **label** badla hai. Website agar sirf label (filename, extension, Content-Type) dekh ke decide karti hai ki file safe hai ya nahi, to attacker sirf label badal ke system ko confuse kar sakta hai.

**Root Cause:** Client-controlled data (filename, extension, MIME type) ko trust kar lena, bina actual file content independently verify kiye.

**Example:** Profile-photo feature sirf JPEG/PNG allow karna chahti hai, lekin weak validation ki wajah se HTML ya SVG bhi accept ho jaati hai.

**Potential Impact:**

- Stored Cross-Site Scripting (XSS)
- Remote Code Execution (RCE) — vulnerable server configs me
- Unauthorized private-file access (IDOR)
- Path Traversal
- ZIP Slip / decompression bombs
- Server-Side Request Forgery (SSRF) — URL-import flows me
- XXE — XML-based upload flows me
- Resource exhaustion / Denial of Service
- Sensitive information disclosure

> **Important:** File accept ho jaana apne aap me high-severity bug nahi hai. Impact ko safely, evidence ke saath demonstrate karna zaroori hai.

---



## 2. Extension Validation

**Typical Impact: 🟠 Medium** (bypass hone par) — agar sirf extension bypass ho jaye lekin server content execute nahi karta, to impact limited hai. Agar bypass ki wajah se server-interpreted file (jaise script, §3.5 ka mechanism dekho) actually **execute** ho jati hai, to rating 🔴 **High** tak ja sakti hai — lekin ye confirm karne ke liye evidence chahiye ki execution hui, sirf accept hona kaafi nahi.

> **Analogy:** Guard sirf dekh raha hai ki banda **uniform** pehna hai ya nahi — andar kaun hai, wo check nahi kar raha. Server bhi sirf filename ke end me `.jpg` dekh ke "image hai" maan leta hai.

> **Simple Example:** Tum ek text file banate ho jisme HTML code likha hai, lekin file ko naam dete ho `profile.jpg`. Server sirf check karta hai "naam `.jpg` se end hota hai? Haan? Accept." Andar asal me HTML hai — server ko fark nahi pada. Agar koi aur page is file ko baad me "HTML hai" samajh ke render kar de, to andar ka content active ho sakta hai, naam kuch bhi ho.

### 2.1 Weak Extension Checks

Kuch applications sirf filename me allowed extension ki *presence* check karte hain, poora filename validate nahi karte.

```text
profile.jpg.php
```

**Check:** Kya server final extension sahi se validate karta hai, aur uploaded content ko execute hone se rokta hai?

> **Historical context:** Ye bypass classically Apache ke purane `mod_mime` multi-extension handling ki wajah se kaam karta tha (jahan `.php` extension anywhere in filename ho to bhi PHP execute ho jata tha). Modern, correctly configured servers me ye generally kaam nahi karta — isliye test karte waqt server stack aur config bhi document karo, sirf "bypass hua" mat likho.

### 2.2 Multiple Extensions

```text
photo.jpg.php   vs   photo.jpg
```

**Check:** Final extension validate ho raha hai, aur file-handling rule consistent hai?

### 2.3 Blacklist Limitations

Blacklist known-dangerous extensions block karti hai (`.php`, `.asp`), lekin server-interpreted formats ki poori list cover karna mushkil hai (`.phtml`, `.php5`, `.cgi`, server config ke hisaab se).

**Fix:** Blacklist ke bajaye **strict allowlist** use karo.

### 2.4 Case Sensitivity

```text
photo.jpg   photo.JPG   photo.JpG
```

**Check:** Sab variants consistently validate hote hain?

---

## 3. MIME-Type Validation

# MIME Type Validation Bypass (File Upload Vulnerability)

## 📌 Introduction
**MIME Type Validation Bypass** is a critical security vulnerability found in file upload functionalities. It occurs when a web application improperly validates the file type during upload, allowing an attacker to bypass restricted file type policies (e.g., uploading a dangerous `.php`, `.exe`, or `.js` file disguised as an image).

---

## 🛠️ How the Attack Works
1. **The Restriction:** A website sets up a rule to only allow safe images (e.g., `image/jpeg`, `image/png`).
2. **The Attacker's Goal:** The attacker wants to upload a malicious payload, such as a web shell (`shell.php`), to gain remote control over the server.
3. **The Bypass:** The attacker intercepts the upload request using a proxy tool (like Burp Suite) and changes the HTTP `Content-Type` header from `application/x-php` to `image/jpeg`. 
4. **The Flaw:** If the server solely trusts the user-supplied `Content-Type` header without verifying the actual content, it accepts and saves the malicious script.

ex-- <img width="1271" height="480" alt="image" src="https://github.com/user-attachments/assets/dbf0f06e-5cc1-45d1-b8ef-db5dbfdf7069" />

## 🎯 Impact & Consequences
* **Remote Code Execution (RCE):** The attacker can execute arbitrary commands on the host server.
* **Full Server Compromise:** Access to sensitive backend databases, source code, and configuration files.
* **Website Defacement:** Replacing original site content with malicious text or images.
* **Malware Distribution:** Hosting and spreading viruses or ransomware to legitimate visitors.

---

## 🛡️ Remediation & Prevention
To properly secure file upload functionalities against MIME-type bypasses, implement the following defenses:

* **Validate Magic Bytes:** Do not rely on the `Content-Type` header or file extension. Verify the actual file content using content-based inspection (e.g., checking file signatures or magic numbers).
* **Enforce Strict Whitelisting:** Implement a strict whitelist of allowed file extensions and MIME types. Reject anything that doesn't match.
* **Rename Uploaded Files:** Change the file names randomly upon saving (e.g., converting `user_file.jpg` to `a8f9c2d3.jpg`) to prevent attackers from predicting the file path.
* **Disable Script Execution:** Store uploaded files in a separate directory or an isolated storage bucket (like AWS S3) where script execution permissions (PHP, ASPX, JS) are strictly disabled.


## 4. Magic Bytes / File Signatures

> **Analogy:** Sona asli hai ya nakli, ye pehchaanne ke liye jeweller acid test karta hai — sirf dekhne se ya label se pata nahi chalta, andar ka "composition" hi sach bolta hai.

Magic bytes file ke start ke characteristic hex bytes hote hain, jo actual format identify karne me madad karte hain — ye filename ya Content-Type se **independent** hote hain, isliye jhooth nahi bol sakte (SVG ka exception neeche dekho).

| Format | Magic Bytes (hex) | Note |
| --- | --- | --- |
| JPEG | `FF D8 FF` | — |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | — |
| GIF | `47 49 46 38` (`GIF8`) | — |
| PDF | `25 50 44 46` (`%PDF`) | — |
| ZIP (and ZIP-based formats like docx/xlsx) | `50 4B 03 04` | — |
| SVG | *koi fixed magic bytes nahi* | SVG plain-text XML hai — signature check se detect nahi hoti, isliye separate XML/content validation chahiye |

**Check:** Server actual content inspect karta hai, ya sirf extension/MIME par rely karta hai?

**Important — "Hybrid" / Polyglot Files:** Signature match hona akela *sufficient* nahi hai. Ek file **valid magic bytes** (e.g., genuine JPEG header `FFD8`), **matching MIME type** (`image/jpeg`), **aur** uske baad hidden malicious content — teeno ek saath ho sakte hain, kyunki magic bytes sirf file ke **shuru** ke kuch bytes check karte hain, poori file ka content nahi. Agar server sirf "shuruaati bytes sahi hain" dekh kar accept kar leta hai, to validation **incomplete** hai.

> **Analogy:** Ek parcel ke upar genuine company ka seal laga ho (signature valid), aur label bhi sahi ho (MIME valid) — lekin andar jo cheez hai wo seal/label se match na kare, ye signature/label check se pata nahi chalega. Poora content verify karna padega.

**Fix:** Signature check ko **trusted parser** (jo poori file structure validate kare, sirf header nahi) **+ format allowlist + storage-location isolation** ke saath combine karo — sirf magic-bytes check kabhi akela sufficient nahi hai.

---

## 5. SVG Upload & Stored XSS
XSS (Cross-Site Scripting) attacks linked to file uploads are a vulnerability frequently encountered during web penetration tests. One of the most common and effective vectors is the use of SVG files.

SVG files are an XML-based image format. It is therefore possible to include JavaScript tags in their structure.

These tags can be executed by the victim’s browser when they are opened, which can allow malicious code to be executed:

<img width="1182" height="449" alt="image" src="https://github.com/user-attachments/assets/f3807092-dc8a-443a-b0ee-35d22be0a290" />
Let’s consider a web application that lets you upload files with the .svg extension:
<img width="1027" height="501" alt="image" src="https://github.com/user-attachments/assets/4aeeccd3-4278-474a-96ba-06ef740174a9" />
 The SVG file is uploaded to the server. However, when a user accesses it via his browser, the malicious content embedded in the file is interpreted and executed. This triggers the execution of JavaScript code, enabling the targeted user’s session cookie, for example, to be retrieved:

<img width="1214" height="683" alt="image" src="https://github.com/user-attachments/assets/e8335c78-6882-4dbb-b9d4-805fe05aadaf" />
In addition, an attacker can upload a booby-trapped SVG image and then share the link to this file with legitimate users. As soon as a user clicks on this link, the JavaScript code contained in the file is executed in the browser. This can allow the attacker to steal sensitive information, such as a session token, or cause any other effect typical of a stored XSS.

In some cases, the malicious file may even be automatically displayed on a public page (for example, the home page of a site or a community space). Users will then be exposed to the attack without needing to click on anything, simply by visiting the page.

## 6. HTML Injection via Uploaded Files

HTML and JavaScript are interpreted by the browser’s rendering engine. When a user requests an HTML page, the browser treats it as content that should be rendered for the requested origin.

If a file upload feature does not properly validate uploaded files, an attacker may be able to upload an HTML file instead of an expected file type, such as an image or document.

For example, suppose an application allows users to upload files and later makes those files publicly accessible without restricting the allowed file types. An attacker could upload a malicious HTML page containing arbitrary HTML, CSS, and JavaScript.

When another user visits the uploaded file, the browser renders it as a normal web page because it is being served from the application’s domain.

This can lead to several security issues, including:

Creating convincing phishing pages that imitate the application’s login screen.
Stealing credentials entered by unsuspecting users.
Displaying misleading or fraudulent content under the trusted domain.
Redirecting users to attacker-controlled websites.
In some cases, leading to Stored Cross-Site Scripting (Stored XSS) if the uploaded HTML is embedded into the application’s pages or served in an unsafe context.

An attacker uploads a file named hacked.html:

<img width="845" height="432" alt="image" src="https://github.com/user-attachments/assets/fcd619c9-f16c-4e98-a258-3b7c2304ba3c" />
If the upload succeeds and the file is publicly accessible, anyone who opens the uploaded file’s URL will see it rendered as a normal HTML page :

<img width="1100" height="526" alt="image" src="https://github.com/user-attachments/assets/2e05cb85-c37b-4d61-a684-988f26d91e04" />
An attacker could make this page closely resemble the legitimate login page of the application. Because it is hosted on the organization’s own domain, users are more likely to trust it and may unknowingly enter sensitive information.

Although this scenario does not necessarily lead to Remote Code Execution (RCE), it is still a serious security issue. By abusing the application’s trusted domain, an attacker can perform phishing attacks, steal user credentials, distribute misleading content, or launch additional attacks against unsuspecting users.

Note: Simply allowing HTML uploads does not automatically result in Stored XSS. Whether it becomes an XSS vulnerability depends on how the uploaded file is served. If users visit the uploaded HTML as a standalone page, the primary impact is typically phishing or content spoofing. Stored XSS usually occurs when the application’s own pages embed or render attacker-controlled HTML within their DOM without proper sanitization.

## 7. Path Traversal / Unsafe Storage Path

The classic technique uses ../ (dot-dot-slash) sequences to navigate up the directory tree:
```plaintext
GET /download?file=../../../etc/passwd
```
Path traversal can be exploited to:
- Read sensitive files — Configuration files, source code, private keys, /etc/passwd, /etc/shadow
- Read application secrets — Database credentials in config files, API keys in .env files
- Write malicious files — Overwrite configuration, plant web shells, modify log files
- Achieve remote code execution — By writing executable files to web-accessible directories or overwriting application code
Path traversal is ranked under OWASP A01:2021 Broken Access Control, the number-one risk category. Its impact is significant because:
1. Direct access to sensitive data — Attackers can read database connection strings, private keys, and other secrets stored on the filesystem
2. Common in file-serving applications — Any feature that serves files based on user input (document downloads, avatar uploads, report generation) is at risk
3. Often leads to full compromise — Reading SSH keys, AWS credentials, or application configs frequently enables complete infrastructure takeover
4. Encoding bypasses are common — Simple blocklist-based defenses are routinely bypassed using URL encoding (%2e%2e%2f), double encoding, null bytes, or Unicode normalization
5. Zip slip attacks — Archive extraction operations that don’t validate member paths can write files to arbitrary locations
An application serves user-uploaded documents:
```plaintext
GET /api/documents?name=report.pdf
```
The server constructs the path as /var/app/uploads/report.pdf. An attacker modifies the request:
```plaintext
GET /api/documents?name=../../../etc/passwd
```
The server resolves /var/app/uploads/../../../etc/passwd to /etc/passwd and returns its contents.
Attackers use various encodings to evade filters:
```plaintext
# URL encoding
GET /api/documents?name=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd

# Double URL encoding
GET /api/documents?name=%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Null byte injection (older runtimes)
GET /api/documents?name=../../../etc/passwd%00.pdf

# Backslash on Windows
GET /api/documents?name=..\..\..\..\windows\win.ini

# Mixed separators
GET /api/documents?name=..%5c..%5c..%5cwindows%5cwin.ini
```
When extracting archives, a malicious zip can contain entries with traversal paths:
```plaintext
../../../../tmp/evil.sh
../../../var/www/html/shell.php
```
If the application extracts these without validating member paths, files are written to arbitrary locations.
## 8. IDOR / Broken Access Control on Files

Ye lo, bilkul tumhare original jaisa hi hai, bas GitHub par paste karne ke liye formatting clean kar di hai.

````
## 🔐 What Makes Access Control So Vulnerable?

Unlike input validation bugs, access control flaws are **logic-based** — they depend on how the application handles roles, permissions, and resources.

Common scenarios include:

- Lack of proper role verification
- Client-side enforcement (never trust the frontend)
- Hidden but unprotected admin panels
- Insecure direct object references (IDOR)

BAC issues often go unnoticed because they don’t always throw errors — they quietly give attackers access to things they shouldn’t see or do.

## 🧪 Where to Look

Look for access control vulnerabilities during recon and functionality testing. Pay close attention to:

- User profile endpoints: `/api/user/profile`, `/user/settings`
- ID-based resources: `/orders/123`, `/documents/456`
- Admin or internal dashboards: `/admin`, `/internal`
- Multi-user portals (e.g., education, healthcare, HRMS)
- Hidden functionality revealed through JavaScript
- Mobile API endpoints or GraphQL mutations

Also, test **both horizontal and vertical privilege escalations**:

- Horizontal = Access another user’s data
- Vertical = Escalate to higher privilege (e.g., normal user → admin)

## 🧪 Common Payloads

These don’t involve traditional fuzzing — instead, they manipulate requests to trick the backend.

## 🔄 IDOR Examples:

```http
GET /invoice/2025
```

Change to:

```http
GET /invoice/2026
```

Or try:

```http
GET /profile?id=123
```

Change to:

```http
GET /profile?id=124
```

## 🔼 Privilege Escalation via Parameter Tampering:

```http
PUT /api/user/update
Content-Type: application/json

{
  "role": "admin"
}
```

## 🕵️ Bypassing Role Checks:

```http
POST /admin/settings
Cookie: user=123; role=admin
```

## 🚫 Bypassing Method Restrictions:

Try using `POST` instead of `GET`, or `PUT` instead of `PATCH`, especially in APIs.

```bash
curl -X POST https://target.com/api/admin/deleteUser/45
```

## ✅ Detection Clues

Since BAC flaws often don’t produce errors, detection requires careful observation and session manipulation.
````

## 9. ZIP Slip & Decompression Bombs

> **Analogy (ZIP Slip):** Ek parcel ke andar chhote-chhote labeled boxes hain, lekin ek box ka label kehta hai "ise mere ghar ke bajaye padosi ke ghar rakh do." Agar delivery system har box ka label blindly follow kare, to wo box galat jagah chala jayega.

**ZIP Slip:** Archive entries ke paths validate nahi hote, isliye extraction intended directory ke bahar ja sakti hai — bilkul path traversal (§9) jaisa concept, bas ye ZIP file ke andar hota hai.

```text
photos.zip
├── photos/photo.png
└── photos/notes.txt
```

Test: normalized path jo extraction-root ke bahar point kare — app use reject karti hai ya nahi?

**Expected rejection:**

```json
{"error": "Archive contains an unsafe path"}
```

**Decompression bomb:** Chhota compressed file jo extract hone par disproportionately bada ho jaye (high compression ratio), resource exhaustion cause kar sakta hai — ye size-limit section (§15) se related hai.

**Fix:**

- Har entry ka normalized path validate karo, extraction-root ke andar hi rakho.
- Absolute paths, traversal, unsafe symlinks reject karo.
- Entry count **aur** expanded (decompressed) size dono par limits lagao.

---

## 10. XXE in XML Uploads

## 📂 Common File Upload Vectors for XXE

- **SVG Images:** SVG files are XML-based. If an application processes uploaded SVG files using an insecure XML parser, external entity processing may lead to XXE vulnerabilities. [[1](https://www.exploit-db.com/docs/49732), [2](https://www.youtube.com/watch?v=z6VriWV1zPo)]

- **Office & Archive Documents (OOXML):** Formats such as `.docx`, `.pptx`, and `.xlsx` are ZIP archives containing XML files, including `[Content_Types].xml` and relationship files. If the server processes these files using an insecure XML parser, malicious XML may trigger XXE. [[1](https://www.youtube.com/watch?v=LZUlw8hHp44)]

- **Audio/Media Metadata:** Some media formats, such as WAV files containing iXML metadata, may include XML data. Vulnerable parsers that process this data insecurely can introduce XXE risks. [[1](https://safe.security/wp-content/uploads/xml-external-entity-xxe-vulnerability-research-paper.pdf)]

---

## 🧪 Exploitation Technique & Payload Flow

### 1. Crafting a Test File (Example: SVG)

Create a test SVG containing an external entity declaration. In an authorized lab, use a harmless local test file to verify whether the XML parser resolves external entities.

**Example XML:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///path/to/harmless-test-file.txt">
]>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <text x="10" y="40" font-size="12">&xxe;</text>
</svg>
```

**What to observe:**

- Does the server accept and process the SVG?
- Does the application render the entity's test content?
- Does the parser return an XML processing error?
- Does the application reject external entity declarations?

A successful upload alone does not prove that XXE exists.

### 2. Testing Archive Files (DOCX / OOXML)

1. Create a test `.docx` document using an office application.
2. Inspect its internal XML files using an archive manager.
3. Identify which XML components the application processes during previewing, conversion, or document extraction.
4. In an authorized lab, use a controlled XML test case to determine whether external entity processing is enabled.
5. Observe the response and server-side processing behavior to confirm whether the parser is vulnerable.

**Important:** OOXML documents contain multiple XML components. Simply modifying a relationship file or `[Content_Types].xml` does not automatically create an XXE vulnerability. The result depends on the specific parser and processing workflow. [[1](https://www.youtube.com/watch?v=LZUlw8hHp44)]

### 3. Execution & Impact Vectors

- **Local File Disclosure (LFD):** An insecure XML parser may resolve an external entity that references a local file. If the resulting content is exposed through the application response or an error message, sensitive information could be disclosed. [[1](https://portswigger.net/web-security/xxe), [2](https://www.hackerone.com/knowledge-center/xxe-complete-guide-impact-examples-and-prevention)]

- **Server-Side Request Forgery (SSRF):** An external entity may cause the server to make a request to an internal service or another network resource. The actual impact depends on network access and parser behavior. [[1](https://www.yeswehack.com/learn-bug-bounty/xml-external-entity-guide-xxe), [2](https://www.hackerone.com/knowledge-center/xxe-complete-guide-impact-examples-and-prevention)]

- **Out-of-Band (OOB) Detection:** If the application does not display the entity's result directly, an authorized tester may use a controlled external interaction endpoint to check whether XML processing triggers an outbound DNS or HTTP request. An outbound interaction alone does not necessarily prove sensitive data exfiltration.


## 11. HTTP PUT Method

# 🔥 HTTP PUT Method Enabled — File Upload Vulnerability

## 📌 What Is HTTP PUT?

The HTTP `PUT` method is used to create or replace a resource at a specified URL.

If a web server accepts unauthorized `PUT` requests, an attacker may be able to upload files to the server without proper authorization.

The impact depends on the server configuration, the destination directory, and how uploaded files are handled.

**Important:** An enabled `PUT` method alone does not confirm a vulnerability. You must verify whether unauthorized users can create or modify resources.

---

## 🎯 Practical Testing Workflow

### 🧪 Step 1: Identify Supported HTTP Methods

Use Burp Suite to intercept a request to an authorized test application.

Send the request to **Repeater** and test the `OPTIONS` method:

```http
OPTIONS / HTTP/1.1
Host: lab.example
Connection: close
```

Observe the response for an `Allow` header.

Example:

```http
HTTP/1.1 200 OK
Allow: GET, HEAD, OPTIONS, PUT
```

**Observation:**

- `PUT` listed in `Allow` suggests that the method may be supported.
- It does not prove that arbitrary file uploads are possible.
- The server may enforce authentication or additional restrictions.

---

### 📤 Step 2: Test File Creation Using PUT

Perform this step only on your own lab or an explicitly authorized target.

Create a harmless text file using a unique filename.

Example request:

```http
PUT /put-test-unique.txt HTTP/1.1
Host: lab.example
Content-Type: text/plain
Content-Length: 26
Connection: close

Authorized PUT test file.
```

Observe the server response.

Possible responses include:

- `200 OK` — the request was processed successfully.
- `201 Created` — a resource may have been created.
- `204 No Content` — the request was processed without a response body.
- `401 Unauthorized` or `403 Forbidden` — access may be restricted.
- `405 Method Not Allowed` — the method is not allowed for that resource.

**Note:** HTTP status codes alone do not establish whether a file was actually created. Verify the result independently.

---

### 🔎 Step 3: Verify the Uploaded File Using GET

Request the exact test filename from the previous step.

```http
GET /put-test-unique.txt HTTP/1.1
Host: lab.example
Connection: close
```

Compare the response body with the harmless text submitted in the `PUT` request.

If the same unique content is returned, you have evidence that the resource is accessible at that URL.

Record the following:

- PUT request and response
- GET request and response
- Exact test filename
- Whether authentication was required
- Whether the resource was created or overwritten

**Important:** A successful `GET` verifies resource accessibility, not remote code execution.

---

## 🧰 Step 4: Analyze the Actual Impact

### 1. Unauthorized File Creation

An unauthenticated user can create a file on the server without authorization.

Potential impact:

- Unauthorized modification of server-hosted content
- Storage abuse
- Exposure of attacker-controlled content to other users

### 2. Unauthorized File Overwriting

If an existing resource can be overwritten, an attacker may be able to modify legitimate content.

The impact depends on the affected resource and the application's integrity requirements.

### 3. Stored Content Exposure

If uploaded content is publicly accessible, other users may be able to retrieve it.

Assess whether the server serves the file as plain text, downloads it, or renders it in a browser.

### 4. Potential Remote Code Execution (RCE)

RCE is **not automatic**.

It may become possible only when the uploaded content reaches a server-side execution path and the relevant runtime processes it.

Do not claim RCE unless it has been safely and explicitly demonstrated in an authorized lab.



## 12. References

- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PortSwigger — File upload vulnerabilities](https://portswigger.net/web-security/file-upload)
- [CWE-434 — Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) — practice lab


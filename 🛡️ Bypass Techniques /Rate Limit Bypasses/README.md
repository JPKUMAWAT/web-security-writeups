Here is your exact notes document rewritten in **easy, simple language**, organized clearly while keeping all of your exact points, techniques, and step-by-step logic intact:

---

# Where to Look for Bugs & Rate Limit Bypasses

### Target Features / Functions to Test

* **Login Pages**
* **Reset Password Forms**
* **Two-Factor Authentication (2FA / OTP Pages)**
* **Confirmation Code / Verification Forms**
* **Sign Up / Registration Pages**

---

### Techniques to Bypass Rate Limits & Security Checks

#### 1. Using Null Bytes & Special Characters

* **Concept:** Insert special encoded characters into inputs (like emails or usernames) to trick the backend logic into treating each request as unique while still sending to the same account.
* **Payloads / Characters:** `%00`, `%0d%0a`, `%09`, `%0C`, `%20`, `%2f0`
* **Test Example:**
1. Perform brute-force testing using `abc@xyz.com`.
2. After a certain number of requests, the server blocks you.
3. Change the input to `abc@xyz.com%00` and retry to test if the rate limit resets or bypasses the check.



---

#### 2. Host Header Injection

* **Concept:** Modify or inject different `Host` headers in the HTTP request to bypass rate limiting or IP-based restrictions.
* **Headers to Modify:**
* `Host: [www.newsite.com](https://www.newsite.com)`
* `Host: localhost`
* `Host: 127.0.0.1`



---

#### 3. Changing Cookies

* **Concept:** If the server tracks failed attempts or rate limits based on your session or cookie value:
* **Test Example:** If the server blocks you after 15 requests, clear or change your session cookie on the 14th request and continue to see if the counter resets.



---

#### 4. Header Spoofing (IP Bypass Headers)

* **Concept:** Add or modify client IP headers to make the server think each request originates from a different IP address.
* **Headers to Use:**
* `X-Forwarded: <IP>`
* `X-Forwarded-For: <IP>`
* `X-Forwarded-Host: <IP>`
* `X-Client-IP: <IP>`
* `X-Remote-IP: <IP>`
* `X-Remote-Addr: <IP>`
* `X-Host: <IP>`
* `X-Originating-IP: <IP>`
* `X-Forwarded-Forwarded-For: <IP>`


* **Multi-Header Techniques:**
* Try adding **two duplicate headers** in the request.
* Try adding custom or nested `X-Forwarded-For` headers with internal IP ranges:
* `X-Forwarded-For:`
* `X-Forwarded-For: 192.168.43.1`

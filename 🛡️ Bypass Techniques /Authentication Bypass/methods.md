Here is your exact notes document rewritten in **easy, simple language**, keeping all the exact topics, steps, and structure intact:

---

# Authentication Bypass

### 1 Direct Access to Post-Authentication URLs

1. Check if pages meant for logged-in users (like `/dashboard` or `/settings`) can be opened directly in a browser without active login cookies.
2. If the page URL can be guessed or brute-forced without needing a session, it leads to direct account takeover.

---

### 2 CAPTCHA Bypass using `X-Forwarded-For`

1. Bypass CAPTCHA protections by injecting a new random IP address into the `X-Forwarded-For` HTTP header on every request.

---

### 3 Lack of Password Confirmation

Test if sensitive account changes require re-entering the current password:

* Change Email Address
* Change Password
* Delete Account
* Manage / Turn off 2FA

---

### 4 Lack of Email Verification

1. Check during sign-up if the application forces users to verify their email address before accessing account features.

---

### 5 No Rate Limiting on Forms

1. Fill out a general form (like contact or feedback) and capture the request in Burp Suite Proxy.
2. Send the request to Burp Intruder.
3. Repeat sending the same request 20–30 times.
4. Check if all form submissions succeed without any blocking or limit errors.

---

### 6 No Rate Limiting or CAPTCHA on Login Page

1. Go to the login page and capture a failed login attempt in Burp Suite Intruder.
2. Set the password field to try multiple random values (brute-force).
3. Observe if the 20th or 30th request still succeeds without triggering an error, lock-out, or CAPTCHA prompt.

---

### 7 Username / Email Address Enumeration

1. Go to password reset, login, or registration pages where email/username inputs are accepted.
2. Enter a valid (existing) account email with a wrong password and note the error message.
3. Enter an invalid (non-existing) email address and note the error message.
4. Check if the application reveals whether an account exists based on differences in:
* **Status codes**
* **Error messages**
* **Response times**



---

### 8 Weak Password Policy

1. Try changing your password to only numbers (e.g., `123456`).
2. Try changing your password to only lowercase letters (e.g., `password`).
3. Try changing your password to common weak words.
4. Try changing your password to short values (e.g., `123`).
5. Check if the application accepts weak passwords without enforcing strong rules.

---

### 9 Weak Registration Implementation over HTTP

1. Intercept the account registration request using Burp Suite.
2. Check if the request is transmitted over plain unencrypted HTTP instead of secure HTTPS.

---

### 10 Unsecure Data Transport

1. Go to the login or form page.
2. Submit the form and capture the request using Burp Suite or Wireshark.
3. Verify whether the sensitive data (passwords, tokens) is encrypted during transmission.

---

### 11 Broken Authentication Session Token Bug

1. Create a user account or log into an existing one.
2. Confirm your email address.
3. Log out of your account and trigger a password reset code request to your email.
4. **Do not use** the password reset code yet.
5. In a new browser tab or window, log back into your account.
6. Go to account settings and manually change your password.
7. Now open your email, grab the password reset code requested in Step 3, and use it.
8. Check if the old reset code still works and overrides the new password.

---

### 12 Broken Authentication and Session Management (Old Email Reset Bug)

1. Create an account registered with an email address (e.g., `a@x.com`).
2. Log out and request a password reset link sent to `a@x.com` (do not use it yet).
3. Log back in, update your email address to `b@x.com`, verify it, and remove `a@x.com`.
4. Log out again, then click the old password reset link mailed to `a@x.com` from Step 2.
5. Check if the old reset link still changes the password for the account, even though the email address was updated.

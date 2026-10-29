Here is a simplified, easy-to-understand version of your 2FA testing checklist while maintaining the exact same structure, points, and step-by-step logic.

---

# Bypassing Two-Factor Authentication

### 1 Flawed two-factor verification logic

Sometimes the application checks credentials in step 1, but fails to check if the same user is completing step 2.

**Step 1:** The user logs in with their credentials:

```http
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com

username=carlos&password=qwerty

```

**Step 2:** The server sets an account cookie and redirects to the second step:

```http
HTTP/1.1 200 OK
Set-Cookie: account=carlos

GET /login-steps/second HTTP/1.1
Cookie: account=carlos

```

**Step 3:** The verification request uses that cookie to determine who is logging in:

```http
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos

verification-code=123456

```

**Bypass:** An attacker logs into their own account, but changes the account cookie to the victim's username during code submission:

```http
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user

verification-code=123456

```

---

### 2 Clickjacking on 2FA Disable Feature

1. Try loading the "Disable 2FA" settings page inside an `<iframe>` on an external site.
2. If the iframe loads, use a social engineering visual overlay to trick the user into clicking the disable button.

---

### 3 Response Manipulation

1. Capture the server's response after entering an incorrect 2FA code.
2. If the response shows `"Success": false`.
3. Change it to `"Success": true` to see if the client-side application unlocks access.

---

### 4 Status Code Manipulation

1. Check if entering an invalid 2FA code returns a `4XX` status code (such as `401 Unauthorized`).
2. Change the status code line to `HTTP/1.1 200 OK` in your proxy to see if it bypasses the screen.

---

### 5 2FA Code Reusability

1. Request a 2FA code and use it successfully.
2. Try submitting that same code a second time to check if it remains valid.
3. Request multiple new 2FA codes and check if older codes stay active or automatically expire.
4. Try reusing a code after a long delay (such as 1 day) to see if time-based expiration is enforced.

---

### 6 CSRF on 2FA Disable Feature

1. Request a 2FA code and complete the verification.
2. Attempt to reuse that code in a automated state-changing request.
3. Check if multiple active codes are generated without invalidating previous ones.
4. Test if old codes can be reused after extended periods (e.g., 24 hours).

---

### 7 Backup Code Abuse

Apply the same testing methodology used on regular 2FA (such as Response Manipulation, Status Code Modification, and Brute-Force) against the backup code validation endpoint.

---

### 8 Enabling 2FA Doesn't Expire Previous Session

1. Log into the same account across two different browsers (Browser A and Browser B).
2. Enable 2FA using Browser A.
3. Refresh Browser B to see if the existing active session is automatically terminated or remains active.

---

### 9 2FA Refer Check Bypass

1. Try navigating directly to a post-2FA dashboard URL without completing the 2FA prompt.
2. If access is blocked, set the HTTP `Referer` header to the 2FA page URL to test for weak referrer-based validation.

---

### 10 2FA Code Leakage in Response

1. Trigger the "Send OTP" or 2FA code request in your browser.
2. Inspect the HTTP response body and headers to see if the generated 2FA code is included in plaintext.

---

### 11 JS File Analysis

1. Trigger a 2FA code request.
2. Analyze all JavaScript files loaded or referenced in the response.
3. Look for client-side validation logic, hardcoded tokens, or debug flags that control the 2FA check.

---

### 12 Lack of Brute-Force Protection

1. Capture the request used to trigger or send a 2FA code.
2. Repeat the request 100–200 times to test if rate limiting is enforced on code generation.
3. Automated test against the 2FA verification endpoint to evaluate rate limiting and account lockout controls.
4. Test initiating code requests simultaneously while attempting verification submissions.

---

### 13 Password Reset/Email Change - 2FA Disable

1. Perform an account password reset or email address update flow.
2. Check if completing the reset or email change automatically disables 2FA on the target account.

---

### 14 Missing 2FA Code Integrity Validation

1. Request a valid 2FA code on your own account.
2. Enter that valid 2FA code into the victim account's 2FA prompt to test if codes are tied strictly to specific sessions.

---

### 15 Direct Request

1. Directly navigate to post-authentication pages (e.g., `/dashboard` or `/settings`) without entering a code.
2. Check if the application allows authenticated access while skipping the 2FA step.
3. Modify the `Referer` header to match the 2FA submission page URL.

---

### 16 Reusing token

1. Check if previously used session or authorization tokens can be resubmitted to bypass active 2FA checks.

---

### 17 Sharing unused tokens

1. Obtain an unused 2FA token generated for Account A and attempt to submit it against Account B.

---

### 18 Leaked Token

1. Check if pre-authentication tokens or session identifier keys are exposed in application responses.

---

### 19 Session permission

1. Initiate the login flow for two separate accounts using the same browser session.
2. Reach the 2FA step for both accounts.
3. Complete 2FA for Account A.
4. Attempt to proceed directly to the post-2FA endpoint for Account B.
5. Test whether the backend relies on a simple global session flag (e.g., `2fa_passed=true`) rather than validating the user identity.

---

### 20 Password reset function

1. Test whether performing a password reset logs the session directly into the application, bypassing 2FA.
2. Check if password reset links or tokens can be reused multiple times.

---

### 21 Lack of Rate limit

Test if the 2FA verification endpoint enforces a maximum threshold for invalid code submissions.

---

### 22 Flow rate limit but no rate limit

Check if rate limiting is applied only to rapid single-thread requests while allowing slower, distributed attempts over extended timeframes.

---

### 23 Re-send code and reset the limit

Test whether triggering a "Resend Code" request resets the failed attempt counter while keeping the original code active.

---

### 24 Client side rate limit bypass

Check if rate limits are enforced purely by client-side JavaScript or local storage values rather than on the server.

---

### 25 Lack of rate limit in the user's account

Check whether sensitive post-login actions (e.g., changing email or password) lack rate limiting on 2FA prompts, even if primary login has limits.

---

### 26 Lack of rate limit re-sending the code via SMS

Test if the system allows unlimited SMS requests without delays or CAPTCHAs, leading to potential SMS gateway exhaustion.

---

### 27 Infinite OTP regeneration

Check if short 4-digit codes can be regenerated indefinitely and submitted in batch attempts.

---

### 28 Guessable cookie

Inspect "Remember This Device" or persistent cookies for predictable formats or weak encoding.

---

### 29 IP address

Test if "Remember Me" features rely solely on client IP addresses for authorization bypasses.

---

### 30 Subdomains

Search for legacy or testing subdomains (e.g., `test.example.com` or `m.example.com`) that use older login forms without 2FA implementation.

---

### 31 APIs

Check if older API versions (such as `/v1/` or `/v2/`) exist alongside modern ones (`/v3/`) that handle authentication without requiring 2FA.

---

### 32 Previous sessions

Verify whether activating 2FA automatically revokes and invalidates all previously issued active sessions across all devices.

---

### 33 Improper access control to backup codes

Check if backup codes can be queried via API endpoints (`GET /api/backup-codes`) without requiring password confirmation.

---

### 34 Information Disclosure

Check if the 2FA prompt page displays sensitive user account details (such as unmasked phone numbers or email addresses) prior to authentication.

---

### 35 Bypass 2FA with null or 000000

Submit empty strings, `null`, or default numeric patterns (like `000000`) in place of the verification code.

---

### 36 Previously created sessions continue being valid after MFA activation

1. Open the target application session across two different devices.
2. On Device A, navigate to security settings and enable 2FA.
3. On Device B, refresh the application and test whether the active session persists without requiring 2FA re-authentication.

---

# Attack Scenarios & PoCs

### 37 Enable 2FA without verifying the email

1. Register an account using a victim's email address.
2. Log into the account prior to completing email verification.
3. Enable 2FA on the unverified account to block the legitimate owner.

---

### 38 Password not checked when disabling 2FA

**Steps:**

1. Log in and enable 2FA under `/settings/auth`.
2. Click the option to disable 2FA.
3. When prompted for both a 2FA code and the account password, enter a valid 2FA code alongside an incorrect password.
4. Submit the form and check if 2FA is successfully disabled despite the invalid password.

---

### 39 "email" MFA mode allows bypassing MFA from victim's device when the device trust is not expired

**Steps:**

1. Enable and configure MFA on the account.
2. Initiate the login process using standard credentials.
3. When the MFA screen appears, enter an arbitrary verification code.
4. Intercept the submission request (`POST /v3/api/login`) in an HTTP proxy tool.
5. Modify the request body parameters:
* Change `"mode": "sms"` to `"mode": "email"`
* Change `"secureLogin": true` to `"secureLogin": false`


6. Forward the modified request to test if authentication completes without phone code validation.

---

### 40 2FA bypass by sending blank code

1. Log into the application and enable 2FA under account settings.
2. Log out of the account.
3. Initiate login again until prompted for the 2FA code.
4. Enter an invalid code and intercept the outgoing `POST` request.
5. Remove the code parameter entirely from the request body before forwarding it to the server.
6. Check if the server accepts the parameterless request and authenticates the session.

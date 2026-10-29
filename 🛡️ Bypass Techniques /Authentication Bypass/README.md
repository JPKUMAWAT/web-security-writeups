Here is the breakdown of those **Authentication Bypass** test cases organized into standard Bug Bounty Priority Levels (**P1 to P5** / Bugcrowd VRT & HackerOne taxonomy) along with their risk ratings:

| Priority | Risk Rating | Vulnerability / Test Case | Bug Bounty Impact |
| --- | --- | --- | --- |
| **P1** | **[Critical]** | **Direct Access to Post-Authentication URLs** | Direct account takeover if sensitive post-auth endpoints lack active session validation. |
| **P1** | **[Critical]** | **Broken Authentication Session Token Bug** | Reusing a pre-issued password reset code after changing credentials allows account hijacking. |
| **P2** | **[High]** | **Broken Email Update Reset Flow** | Using an old password reset link sent to a former email address successfully resets access on a newly updated account email. |
| **P2** | **[High]** | **Lack of Password Confirmation** | Critical actions (changing email, changing password, deleting account, turning off 2FA) can be performed without verifying the current password. |
| **P2** | **[High]** | **Weak Registration Implementation over HTTP** | Transmitting account credentials over unencrypted plaintext HTTP exposes data to network interception. |
| **P2** | **[High]** | **Unsecure Data Transport** | Transmitting sensitive form data (passwords, tokens) without proper encryption during transport. |
| **P3** | **[Medium]** | **No Rate Limiting or CAPTCHA on Login Page** | Allows automated credential stuffing or brute-force attacks without triggering account lockouts or security delays. |
| **P3** | **[Medium]** | **Username / Email Address Enumeration** | Discrepancies in error messages, status codes, or response times reveal whether an email or username exists in the system. |
| **P3** | **[Medium]** | **Lack of Email Verification** | Users can register accounts with arbitrary/unverified email addresses, enabling potential account squatting or impersonation. |
| **P3** | **[Medium]** | **Weak Password Policy** | Accepting short, purely numeric, or easily guessable passwords increases vulnerability to credential guessing. |
| **P4** | **[Low]** | **CAPTCHA Bypass using `X-Forwarded-For**` | Manipulating the `X-Forwarded-For` header bypasses simple IP-bound CAPTCHA checks. |
| **P4** | **[Low]** | **No Rate Limiting on Generic Forms** | Submitting contact or feedback forms repeatedly without rate limits (can lead to spam or minor resource usage). |

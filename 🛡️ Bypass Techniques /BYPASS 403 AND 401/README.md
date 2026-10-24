# HTTP 401 & 403 — Access Control Testing

## 1. HTTP 401 Unauthorized

**Definition:** HTTP 401 occurs when authentication credentials are missing, invalid, or not accepted by the server.

**Simple example:** A user tries to access a private API without logging in.

### Common Causes
- Missing or invalid authentication token
- Expired session or token
- Incorrect authentication format
- Authentication misconfiguration

### Security Testing Methods
- **Token & Session Testing:** Verify that missing, invalid, and expired credentials are rejected.
- **HTTP Method Testing:** Check that authentication is enforced for every supported method.
- **Header Testing:** Review whether authentication depends on untrusted headers.
- **Path Normalization:** Verify that route variations do not skip authentication.
- **URL Override Testing:** Check whether proxy rewrites apply authentication consistently.
- **Protocol Testing:** Confirm that authentication remains enforced across supported HTTP versions.

**Security Goal:** Verify that protected resources cannot be accessed without valid authentication.

---

## 2. HTTP 403 Forbidden

**Definition:** HTTP 403 occurs when the server refuses a request because access is not permitted. The user may already be authenticated.

**Simple example:** A normal user tries to access an administrator-only page.

### Common Causes
- Insufficient user permissions
- Incorrect role-based access control
- Resource ownership checks missing
- Proxy or firewall access rules
- Inconsistent route authorization

### Security Testing Methods
- **Role Testing:** Verify that users can access only resources permitted for their roles.
- **HTTP Method Testing:** Check authorization for every supported method.
- **Header Testing:** Verify that client-controlled headers cannot grant unauthorized access.
- **Path Normalization:** Check that route variations use consistent authorization rules.
- **URL Override Testing:** Verify that rewritten routes receive the correct authorization checks.
- **Protocol Testing:** Confirm that authorization remains enforced across supported HTTP versions.

**Security Goal:** Verify that users cannot access resources or perform actions beyond their permissions.

---

## 3. Common 401/403 Testing Concepts

| Technique | What to verify |
|---|---|
| HTTP methods | Authentication and authorization apply to every supported method |
| Headers | Client-controlled headers do not grant trust |
| Path variations | Routing and access-control rules remain consistent |
| URL overrides | Rewritten requests receive the correct security checks |
| Proxy configuration | Trusted proxy metadata is handled securely |
| Protocol differences | Security rules remain consistent across supported protocols |
| Request smuggling review | HTTP components interpret request boundaries consistently |

*Note: Request smuggling is a specialized, higher-risk test that requires an explicitly authorized test environment.*

---

## 4. Testing Workflow

1. Confirm authorization and testing scope.
2. Establish a baseline request and response.
3. Test with approved accounts having different permissions.
4. Change one request property at a time.
5. Compare status codes, response content, and behavior.
6. Verify whether protected data or functionality becomes accessible.
7. Document evidence and recommend remediation.

---

## 5. Important Differences

| HTTP 401 | HTTP 403 |
|---|---|
| Authentication problem | Authorization problem |
| “Who are you?” | “Are you allowed?” |
| Check credentials and sessions | Check permissions and resource access |

**Important:** A status-code change alone does not prove a vulnerability. Verify actual unauthorized access and test only systems you own or have explicit permission to assess.

## Quick Summary

- **401:** Verify authentication controls.
- **403:** Verify authorization controls.
- Some testing techniques overlap, but their purpose differs.
- Secure applications enforce access control on the server for every relevant request.

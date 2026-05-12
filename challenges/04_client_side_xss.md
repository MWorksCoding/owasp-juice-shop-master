# Challenge 4 — Client-side XSS Protection (Stored XSS)

| Field | Detail |
|---|---|
| **OWASP Category** | A03 – Injection (Cross-Site Scripting) |
| **Severity** | Critical |
| **Technique** | Stored XSS via direct API injection into the email field |

---

## Background

Cross-Site Scripting (XSS) allows an attacker to inject malicious JavaScript into a web application that is then executed in other users' browsers. Stored XSS is the most dangerous variant — the payload is persisted in the database and fires automatically for every user who loads the affected page, without any further attacker interaction.

Client-side input validation (form field restrictions, format checks) is a UX feature, not a security control. Any attacker with an intercepting proxy can bypass it entirely by modifying the request before it reaches the server.

---

## Discovery

The registration form validated the email field format client-side and would have rejected an XSS payload through the UI. By intercepting the POST request to `/api/Users/` with Burp Suite, the email value was replaced with an iframe payload before the server received it.

---

## Exploit

The intercepted registration request body was modified:

```json
{
  "email": "<iframe src=\"javascript:alert(`xss`)\">",
  "password": "abc123",
  "passwordRepeat": "abc123",
  ...
}
```

The server returned HTTP 201 and stored the payload verbatim. Navigating to `/#/administration` as an admin triggered the JavaScript alert immediately — the payload executed in the admin's browser when the user list was rendered.

---

## Impact

- Any anonymous visitor can plant the payload — no authentication required to inject
- The payload executes automatically in every administrator's browser on their next visit to the admin panel
- Enables admin session token theft, account takeover, and full administrative access without any victim interaction
- Persists until the malicious account is manually deleted

---

## Video

[Watch on Loom](#) [Client-side XSS Protection](https://www.loom.com/share/3f2dea21fc15422c801cfbf4cc8d5e58)

---

## Recommendation

- Validate email field format server-side — reject any value that does not conform to RFC 5322
- Apply output encoding to all user-supplied fields rendered as HTML
- Implement a Content Security Policy (CSP) that disallows inline scripts and `javascript:` URI schemes
- Never rely on client-side validation as a security boundary

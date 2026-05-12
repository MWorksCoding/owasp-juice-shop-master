# Challenge 1 — Login Admin (SQL Injection)

| Field | Detail |
|---|---|
| **OWASP Category** | A03 – Injection |
| **Severity** | Critical |
| **Technique** | SQL Injection via login form |

---

## Background

SQL Injection occurs when user input is concatenated directly into a SQL query without sanitization. The database cannot distinguish between the developer's intended query structure and attacker-supplied data. By injecting SQL syntax into an input field, an attacker can manipulate the query logic entirely.

A well-known basic test is submitting `OR ""=""` as the username or password. Since `""=""` is always true, this forces the query to return a result regardless of what credentials are stored — effectively bypassing authentication. This is the starting point for understanding how SQL Injection works in practice.

---

## Discovery

### Step 1 — Confirm the injection point

To verify that the login form is vulnerable to SQL Injection, a single quote `'` was submitted as both the email and password value. Burp Suite was used to intercept the POST request and inspect the server response.

The server returned HTTP 500 with a raw database error in the response body:

```json
{
  "error": {
    "message": "SQLITE_ERROR: unrecognized token: \"3590cb8af0bbb9e78c343b52b93773c9\"",
    "sql": "SELECT * FROM Users WHERE email = ''' AND password = '3590cb8af0bbb9e78c343b52b93773c9' AND deletedAt IS NULL"
  }
}
```

This response confirms two things:

- The input is injected directly into the SQL query without sanitization — the unbalanced quote broke the SQL syntax
- The full query structure is exposed in the error response, revealing exactly how to craft a bypass payload

### Step 2 — Analyze the query structure

The leaked query is:

```sql
SELECT * FROM Users WHERE email = '[INPUT]' AND password = '[HASHED_INPUT]' AND deletedAt IS NULL
```

The application uses SQLite. In SQLite, `--` is the comment operator — everything after it is ignored by the database engine. This makes it possible to comment out the password check entirely.

---

## Exploit

Using the knowledge from the error response, the following payload was crafted:

**Email:** `' OR 1=1 --`  
**Password:** `anything`

This transforms the executed query into:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1 --' AND password = '...' AND deletedAt IS NULL
```

Three things happen:

- The opening `'` closes the string literal, breaking out of the data context
- `OR 1=1` appends a condition that is always true, causing the `WHERE` clause to match every row
- `--` comments out the rest of the query, including the password and `deletedAt` checks

The database evaluates the condition as true for all rows and returns the first result — the administrator account.

---

## Impact

- Full authentication bypass without valid credentials
- Access to the administrator account and all privileged functionality
- The verbose error response accelerated discovery by leaking the full query structure, reducing exploitation to a single crafted request

---

## Video

[Watch on Loom](#) [Login Admin](https://www.loom.com/share/06c580617fc44d68b21e6dc2255dd3c0)

---

## Recommendation

- Use parameterized queries (prepared statements) for all database interactions — never concatenate user input into SQL strings
- Never expose raw SQL error messages in HTTP responses — log errors server-side only
- Implement server-side input validation as a secondary defense layer
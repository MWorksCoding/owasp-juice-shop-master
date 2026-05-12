# Juice Shop Master

This project documents the independent discovery and exploitation of security vulnerabilities in the OWASP Juice Shop — a deliberately insecure web application designed for security training. All findings were produced in a local, self-hosted environment for educational purposes exclusively. No real systems, real user data, or real credentials were used at any point.

> **Disclaimer:** The content of this documentation is intended purely for educational purposes. The techniques described must not be applied to any system without explicit written authorization.

---

## Table of Contents

- [Quickstart](#quickstart)
- [Challenges](#challenges)
  - [Challenge 1 — Login Admin (SQL Injection)](#challenge-1--login-admin-sql-injection)
  - [Challenge 2 — Admin Registration (Mass Assignment)](#challenge-2--admin-registration-mass-assignment)
  - [Challenge 3 — Meta Geo Stalking (OSINT)](#challenge-3--meta-geo-stalking-osint)
  - [Challenge 4 — Client-side XSS Protection (Stored XSS)](#challenge-4--client-side-xss-protection-stored-xss)
- [Security Notes](#security-notes)

---

## Quickstart

### Prerequisites

- [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) running locally
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)
- Firefox or Chrome with proxy configured to Burp Suite (`127.0.0.1:8080`)
- Optional tools: `exiftool`, `sqlmap`, `haiti`

### Run Juice Shop

```bash
git clone https://github.com/juice-shop/juice-shop.git
cd juice-shop
npm install
npm start
```

Navigate to `http://localhost:3000`.

---

## Challenges

### Challenge 1 — Login Admin (SQL Injection)

**Category:** A03 – Injection | **Severity:** Critical

The login form passes unsanitized user input directly into a SQL query. By injecting `' OR 1=1 --` as the email value, the query logic is manipulated to bypass authentication entirely and log in as the administrator without valid credentials.

→ [Full documentation](./challenges/01_login_admin.md)

---

### Challenge 2 — Admin Registration (Mass Assignment)

**Category:** A04 – Insecure Design / A01 – Broken Access Control | **Severity:** Critical

The registration API returns the full internal user object including the `role` field. By intercepting the registration request and injecting `"role": "admin"`, any anonymous visitor can create an account with full administrator privileges — no prior access required.

→ [Full documentation](./challenges/02_admin_registration.md)

---

### Challenge 3 — Meta Geo Stalking (OSINT)

**Category:** A07 – Identification and Authentication Failures | **Severity:** High

A user's email address is found in public product reviews. Their uploaded Photo Wall image contains GPS coordinates in the EXIF metadata. Extracting the coordinates with `exiftool` and resolving them in Google Maps reveals the answer to their security question, enabling a full password reset without any technical exploitation.

→ [Full documentation](./challenges/03_meta_geo_stalking.md)

---

### Challenge 4 — Client-side XSS Protection (Stored XSS)

**Category:** A03 – Injection (XSS) | **Severity:** Critical

The registration UI validates the email field client-side only. By intercepting the POST request and replacing the email with an XSS iframe payload, the server stores the malicious value without server-side validation. The payload executes in every administrator's browser when they visit the admin panel — enabling session theft and account takeover from an unauthenticated starting position.

→ [Full documentation](./challenges/04_client_side_xss.md)

---

## Security Notes

- All testing was conducted against a self-hosted local instance of OWASP Juice Shop
- No real personal data, credentials, or systems were used or targeted
- No SSH keys, passwords, tokens, or IP addresses are stored in this repository
- JWT tokens visible in documentation originate from the local test environment only
- The techniques described are illegal when applied to systems without explicit written authorization


## Project Checklist

You can find a detailed checklist for this project in PDF format:

- [Download the Checklist](./docs/checklist.pdf)
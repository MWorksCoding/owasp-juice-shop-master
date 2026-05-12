# Challenge 3 — Meta Geo Stalking (OSINT)

| Field | Detail |
|---|---|
| **OWASP Category** | A07 – Identification and Authentication Failures |
| **Severity** | High |
| **Technique** | EXIF metadata extraction + geographic OSINT lookup |

---

## Background

Digital images contain embedded EXIF metadata written automatically by the capturing device. This can include GPS coordinates if location services were enabled. When applications serve raw uploaded images without stripping metadata, this information is publicly accessible to anyone who downloads the file.

Security questions rely on answers that are both memorable and secret. When the answer is derivable from publicly available data — such as image metadata — the security question provides no real protection.

---

## Discovery

**Step 1 — Identify the target**

The email address `john@juice-sh.op` was found in product reviews without authentication.

**Step 2 — Retrieve the security question**

Entering the email in the password reset flow revealed the question:

```
What's your favorite place to go hiking?
```

**Step 3 — Locate the image**

A photo titled "I love going hiking here" by user `j0hNny` was found on the public Photo Wall and downloaded.

**Step 4 — Extract GPS metadata**

```bash
exiftool favorite-hiking-place.png
# GPS Position: 36 deg 57' 31.38" N, 84 deg 20' 53.58" W
```

**Step 5 — Resolve the location**

The coordinates were entered into Google Maps:

```
36° 57' 31.38" N, 84° 20' 53.58" W
```

Zooming out revealed the named area: **Daniel Boone National Forest**, Kentucky, USA.

---

## Exploit

`Daniel Boone National Forest` was submitted as the security question answer. A new password was set and the account was fully taken over — no technical exploitation required.

---

## Impact

- Full account takeover using only public information and a browser
- No authentication, no tools, no interaction with the victim required
- The attack leaves no server-side traces

---

## Video

[Watch on Loom](#) [Meta Geo Stalking ]https://www.loom.com/share/3f2dea21fc15422c801cfbf4cc8d5e58

---

## Recommendation

- Strip EXIF metadata from all user-uploaded images server-side before storing or serving them
- Replace security questions with time-limited email-based password reset links
- Do not expose full email addresses in public product reviews

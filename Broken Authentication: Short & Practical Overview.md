```markdown
# 🔑 Broken Authentication: Short & Practical Overview

Authentication flaws are bugs in the login or identity flow that let an attacker act as another user, often leading to full account takeover.

---

## 🔐 Auth vs. AuthZ (Quick)

- **Authentication** = “Who are you?”  
  Example: Carlos logs in with email + password and the app checks if the credentials match a real account. [web:14]

- **Authorization** = “What are you allowed to do?”  
  Example: After login, Carlos can view his own profile but gets a 403 if he tries to access `/admin`. [web:17]

---

## 🧱 3 Ways Systems Identify You

- **Something you know** – Password, PIN, security question.  
- **Something you have** – Phone with OTP, hardware key, token.  
- **Something you are** – Fingerprint, Face ID, biometrics. [web:14]

Good security often combines 2+ factors (MFA).

---

## 🪓 Two Common Break Points

### 1️⃣ Weak Protections (Brute Force Style)

The login form technically “works”, but has weak defenses.

Example:  
- Site allows unlimited password attempts.  
- Attacker runs a script to try thousands of common passwords against `carlos@example.com` until one works. [web:15]

Fixes:
- Rate limiting and lockouts.
- CAPTCHA or step-up verification after several failures. [web:1]

---

### 2️⃣ Logic Flaws (Broken Implementation)

The login or session logic itself is wrong.

Example (Insecure session handling):  
- After login, the app sets a cookie `session_id=123`.  
- IDs are predictable (e.g., `124` is admin).  
- Attacker logs in as a normal user, changes cookie to `session_id=124`, and becomes admin because the server only trusts the ID, not the actual user binding. [web:17]

Fixes:
- Use long, random, unguessable session tokens.
- Always bind sessions/tokens to the real authenticated user and validate on every request. [web:14]

---

## 🚨 Short Example: Password Reset Poisoning

Password reset flows are often the weakest link.

Simple attack flow: [web:18][web:15]  
1. Attacker submits a “forgot password” request for the victim.  
2. They modify a header like `Host` so the app builds the reset link with `https://evil-site.com/reset?...`.  
3. The victim receives a legitimate-looking email but the link points to the attacker’s domain.  
4. Victim clicks; attacker’s server captures the reset token and uses it to reset the victim’s password.

Root cause: the app trusts user-controlled data (like `Host`) when building security-critical URLs.

Fixes:
- Use a hardcoded application base URL on the server.
- Never rely on unvalidated headers to construct password reset links. [web:18]

---

## 🛡️ Minimal Best Practices

- Enforce rate limiting and account lockouts on login. [web:1]  
- Use strong password hashing (Argon2, bcrypt, scrypt). [web:1]  
- Enable mandatory MFA for sensitive accounts. [web:14]  
- Treat password reset and magic-link flows as highly sensitive logic. [web:18]  
- Use secure, random, short-lived session tokens with proper logout invalidation. [web:14]

```

[1](https://github.com/adam-p/markdown-here/wiki/markdown-cheatsheet)
[2](https://www.freecodecamp.org/news/github-flavored-markdown-syntax-examples/)
[3](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/quickstart-for-writing-on-github)
[4](https://www.markdownguide.org/basic-syntax/)
[5](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
[6](https://www.sailpoint.com/identity-library/difference-between-authentication-and-authorization)
[7](https://www.acunetix.com/blog/articles/password-reset-poisoning/)
[8](https://google.github.io/styleguide/docguide/style.html)
[9](https://www.geeksforgeeks.org/computer-networks/difference-between-authentication-and-authorization/)
[10](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning)

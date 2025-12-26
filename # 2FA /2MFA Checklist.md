# 2FA / MFA Account Takeover Testing Checklist

## 1. Mapping 2FA Surface

- [ ] Identify all flows that use 2FA:
  - [ ] Login
  - [ ] Password reset / account recovery
  - [ ] New device / new location verification
  - [ ] High-risk actions (change email/phone, withdrawal, etc.)
- [ ] Identify all 2FA factors:
  - [ ] SMS / phone call
  - [ ] Email OTP
  - [ ] TOTP (Google Authenticator, Authy, etc.)
  - [ ] Push-based (in-app, FIDO, WebAuthn)
  - [ ] Backup codes / recovery methods
- [ ] Identify all 2FA management endpoints:
  - [ ] Enable / setup 2FA
  - [ ] Disable 2FA
  - [ ] Reset / change 2FA device
  - [ ] Add / edit phone number or authenticator app
  - [ ] Backup codes generation / viewing
- [ ] Check if 2FA is **enforced** or **optional** for:
  - [ ] All accounts
  - [ ] Privileged accounts
  - [ ] Sensitive actions

---

## 2. Phone / Email Reuse & Ownership

- [ ] Can the **same phone number** be registered to multiple accounts?
- [ ] Can the **same email** be used for multiple 2FA profiles?
- [ ] When adding a new phone/email:
  - [ ] Is the **old factor still valid** without re-verification?
  - [ ] Is there a **verification step** (OTP) before linking?
- [ ] On change of phone/email:
  - [ ] Is the user forced to confirm via the **old factor**?
  - [ ] Is an alert sent to the account email / previous factor?
- [ ] Try:
  - [ ] Register victim’s phone/email on attacker account.
  - [ ] Observe if 2FA codes now work for both accounts or allow takeover.

---

## 3. OTP / Code Weaknesses

- [ ] OTP **length** and complexity:
  - [ ] Too short (4–6 digits only)?
- [ ] OTP **lifetime**:
  - [ ] Is it valid for too long?
- [ ] OTP **reuse**:
  - [ ] Can the same OTP be reused multiple times?
- [ ] Brute force protection:
  - [ ] Any rate limiting on OTP submission?
  - [ ] Account lockout or cooldown after several failures?
  - [ ] IP-based throttling or device-based?
- [ ] Response behavior:
  - [ ] Different messages or response codes for valid vs invalid OTPs?
  - [ ] Timing differences that can be used as an oracle?
- [ ] Check if OTP is tied to:
  - [ ] Specific session / device
  - [ ] Specific user
  - [ ] Or can it be used cross-session / cross-user

---

## 4. Logic Bypass & Flow Skipping

- [ ] Try to **skip 2FA step**:
  - [ ] Directly access post-login pages after password auth.
  - [ ] Replay session cookies from after successful 2FA.
- [ ] Check for weak state management:
  - [ ] Is 2FA status stored in a client-side cookie/parameter?
  - [ ] Can you flip a flag like `mfa=0` → `mfa=1`?
- [ ] Alternate flows:
  - [ ] Mobile vs web app behavior (API vs browser).
  - [ ] Federated login providers (Google, SSO) avoiding 2FA.
- [ ] Check “Remember this device”:
  - [ ] Is the device token guessable or reusable?
  - [ ] Can attacker steal or replay device tokens?

---

## 5. 2FA Disable / Reset Abuse

- [ ] To disable 2FA, does the app require:
  - [ ] Current password?
  - [ ] Current 2FA code?
  - [ ] Only being logged in (session hijack risk)?
- [ ] To reset/change 2FA factor, does the app:
  - [ ] Verify via existing factor (phone/email/TOTP)?
  - [ ] Accept only password, or weaker info (DOB, security questions)?
- [ ] Test for:
  - [ ] CSRF on disable/reset 2FA endpoints.
  - [ ] Clickjacking on 2FA settings pages.
  - [ ] IDOR on 2FA management APIs (changing 2FA for another user).

---

## 6. Backup & Recovery Mechanisms

- [ ] Backup codes:
  - [ ] Are they shown again after initial generation?
  - [ ] Can they be regenerated without verifying existing factor?
  - [ ] Are they long and random enough?
- [ ] “Login without 2FA” options:
  - [ ] Email-only login links.
  - [ ] Security questions.
  - [ ] Support-based reset (via ticket/email).
- [ ] Test if backup flows **bypass 2FA entirely** with:
  - [ ] Just email access.
  - [ ] Easily guessed or OSINT-able information.

---

## 7. Phishing / MITM-Susceptible Flows

- [ ] Are OTPs or approve/deny prompts delivered over:
  - [ ] SMS (SIM swap / number recycling risk)?
  - [ ] Email (already compromised mailbox)?
- [ ] Check if:
  - [ ] Same OTP can be used from any IP/device.
  - [ ] No binding between OTP and originating session.
- [ ] Evaluate push-based MFA:
  - [ ] Can “MFA fatigue” be abused (spam push approvals)?

---

## 8. API-Level & Mobile App Issues

- [ ] Enumerate 2FA-related API endpoints:
  - [ ] `/login`, `/login2`, `/mfa/verify`, `/mfa/enable`, `/mfa/disable`, etc.
- [ ] Compare web and mobile behavior:
  - [ ] Does mobile app enforce 2FA the same way?
  - [ ] Any undocumented endpoints that skip 2FA checks?
- [ ] Check for:
  - [ ] Missing authentication on 2FA APIs.
  - [ ] Weak binding between session and OTP verification request.

---

## 9. Session & Token Handling

- [ ] After enabling/disabling 2FA:
  - [ ] Are old sessions invalidated?
- [ ] After successful 2FA:
  - [ ] Is a new session token issued?
  - [ ] Any session fixation risk?
- [ ] Remember-me and device cookies:
  - [ ] Are they protected with Secure/HttpOnly/SameSite flags?
  - [ ] Are they easily guessable or brute-forceable?

---

## 10. Reporting & Severity Notes

When you find an issue:

- [ ] Clearly show how 2FA can be **bypassed or weakened** to achieve:
  - [ ] Account takeover (ATO)
  - [ ] Privilege escalation
  - [ ] Unauthorized access to sensitive actions
- [ ] Map against:
  - [ ] Common 2FA bypass categories (improper implementation, logic flaws, weak backup methods).
  - [ ] Program’s payout guidelines for MFA/ATO if available.

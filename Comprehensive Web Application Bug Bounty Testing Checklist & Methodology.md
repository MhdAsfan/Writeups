
# Comprehensive Web Application Bug Bounty Testing Checklist & Methodology

This document provides a deep, systematic approach to web application penetration testing and bug bounty hunting. It covers reconnaissance, vulnerability assessment, exploitation, and reporting across all major OWASP categories and common attack vectors.

---

## Summary

---

## Phase 1: Reconnaissance & Information Gathering

### 1.1 Passive Reconnaissance

**Domain & Network Enumeration / URL & Endpoint Discovery / Technology Stack Identification**
- [ ] Use Wappalyzer, BuiltWith, Shodan
- [ ] Identify web framework, CMS, server software
- [ ] Check HTTP headers for version information
- [ ] Enumerate SSL/TLS certificates (crt.sh, SSL Labs)
- [ ] Identify third-party services and APIs
- [ ] Check `robots.txt`, `sitemap.xml`, `.well-known/` directories
- [ ] Identify all subdomains using: `subfinder`, `assetfinder`, `amass`  
  - DNS brute-forcing: `dnsx`, `dnsgen`  
  - CNAME enumeration and alias discovery  
  - Reverse DNS lookups
- [ ] Check WHOIS records for organizational info
- [ ] Identify IP ranges and ASN information
- [ ] Look for forgotten/deprecated subdomains on old hosting
- [ ] Search for archived versions on Wayback Machine
- [ ] Monitor GitHub for exposed credentials and API endpoints
- [ ] Use `waybackurls` to find historical endpoints
- [ ] Crawl with `hakrawler`, `gau` (Google, Archive, URLScan)
- [ ] Check for API documentation (Swagger, OpenAPI, GraphQL)
- [ ] Brute-force common paths and extensions
- [ ] Identify admin panels, backup files, config files
- [ ] Look for `.git`, `.svn`, `.env`, `.DS_Store` exposed

**API Enumeration**
- [ ] Identify all API endpoints
- [ ] Document API authentication mechanisms
- [ ] Check for API versioning (v1, v2, etc.)
- [ ] Test for rate limiting and throttling
- [ ] Look for API keys in responses and headers
- [ ] Test for account registration bypass
- [ ] Check email verification mechanisms
- [ ] Look for mass assignment/parameter pollution
- [ ] Test for weak password reset tokens
- [ ] Verify CAPTCHA bypass possibilities
- [ ] Check for privilege escalation via registration

### 1.2 Active Reconnaissance

**Network Scanning**
- [ ] Port scanning: `nmap`, `masscan`
- [ ] Service enumeration and version detection
- [ ] Check for open management interfaces (RDP, SSH, etc.)
- [ ] Identify cloud storage buckets (S3, GCS, Azure Blob)
- [ ] Test for default credentials (admin/admin, test/test, etc.)
- [ ] Check password strength requirements

**Vulnerability Testing**
- [ ] Test for SQL injection in login forms
- [ ] Test for NoSQL injection in login forms
- [ ] Look for username enumeration vulnerabilities
- [ ] Check for response timing differences (timing attacks)
- [ ] Test for brute-force protection and account lockout
- [ ] Verify rate limiting on authentication endpoints
- [ ] Fingerprint web server and version
- [ ] Test for HTTP methods (GET, POST, PUT, DELETE, TRACE, OPTIONS)
- [ ] Identify allowed HTTP methods per endpoint
- [ ] Check for TRACE method enabled (HTTP Response Splitting)
- [ ] Enumerate virtual hosts and hosted applications

---

## Phase 2: Authentication & Session Management

### 2.1 Authentication Testing

**OAuth & Third-Party Auth**
- [ ] Verify OAuth token validation
- [ ] Check redirect URI validation (open redirect)
- [ ] Test for code/token replay attacks
- [ ] Verify scope limitations are enforced
- [ ] Check for state parameter validation
- [ ] Test for cross-site request forgery in auth flow
- [ ] Verify logout functionality clears all sessions
- [ ] Test for session replay attacks
- [ ] Check for cross-domain session sharing

**Session Handling**
- [ ] Verify privilege level enforced per session
- [ ] Test for session validation on sensitive operations
- [ ] Identify all numeric/sequential IDs (user ID, order ID, etc.)
- [ ] Test parameter manipulation (Direct, Base64, Hash, UUIDs)
- [ ] Check all HTTP methods (GET, POST, PUT, PATCH, DELETE)
- [ ] Verify access control on API endpoints
- [ ] Check GraphQL for enumeration of other users' data
- [ ] Analyze session token entropy and randomness
- [ ] Check token storage mechanism (cookies, localStorage, sessionStorage)
- [ ] Verify secure and httpOnly flags on cookies
- [ ] Check SameSite cookie attribute
- [ ] Test for session fixation vulnerabilities
- [ ] Look for session token prediction/brute-force
- [ ] Check token expiration and timeout behavior
- [ ] Test for concurrent session limits

---

## Phase 3: Authorization & Access Control

### 3.1 Broken Access Control (IDOR)
- [ ] Extract database version and structure
- [ ] Enumerate database users and privileges
- [ ] Dump sensitive data (users, credentials, PII)
- [ ] Check for second-order SQL injection
- [ ] Test for SQL injection in different contexts (WHERE, ORDER BY, INSERT, UPDATE)
- [ ] Verify protection mechanisms (WAF bypass)
- [ ] Test input fields for SQL injection (form inputs, URL parameters, headers, API)
- [ ] Map all endpoints requiring authentication
- [ ] Test unauthenticated access to protected resources
- [ ] Verify proper authorization checks on all endpoints
- [ ] Test bypass via path manipulation, case sensitivity, null byte injection, role/privilege tampering
- [ ] Check direct admin panel access
- [ ] Verify privilege checks on sensitive operations
- [ ] Test for hidden parameters (role=admin, is_admin=true)
- [ ] Check for parameter pollution
- [ ] Test for privilege escalation via file upload/manipulation
- [ ] Look for JWT/token manipulation to elevate privileges

---

## Phase 4: Input Validation & Injection Attacks

### 4.1 SQL Injection
- [ ] Out-of-band SQL injection (DNS exfiltration)
- [ ] Stacked queries for arbitrary command execution
- [ ] Test OS command execution via SQL (MSSQL, MySQL, PostgreSQL)
- [ ] Test JSON/MongoDB injection
- [ ] Check operator injection in API parameters
- [ ] Test JavaScript injection in MongoDB
- [ ] Verify protection mechanisms
- [ ] Identify functions that execute system commands
- [ ] Test input parameters for injection (command separators, substitution, newline injection)
- [ ] Test blind command injection with time delays
- [ ] Attempt out-of-band exfiltration (DNS, HTTP callbacks)
- [ ] Check for filter bypasses (case variation, encoding, null bytes, variable expansion)

### 4.2 NoSQL Injection
- [ ] Test LDAP filters for injection
- [ ] Bypass authentication with wildcard
- [ ] Extract directory information
- [ ] Enumerate users and groups
- [ ] Test Angular, Vue, React templates for injection
- [ ] Exploit prototype pollution in JavaScript

### 4.3 Command Injection
- [ ] Exploit OS command execution
- [ ] Test blind command injection
- [ ] Attempt out-of-band exfiltration

### 4.4 Template Injection (SSTI)
- [ ] Identify template engines (Jinja2, Freemarker, Velocity, etc.)
- [ ] Test for injection `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`
- [ ] Exploit for RCE (Python, Java, PHP examples)

---

*(The checklist continues similarly for XSS, CSRF, Security Misconfigurations, File Uploads, Cryptographic Failures, Business Logic Flaws, SSRF, Deserialization, Component Vulnerabilities, API-Specific Vulnerabilities, Subdomain Takeover, Advanced Techniques, Exploitation & Proof of Concept, Reporting & Disclosure, and Continuous Monitoring.)*

---

## Appendix: Tool Reference

**Reconnaissance Tools**
- Wappalyzer, BuiltWith, Shodan  
- crt.sh, SSL Labs  
- subfinder, assetfinder, amass  
- Waybackurls, gau, hakrawler  

**Scanning & Enumeration**
- Nmap, masscan  
- Nikto, dirbuster  
- Nuclei, Burp Suite, OWASP ZAP  

**Exploitation Tools**
- Burp Suite, SQLmap, NoSQL injection tools  
- XXE payloads, SSTI templates  
- JWT tools (jwt.io, jwtcat)  
- Ysoserial (Java deserialization)  
- Curl, Postman, Insomnia  
- Python & Bash scripting  

**Utility & Documentation**
- Jira, GitHub Issues  
- Markdown, LaTeX  
- Screenshot tools (Flameshot, Snagit)  
- Video recording (OBS, Loom)

---


# OWASP Testing & Bug Bounty Interview Guide
## Simplified Edition - Quick Reference

---

## **Part 1: OWASP Testing Framework Basics**

### What is OWASP Testing Framework?

OWASP provides a structured approach to find security vulnerabilities in web applications throughout the development lifecycle.

**Core Principle:** Test security early and often, not just at the end.

---

### The 5 Phases of OWASP Testing

**Phase 1: Before Development**
- Plan security requirements
- Identify what needs protection
- Map potential threats

**Phase 2: During Design**
- Review architecture documents
- Validate security controls
- Create threat models

**Phase 3: During Development**
- Code review and static analysis
- Developer security testing
- Fix issues as you code

**Phase 4: During Deployment**
- Penetration testing
- Configuration checks
- Vulnerability scanning

**Phase 5: After Deployment**
- Continuous monitoring
- Regular security audits
- Patch management

---

### Why Test Early?

**Cost to Fix Bugs:**
- Development phase: $1,000
- Testing phase: $15,000
- Production phase: $50,000 - $500,000

**Key Takeaway:** Finding bugs early saves money and protects users.

---

## **Part 2: Common Vulnerability Types**

### OWASP Top 10 (2021) - Quick Reference

**1. Broken Access Control**
- Users access data they shouldn't
- Example: Change user ID in URL to see other's data

**2. Cryptographic Failures**
- Weak encryption or no encryption
- Example: Storing passwords in plain text

**3. Injection (SQL, Command, etc.)**
- Malicious code inserted into application
- Example: `admin' OR '1'='1` in login field

**4. Insecure Design**
- Missing security controls in design
- Example: No rate limiting on login

**5. Security Misconfiguration**
- Default settings left unchanged
- Example: Default admin password still active

**6. Vulnerable Components**
- Using outdated libraries with known bugs
- Example: Old jQuery version with XSS vulnerability

**7. Authentication Failures**
- Weak login mechanisms
- Example: Predictable session tokens

**8. Data Integrity Failures**
- No verification of data/updates
- Example: Software updates without signature checks

**9. Logging Failures**
- Not recording security events
- Example: Failed logins not logged

**10. Server-Side Request Forgery (SSRF)**
- Server makes requests to attacker-controlled URLs
- Example: Accessing internal admin panel via server

---

## **Part 3: Bug Bounty Basics**

### What Makes a Finding "Unique"?

**Unique = Requires separate fix**

**Example Scenario:**
- 5 SQL injections in different endpoints
- Each uses different code → 5 unique findings
- All use same function → 1 finding

**Bugcrowd Rule:** "Touch the code, pay the bug"

---

### Bug Bounty vs Penetration Testing

| Aspect | Bug Bounty | Penetration Testing |
|--------|------------|---------------------|
| Timeline | Ongoing | Fixed (1-4 weeks) |
| Testers | Global community | Specific team |
| Payment | Per vulnerability | Fixed fee |
| Environment | Production | Usually staging |
| Coverage | Very high | Systematic |

---

### The 100-Hour Rule

**Spend 100+ hours on one target for best results**

**Why it works:**
- 0-20 hours: Easy bugs
- 20-50 hours: Overlooked areas
- 50-100 hours: Pattern recognition
- 100+ hours: Deep vulnerabilities

**Key Insight:** Stay focused on one target instead of jumping between many.

---

## **Part 4: Testing Real-World Scenarios**

### Scenario 1: Testing an E-commerce Site

**Step 1: Reconnaissance**
- Find subdomains: `subfinder -d example.com`
- Discover endpoints: `dirsearch -u https://example.com`

**Step 2: Find Interesting Endpoints**
- `/fileupload` - File upload functionality
- `/api/orders` - Order management
- `/admin` - Admin panel

**Step 3: Test for IDOR (Access Control)**
```
GET /api/orders/user/123  → Your orders
GET /api/orders/user/124  → Try other user's orders
```

**If successful = CRITICAL vulnerability**

**Step 4: Test for SQL Injection**
```
Search: product' OR '1'='1
If all products return = SQL Injection found
```

**Step 5: Test for XSS**
```
Comment: <script>alert('XSS')</script>
If script runs = XSS vulnerability
```

---

### Scenario 2: File Upload LFI (Local File Inclusion)

**Target:** File upload endpoint

**Step 1: Basic Upload Test**
```bash
curl -X POST https://example.com/fileupload/ \
-F "file=@test.txt"
```

**Step 2: Try Path Traversal**
```bash
curl -X POST https://example.com/fileupload/ \
-F "file=@../../../etc/passwd"
```

**Step 3: If Successful, Access Sensitive Files**
```
/etc/passwd
/etc/shadow
/etc/nginx/nginx.conf
/etc/mysql/my.cnf
```

**Severity:** CRITICAL - Can read system files

**Bounty Range:** $3,000 - $10,000

---

## **Part 5: Common Vulnerabilities Explained**

### 1. SQL Injection

**What it is:** Inserting SQL commands into input fields

**Vulnerable Code:**
```python
query = f"SELECT * FROM users WHERE username='{input}'"
```

**Attack:**
```
Input: admin' OR '1'='1
Result: Bypasses login, returns all users
```

**Fix:** Use parameterized queries
```python
query = "SELECT * FROM users WHERE username=?"
cursor.execute(query, (input,))
```

---

### 2. Cross-Site Scripting (XSS)

**What it is:** Injecting JavaScript that runs in victim's browser

**Three Types:**

**Reflected XSS:**
```
URL: example.com/search?q=<script>alert('XSS')</script>
```

**Stored XSS:**
```
Comment: <img src=x onerror="alert('XSS')">
Saved to database, executes for all users
```

**DOM-based XSS:**
```
JavaScript reads URL and inserts without sanitizing
```

**Test Payloads:**
```html
<script>alert('XSS')</script>
<img src=x onerror="alert('XSS')">
<svg onload="alert('XSS')">
```

---

### 3. IDOR (Insecure Direct Object Reference)

**What it is:** Accessing other users' data by changing IDs

**Example:**
```
Your profile: /api/user/1000
Try: /api/user/1001, /api/user/1002, /api/user/1003
```

**If you see other users' data = IDOR vulnerability**

**Testing Steps:**
1. Create two accounts (if possible)
2. Note their user IDs
3. Try accessing Account B data while logged in as Account A
4. If successful, report as CRITICAL

---

### 4. SSRF (Server-Side Request Forgery)

**What it is:** Making the server request internal URLs

**Vulnerable Endpoint:**
```
POST /api/fetch
Body: {"url": "https://example.com/file.pdf"}
```

**Attack:**
```
POST /api/fetch
Body: {"url": "http://127.0.0.1:8080/admin"}
```

**Result:** Access internal admin panel through server

**Common Targets:**
- `http://127.0.0.1:admin` - Internal admin
- `http://169.254.169.254/` - AWS metadata
- `file:///etc/passwd` - Local files

---

## **Part 6: How to Report Vulnerabilities**

### Professional Report Structure

**1. Clear Title**
```
✓ GOOD: "SQL Injection in /search - Unauthenticated Access"
✗ BAD: "Security Bug Found"
```

**2. Severity**
```
CRITICAL / HIGH / MEDIUM / LOW
Include CVSS score if possible
```

**3. Description (2-3 sentences)**
```
Explain what's vulnerable and why it matters
```

**4. Steps to Reproduce**
```
1. Go to https://example.com/search
2. Enter: ' OR '1'='1
3. Click Search
4. Observe all products returned
```

**5. Proof of Concept**
```bash
curl -X GET "https://example.com/search?q=' OR '1'='1"
```

**6. Impact**
```
- All user data accessible
- 500,000+ users affected
- Data can be modified/deleted
```

**7. Remediation**
```
Use parameterized queries
Implement input validation
Add rate limiting
```

**8. Screenshots**
- Show the vulnerability in action
- Include request/response data

---

### Report Example

```
═══════════════════════════════════════════
VULNERABILITY REPORT
═══════════════════════════════════════════

Title: SQL Injection in Product Search
Severity: CRITICAL (CVSS 9.8)
Date: 2025-11-20

DESCRIPTION:
The /search endpoint doesn't sanitize user input,
allowing SQL injection attacks that expose the
entire database.

STEPS TO REPRODUCE:
1. Visit https://example.com/search
2. Enter: ' OR '1'='1' --
3. All products returned (bypass confirmed)

IMPACT:
- Complete database access
- 500,000+ users affected
- Credentials exposed

REMEDIATION:
Use parameterized queries instead of string
concatenation in SQL statements.
═══════════════════════════════════════════
```

---

## **Part 7: Responsible Disclosure**

### Do's and Don'ts

**✅ DO:**
- Use official bug bounty platforms
- Test only what's necessary
- Report immediately when found
- Keep findings confidential
- Document everything
- Wait for fix before public disclosure

**❌ DON'T:**
- Access more data than needed for proof
- Modify or delete data
- Share findings publicly before fix
- Test on production without permission
- Disrupt services
- Sell exploits to third parties

---

### Disclosure Timeline

**Standard Timeline:**
- Day 1: Report vulnerability
- Day 3: Program acknowledges
- Day 7: Program confirms issue
- Day 30: Patch developed
- Day 90: Patch deployed
- Day 120: Public disclosure allowed

---

## **Part 8: Career Path**

### Bug Bounty Hunter Progression

**Month 1-3: Beginner**
- Learn OWASP Top 10
- Practice on labs (PortSwigger Academy)
- 0-1 findings expected
- Focus: Learning fundamentals

**Month 3-6: Intermediate**
- Choose a specialization (SQLi, XSS, IDOR)
- 2-5 findings per program
- Focus: Building methodology

**Month 6-12: Advanced**
- Master business logic testing
- 5-10 findings per program
- Focus: Deep testing, chaining vulnerabilities

**Year 1+: Expert**
- Find critical vulnerabilities consistently
- $1,000-$5,000/month potential
- Focus: Unique findings, advanced techniques

---

### Essential Skills to Master

**Technical Skills:**
- HTTP/HTTPS protocol
- Web technologies (HTML, JavaScript, SQL)
- Linux command line
- Burp Suite / OWASP ZAP
- Python scripting (basic)

**Soft Skills:**
- Clear technical writing
- Patience and persistence
- Ethical mindset
- Continuous learning
- Communication with security teams

---

## **Part 9: Quick Testing Checklist**

### Before Submitting a Report

- [ ] Can I reproduce it 100% reliably?
- [ ] Is it truly a security issue?
- [ ] What's the real-world impact?
- [ ] Have I documented all steps clearly?
- [ ] Did I check if it's already reported?
- [ ] Is my proof-of-concept clear?
- [ ] Have I included screenshots?
- [ ] Did I suggest remediation?
- [ ] Is my report professionally written?
- [ ] Did I avoid accessing unnecessary data?

---

### Program Selection Checklist

- [ ] Does program have good reputation?
- [ ] Are bounty amounts competitive?
- [ ] Is scope broad enough?
- [ ] How fast do they respond?
- [ ] What's the duplicate rate?
- [ ] Are there recent payouts?
- [ ] Is it beginner-friendly?

---

## **Part 10: Essential Tools**

### Reconnaissance Tools
- **subfinder** - Subdomain enumeration
- **assetfinder** - Find assets and subdomains
- **amass** - Attack surface mapping

### Content Discovery
- **dirsearch** - Directory/file brute-forcing
- **ffuf** - Fast fuzzing tool
- **gobuster** - URL/DNS brute-forcing

### Testing Tools
- **Burp Suite** - Intercept and modify requests
- **OWASP ZAP** - Web app security scanner
- **sqlmap** - SQL injection automation
- **nuclei** - Vulnerability scanner

### Useful Commands

**Subdomain Enumeration:**
```bash
subfinder -d example.com -o subdomains.txt
```

**Content Discovery:**
```bash
dirsearch -u https://example.com -w wordlist.txt
```

**File Upload Test:**
```bash
curl -X POST https://example.com/upload \
-F "file=@test.txt"
```

**IDOR Test:**
```bash
curl -H "Authorization: Bearer TOKEN" \
https://example.com/api/user/1001
```

---

## **Quick Reference Card**

### CVSS Severity Scale

| Score | Rating | Bounty Range |
|-------|--------|--------------|
| 9.0-10.0 | CRITICAL | $5,000-$50,000 |
| 7.0-8.9 | HIGH | $1,000-$5,000 |
| 4.0-6.9 | MEDIUM | $250-$1,000 |
| 0.1-3.9 | LOW | $50-$250 |

### Common Ports to Check

- 80/443 - HTTP/HTTPS
- 8080/8443 - Alternative HTTP
- 3306 - MySQL
- 5432 - PostgreSQL
- 27017 - MongoDB
- 6379 - Redis
- 22 - SSH
- 21 - FTP

### HTTP Status Codes

- 200 - OK
- 301/302 - Redirect
- 401 - Unauthorized
- 403 - Forbidden
- 404 - Not Found
- 500 - Server Error

---

## **Key Principles for Success**

1. **Methodology > Tools**
   - Understanding beats automation

2. **Patience > Speed**
   - 100+ hours on one target

3. **Manual > Automated**
   - Avoid duplicate findings

4. **Unique > Common**
   - Focus on what others miss

5. **Business Logic > Generic**
   - High-impact findings

6. **Responsible > Reckless**
   - Ethical testing always

7. **Documentation > Speed**
   - Clear reports get paid faster

---

## **Final Tips**

### For Beginners
- Start with easy programs (VDP - Vulnerability Disclosure Programs)
- Read 100+ writeups before starting
- Practice on intentionally vulnerable apps (DVWA, WebGoat)
- Join bug bounty communities (Discord, Reddit, Twitter)
- Don't expect earnings for first 3 months

### For Interviews
- Know OWASP Top 10 by heart
- Understand one vulnerability deeply (become specialist)
- Prepare real-world examples from your experience
- Practice explaining technical concepts simply
- Have a portfolio of disclosed vulnerabilities

### For Career Growth
- Build a strong GitHub profile with security tools
- Write technical blog posts about findings
- Contribute to open-source security projects
- Get certifications (CEH, OSCP, OSWE)
- Network with other security researchers

---

## **Resources**

**Learning Platforms:**
- PortSwigger Web Security Academy (Free)
- OWASP Testing Guide
- HackerOne Hacktivity
- Bugcrowd University
- PentesterLab

**Bug Bounty Platforms:**
- HackerOne
- Bugcrowd
- Intigriti
- YesWeHack
- Synack

**Practice Labs:**
- DVWA (Damn Vulnerable Web App)
- WebGoat
- Hack The Box
- TryHackMe
- PentesterLab

---

**Remember:** Bug bounty hunting requires patience, ethics, and continuous learning. Your first bounty might take 2-3 months, but the skills you build last a lifetime.

**Good luck and happy hunting! 🎯**

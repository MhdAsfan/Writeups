# Google Dorks List for Security Reconnaissance

## Overview
This README provides a curated list of Google Dorks (advanced search queries, also known as "Google Hacking" techniques) designed for ethical security reconnaissance. These dorks help identify potential weaknesses, exposed sensitive information, and misconfigurations in web applications and systems—aligning with security frameworks like NIST CSF (Identify function) and ISO 27001 (risk assessment controls).

Google Dorking leverages search operators (e.g., `inurl:`, `filetype:`, `intitle:`) to uncover publicly indexed data that shouldn't be exposed. This tool is invaluable for penetration testing, bug bounty hunting, and proactive vulnerability scanning.

**Key Operators Quick Reference:**
| Operator | Description | Example |
|----------|-------------|---------|
| `site:` | Limit to a specific domain | `site:example.com` |
| `inurl:` | Search in URL | `inurl:admin` |
| `intitle:` | Search in page title | `intitle:"index of"` |
| `filetype:` | Specific file type | `filetype:pdf` |
| `intext:` | Search in page text | `intext:"password"` |
| `cache:` | View cached page | `cache:example.com` |
| `related:` | Find similar sites | `related:example.com` |

## Disclaimer and Ethical Use
- **Educational Purposes Only:** This list is for authorized use in penetration testing, red teaming, bug bounties, or self-assessments. Unauthorized scanning may violate laws (e.g., CFAA in the US) or terms of service.
- **Responsible Disclosure:** If you find vulnerabilities, report them via proper channels (e.g., HackerOne, company security teams). Do not exploit.
- **Anonymity and Legality:** Use VPNs/Tor for privacy. Always obtain permission for third-party targets.
- **Mitigation Tips:** Protect your site with `robots.txt`, proper indexing controls, and regular Google Search Console audits.

For more, consult the [Google Hacking Database (GHDB)](https://www.exploit-db.com/google-hacking-database).

## Dork Categories and Examples
Dorks are grouped by GHDB-inspired categories. Replace `site:example.com` with your target domain. Test responsibly.

### 1. Directory Listings (Exposed File Structures)
These reveal open directories that may leak file listings.
- `intitle:"index of" "parent directory" site:example.com` – Finds Apache/Nginx directory indexes.
- `intitle:"index of" inurl:backup site:example.com` – Targets backup folders.

### 2. Sensitive Files (Documents, Configs, Logs)
Uncover exposed configs, logs, or databases.
- `filetype:log intext:"password" site:example.com` – Searches for logs with credentials.
- `filetype:sql "insert into" site:example.com` – Dumps SQL database files.
- `filetype:env intext:"DB_PASSWORD" site:example.com` – Environment files with secrets (e.g., .env).
- `inurl:wp-config.php site:example.com` – WordPress config files.

### 3. Login Portals and Admin Panels
Locate authentication pages for further testing.
- `inurl:login | inurl:signin | intitle:login | intitle:signin | inurl:secure site:example.com` – Common login forms.
- `intitle:"admin login" inurl:panel site:example.com` – Admin interfaces.
- `inurl:/admin filetype:html site:example.com` – HTML admin pages.

### 4. Error Messages and Vulnerabilities
Spot debug info or error pages revealing stack traces.
- `intext:"PHP Parse error" site:example.com` – PHP errors with code paths.
- `intitle:"error" inurl:debug site:example.com` – Debug modes enabled.
- `inurl:"sql syntax near" site:example.com` – SQL injection hints.

### 5. Files Containing Juicy Info (Passwords, Usernames)
Direct credential hunting.
- `filetype:txt "password" | "username" site:example.com` – Text files with creds.
- `intext:"api_key" filetype:js site:example.com` – JavaScript with API keys.
- `intitle:"index of" inurl:passwd site:example.com` – Unix password files.

### 6. Online Devices and Cameras
For IoT/embedded recon (use sparingly).
- `inurl:/view.shtml intitle:"Live View / - AXIS" site:example.com` – AXIS cameras.
- `inurl:/control/userimage.html intitle:"Toshiba Network Camera" site:example.com` – Toshiba cams.

### 7. Network/Vulnerability Data
Find exposed tools or reports.
- `filetype:pdf "vulnerability assessment" site:example.com` – Security reports.
- `inurl:shodan site:example.com` – Shodan-indexed devices.

## How to Use This List
1. **Manual Testing:** Copy a dork into Google Search, append `site:target.com`, and review results. Use incognito mode.
2. **Automation:** Integrate with tools like:
   - [DorkRecon](https://github.com/Priyank-CyberK/DorkRecon) – Customizable dork runner.
   - [BigBountyRecon](https://github.com/Viralmaniar/BigBountyRecon) – 58+ techniques with dorks.
   - [GoogleDorker](https://github.com/RevoltSecurities/GoogleDorker) – Async dorking for bug hunting.
3. **Customization:** Edit `dorks.txt` in tools or add to this repo via PR.
4. **Advanced:** Chain operators, e.g., `intitle:"index of" filetype:log site:example.com`.

## Contributing
- Fork the repo and submit PRs with new dorks (include category, explanation, and source).
- Follow GHDB submission guidelines for quality.

## Resources
- [Google Hacking Database (GHDB)](https://www.exploit-db.com/google-hacking-database) – Official dork archive.
- [OWASP: Conduct Search Engine Discovery](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage) – Testing guide.
- [Google Dorks Cheat Sheet 2025](https://www.stationx.net/google-dorks-cheat-sheet/) – Updated operators.

Stay ethical, stay secure! Last updated: November 09, 2025.

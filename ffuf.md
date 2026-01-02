## What ffuf is used for

ffuf (**Fuzz Faster U Fool**) is a fast web fuzzer used to:  

- Discover **hidden directories/files** (content discovery).[1][2]
- Fuzz **parameters and values** for weird behavior and bugs.[3]
- Bruteforce **endpoints in APIs** (methods, IDs, resources).[2]
- Test **hostnames, virtual hosts, subdomains**.  
- Automate **“try many inputs, watch for interesting responses”** workflows.[4][5]

Anywhere you can put a variable piece into a URL, header, body, or parameter, you can put `FUZZ` and let ffuf iterate a wordlist.

***

## Basic pattern: `FUZZ` placeholder

Core idea:

```bash
ffuf -w WORDLIST -u "https://target.com/FUZZ"
```

- `FUZZ` is replaced by each word in `WORDLIST`.[5][4]
- You then filter by status code, size, words, etc.

***

## 1. Directory / file fuzzing

Find hidden paths like `/admin`, `/backup.zip`, `/api/`:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
     -u https://target.com/FUZZ \
     -mc 200,301,302
```

- `-mc` → show only 200/301/302.[1][2]

***

## 2. Parameter name fuzzing

Find **hidden GET parameters**:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u "https://target.com/page.php?FUZZ=value" \
     -mc 200
```

Find **hidden POST JSON keys**:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -X POST \
     -H "Content-Type: application/json" \
     -u https://target.com/api/v1/profile \
     -d '{"FUZZ":"test"}' \
     -mc 200
```

Watch for changes in response length/status when parameter exists vs ignored.[2][3]

***

## 3. Parameter VALUE fuzzing

Useful for logic bugs, LFI, SSTI, etc.

Example: fuzz `doc` value:

```bash
ffuf -w /usr/share/seclists/Fuzzing/fuzz-Bo0oM.txt \
     -u "http://target.com/view?doc=FUZZ" \
     -mc 200
```

Example: fuzz JSON value for `user`:

```bash
ffuf -w /path/to/payloads.txt \
     -X POST \
     -H "Content-Type: application/json" \
     -u https://target.com/api/login \
     -d '{"user":"FUZZ","password":"pass"}' \
     -mc 200,403
```

***

## 4. API endpoint fuzzing

Fuzz paths inside an API:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/api/endpoints.txt \
     -u https://api.target.com/FUZZ \
     -mc 200,401,403
```

Fuzz deeper (e.g. `/v1/FUZZ/list`):

```bash
ffuf -w api_words.txt \
     -u https://api.target.com/v1/FUZZ/list \
     -mc 200
```

***

## 5. Virtual host / Host header fuzzing

Check vhosts on same IP:

```bash
ffuf -w hosts.txt \
     -u http://IP/ \
     -H "Host: FUZZ.target.com" \
     -mc 200,302
```

Good for **hidden admin apps** or multi-tenant panels.

***

## 6. Filtering and making output usable

Filter by status:

```bash
ffuf -w wordlist.txt -u https://target.com/FUZZ -mc 200,301,302
```

Filter out noise sizes:

```bash
ffuf -w wordlist.txt -u https://target.com/FUZZ -fc 404 -fs 1234
```

Save results:

```bash
ffuf -w wordlist.txt -u https://target.com/FUZZ \
     -mc 200 \
     -of json -o ffuf_results.json
```

Auto-calibrate:

```bash
ffuf -w wordlist.txt -u https://target.com/FUZZ -ac
```

This helps ignore uniform “not found” responses automatically.[6][7][5]

***

## How to think about “fuzz”

You **fuzz** when you don’t know:

- the **path** → directory/file fuzzing.  
- the **parameter name** → param name fuzzing.  
- the **parameter value** → payload/value fuzzing.  
- the **host** → vhost/Subdomain/Host header fuzzing.

ffuf gives you the engine; the **smart part** is:

- choosing the right **wordlist** for the tech/target.  
- focusing your fuzzing (e.g. only auth URLs, only API URLs).  
- interpreting **differences** in responses (size, code, body).  

[1](https://infosecwriteups.com/content-discovery-with-ffuf-5bc81d2d8db6)
[2](https://hacklido.com/blog/969-directory-fuzzing-and-hidden-resources-using-ffuf-effectively)
[3](https://agentsteal.com/fuzz-parameters-directories-more-with-ffuf/)
[4](https://www.briskinfosec.com/tooloftheday/toolofthedaydetail/Fuzz-Faster-U-Fool-Tool-to-Fuzzing-Get-and-Post-data)
[5](https://hackviser.com/tactics/tools/ffuf)
[6](https://hackzone.in/blog/ffuf-bug-bounty-ultimate-guide/)
[7](https://blog.leetsec.in/stop-fuzzing-blindly-the-ultimate-guide-to-ffuf-bce8e0cdb4bd)
[8](https://github.com/0xPugal/fuzz4bounty)
[9](https://github.com/tamimhasan404/FFUF-Tips-And-Tricks)
[10](https://wimma-capstone.pages.labranet.jamk.fi/support-material/4.%20TEST/3.%20Tools%20for%20testing/Tool%20-%20Ffuf/ffuf_guide/)

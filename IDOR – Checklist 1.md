
# 🔐 IDOR (Insecure Direct Object Reference) –  Checklist

A practical, real-world checklist for bug bounty hunters and penetration testers.

---

## 1. Asset & Entry Point Discovery
Identify where IDORs are most likely to exist.

- [ ] Authenticated endpoints
- [ ] Unauthenticated endpoints
- [ ] REST APIs
- [ ] GraphQL APIs
- [ ] Mobile application APIs
- [ ] Background XHR / fetch requests
- [ ] WebSocket endpoints
- [ ] File download / view endpoints
- [ ] Export features (PDF, CSV, Excel)
- [ ] Admin or internal endpoints

---

## 2. Object Identifier Identification
Find anything that references an object.

### Common Identifiers
- [ ] `id`
- [ ] `user_id`
- [ ] `uid`
- [ ] `account_id`
- [ ] `order_id`
- [ ] `invoice_id`
- [ ] `transaction_id`
- [ ] `document_id`
- [ ] `file_id`
- [ ] `message_id`
- [ ] `ticket_id`

### Hidden / Indirect Locations
- [ ] URL path parameters (`/users/123`)
- [ ] JSON / XML request bodies
- [ ] HTTP headers
- [ ] Cookies
- [ ] JWT payloads
- [ ] Base64-encoded values
- [ ] Encrypted-looking fields

---

## 3. Access Control Validation
Test whether ownership is enforced.

- [ ] User A accessing User B’s data
- [ ] Same role → different user
- [ ] Lower privilege → higher privilege
- [ ] Higher privilege → lower privilege
- [ ] Logged-in → logged-out
- [ ] Fresh account → old account
- [ ] Free user → paid user

---

## 4. ID Manipulation Techniques

### Basic Manipulation
- [ ] Increment / decrement IDs
- [ ] Random valid IDs
- [ ] Reuse IDs from other endpoints
- [ ] IDs from emails, PDFs, notifications

### Advanced Manipulation
- [ ] Swap ID types (`user_id` → `account_id`)
- [ ] Replace UUID with numeric ID
- [ ] Replace numeric ID with UUID
- [ ] `null` values
- [ ] Empty values
- [ ] Negative numbers
- [ ] Very large numbers
- [ ] Duplicate parameters (`id=1&id=2`)
- [ ] Array injection (`id[]=1`)

### Encoding Tricks
- [ ] URL encoding
- [ ] Double URL encoding
- [ ] Base64 encode / decode
- [ ] JSON structure tampering

---

## 5. HTTP Method Abuse
Same endpoint, different behavior.

- [ ] GET → POST
- [ ] POST → PUT
- [ ] PUT → PATCH
- [ ] DELETE access checks
- [ ] OPTIONS / HEAD testing

---

## 6. Mass Assignment + IDOR
High-impact combination.

- [ ] Change object `id` in update requests
- [ ] Inject hidden parameters
- [ ] Modify restricted fields
- [ ] Change ownership or roles

---

## 7. State-Changing Actions (High Severity)
Focus on actions, not just reads.

- [ ] Update profile data
- [ ] Change email
- [ ] Change password
- [ ] Cancel subscriptions
- [ ] Delete accounts
- [ ] Approve / reject actions
- [ ] Mark items as paid / shipped
- [ ] Transfer ownership
- [ ] Assign or change roles

---

## 8. Indirect IDORs (Often Missed)
IDs that don’t look like IDs.

- [ ] Username-based access
- [ ] Email-based access
- [ ] Filenames (`invoice_123.pdf`)
- [ ] Order references
- [ ] Slugs
- [ ] Predictable document names

---

## 9. Chained IDORs
One bug feeds another.

- [ ] Leak object IDs from one endpoint
- [ ] Reuse leaked IDs elsewhere
- [ ] Escalate impact via chaining

---

## 10. Multi-Step Workflow Abuse
Validate checks at every step.

- [ ] Skip workflow steps
- [ ] Reorder steps
- [ ] Reuse incomplete object IDs
- [ ] Inject another user’s ID mid-flow

---

## 11. API vs Web Inconsistencies
Backend trust issues.

- [ ] UI blocks but API allows access
- [ ] Mobile API less restrictive
- [ ] Older API versions (`/v1`, `/v2`)
- [ ] Hidden or undocumented endpoints

---

## 12. Authorization Bypass Signals
Red flags during testing.

- [ ] `200 OK` with unauthorized data
- [ ] Partial data leakage
- [ ] Response length differences
- [ ] Error message changes
- [ ] Timing differences
- [ ] Blind success (action works silently)

---

## 13. Impact Verification
Make the issue undeniable.

- [ ] Two separate user accounts
- [ ] Cross-account access proof
- [ ] Cross-account modification proof
- [ ] Before / after evidence
- [ ] Sensitive data exposure

---

## 14. Reporting Checklist
Avoid rejection during triage.

- [ ] Clear vulnerable endpoint
- [ ] Full request & response
- [ ] Step-by-step reproduction
- [ ] Two-account demonstration
- [ ] Business impact explanation
- [ ] Clear authorization failure explanation

---

## 15. Mental Model
Remember this while hunting.

- [ ] Server trusts user-supplied IDs
- [ ] Ownership is not revalidated per request
- [ ] Authorization ≠ authentication

If these exist, IDOR likely exists.

---

# Quick IDOR Checklist (Easy Pass)

- [ ] **Identify object IDs**
  - Look for parameters like `id`, `user_id`, `order_id`, `invoice`, `file`, etc. in URLs, bodies, and JSON.  

- [ ] **Change the ID value**
  - Try simple tampering: `123 → 124`, `123 → 1`, `123 → 999999`.  
  - Observe: status code, response size, and content changes.  

- [ ] **Test as another user**
  - Create **User A** and **User B**.  
  - Capture a request as User A, then replay it with User B’s session and modified ID.  

- [ ] **Check for unauthorized data**
  - Do you see another user’s profile, orders, tickets, or files without errors?  
  - If yes, capture clear before/after evidence (both users) for reporting.  

- [ ] **Try non‑sequential / opaque IDs**
  - If IDs look random (UUID, hashes), paste one ID from User A’s data into a request as User B.  

- [ ] **Confirm impact**
  - Can you **read**, **update**, or **delete** someone else’s data?  
  - Map it to vertical (role) or horizontal (same role) access control failure.

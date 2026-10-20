# 🧪 QA Test Plan & Test Cases Matrix

> **Project Name:** [App Name]  
> **Lead QA Engineer:** [QA Lead Name]  
> **Test Cycle:** Sprint 12 / Release v1.0.0 (Candidate 1)  
> **Status:** Active / Execution in Progress  

---

## 1. Testing Scope & Strategy

Yeh document ensure karta hai ki application production me bina kisi critical bug ke deploy ho sake:

- **In-Scope Testing:**
  - Functional Testing (Happy path, Negative flows, Edge cases)
  - UI/UX & Responsive Testing across various screen resolutions
  - API & Contract Testing (Postman / Newman automation)
  - Network Interruption & Offline Handling (Airplane mode, slow 2G)
  - Performance & Load Testing (Simulating 3,000 concurrent active users)
  - Security Testing (OWASP Mobile Top 10 checklist)

---

## 2. Test Environment & Device Matrix

| Device Brand / Model | OS Version | Screen Resolution | Purpose |
|----------------------|------------|-------------------|---------|
| iPhone 15 Pro | iOS 17.4 | 2556 x 1179 (Retina) | High-end iOS benchmark |
| iPhone 11 | iOS 16.0 | 1792 x 828 | Legacy iOS performance & notch handling |
| Samsung Galaxy S23 | Android 14 (OneUI 6) | 2340 x 1080 | Flagship Android standard |
| Xiaomi Redmi Note 12 | Android 12 (MIUI 14) | 2400 x 1080 | Budget Android benchmark (Memory constraint testing) |

---

## 3. Detailed Test Cases Matrix

| Test ID | Module | Test Scenario Description | Pre-Conditions | Test Steps | Expected Result | Severity |
|---------|--------|---------------------------|----------------|------------|-----------------|----------|
| **TC-AUTH-01** | Login | Valid OTP phone verification | App installed, active SIM | 1. Enter 10-digit number<br>2. Tap 'Get OTP'<br>3. Enter received 4-digit code | User logged in, navigated to Home screen within 2s | `Blocker` |
| **TC-AUTH-02** | Login | Invalid OTP error handling | OTP received on phone | 1. Enter wrong 4-digit OTP (e.g. 0000)<br>2. Tap 'Verify' | Inline red error: *"Invalid code"*, retry attempts counter decreases | `Critical` |
| **TC-AUTH-03** | Login | Resend OTP cooldown timer | OTP screen open | 1. Observe 'Resend OTP' button | Button remains disabled for 60 seconds with countdown | `Medium` |
| **TC-CART-01** | Cart | Item quantity increment/decrement | Restaurant menu open | 1. Tap '+ Add'<br>2. Tap '+' button 3 times<br>3. Tap '-' button 1 time | Cart item quantity updates from 1 -> 4 -> 3. Bill calculates accurately | `Major` |
| **TC-CART-02** | Cart | Items from multiple restaurants conflict | Cart already has items from Restaurant A | 1. Navigate to Restaurant B<br>2. Tap '+ Add' on dish | Bottom sheet alert: *"Clear current cart to add items from another restaurant?"* | `Critical` |
| **TC-PAY-01** | Checkout | Payment failure recovery | Items in cart | 1. Proceed to pay<br>2. Cancel UPI payment on GPay prompt | App returns to checkout screen gracefully with *"Payment cancelled. Try again."* Cart remains intact | `Critical` |
| **TC-NET-01** | Network | Network lost during live map tracking | Active order in progress | 1. Turn on Airplane mode while viewing driver map | Yellow banner appears: *"Offline. Reconnecting..."* Driver marker does not jump or crash | `Major` |

---

## 4. Defect Severity Definitions & SLAs

- **Blocker (P0):** App crashes on launch, payment deduction without order creation, login completely broken. *(Fix SLA: < 4 hours)*
- **Critical (P1):** Major core feature broken with no workaround (e.g., GPS map not rendering). *(Fix SLA: < 24 hours)*
- **Major (P2):** Secondary feature impaired (e.g., promo coupon auto-apply fails, but manual entry works). *(Fix SLA: Next sprint release)*
- **Minor / Trivial (P3):** UI alignment, spelling typo, minor color mismatch. *(Fix SLA: Backlog)*

---

## 5. QA Signoff Gate Criteria

Release production me tabhi jayegi jab:
1. 0 Blocker bugs open hon.
2. 0 Critical bugs open hon.
3. Test case execution pass rate **>= 98%** ho.
4. Stress test me p95 latency **< 200ms** at 3,000 concurrent virtual users ho.

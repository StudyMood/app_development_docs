# 💼 Business Requirements Document (BRD)

> **Project Name:** [e.g., QuickBite On-Demand Platform]  
> **Document Owner:** Business Analyst / Lead Product Manager  
> **Stakeholders:** Executive Leadership, Head of Engineering, Finance, Operations  
> **Version:** 1.0  
> **Approval Date:** YYYY-MM-DD  

---

## 1. Project Background & Business Goals
*Yeh project kyun shuru kiya ja raha hai aur company ke overall goals me kaise fit hota hai?*

- **Business Background:** [Company chahti hai ki hyperlocal delivery market me corporate niche capture kare.]
- **Primary Business Objectives:**
  - Launch an MVP in 2 tech parks within 4 months.
  - Achieve 15,000 monthly active orders within 6 months of launch.
  - Reach gross margin positivity (>18%) on deliveries within 9 months.

---

## 2. Project Scope (In-Scope vs Out-of-Scope)
*Scope creep se bachne ke liye clearly define karein ki kya banega aur kya nahi banega.*

### In-Scope (Phase 1 MVP)
- Customer Mobile App (iOS & Android) for ordering food.
- Driver Partner App for accepting and fulfilling deliveries.
- Restaurant Partner Tablet App / Web Portal for order management.
- Super Admin Dashboard for operations, menu approval, and refund management.
- Payment Gateway integration (UPI, Credit/Debit cards, Wallets).
- Push notification system for real-time order alerts.

### Out-of-Scope (Deferred to Phase 2/3)
- Multi-currency / International payment support.
- Scheduled orders for next day delivery.
- Voice-based search and AI conversational ordering.
- Third-party courier aggregation (Dunzo/Shadowfax fallback).

---

## 3. Business Success Metrics & KPIs
*Kaise pata chalega ki project successful raha?*

| Metric Category | KPI | Target (Month 6) |
|-----------------|-----|------------------|
| **Acquisition** | New User Signups | 25,000 users |
| **Engagement** | Monthly Active Users (MAU) | 12,000 MAU |
| **Operational** | Average Order Delivery Time | < 18 minutes |
| **Financial** | Average Order Value (AOV) | ₹280 INR |
| **Satisfaction** | Customer CSAT / Net Promoter Score (NPS) | > 65 NPS |

---

## 4. Key Business Rules & Policies
*System ko kin business rules ke mutabiq behave karna padega?*

1. **Order Cancellation Window:** User order place hone ke baad sirf 60 seconds tak cancel kar sakta hai (restaurant kitchen cooking start karne se pehle).
2. **Refund Policy:** Agar order delivery guaranteed SLA (e.g. 25 min) se 15 minute zyada late ho jaye, to automatically 50% cashback wallet me credit hoga.
3. **Driver Allocation Rule:** Order sirf us delivery partner ko assign hoga jo restaurant ke 1.5 km radius me active aur unassigned ho.
4. **Commission Deduction:** Har Sunday midnight par automatically restaurant ka settlement calculate hoga minus commission.

---

## 5. Stakeholder Matrix & RACI
*Kaun Responsible, Accountable, Consulted aur Informed hai?*

| Role / Function | Name / Department | Responsibility in Project |
|-----------------|-------------------|----------------------------|
| **Sponsor / Exec** | VP of Business | Budget approval, high-level signoff |
| **Product Owner** | Senior PM | Feature roadmap, PRD, user acceptance |
| **Tech Lead** | Principal Architect | Architecture, engineering delivery, infra |
| **Operations Lead** | Head of Ops | Driver fleet management, kitchen onboarding |
| **Compliance/Legal**| Legal Counsel | Data privacy, FSSAI compliance, terms |

---

## 6. Budget, Timeline & Financial Projections
*Project ka total estimated cost aur milestone schedule:*

- **Approved Initial Budget:** ₹45,00,000 INR ($55,000 USD)
- **Timeline Milestones:**
  - *Month 1:* Requirements, Design & Architecture signoff
  - *Month 2-3:* Core backend, customer app, partner dashboard build
  - *Month 3.5:* Integration testing, security audit & beta testing
  - *Month 4:* Production launch in Tech Park Alpha

---

## 7. Assumptions, Dependencies & Risk Assessment

| Risk Description | Severity | Probability | Mitigation Strategy |
|------------------|----------|-------------|---------------------|
| Payment Gateway downtime during peak lunch hours | High | Medium | Dual PG failover (Razorpay primary, Cashfree backup) |
| Delivery partner shortage during heavy rain | High | High | Dynamic rain incentive bonus for delivery fleet |
| Google Maps API billing spike | Medium | Medium | In-memory Redis caching for geocoding & route caching |

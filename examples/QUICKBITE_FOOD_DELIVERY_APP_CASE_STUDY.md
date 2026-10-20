# 🍔 Complete Real-World Case Study: "QuickBite" Food Delivery App

> **Overview:** Yeh ek complete end-to-end practical example hai jo dikhata hai ki ek real-world production mobile app (jaise Swiggy, Zomato, UberEats) ke liye documents kaise likhe jate hain.  
> Is case study me **MRD ➔ BRD ➔ PRD ➔ HLD ➔ LLD ➔ API Spec ➔ QA Test Cases ➔ DevOps** sabhi ek sath interconnected dikhaye gaye hain.

---

## 1. QuickBite MRD (Market Research Snapshot)

- **Problem:** Bangalore ke Electronic City & Whitefield tech parks me 1.5 lakh IT employees hain. Lunch break 1:00 PM se 2:00 PM hota hai. Existing delivery apps 45-60 min leti hain, jisse log order nahi kar pate. Cafeteria me 20-25 min ki lambi line hoti hai.
- **Solution:** 15-Minute Guaranteed Delivery from curated cloud-kitchen pods placed directly inside and adjacent to major tech campuses.
- **Customer Persona:**
  - *Name:* Aditi Nair (Age: 26, UI Designer at MNC)
  - *Habit:* Lunch 1:15 PM par chahiye; time bacha kar thoda relax karna chahti hai.
  - *Willingness to pay:* ₹200-₹350 per lunch meal.
- **USP:** "Order at 1:00 PM, Eat at 1:15 PM at your corporate reception or desk."

---

## 2. QuickBite BRD (Business Requirements Snapshot)

- **Revenue Streams:**
  - 15% platform commission on orders from restaurant partners.
  - QuickBite Pass Subscription: ₹149/month (Free delivery + VIP lunch priority).
- **Core Business Rules:**
  1. Guaranteed Delivery SLA: 20 minutes max. Agar late hua to ₹50 cashback in QuickBite Wallet.
  2. Kitchen preparation SLA: Maximum 7 minutes (Hot & Ready dishes only).
  3. Minimum Order Value: ₹99.

---

## 3. QuickBite PRD (Product Requirements & User Stories)

### User Story: 1-Tap Lunch Re-Order & Live Tracking

> **As an** office employee who has a favorite regular lunch,  
> **I want to** re-order my daily meal in 1-tap from the home screen,  
> **So that** I don't waste 5 minutes searching through menus every single day.

#### Acceptance Criteria
1. Home screen par *"Your Usual Lunch"* card dynamically display hoga agar user ne past 30 days me 3+ baar same dish order ki ho.
2. Tapping "Quick Re-order" automatically cart populate karega aur direct saved default corporate address select karega.
3. User swipe-to-pay karega (UPI AutoPay / Saved Card).
4. Order confirm hone ke baad live map render hoga jisme:
   - Cloud Kitchen marker (Red)
   - Delivery Bike marker (Orange) with live heading angle
   - Office Building gate drop marker (Green)
5. ETA dynamically recalculate hoga har 10 seconds me based on rider GPS coordinates.

---

## 4. QuickBite HLD (System Architecture)

```mermaid
graph TB
    subgraph Mobile["Mobile Apps (Flutter)"]
        UserApp[Consumer App - iOS & Android]
        DriverApp[Driver Partner App]
        KitchenApp[Kitchen Order Display System - Android Tablet]
    end

    subgraph CDN_Gateway["Traffic Routing & Gateway"]
        CF[Cloudflare CDN + WAF]
        Kong[Kong API Gateway]
    end

    subgraph Microservices["Backend Services (AWS EKS)"]
        AuthSvc[Auth & Profile Service - Node.js]
        CatalogSvc[Restaurant Catalog Service - Go]
        OrderSvc[Order Processing Service - NestJS]
        PaymentSvc[Payment Gateway Service - Python]
        GeoSvc[Geospatial Dispatch Engine - Go]
        NotifSvc[Push Notification Service - Node.js]
    end

    subgraph DataStores["Databases & Queues"]
        PG[(PostgreSQL Primary RDS)]
        RedisCache[(Redis Cluster - Menus & GeoIndex)]
        KafkaCluster[Apache Kafka Event Bus]
        S3Storage[(AWS S3 - Static Assets)]
    end

    UserApp & DriverApp & KitchenApp --> CF
    CF --> Kong
    Kong --> AuthSvc
    Kong --> CatalogSvc
    Kong --> OrderSvc
    Kong --> PaymentSvc

    OrderSvc --> PG
    OrderSvc --> KafkaCluster
    KafkaCluster --> GeoSvc
    KafkaCluster --> NotifSvc

    GeoSvc <--> RedisCache
    CatalogSvc <--> RedisCache
```

---

## 5. QuickBite LLD (Database Schema DDL)

```sql
-- Users Table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name VARCHAR(100) NOT NULL,
    phone_number VARCHAR(15) UNIQUE NOT NULL,
    corporate_email VARCHAR(120),
    wallet_balance NUMERIC(8, 2) DEFAULT 0.00,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Active Orders Table
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    pod_id UUID NOT NULL, -- Hyperlocal kitchen pod
    status VARCHAR(30) NOT NULL, -- CREATED, PREPARING, DISPATCHED, DELIVERED
    subtotal NUMERIC(8,2) NOT NULL,
    delivery_fee NUMERIC(6,2) DEFAULT 0.00,
    total_amount NUMERIC(8,2) NOT NULL,
    delivery_lat NUMERIC(9,6) NOT NULL,
    delivery_lng NUMERIC(9,6) NOT NULL,
    desk_note VARCHAR(150), -- e.g. "Tower B, 4th Floor, Desk #402"
    placed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Driver Telemetry Table (TimescaleDB / Hypertable for high frequency GPS pings)
CREATE TABLE driver_locations (
    driver_id UUID NOT NULL,
    recorded_at TIMESTAMP WITH TIME ZONE NOT NULL,
    coordinates POINT NOT NULL,
    speed_kmh NUMERIC(4, 1),
    heading_deg NUMERIC(5, 2)
);
```

---

## 6. QuickBite API Specification (Key Endpoints)

### Place Instant Re-Order
```http
POST /v1/orders/quick-reorder
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json

{
  "previous_order_id": "ord_8829-1a-2026",
  "delivery_address_id": "addr_corporate_desk_402",
  "payment_mode": "UPI_INTENT"
}
```

#### Successful Response (201 Created)
```json
{
  "success": true,
  "data": {
    "order_id": "ord_99014",
    "status": "PREPARING",
    "estimated_delivery_time": "12 mins",
    "delivery_slot": "1:14 PM",
    "kitchen_pod": "Electronic City Phase 1 Pod #3",
    "assigned_driver": {
      "name": "Ramesh Kumar",
      "phone_masked": "+91-98765-XXXXX",
      "vehicle_number": "KA-01-EQ-9921",
      "rating": 4.9
    }
  }
}
```

---

## 7. QuickBite QA Test Cases (Critical Flows)

| Test ID | Module | Test Scenario | Steps | Expected Result | Pass/Fail |
|---------|--------|---------------|-------|-----------------|-----------|
| **QB-TC-01** | Reorder | 1-Tap Quick Reorder from Home | 1. Tap 'Re-order' on Lunch card.<br>2. Biometric authenticate. | Order created in under 3 seconds; Live map opens. | `PASS` |
| **QB-TC-02** | Geofence | Delivery outside service boundary | 1. Set location 10km away from nearest pod. | Screen shows: *"Service arriving soon in your area! Join waitlist."* Cart checkout disabled. | `PASS` |
| **QB-TC-03** | SLA Refund | Order delivers in 23 min (>20 min SLA) | 1. Driver marks order delivered at 23rd min. | System automatically dispatches ₹50 wallet cashback SMS & in-app notification within 60s. | `PASS` |

---

## 8. QuickBite DevOps & Deployment Pipeline

- **Hosting:** AWS Mumbai (`ap-south-1`)
- **Containers:** Docker + Amazon EKS (Kubernetes 1.29)
- **CI/CD:** GitHub Actions with automated linting, SonarQube code analysis, and Canary deployment.
- **Monitoring:** Datadog for APM metrics + Sentry for Flutter mobile crash tracking.

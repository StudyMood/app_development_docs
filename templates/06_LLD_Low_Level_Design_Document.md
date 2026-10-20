# ⚙️ Low-Level Design (LLD) Document

> **Document Owner:** Lead Backend Engineer / Senior Mobile Engineer  
> **Target Audience:** Software Developers, Code Reviewers, QA Automation  
> **Status:** Approved  
> **Version:** 1.0  

---

## 1. Database Schema & Entity-Relationship (ER) Diagram

PostgreSQL database tables, primary/foreign keys aur relationships ka design:

```mermaid
erDiagram
    USERS ||--o{ ORDERS : places
    USERS ||--o{ ADDRESSES : owns
    RESTAURANTS ||--o{ MENU_ITEMS : contains
    RESTAURANTS ||--o{ ORDERS : receives
    ORDERS ||--|{ ORDER_ITEMS : contains
    ORDERS ||--|| PAYMENTS : has
    ORDERS ||--o| DELIVERIES : assigned_to
    DRIVERS ||--o{ DELIVERIES : fulfills

    USERS {
        uuid id PK
        string full_name
        string phone_number UK
        string email UK
        enum status
        timestamp created_at
    }

    ADDRESSES {
        uuid id PK
        uuid user_id FK
        string tag
        string building_name
        decimal latitude
        decimal longitude
        boolean is_default
    }

    RESTAURANTS {
        uuid id PK
        string name
        decimal rating
        decimal latitude
        decimal longitude
        boolean is_active
    }

    MENU_ITEMS {
        uuid id PK
        uuid restaurant_id FK
        string item_name
        decimal price
        boolean is_veg
        boolean in_stock
    }

    ORDERS {
        uuid id PK
        uuid user_id FK
        uuid restaurant_id FK
        decimal total_amount
        enum order_status
        timestamp created_at
    }

    ORDER_ITEMS {
        uuid id PK
        uuid order_id FK
        uuid menu_item_id FK
        integer quantity
        decimal unit_price
    }

    PAYMENTS {
        uuid id PK
        uuid order_id FK
        string transaction_ref UK
        enum payment_method
        enum payment_status
        decimal amount
    }

    DELIVERIES {
        uuid id PK
        uuid order_id FK
        uuid driver_id FK
        enum delivery_status
        decimal pickup_lat
        decimal drop_lat
        timestamp delivered_at
    }
```

### 1.1 Key Database Table DDL (PostgreSQL Sample)

```sql
-- Orders Table Definition
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    restaurant_id UUID NOT NULL REFERENCES restaurants(id) ON DELETE RESTRICT,
    total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount >= 0),
    delivery_fee NUMERIC(6, 2) DEFAULT 0.00,
    order_status VARCHAR(30) NOT NULL DEFAULT 'CREATED',
    delivery_address JSONB NOT NULL,
    cancellation_reason VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indexing Strategy for Fast Lookups
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(order_status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

---

## 2. Order Lifecycle Finite State Machine (FSM)

Order kisi bhi time kis state me ho sakta hai aur kaunse valid transitions allowed hain:

```mermaid
stateDiagram-v2
    [*] --> CREATED: User taps Checkout
    CREATED --> PAYMENT_PENDING: Payment Intent generated
    PAYMENT_PENDING --> PAYMENT_FAILED: Transaction rejected
    PAYMENT_FAILED --> PAYMENT_PENDING: Retry payment
    PAYMENT_FAILED --> CANCELLED: Timeout / User aborts

    PAYMENT_PENDING --> ORDER_CONFIRMED: Payment Webhook Received
    ORDER_CONFIRMED --> PREPARING: Kitchen accepts order
    ORDER_CONFIRMED --> CANCELLED: Kitchen rejects / Out of stock

    PREPARING --> READY_FOR_PICKUP: Food packaged
    READY_FOR_PICKUP --> OUT_FOR_DELIVERY: Rider picks up food
    OUT_FOR_DELIVERY --> DELIVERED: OTP entered at customer desk
    OUT_FOR_DELIVERY --> FAILED_DELIVERY: Customer unreachable

    DELIVERED --> [*]
    CANCELLED --> [*]
    FAILED_DELIVERY --> [*]
```

---

## 3. Design Patterns Applied

### 3.1 Strategy Pattern for Payment Gateways
Alag-alag payment providers (Razorpay, Stripe, Cashfree) ko dynamically switch karne ke liye **Strategy Pattern** use kiya jata hai:

```typescript
// Payment Strategy Interface
export interface IPaymentStrategy {
  createPaymentIntent(amount: number, orderId: string): Promise<PaymentResponse>;
  verifyPaymentSignature(payload: WebhookPayload): Promise<boolean>;
  refund(transactionId: string, amount: number): Promise<RefundResponse>;
}

// Razorpay Implementation
export class RazorpayStrategy implements IPaymentStrategy {
  async createPaymentIntent(amount: number, orderId: string): Promise<PaymentResponse> {
    // Razorpay SDK Call
    return { provider: 'RAZORPAY', orderId, status: 'INITIATED' };
  }
  // ... verify and refund
}

// Strategy Context Resolver
export class PaymentContext {
  private strategy: IPaymentStrategy;

  constructor(strategy: IPaymentStrategy) {
    this.strategy = strategy;
  }

  public setStrategy(strategy: IPaymentStrategy) {
    this.strategy = strategy;
  }

  public async processPayment(amount: number, orderId: string) {
    return this.strategy.createPaymentIntent(amount, orderId);
  }
}
```

---

## 4. Core Algorithms & Logic: Rider Assignment

Nearest delivery partner assign karne ka logic (Geospatial Radius Matching):

```python
# Nearest Available Driver Search Algorithm
def find_optimal_driver(restaurant_lat, restaurant_lng, max_radius_km=2.0):
    """
    1. Query Redis GeoSet for drivers with status 'AVAILABLE' within radius.
    2. Rank drivers based on:
       Score = (0.6 * Distance_km) + (0.4 * Wait_time_minutes)
    3. Send notification ping to the top-ranked driver.
    4. Wait 15 seconds for acceptance. If timeout, cascade to next driver.
    """
    nearby_drivers = redis_client.geosearch(
        name="drivers:online",
        longitude=restaurant_lng,
        latitude=restaurant_lat,
        radius=max_radius_km,
        unit="km",
        withdist=True,
        count=5
    )

    if not nearby_drivers:
        # Expand search radius or notify dispatch team
        return None

    return nearby_drivers[0]  # Best match driver
```

---

## 5. Standard Error Handling & Domain Error Codes

Frontend aur backend ke uniform error mapping ke liye standard format:

```json
{
  "success": false,
  "error": {
    "code": "ERR_PAYMENT_FAILED",
    "message": "Your bank card was declined due to insufficient balance.",
    "field": "payment_method",
    "timestamp": "2026-10-20T10:15:30Z",
    "tracking_id": "req-9884a-1120-f56"
  }
}
```

# 🌐 RESTful API Specification (OpenAPI / Swagger Contract)

> **API Base Version:** v1  
> **Environment URLs:**  
> - **Development:** `https://dev-api.quickbite.internal/v1`  
> - **Staging:** `https://staging-api.quickbite.com/v1`  
> - **Production:** `https://api.quickbite.com/v1`  
> **Security Protocol:** HTTPS TLS 1.3  
> **Default Content-Type:** `application/json`  

---

## 1. Authentication & Common Headers

Har authenticated request me client ko Bearer JWT token bhejna zaroori hai:

```http
Authorization: Bearer <YOUR_JWT_ACCESS_TOKEN>
Content-Type: application/json
X-Client-Platform: android | ios | web
X-App-Version: 1.2.0
X-Request-ID: e8b940e7-3b24-4f0e-b8d2-19266158e234
```

### Rate Limiting Headers (RFC 6585)
- `X-RateLimit-Limit`: `100` (requests per minute)
- `X-RateLimit-Remaining`: `94`
- `X-RateLimit-Reset`: `1729418400` (Unix timestamp)

---

## 2. Standard API Response Structure

Sabhi endpoints uniform JSON format return karenge:

### Successful Response Format
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total_count": 142
  }
}
```

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The restaurant with ID 48f9-12a does not exist or is currently inactive.",
    "details": []
  }
}
```

---

## 3. Core API Endpoints

### 3.1 Authentication: Request OTP
- **Endpoint:** `POST /auth/request-otp`
- **Description:** User ke mobile phone number par SMS OTP send karta hai.

#### Request Body
```json
{
  "phone_number": "+919876543210"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "request_id": "otp_req_998124",
    "resend_available_in_seconds": 60,
    "message": "OTP sent successfully to registered mobile number."
  }
}
```

---

### 3.2 Restaurants: List Hyperlocal Restaurants
- **Endpoint:** `GET /restaurants/nearby`
- **Description:** Customer ke current latitude & longitude ke aaspas ke restaurants return karta hai.
- **Query Parameters:**
  - `latitude` (float, required): e.g., `12.9716`
  - `longitude` (float, required): e.g., `77.5946`
  - `radius_km` (float, optional, default: `3.0`)
  - `is_veg` (boolean, optional): e.g., `true`

#### Response (200 OK)
```json
{
  "success": true,
  "data": [
    {
      "id": "e4b2d184-7a1a-493a-86a0-bf88289bfbb1",
      "name": "Bikanervala Express",
      "rating": 4.6,
      "total_reviews": 1280,
      "estimated_delivery_time_mins": 14,
      "distance_km": 0.8,
      "hero_image_url": "https://cdn.quickbite.com/restaurants/bikaner.webp",
      "cuisines": ["North Indian", "Thali", "Snacks"],
      "is_pure_veg": true
    }
  ]
}
```

---

### 3.3 Orders: Create & Place Order
- **Endpoint:** `POST /orders`
- **Headers:** `Authorization: Bearer <TOKEN>`
- **Description:** New food order initiate karta hai aur payment gateway intent generate karta hai.

#### Request Body
```json
{
  "restaurant_id": "e4b2d184-7a1a-493a-86a0-bf88289bfbb1",
  "items": [
    {
      "menu_item_id": "item-9918-paneer-thali",
      "quantity": 2,
      "customizations": ["Extra Roti", "Less Spicy"]
    }
  ],
  "delivery_address_id": "addr-8812-desk-4th-floor",
  "payment_method": "UPI",
  "promo_code": "QUICKFIRST"
}
```

#### Response (201 Created)
```json
{
  "success": true,
  "data": {
    "order_id": "ord_8829410_2026",
    "order_status": "PAYMENT_PENDING",
    "total_bill": {
      "item_total": 450.00,
      "delivery_fee": 0.00,
      "taxes": 22.50,
      "discount": 50.00,
      "grand_total": 422.50
    },
    "payment_gateway": {
      "provider": "RAZORPAY",
      "razorpay_order_id": "order_Rzp_88194",
      "amount_in_paise": 42250,
      "currency": "INR"
    }
  }
}
```

---

### 3.4 Webhooks: Payment Gateway Callback
- **Endpoint:** `POST /webhooks/payments/razorpay`
- **Security:** HMAC-SHA256 signature verification in `X-Razorpay-Signature` header.
- **Description:** Payment successful ya failed hone par real-time webhook callback.

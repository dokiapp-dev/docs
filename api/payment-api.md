# Payment API

The Payment API allows you to process payments, manage transactions, and handle payment methods for your applications.

## Base URL

```
https://api.dokiapp.com/v1/payments
```

## Authentication

All Payment API endpoints require authentication using an API key in the request header:

```
Authorization: Bearer YOUR_API_KEY
```

---

## Endpoints

### 1. Create Payment

Create a new payment transaction.

**Endpoint:** `POST /payments`

**Request Payload:**

```json
{
  "amount": 1000,
  "currency": "USD",
  "payment_method_id": "pm_1234567890",
  "description": "Payment for order #12345",
  "metadata": {
    "order_id": "12345",
    "customer_id": "cust_abc123"
  }
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | integer | Yes | Payment amount in cents (e.g., 1000 = $10.00) |
| `currency` | string | Yes | Three-letter ISO currency code (e.g., USD, EUR, GBP) |
| `payment_method_id` | string | Yes | ID of the payment method to charge |
| `description` | string | No | Description of the payment |
| `metadata` | object | No | Set of key-value pairs for storing additional information |

**Success Response (201 Created):**

```json
{
  "id": "pay_1234567890abcdef",
  "amount": 1000,
  "currency": "USD",
  "status": "succeeded",
  "payment_method_id": "pm_1234567890",
  "description": "Payment for order #12345",
  "metadata": {
    "order_id": "12345",
    "customer_id": "cust_abc123"
  },
  "created_at": "2025-11-06T19:54:00Z"
}
```

**HTTP Error Codes:**

| Code | Description |
|------|-------------|
| `400` | Bad Request - Invalid parameters or missing required fields |
| `401` | Unauthorized - Invalid or missing API key |
| `402` | Payment Required - Insufficient funds or payment method declined |
| `404` | Not Found - Payment method not found |
| `409` | Conflict - Duplicate payment attempt |
| `422` | Unprocessable Entity - Invalid payment method or currency not supported |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error - Something went wrong on our end |

---

### 2. Retrieve Payment

Retrieve details of a specific payment.

**Endpoint:** `GET /payments/{payment_id}`

**URL Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `payment_id` | string | Yes | The ID of the payment to retrieve |

**Success Response (200 OK):**

```json
{
  "id": "pay_1234567890abcdef",
  "amount": 1000,
  "currency": "USD",
  "status": "succeeded",
  "payment_method_id": "pm_1234567890",
  "description": "Payment for order #12345",
  "metadata": {
    "order_id": "12345",
    "customer_id": "cust_abc123"
  },
  "created_at": "2025-11-06T19:54:00Z",
  "updated_at": "2025-11-06T19:54:05Z"
}
```

**HTTP Error Codes:**

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing API key |
| `404` | Not Found - Payment not found |
| `500` | Internal Server Error - Something went wrong on our end |

---

### 3. List Payments

Retrieve a list of payments.

**Endpoint:** `GET /payments`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | integer | No | Number of payments to return (default: 10, max: 100) |
| `offset` | integer | No | Number of payments to skip (default: 0) |
| `status` | string | No | Filter by status: `pending`, `succeeded`, `failed`, `canceled` |
| `created_after` | string | No | Filter payments created after this timestamp (ISO 8601 format) |
| `created_before` | string | No | Filter payments created before this timestamp (ISO 8601 format) |

**Success Response (200 OK):**

```json
{
  "data": [
    {
      "id": "pay_1234567890abcdef",
      "amount": 1000,
      "currency": "USD",
      "status": "succeeded",
      "payment_method_id": "pm_1234567890",
      "description": "Payment for order #12345",
      "created_at": "2025-11-06T19:54:00Z"
    },
    {
      "id": "pay_0987654321fedcba",
      "amount": 2500,
      "currency": "USD",
      "status": "succeeded",
      "payment_method_id": "pm_0987654321",
      "description": "Payment for order #12346",
      "created_at": "2025-11-06T18:30:00Z"
    }
  ],
  "has_more": true,
  "total_count": 42
}
```

**HTTP Error Codes:**

| Code | Description |
|------|-------------|
| `400` | Bad Request - Invalid query parameters |
| `401` | Unauthorized - Invalid or missing API key |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error - Something went wrong on our end |

---

### 4. Cancel Payment

Cancel a pending payment.

**Endpoint:** `POST /payments/{payment_id}/cancel`

**URL Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `payment_id` | string | Yes | The ID of the payment to cancel |

**Request Payload:**

```json
{
  "reason": "Customer requested cancellation"
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | No | Reason for canceling the payment |

**Success Response (200 OK):**

```json
{
  "id": "pay_1234567890abcdef",
  "amount": 1000,
  "currency": "USD",
  "status": "canceled",
  "payment_method_id": "pm_1234567890",
  "description": "Payment for order #12345",
  "canceled_at": "2025-11-06T20:00:00Z",
  "cancellation_reason": "Customer requested cancellation"
}
```

**HTTP Error Codes:**

| Code | Description |
|------|-------------|
| `400` | Bad Request - Payment cannot be canceled (already succeeded or failed) |
| `401` | Unauthorized - Invalid or missing API key |
| `404` | Not Found - Payment not found |
| `409` | Conflict - Payment is already canceled |
| `500` | Internal Server Error - Something went wrong on our end |

---

### 5. Refund Payment

Refund a completed payment.

**Endpoint:** `POST /payments/{payment_id}/refund`

**URL Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `payment_id` | string | Yes | The ID of the payment to refund |

**Request Payload:**

```json
{
  "amount": 500,
  "reason": "Product returned"
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | integer | No | Amount to refund in cents. If not provided, refunds the full amount |
| `reason` | string | No | Reason for the refund |

**Success Response (200 OK):**

```json
{
  "id": "ref_1234567890abcdef",
  "payment_id": "pay_1234567890abcdef",
  "amount": 500,
  "currency": "USD",
  "status": "succeeded",
  "reason": "Product returned",
  "created_at": "2025-11-06T20:15:00Z"
}
```

**HTTP Error Codes:**

| Code | Description |
|------|-------------|
| `400` | Bad Request - Invalid refund amount or payment cannot be refunded |
| `401` | Unauthorized - Invalid or missing API key |
| `404` | Not Found - Payment not found |
| `409` | Conflict - Payment has already been fully refunded |
| `422` | Unprocessable Entity - Refund amount exceeds available balance |
| `500` | Internal Server Error - Something went wrong on our end |

---

## Payment Statuses

| Status | Description |
|--------|-------------|
| `pending` | Payment is being processed |
| `succeeded` | Payment completed successfully |
| `failed` | Payment failed |
| `canceled` | Payment was canceled before completion |
| `refunded` | Payment was refunded (partially or fully) |

---

## Common Error Response Format

All error responses follow this format:

```json
{
  "error": {
    "code": "payment_method_invalid",
    "message": "The payment method provided is invalid or has expired",
    "type": "invalid_request_error",
    "param": "payment_method_id"
  }
}
```

**Error Object Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Machine-readable error code |
| `message` | string | Human-readable error message |
| `type` | string | Error category (e.g., `invalid_request_error`, `authentication_error`, `api_error`) |
| `param` | string | Parameter that caused the error (if applicable) |

---

## Rate Limiting

The Payment API has the following rate limits:

- **Standard**: 100 requests per minute per API key
- **Burst**: 20 requests per second

When rate limits are exceeded, you'll receive a `429 Too Many Requests` response with a `Retry-After` header indicating when you can retry.

---

## Testing

For testing purposes, use the following test API key:

```
sk_test_1234567890abcdefghijklmnop
```

Test mode allows you to simulate payments without processing real transactions.

### Test Payment Methods

| Payment Method ID | Result |
|-------------------|--------|
| `pm_test_success` | Payment succeeds |
| `pm_test_decline` | Payment is declined |
| `pm_test_insufficient_funds` | Payment fails due to insufficient funds |
| `pm_test_expired` | Payment method is expired |

---

## Examples

### Example 1: Create a Payment

```bash
curl -X POST https://api.dokiapp.com/v1/payments \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1000,
    "currency": "USD",
    "payment_method_id": "pm_1234567890",
    "description": "Payment for order #12345"
  }'
```

### Example 2: Retrieve a Payment

```bash
curl -X GET https://api.dokiapp.com/v1/payments/pay_1234567890abcdef \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Example 3: List Payments with Filters

```bash
curl -X GET "https://api.dokiapp.com/v1/payments?status=succeeded&limit=20" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Example 4: Refund a Payment

```bash
curl -X POST https://api.dokiapp.com/v1/payments/pay_1234567890abcdef/refund \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 500,
    "reason": "Product returned"
  }'
```

---

## Webhooks

To receive real-time notifications about payment events, configure webhooks in your dashboard. The following events are available:

- `payment.created` - A payment was created
- `payment.succeeded` - A payment completed successfully
- `payment.failed` - A payment failed
- `payment.canceled` - A payment was canceled
- `payment.refunded` - A payment was refunded

Webhook configuration and detailed event payloads can be managed through your dashboard.

---

## Support

For questions or issues with the Payment API, please contact our developer support team at dev-support@dokiapp.com.

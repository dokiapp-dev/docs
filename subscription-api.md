---
title: Subscription API
layout: page
---

# Subscription API (DRAFT)

### Endpoint URLs:

| Environment | Root Endpoint URL               |
| ----------- | -----------------------------   |
| Staging     | `https://cortex-dev.dokiapp.hu` |
| Production  | `https://cortex.dokiapp.hu`     |

### Sample `JSON` payload:

HTTP Method: `POST`

Path: `/v1/subscriptions`

```json
{
  "user": {
    "email": "kolompar.mario@gmail.com",
    "fullName": "Kolompár Márió",
    "phoneNumber": "+36301234567"
  },
  "billing": {
    "billingName": "Kolompár Márió",
    "companyName": null, // only if corporate
    "taxNumber": null, // only if coporate
    "country": "Magyarország",
    "zip": "1055",
    "city": "Budapest",
    "addressLine1": "Fő utca 10.",
    "addressLine2": null // optional
  },
  "subscription": {
    "planCode": "AYCM", // fix for 'All You Can Move'
    "billingCycle": "recurring",   // 'recurring' → 12 monthly installments; 'upfront' → single one-time payment
    "currency": "HUF", // for now, only HUF is accepted
    "locale": "hu" // 'hu' or 'en'
  }
}
```

### Response Codes


| Status Code | Name                    | When Used / Meaning                                                                 |
|-------------|-------------------------|--------------------------------------------------------------------------------------|
| **201**     | Created                 | ✅ Subscription was successfully created; will return the redirect URL to the Payment provider where the user can enter card details and pay.      |
| **400**     | Bad Request             | ❌ Payload is invalid — missing fields, invalid format, or validation error.          |
| **404**     | Not Found               | ❌ Related resource not found (e.g., unknown `planCode`).                               |
| **409**     | Conflict                | ❌ Resource conflict — e.g., subscription already exists for this user.               |
| **500**     | Internal Server Error   | ❌ Unexpected server error.                                                          |
| **503**     | Service Unavailable     | ❌ System is down or overloaded; try later.                                           |

### [TODO] Next Steps
- After capturing payload, we need to initiate a transaction via Nevogate
- We return the redirect URL where the user needs to navigate, to enter card details (e.g. SimplePay UI)
- User completes transaction, gets redirected back to WordPress Landing with a `transactionId`
- WordPress Landing will need to call another endpoint to check payment status
- If successful, we need to show details about the payment (hard requirement by SimplePay)
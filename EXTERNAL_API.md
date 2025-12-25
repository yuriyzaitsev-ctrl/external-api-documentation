# External API Documentation

> **⚠️ IMPORTANT - DRAFT VERSION**
>
> This is a draft version of the External API documentation. Please note:
> - Fields may be added or removed in the final version
> - All URLs are draft/placeholder - actual URLs will be provided later
> - API endpoints and response structures are subject to change
> - This documentation is for integration planning purposes only

## Table of Contents
1. [Initiate loan application](#1-initiate-loan-application)
2. [Get auth token endpoint](#2-get-auth-token-endpoint)
3. [Loan applications endpoints](#3-loan-applications-endpoints)
4. [Decision endpoints](#4-decision-endpoints)

# 1. Initiate loan application

## 1.1. Overview

After a loan application is assigned to a partner, the user will be redirected to the partner's system via the Partner System Link configured in the product settings within our platform.

## 1.2. Redirect URL Format

```
https://partner.com/path?merchant_id={merchant_uuid}&loan_application_id={loan_application_uuid}
```

**Query Parameters:**
- `merchant_id`: Secure UUID identifying the merchant in our system (e.g., `f47ac10b-58cc-4372-a567-0e02b2c3d479`)
- `loan_application_id`: Secure UUID identifying the loan application in our system (e.g., `e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d`)

## 1.3. Security

- **UUIDs are cryptographically secure** identifiers that cannot be easily guessed or enumerated
- **Each UUID is unique** and serves as a secure reference to the merchant and loan application
- Partners should **validate these UUIDs** via External API endpoints before processing
- UUIDs are **immutable** for the lifetime of the entity
- **Store loan_application_id and merchant_id** in your system to handle cases where the user navigates to the redirect URL multiple times

## 1.4. Example Redirect

```
https://partner.com/loan-processing?merchant_id=f47ac10b-58cc-4372-a567-0e02b2c3d479&loan_application_id=e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d
```

# 2. Get auth token endpoint

## 2.1. Overview

Obtain OAuth 2.0 access token for External API authentication using Client Credentials grant type.

## 2.2. Endpoint

```
POST https://auth.platform.com/realms/Integration/protocol/openid-connect/token
```

## 2.3. Request

**Headers:**
- `Content-Type: application/x-www-form-urlencoded`

**Body parameters:**
- `client_id` (required): Integration client ID, e.g. `a7f3c8b9-4d2e-4f1a-8c3b-9e5d7a2f4b6c`, will be provided by platform
- `client_secret` (required): Client secret, will be provided by platform
- `grant_type` (required): `client_credentials`

**Example:**

```bash
curl -X POST "https://auth.platform.com/realms/Integration/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=a7f3c8b9-4d2e-4f1a-8c3b-9e5d7a2f4b6c" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "grant_type=client_credentials"
```

## 2.4. Response

**Success (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ1Z2Y3ZzM2SVZ1WnpzaDNrUUc5Rlk4WVVhU3RpVUxudEQyOWszVVduTi1vIn0...",
  "expires_in": 300,
  "refresh_expires_in": 0,
  "token_type": "Bearer",
  "not-before-policy": 0,
  "scope": ""
}
```

**Fields:**
- `access_token`: JWT token for API requests
- `expires_in`: Expiration in seconds (default: 300)
- `token_type`: Always "Bearer"

## 2.5. Using Token

Include in Authorization header:

```
Authorization: Bearer {access_token}
```

**Example:**

```bash
curl -X GET "https://platform.com/api/external/v1/loan-applications" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ1Z2Y3ZzM2SVZ1WnpzaDNrUUc5Rlk4WVVhU3RpVUxudEQyOWszVVduTi1vIn0..."
```

## 2.6. Best Practices

- Cache token until expiration
- Refresh before expiration
- Never expose client credentials in client-side code
- Store credentials securely (environment variables, secure vault)

# 3. Loan applications endpoints

## 3.1. Get Loan Applications List

### 3.1.1. Overview

Retrieve paginated list of loan applications with optional status filtering. Returns only loan applications assigned to the authenticated external client (partner).

### 3.1.2. Endpoint

```
GET https://platform.com/api/external/v1/loan-applications
```

### 3.1.3. Request

**Headers:**
- `Authorization: Bearer {access_token}`

**Query Parameters:**
- `page` (optional, Integer): Page number, zero-based (default: `0`)
- `size` (optional, Integer): Page size (default: `20`, max: `100`)
- `sort` (optional, String): Sort field and direction, e.g., `createdAt,DESC` (default: `createdAt,DESC`)
- `filter.statuses` (optional, String[]): Filter by loan application statuses. Multiple values supported

**Available Statuses:**
TBD (To Be Defined)

### 3.1.4. Example Request

```bash
curl -X GET "https://platform.com/api/external/v1/loan-applications?page=0&size=20&filter.statuses=ASSIGNED&filter.statuses=APPROVED" \
  -H "Authorization: Bearer {access_token}"
```

### 3.1.5. Response

**Success (200 OK):**

```json
{
  "content": [
    {
      "id": "e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d",
      "createdAt": "2024-01-15T10:30:00Z",
      "number": "LA-2024-001",
      "status": "ASSIGNED",
      "amount": 100000.00,
      "merchant": {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "createdAt": "2023-06-15T10:00:00Z",
        "number": "M-123456",
        "name": "Merchant Business Name",
        "joiningDate": "2023-06-15T10:00:00Z",
        "tenureDays": 500,
        "totalBranches": 5,
        "lastOrderDate": "2024-12-18T15:30:00Z",
        "metrics": {
          "totalCustomers": 1500,
          "totalOrders": 3200,
          "totalAmount": 125000.50,
          "aov": 250.00,
          "oneTimeSharePct": 35.5,
          "repeatSharePct": 64.5
        },
        "sales": [
          {
            "type": "MONTHLY",
            "periodStart": "2024-01-01T00:00:00Z",
            "periodEnd": "2024-01-31T23:59:59Z",
            "totalCustomers": 500,
            "totalOrders": 1200,
            "totalAmount": 45000.00,
            "aov": 250.00,
            "oneTimeSharePct": 38.0,
            "repeatSharePct": 62.0
          },
          {
            "type": "WEEKLY",
            "periodStart": "2024-01-01T00:00:00Z",
            "periodEnd": "2024-01-07T23:59:59Z",
            "totalCustomers": 150,
            "totalOrders": 350,
            "totalAmount": 12000.00,
            "aov": 250.00,
            "oneTimeSharePct": 40.0,
            "repeatSharePct": 60.0
          }
        ]
      },
      "phoneNumber": "+966501234567",
      "secretKey": "a3f8c9b2-4d1e-4f2a-8c3b-9e5d7a2f4b6c",
      "assignedPartner": {
        "id": "d8c9f3b2-4a1e-4f2a-8c3b-9e5d7a2f4b6c",
        "createdAt": "2023-01-10T08:00:00Z",
        "number": "P-001",
        "arabicName": "بنك الشريك",
        "englishName": "Partner Bank",
        "partnerType": "BANK",
        "crNumber": "1234567890",
        "licenseNumber": "LIC-123456",
        "country": "SA",
        "specialization": ["CONSUMER_FINANCE", "BUSINESS_LOANS"],
        "description": "Leading financial institution",
        "erpCode": "ERP-001",
        "jahezComissionType": "PERCENTAGE",
        "jahezComissionAmount": 0,
        "jahezComissionPct": 2.5,
        "contact": {
          "contactPersonInfo": "John Doe, Manager",
          "supportEmail": "support@partner.com",
          "supportPhone": "+966501234567"
        },
        "products": [
          {
            "id": "c7b8a2f1-3d4e-4a5b-9c6d-8e7f5a4b3c2d",
            "number": "PROD-001",
            "name": "Business Loan",
            "description": "Flexible business financing",
            "offeringTypes": ["LOAN", "CREDIT_LINE"],
            "minAmount": 10000.00,
            "maxAmount": 500000.00,
            "minTerms": 6,
            "maxTerms": 36,
            "minMurabahaRatePct": 5.0,
            "maxMurabahaRatePct": 12.0,
            "partnerSystemLink": "https://partner.com/loan-processing"
          }
        ]
      },
      "product": {
        "id": "c7b8a2f1-3d4e-4a5b-9c6d-8e7f5a4b3c2d",
        "number": "PROD-001",
        "name": "Business Loan",
        "description": "Flexible business financing",
        "offeringTypes": ["LOAN", "CREDIT_LINE"],
        "minAmount": 10000.00,
        "maxAmount": 500000.00,
        "minTerms": 6,
        "maxTerms": 36,
        "minMurabahaRatePct": 5.0,
        "maxMurabahaRatePct": 12.0,
        "partnerSystemLink": "https://partner.com/loan-processing"
      },
      "declineInfo": [],
      "initialOffer": null
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20,
    "sort": {
      "sorted": true,
      "unsorted": false,
      "empty": false
    },
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalElements": 150,
  "totalPages": 8,
  "size": 20,
  "number": 0,
  "first": true,
  "last": false,
  "numberOfElements": 20,
  "empty": false
}
```

### 3.1.6. Response Fields

**Root Level:**
- `content`: Array of loan application objects
- `totalElements`: Total number of matching loan applications
- `totalPages`: Total number of pages
- `size`: Page size
- `number`: Current page number (zero-based)
- `first`: True if this is the first page
- `last`: True if this is the last page

**LoanApplicationExternalDto:**
- `id` (UUID): Unique loan application identifier
- `createdAt` (ISO 8601): Creation timestamp
- `number` (String): Human-readable loan application number
- `status` (String): Current application status
- `amount` (Decimal): Requested loan amount
- `merchant` (MerchantExternalDto): Complete merchant information
- `phoneNumber` (String): Customer contact phone number
- `secretKey` (UUID): Secure key for loan application
- `assignedPartner` (PartnerExternalDto): Complete partner information
- `product` (ProductExternalDto): Product selected for this loan application
- `declineInfo` (Array): Decline history (if applicable)
- `initialOffer` (Object): Initial offer details (if available)

**MerchantExternalDto:**
- `id` (UUID): Unique merchant identifier
- `createdAt` (ISO 8601): Merchant creation timestamp
- `number` (String): Merchant reference number
- `name` (String): Business name
- `joiningDate` (ISO 8601): Date when merchant joined platform
- `tenureDays` (Long): Days since joining
- `totalBranches` (Long): Number of business branches
- `lastOrderDate` (ISO 8601): Last order timestamp
- `metrics` (MetricsExternal): Aggregate business metrics
- `sales` (SalesExternal[]): Historical sales data

**MetricsExternal:**
- `totalCustomers` (Long): Total unique customers
- `totalOrders` (Long): Total number of orders
- `totalAmount` (Decimal): Total sales amount
- `aov` (Decimal): Average order value
- `oneTimeSharePct` (Decimal): One-time customer percentage
- `repeatSharePct` (Decimal): Repeat customer percentage

**SalesExternal:**
- `type` (String): Period type (MONTHLY, WEEKLY, YEARLY)
- `periodStart` (ISO 8601): Period start date
- `periodEnd` (ISO 8601): Period end date
- `totalCustomers` (Long): Customers in period
- `totalOrders` (Long): Orders in period
- `totalAmount` (Decimal): Sales amount in period
- `aov` (Decimal): Average order value in period
- `oneTimeSharePct` (Decimal): One-time customers percentage
- `repeatSharePct` (Decimal): Repeat customers percentage

**PartnerExternalDto:**
- `id` (UUID): Unique partner identifier
- `createdAt` (ISO 8601): Partner creation timestamp
- `number` (String): Partner reference number
- `arabicName` (String): Partner name in Arabic
- `englishName` (String): Partner name in English
- `partnerType` (String): Type of partner (BANK, NBFI, etc.)
- `crNumber` (String): Commercial registration number
- `licenseNumber` (String): License number
- `country` (String): Country code (ISO 3166-1 alpha-2)
- `specialization` (String[]): Areas of specialization
- `description` (String): Partner description
- `erpCode` (String): ERP system code
- `jahezComissionType` (String): Commission type (PERCENTAGE, FIXED)
- `jahezComissionAmount` (Decimal): Fixed commission amount
- `jahezComissionPct` (Decimal): Percentage commission
- `contact` (PartnerExternalContact): Contact information
- `products` (ProductExternalDto[]): Available products

**ProductExternalDto:**
- `id` (UUID): Unique product identifier
- `number` (String): Product reference number
- `name` (String): Product name
- `description` (String): Product description
- `offeringTypes` (String[]): Types of offerings
- `minAmount` (Decimal): Minimum loan amount
- `maxAmount` (Decimal): Maximum loan amount
- `minTerms` (Decimal): Minimum loan term (months)
- `maxTerms` (Decimal): Maximum loan term (months)
- `minMurabahaRatePct` (Decimal): Minimum murabaha rate
- `maxMurabahaRatePct` (Decimal): Maximum murabaha rate
- `partnerSystemLink` (String): URL to partner's system

## 3.2. Get Loan Application by ID

### 3.2.1. Overview

Retrieve a single loan application by its UUID. Returns the loan application only if it is assigned to the authenticated external client (partner).

### 3.2.2. Endpoint

```
GET https://platform.com/api/external/v1/loan-applications/{id}
```

### 3.2.3. Request

**Headers:**
- `Authorization: Bearer {access_token}`

**Path Parameters:**
- `id` (required, UUID): Loan application unique identifier

### 3.2.4. Example Request

```bash
curl -X GET "https://platform.com/api/external/v1/loan-applications/e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d" \
  -H "Authorization: Bearer {access_token}"
```

### 3.2.5. Response

**Success (200 OK):**

```json
{
  "id": "e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d",
  "createdAt": "2024-01-15T10:30:00Z",
  "number": "LA-2024-001",
  "status": "ASSIGNED",
  "amount": 100000.00,
  "merchant": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "createdAt": "2023-06-15T10:00:00Z",
    "number": "M-123456",
    "name": "Merchant Business Name",
    "joiningDate": "2023-06-15T10:00:00Z",
    "tenureDays": 500,
    "totalBranches": 5,
    "lastOrderDate": "2024-12-18T15:30:00Z",
    "metrics": {
      "totalCustomers": 1500,
      "totalOrders": 3200,
      "totalAmount": 125000.50,
      "aov": 250.00,
      "oneTimeSharePct": 35.5,
      "repeatSharePct": 64.5
    },
    "sales": [
      {
        "type": "MONTHLY",
        "periodStart": "2024-01-01T00:00:00Z",
        "periodEnd": "2024-01-31T23:59:59Z",
        "totalCustomers": 500,
        "totalOrders": 1200,
        "totalAmount": 45000.00,
        "aov": 250.00,
        "oneTimeSharePct": 38.0,
        "repeatSharePct": 62.0
      }
    ]
  },
  "phoneNumber": "+966501234567",
  "secretKey": "a3f8c9b2-4d1e-4f2a-8c3b-9e5d7a2f4b6c",
  "assignedPartner": {
    "id": "d8c9f3b2-4a1e-4f2a-8c3b-9e5d7a2f4b6c",
    "createdAt": "2023-01-10T08:00:00Z",
    "number": "P-001",
    "arabicName": "بنك الشريك",
    "englishName": "Partner Bank",
    "partnerType": "BANK",
    "crNumber": "1234567890",
    "licenseNumber": "LIC-123456",
    "country": "SA",
    "specialization": ["CONSUMER_FINANCE", "BUSINESS_LOANS"],
    "description": "Leading financial institution",
    "erpCode": "ERP-001",
    "jahezComissionType": "PERCENTAGE",
    "jahezComissionAmount": 0,
    "jahezComissionPct": 2.5,
    "contact": {
      "contactPersonInfo": "John Doe, Manager",
      "supportEmail": "support@partner.com",
      "supportPhone": "+966501234567"
    },
    "products": [
      {
        "id": "c7b8a2f1-3d4e-4a5b-9c6d-8e7f5a4b3c2d",
        "number": "PROD-001",
        "name": "Business Loan",
        "description": "Flexible business financing",
        "offeringTypes": ["LOAN", "CREDIT_LINE"],
        "minAmount": 10000.00,
        "maxAmount": 500000.00,
        "minTerms": 6,
        "maxTerms": 36,
        "minMurabahaRatePct": 5.0,
        "maxMurabahaRatePct": 12.0,
        "partnerSystemLink": "https://partner.com/loan-processing"
      }
    ]
  },
  "product": {
    "id": "c7b8a2f1-3d4e-4a5b-9c6d-8e7f5a4b3c2d",
    "number": "PROD-001",
    "name": "Business Loan",
    "description": "Flexible business financing",
    "offeringTypes": ["LOAN", "CREDIT_LINE"],
    "minAmount": 10000.00,
    "maxAmount": 500000.00,
    "minTerms": 6,
    "maxTerms": 36,
    "minMurabahaRatePct": 5.0,
    "maxMurabahaRatePct": 12.0,
    "partnerSystemLink": "https://partner.com/loan-processing"
  },
  "declineInfo": [],
  "initialOffer": null
}
```

**Error Responses:**

**401 Unauthorized:**
```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid or expired access token"
}
```

**403 Forbidden:**
```json
{
  "status": 403,
  "error": "Forbidden",
  "message": "Access denied"
}
```

**404 Not Found:**
```json
{
  "status": 404,
  "error": "Not Found",
  "message": "Loan application not found or not assigned to your partner"
}
```

### 3.2.6. Use Cases

**Validate Redirect Parameters:**
When a user is redirected from our platform with `merchant_id` and `loan_application_id` parameters, use this endpoint to:
1. Validate the UUIDs
2. Retrieve complete merchant information
3. Retrieve loan application details
4. Begin your internal processing

**Example:**
```bash
# User redirected with: 
# https://partner.com/process?merchant_id=f47ac10b-58cc-4372-a567-0e02b2c3d479&loan_application_id=e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d

# Validate and fetch details:
curl -X GET "https://platform.com/api/external/v1/loan-applications/e9b2d8c4-1a3f-4b7e-9c2d-6f8a5e3b1c0d" \
  -H "Authorization: Bearer {access_token}"

# Response will include merchant with id: f47ac10b-58cc-4372-a567-0e02b2c3d479
```

## 3.3. Authentication

All endpoints require Bearer token authentication. Include the access token in the Authorization header:

```
Authorization: Bearer {access_token}
```

Obtain the access token using the token endpoint (see section 1).

# 4. Decision endpoints

TBD (To Be Defined)

## 4.1. Error Handling

All error responses follow standard HTTP status codes:

- **404 Not Found**: When requesting a specific loan application by ID that doesn't exist or is not assigned to your partner
- **401 Unauthorized**: Invalid or expired access token
- **403 Forbidden**: Access denied

When requesting the loan applications list, non-existent or inaccessible applications will return an empty `content` array rather than an error.


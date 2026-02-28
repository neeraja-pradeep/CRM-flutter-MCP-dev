# NexoCRM Django API Documentation

**Base URL:** `http://156.67.104.149:8013`

**Version:** 1.0.0

**Last Updated:** January 26, 2026

---

## Table of Contents

1. [Authentication](#authentication)
2. [CRM - Leads](#crm---leads)
3. [CRM - Customers](#crm---customers)
4. [CRM - Tasks](#crm---tasks)
5. [CRM - Communications](#crm---communications)
6. [CRM - Issues](#crm---issues)
7. [CRM - Organizations](#crm---organizations)
8. [CRM - Chat AI Assistant](#crm---chat-ai-assistant)
9. [Projects Management](#projects-management)
10. [Email Management](#email-management)
11. [User Management](#user-management)
12. [Role Management](#role-management)
13. [Team Management](#team-management)
14. [Hierarchy Management](#hierarchy-management)
15. [Permissions Management](#permissions-management)
16. [Organization Management](#organization-management)
17. [Access Control](#access-control)
18. [Error Codes](#error-codes)
19. [Rate Limiting](#rate-limiting)

---

## Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <access_token>
```

### 1.1 Organization Registration

**Endpoint:** `POST /api/v1/organizations/register/`

**Authentication:** None (Public endpoint)

**Rate Limit:** 10 requests per minute per IP

**Description:** Register a new organization with an admin user. Creates organization, admin user, and seeds default roles and permissions.

**Request Body:**
```json
{
  "org_name": "Demo Organization",
  "subdomain": "demo-org",
  "first_name": "Admin",
  "last_name": "User",
  "email": "admin@demoorg.com",
  "password": "SecurePassword123!"
}
```

**Request Fields:**
- `org_name` (string, required): Organization name (max 200 characters)
- `subdomain` (string, required): Unique subdomain (min 3 characters, alphanumeric and hyphens only)
- `first_name` (string, required): Admin user's first name (max 150 characters)
- `last_name` (string, optional): Admin user's last name (max 150 characters)
- `email` (string, required): Valid email address
- `password` (string, required): Password (min 8 chars, must contain uppercase, lowercase, number, and special character)

**Success Response (201 Created):**
```json
{
  "message": "Organization and Admin user created successfully.",
  "organization": {
    "id": 1,
    "name": "Demo Organization",
    "subdomain": "demo-org"
  },
  "user": {
    "id": 1,
    "email": "admin@demoorg.com",
    "full_name": "Admin User"
  }
}
```

**Error Responses:**

**400 Bad Request:**
```json
{
  "subdomain": ["This subdomain is already taken."],
  "email": ["A user with this email already exists."]
}
```

**400 Bad Request (Password Validation):**
```json
{
  "password": [
    "Password must be at least 8 characters long.",
    "Password must contain at least one uppercase letter."
  ]
}
```

---

### 1.2 Login

**Endpoint:** `POST /api/v1/auth/login/`

**Authentication:** None

**Rate Limit:** 10 requests per minute per IP

**Description:** Login with email and password. Returns JWT access and refresh tokens. Account will be locked after 5 failed attempts in 1 hour.

**Request Body:**
```json
{
  "email": "admin@demoorg.com",
  "password": "SecurePassword123!"
}
```

**Request Fields:**
- `email` (string, required): User's email address
- `password` (string, required): User's password

**Success Response (200 OK):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "admin@demoorg.com",
    "full_name": "Admin User",
    "organizations": [
      {
        "id": 1,
        "name": "Demo Organization",
        "subdomain": "demo-org",
        "is_primary": true
      }
    ]
  }
}
```

**Error Responses:**

**401 Unauthorized:**
```json
{
  "detail": "No active account found with the given credentials"
}
```

**429 Too Many Requests (Account Locked):**
```json
{
  "error": "Account locked due to too many failed login attempts. Please try again in 1 hour or reset your password."
}
```

---

### 1.3 Refresh Token

**Endpoint:** `POST /api/v1/auth/login/refresh/`

**Authentication:** None

**Description:** Refresh the access token using a valid refresh token.

**Request Body:**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Success Response (200 OK):**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error Response (401 Unauthorized):**
```json
{
  "detail": "Token is invalid or expired",
  "code": "token_not_valid"
}
```

---

### 1.4 Get Current User (Me)

**Endpoint:** `GET /api/v1/auth/me/`

**Authentication:** Required

**Rate Limit:** 60 requests per minute per user

**Description:** Returns the current authenticated user's details.

**Success Response (200 OK):**
```json
{
  "id": 1,
  "email": "admin@demoorg.com",
  "username": "admin@demoorg.com",
  "first_name": "Admin",
  "last_name": "User",
  "full_name": "Admin User",
  "profile": {
    "phone": "+1234567890",
    "mobile": "+1234567890",
    "user_image": "https://example.com/image.jpg",
    "timezone": "UTC",
    "language": "en",
    "gender": "Male",
    "birth_date": "1990-01-01",
    "location": "New York, USA",
    "bio": "System Administrator",
    "preferences": {
      "theme": "dark",
      "notifications": true
    }
  },
  "organizations": [
    {
      "id": 1,
      "name": "Demo Organization",
      "subdomain": "demo-org",
      "is_primary": true
    }
  ]
}
```

---

### 1.5 Logout

**Endpoint:** `POST /api/v1/auth/logout/`

**Authentication:** Required

**Rate Limit:** 10 requests per minute per IP

**Description:** Blacklist the refresh token to logout the user.

**Request Body:**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Success Response (200 OK):**
```json
{
  "message": "Successfully logged out."
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Invalid token."
}
```

---

### 1.6 Register Internal User

**Endpoint:** `POST /api/v1/auth/register/`

**Authentication:** Required

**Permission:** `create_user`

**Rate Limit:** 10 requests per minute per IP

**Description:** Internal user registration by admins. Creates a new user within the organization.

**Request Body:**
```json
{
  "email": "newuser@demoorg.com",
  "password": "SecurePassword123!",
  "first_name": "New",
  "last_name": "User",
  "role_ids": [1, 2]
}
```

**Request Fields:**
- `email` (string, required): User's email address
- `password` (string, required): Password (min 8 chars with complexity requirements)
- `first_name` (string, required): User's first name
- `last_name` (string, optional): User's last name
- `role_ids` (array, optional): Array of role IDs to assign

**Success Response (201 Created):**
```json
{
  "id": 2,
  "email": "newuser@demoorg.com",
  "username": "newuser@demoorg.com",
  "first_name": "New",
  "last_name": "User",
  "full_name": "New User",
  "profile": null,
  "organizations": [
    {
      "id": 1,
      "name": "Demo Organization",
      "subdomain": "demo-org",
      "is_primary": true
    }
  ]
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "User with this email already exists"
}
```

---

### 1.7 Password Reset Request

**Endpoint:** `POST /api/v1/auth/password-reset/`

**Authentication:** None

**Description:** Request a password reset email with token (expires in 1 hour).

**Request Body:**
```json
{
  "email": "admin@demoorg.com"
}
```

**Success Response (200 OK):**
```json
{
  "message": "If an account exists with this email, a password reset link has been sent."
}
```

**Note:** Always returns 200 OK to prevent email enumeration.

---

### 1.8 Password Reset Confirm

**Endpoint:** `POST /api/v1/auth/password-reset/confirm/`

**Authentication:** None

**Description:** Confirm password reset with token and new password.

**Request Body:**
```json
{
  "uid": "MQ",
  "token": "abc123-def456",
  "password": "NewSecurePassword123!"
}
```

**Success Response (200 OK):**
```json
{
  "message": "Password has been reset successfully."
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Invalid or expired reset token"
}
```

---

## CRM - Leads

### 2.1 List Leads

**Endpoint:** `GET /api/v1/crm/leads/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** List all leads with pagination and comprehensive filtering options. Applies record-level permissions based on user role.

**Query Parameters:**

#### Pagination & Dynamic Fields
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Items per page (default: 20, max: 100)
- `fields` (string, optional): Comma-separated list of fields to include (e.g., `lead_name,email,status`)
  - Supports nested fields with dot notation: `assignees.full_name,assignees.email`
- `ordering` (string, optional): Sort field (e.g., `-created_at`, `lead_name`)

#### Date Filters
- `created_at` (datetime, optional): Exact creation date
- `created_at_after` (datetime, optional): Created after this date (format: `YYYY-MM-DD` or `YYYY-MM-DDTHH:MM:SS`)
- `created_at_before` (datetime, optional): Created before this date
- `updated_at` (datetime, optional): Exact update date
- `updated_at_after` (datetime, optional): Updated after this date
- `updated_at_before` (datetime, optional): Updated before this date
- `converted_on` (datetime, optional): Exact conversion date
- `converted_on_after` (datetime, optional): Converted after this date
- `converted_on_before` (datetime, optional): Converted before this date

#### Status & Classification Filters
- `status` (integer, optional): Filter by status ID
- `status__in` (string, optional): Filter by multiple status IDs (comma-separated, e.g., `1,2,3`)
- `status_name` (string, optional): Filter by status name (exact match, case-insensitive, e.g., `New Lead`)
- `status_name__icontains` (string, optional): Filter by status name (partial match, e.g., `progress`)
- `lead_source` (integer, optional): Filter by lead source ID
- `lead_source__in` (string, optional): Filter by multiple lead source IDs
- `lead_source_name` (string, optional): Filter by lead source name (exact match, case-insensitive)
- `lead_source_name__icontains` (string, optional): Filter by lead source name (partial match)
- `territory` (integer, optional): Filter by territory ID
- `territory__in` (string, optional): Filter by multiple territory IDs
- `territory_name` (string, optional): Filter by territory name (exact match, case-insensitive)
- `territory_name__icontains` (string, optional): Filter by territory name (partial match)
- `product` (integer, optional): Filter by product ID
- `product__in` (string, optional): Filter by multiple product IDs

#### Owner & Assignee Filters
- `lead_owner` (integer, optional): Filter by owner user ID
- `lead_owner__in` (string, optional): Filter by multiple owner user IDs
- `assignees` (integer, optional): Filter by assignee user ID (leads assigned to this user)
- `assignees__in` (string, optional): Filter by multiple assignee user IDs

#### Boolean Filters
- `is_converted` (boolean, optional): Filter by conversion status (`true` or `false`)
- `unsubscribed` (boolean, optional): Filter by unsubscribe status
- `do_not_disturb` (boolean, optional): Filter by DND status

#### Text Filters
- `lead_name` (string, optional): Exact lead name match
- `lead_name__icontains` (string, optional): Lead name contains (case-insensitive)
- `first_name` (string, optional): Exact first name match
- `first_name__icontains` (string, optional): First name contains
- `last_name` (string, optional): Exact last name match
- `last_name__icontains` (string, optional): Last name contains
- `email` (string, optional): Exact email match
- `email__icontains` (string, optional): Email contains
- `phone` (string, optional): Exact phone match
- `phone__icontains` (string, optional): Phone contains
- `mobile_no` (string, optional): Exact mobile number match
- `mobile_no__icontains` (string, optional): Mobile number contains
- `industry` (string, optional): Exact industry match (case-insensitive)
- `industry__icontains` (string, optional): Industry contains
- `organization_name` (string, optional): Exact organization name match (case-insensitive)
- `organization_name__icontains` (string, optional): Organization name contains
- `website` (string, optional): Exact website match
- `website__icontains` (string, optional): Website contains
- `salutation` (string, optional): Exact salutation match

#### Numeric Range Filters
- `lead_score` (integer, optional): Exact lead score
- `lead_score_min` (integer, optional): Minimum lead score (inclusive)
- `lead_score_max` (integer, optional): Maximum lead score (inclusive)
- `annual_revenue` (decimal, optional): Exact annual revenue
- `annual_revenue_min` (decimal, optional): Minimum annual revenue
- `annual_revenue_max` (decimal, optional): Maximum annual revenue
- `no_of_employees` (string, optional): Number of employees

#### Multi-Field Search
- `search` (string, optional): Search across multiple fields simultaneously:
  - lead_name, first_name, last_name, email, phone, mobile_no, organization_name
  - Example: `?search=john` will find leads with "john" in any of these fields

**Example Requests:**

1. Basic listing with pagination:
```
GET /api/v1/crm/leads/?page=1&page_size=20&ordering=-created_at
```

2. Filter by status ID and conversion:
```
GET /api/v1/crm/leads/?status=1&is_converted=false&ordering=-created_at
```

2b. Filter by status name and conversion:
```
GET /api/v1/crm/leads/?status_name=New%20Lead&is_converted=false&ordering=-created_at
```

3. Filter by date range:
```
GET /api/v1/crm/leads/?created_at_after=2024-01-01&created_at_before=2024-12-31
```

4. Filter by multiple statuses and owner:
```
GET /api/v1/crm/leads/?status__in=1,2,3&lead_owner=5
```

5. Filter by assignees in Technology industry:
```
GET /api/v1/crm/leads/?assignees__in=3,4&industry=Technology
```

6. Filter by lead score range with high-value leads:
```
GET /api/v1/crm/leads/?lead_score_min=70&annual_revenue_min=500000
```

7. Search across multiple fields:
```
GET /api/v1/crm/leads/?search=acme
```

8. Combine filters with dynamic fields:
```
GET /api/v1/crm/leads/?fields=lead_name,email,status,assignees.full_name&status=1&is_converted=false&page_size=50
```

9. Get unconverted leads from specific product and territory:
```
GET /api/v1/crm/leads/?product=5&territory=10&is_converted=false
```

10. Complex filter combining multiple criteria:
```
GET /api/v1/crm/leads/?status__in=1,2&lead_score_min=50&created_at_after=2024-01-01&assignees=3&industry__icontains=tech
```

11. Filter by status name (partial match):
```
GET /api/v1/crm/leads/?status_name__icontains=progress
```

12. Filter by lead source and territory names:
```
GET /api/v1/crm/leads/?lead_source_name=Website&territory_name=North%20Region
```

**Success Response (200 OK):**
```json
{
  "count": 150,
  "next": "http://156.67.104.149:8013/api/v1/crm/leads/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "nexocrm-lead-1",
      "salutation": "Mr.",
      "first_name": "John",
      "last_name": "Doe",
      "lead_name": "John Doe",
      "email": "john.doe@example.com",
      "mobile_no": "+1234567890",
      "phone": "+1234567890",
      "whatsapp_no": "+1234567890",
      "organization_name": "Example Corp",
      "website": "https://example.com",
      "no_of_employees": "50-100",
      "annual_revenue": "1000000.00",
      "industry": "Technology",
      "territory": 1,
      "status": "New Lead",
      "lead_source": 1,
      "lead_owner": 1,
      "lead_score": 75,
      "assignees": [1, 2],
      "is_converted": false,
      "converted_to_deal": null,
      "converted_on": null,
      "unsubscribed": false,
      "do_not_disturb": false,
      "notes": "Interested in enterprise solution",
      "custom_fields": {
        "budget": "100000",
        "timeline": "Q1 2026"
      },
      "organization": 1,
      "owner": 1,
      "created_at": "2026-01-15T10:30:00Z",
      "updated_at": "2026-01-20T15:45:00Z"
    }
  ]
}
```

---

### 2.2 Get Lead

**Endpoint:** `GET /api/v1/crm/leads/{id}/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** Retrieve a specific lead by ID.

**Path Parameters:**
- `id` (integer, required): Lead ID

**Success Response (200 OK):**
```json
{
  "id": 1,
  "name": "nexocrm-lead-1",
  "salutation": "Mr.",   //needed
  "first_name": "John", //needed
  "last_name": "Doe", //needed
  "lead_name": "John Doe",  //needed
  "email": "john.doe@example.com",
  "mobile_no": "+1234567890", //needed for dialer
  "phone": "+1234567890",
  "whatsapp_no": "+1234567890",
  "organization_name": "Example Corp",
  "website": "https://example.com",
  "no_of_employees": "50-100",
  "annual_revenue": "1000000.00",
  "industry": "Technology",
  "territory": 1,
  "status": "New Lead",
  "lead_source": 1,
  "lead_owner": 1,
  "lead_score": 75,
  "assignees": [1, 2],
  "is_converted": false,
  "converted_to_deal": null,
  "converted_on": null,
  "unsubscribed": false,
  "do_not_disturb": false,
  "notes": "Interested in enterprise solution",
  "custom_fields": {
    "budget": "100000",
    "timeline": "Q1 2026"
  },
  "organization": 1,
  "owner": 1,
  "created_at": "2026-01-15T10:30:00Z",   //needed
  "updated_at": "2026-01-20T15:45:00Z",
  "product_name": "product"
}
```

**Error Response (404 Not Found):**
```json
{
  "detail": "Not found."
}
```

---

### 2.3 Create Lead

**Endpoint:** `POST /api/v1/crm/leads/`

**Authentication:** Required

**Permission:** `create_lead`

**Description:** Create a new lead. If no status is provided, the system automatically assigns the default status (where `is_default=true` for the organization). The response returns the status name instead of the ID.

**Request Body:**
```json
{
  "salutation": "Mr.",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@example.com",
  "mobile_no": "+1234567890",
  "phone": "+1234567890",
  "whatsapp_no": "+1234567890",
  "organization_name": "Example Corp",
  "website": "https://example.com",
  "no_of_employees": "50-100",
  "annual_revenue": "1000000.00",
  "industry": "Technology",
  "territory": 1,
  "status_id": 1,
  "lead_source": 1,
  "lead_owner": 1,
  "lead_score": 75,
  "assignees": [1, 2],
  "notes": "Interested in enterprise solution",
  "custom_fields": {
    "budget": "100000",
    "timeline": "Q1 2026"
  }
}
```

**Required Fields:**
- `first_name` (string): Lead's first name

**Optional Fields:**
- `status_id` (integer): Lead status ID. If omitted, the system automatically assigns the default status for your organization
- All other fields are optional

**Important Notes:**
- Use `status_id` (not `status`) when creating or updating leads
- The response will return `status` as the status name (e.g., "New Lead")
- If no `status_id` is provided, the default status is automatically assigned
- `lead_name` is auto-generated (e.g., "LEAD-00001") if not provided

**Success Response (201 Created):**
```json
{
  "id": 2,
  "name": "nexocrm-lead-2",
  "salutation": "Mr.",
  "first_name": "John",
  "last_name": "Doe",
  "lead_name": "John Doe",
  "email": "john.doe@example.com",
  "mobile_no": "+1234567890",
  "phone": "+1234567890",
  "whatsapp_no": "+1234567890",
  "organization_name": "Example Corp",
  "website": "https://example.com",
  "no_of_employees": "50-100",
  "annual_revenue": "1000000.00",
  "industry": "Technology",
  "territory": 1,
  "status": "New Lead",
  "lead_source": 1,
  "lead_owner": 1,
  "lead_score": 75,
  "assignees": [1, 2],
  "is_converted": false,
  "organization": 1,
  "owner": 1,
  "created_at": "2026-01-26T10:30:00Z",
  "updated_at": "2026-01-26T10:30:00Z"
}
```

---

### 2.4 Update Lead

**Endpoint:** `PUT /api/v1/crm/leads/{id}/`

**Authentication:** Required

**Permission:** `update_lead`

**Description:** Update a lead (full update).

**Path Parameters:**
- `id` (integer, required): Lead ID

**Request Body:** Same as Create Lead (all fields required)

**Success Response (200 OK):** Same structure as Get Lead

---

### 2.5 Partial Update Lead

**Endpoint:** `PATCH /api/v1/crm/leads/{id}/`

**Authentication:** Required

**Permission:** `update_lead`

**Description:** Partially update a lead.

**Request Body:**
```json
{
  "status_id": 2,
  "lead_score": 85,
  "notes": "Follow up scheduled for next week"
}
```

**Note:** Use `status_id` to update the lead status. The response will return `status` as the status name.

**Success Response (200 OK):** Same structure as Get Lead

---

### 2.6 Delete Lead

**Endpoint:** `DELETE /api/v1/crm/leads/{id}/`

**Authentication:** Required

**Permission:** `delete_lead`

**Description:** Delete a lead.

**Success Response (204 No Content):** Empty response

---

### 2.7 Convert Lead to Customer

**Endpoint:** `POST /api/v1/crm/leads/{id}/convert/`

**Authentication:** Required

**Permission:** `convert_lead`

**Description:** Convert a lead to a customer.

**Request Body:**
```json
{}
```

**Success Response (201 Created):**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com",
  "phone": "+1234567890",
  "address": "123 Main St, New York, NY 10001",
  "status": "Active",
  "organization": 1,
  "created_at": "2026-01-26T10:30:00Z",
  "updated_at": "2026-01-26T10:30:00Z"
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Lead is already converted"
}
```

---

### 2.7b Get Lead Schema

**Endpoint:** `GET /api/v1/crm/leads/schema/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** Returns the Lead model schema with field types and attributes. This endpoint provides metadata about the Lead model structure, optionally filtered based on user's view settings.

**Query Parameters:**
- `view_type` (string, optional): View type to filter fields - `list`, `detail`, or `form` (default: `list`)
- `include_all` (boolean, optional): Set to `true` to include all fields regardless of user settings (default: `false`)

**Use Cases:**
- Generate dynamic forms based on Lead model structure
- Build custom UI components that adapt to user's configured view settings
- Validate data before submission
- Display field-specific help text and validation rules

**Example Requests:**

1. Get schema for list view (respects user settings):
```
GET /api/v1/crm/leads/schema/?view_type=list
```

2. Get all fields regardless of user settings:
```
GET /api/v1/crm/leads/schema/?include_all=true
```

3. Get schema for form view:
```
GET /api/v1/crm/leads/schema/?view_type=form
```

**Success Response (200 OK):**
```json
{
  "model": "Lead",
  "view_type": "list",
  "has_user_settings": true,
  "fields": {
    "lead_id": {
      "name": "lead_id",
      "verbose_name": "lead id",
      "type": "biginteger",
      "required": false,
      "nullable": true,
      "editable": true
    },
    "first_name": {
      "name": "first_name",
      "verbose_name": "first name",
      "type": "string",
      "max_length": 140,
      "required": true,
      "nullable": false,
      "editable": true
    },
    "last_name": {
      "name": "last_name",
      "verbose_name": "last name",
      "type": "string",
      "max_length": 140,
      "required": false,
      "nullable": true,
      "editable": true
    },
    "email": {
      "name": "email",
      "verbose_name": "email",
      "type": "string",
      "max_length": 140,
      "required": false,
      "nullable": true,
      "editable": true
    },
    "mobile_no": {
      "name": "mobile_no",
      "verbose_name": "mobile no",
      "type": "string",
      "max_length": 20,
      "required": false,
      "nullable": true,
      "editable": true
    },
    "status": {
      "name": "status",
      "verbose_name": "status",
      "type": "foreignkey",
      "related_model": "LeadStatus",
      "related_field": "lead_status_id",
      "required": false,
      "nullable": true,
      "editable": true
    },
    "lead_score": {
      "name": "lead_score",
      "verbose_name": "lead score",
      "type": "integer",
      "required": false,
      "nullable": false,
      "editable": true,
      "default": 0
    },
    "annual_revenue": {
      "name": "annual_revenue",
      "verbose_name": "annual revenue",
      "type": "decimal",
      "max_digits": 18,
      "decimal_places": 2,
      "required": false,
      "nullable": true,
      "editable": true
    },
    "is_converted": {
      "name": "is_converted",
      "verbose_name": "is converted",
      "type": "boolean",
      "required": false,
      "nullable": false,
      "editable": true,
      "default": false
    },
    "assignees": {
      "name": "assignees",
      "verbose_name": "assignees",
      "type": "manytomany",
      "related_model": "User",
      "required": false,
      "nullable": false,
      "editable": true
    },
    "custom_fields": {
      "name": "custom_fields",
      "verbose_name": "custom fields",
      "type": "json",
      "required": false,
      "nullable": true,
      "editable": true
    },
    "created_at": {
      "name": "created_at",
      "verbose_name": "created at",
      "type": "datetime",
      "required": false,
      "nullable": false,
      "editable": false
    }
  },
  "user_settings": {
    "columns": [
      {
        "field": "lead_name",
        "label": "Lead Name",
        "visible": true,
        "order": 1,
        "width": 200
      },
      {
        "field": "email",
        "label": "Email",
        "visible": true,
        "order": 2,
        "width": 250
      },
      {
        "field": "status",
        "label": "Status",
        "visible": true,
        "order": 3,
        "width": 150
      },
      {
        "field": "lead_score",
        "label": "Score",
        "visible": true,
        "order": 4,
        "width": 100
      },
      {
        "field": "created_at",
        "label": "Created",
        "visible": false,
        "order": 5,
        "width": 180
      }
    ],
    "sorting": {
      "field": "created_at",
      "direction": "desc"
    },
    "filters": {
      "status": [1, 2],
      "is_converted": false
    }
  }
}
```

**Response Fields:**
- `model` (string): Model name ("Lead")
- `view_type` (string): The requested view type
- `has_user_settings` (boolean): Whether user-specific view settings exist
- `fields` (object): Dictionary of field definitions keyed by field name
  - Each field contains:
    - `name` (string): Field name
    - `verbose_name` (string): Human-readable field name
    - `type` (string): Field type (`string`, `integer`, `biginteger`, `decimal`, `float`, `boolean`, `datetime`, `date`, `time`, `json`, `foreignkey`, `manytomany`, `text`)
    - `required` (boolean): Whether the field is required
    - `nullable` (boolean): Whether the field accepts null values
    - `editable` (boolean): Whether the field can be edited
    - Type-specific attributes:
      - String fields: `max_length`
      - Decimal fields: `max_digits`, `decimal_places`
      - Boolean fields: `default`
      - Foreign key fields: `related_model`, `related_field`
      - Many-to-many fields: `related_model`
      - Fields with choices: `choices` array
- `user_settings` (object, optional): User's view configuration if available
  - `columns` (array): Column configuration (field, label, visibility, order, width)
  - `sorting` (object): Default sorting preferences
  - `filters` (object): Saved filter values

**Field Types:**
- `string`: CharField - text with max_length
- `text`: TextField - unlimited text
- `integer`: IntegerField - whole numbers
- `biginteger`: BigIntegerField - large whole numbers
- `decimal`: DecimalField - precise decimal numbers
- `float`: FloatField - floating-point numbers
- `boolean`: BooleanField - true/false
- `datetime`: DateTimeField - date and time
- `date`: DateField - date only
- `time`: TimeField - time only
- `json`: JSONField - JSON data
- `foreignkey`: ForeignKey - relation to another model
- `manytomany`: ManyToManyField - multiple relations

**Notes:**
- When `include_all=false` (default), only fields configured in user's view settings are returned
- When `include_all=true`, all model fields are returned
- Fields marked as `editable=false` are typically auto-generated or read-only
- The `user_settings` section is only included if the user has custom view settings configured

---

### 2.8 List Lead Statuses

**Endpoint:** `GET /api/v1/crm/lead-statuses/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** List all lead statuses for the organization. Each organization can have multiple lead statuses, but only one can be marked as `is_default=true`.

**Success Response (200 OK):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "lead_status_id": 1,
      "name": "New",
      "color": "#3498db",
      "position": 1,
      "is_active": true,
      "is_default": true,
      "is_converted": false,
      "organization": 1,
      "created_at": "2026-01-15T10:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

### 2.9 Create Lead Status

**Endpoint:** `POST /api/v1/crm/lead-statuses/`

**Authentication:** Required

**Permission:** `create_lead`

**Description:** Create a new lead status. When setting `is_default=true`, the system automatically sets all other statuses in the organization to `is_default=false` to ensure only one default status exists.

**Request Body:**
```json
{
  "name": "Qualified",
  "color": "#2ecc71",
  "position": 2,
  "is_active": true,
  "is_default": false,
  "is_converted": false
}
```

**Field Descriptions:**
- `name` (string, required): Status name (must be unique within the organization)
- `color` (string, optional): Hex color code for UI display (e.g., "#2ecc71")
- `position` (integer, optional): Display order position (default: 0)
- `is_active` (boolean, optional): Whether the status is active (default: true)
- `is_default` (boolean, optional): Whether this is the default status for new leads (default: false)
  - **Important:** Only one status per organization can be `is_default=true`
  - Setting a new status as default automatically unsets the previous default
- `is_converted` (boolean, optional): Whether this status indicates a converted lead (default: false)

**Success Response (201 Created):** Same structure as list response

**Important Notes:**
- Each organization can only have ONE default status at a time
- When you set a status as `is_default=true`, all other statuses are automatically updated to `is_default=false`
- The default status is automatically assigned to new leads when no status is specified
- Status names must be unique within an organization

---

### 2.10 List Lead Sources

**Endpoint:** `GET /api/v1/crm/lead-sources/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** List all lead sources for the organization.

**Success Response (200 OK):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "lead_source_id": 1,
      "name": "Website",
      "is_active": true,
      "description": "Leads from website contact form",
      "organization": 1,
      "created_at": "2026-01-15T10:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

### 2.11 Create Lead Source

**Endpoint:** `POST /api/v1/crm/lead-sources/`

**Authentication:** Required

**Permission:** `create_lead`

**Request Body:**
```json
{
  "name": "Referral",
  "is_active": true,
  "description": "Customer referrals"
}
```

**Success Response (201 Created):** Same structure as list response

---

### 2.12 List Territories

**Endpoint:** `GET /api/v1/crm/territories/`

**Authentication:** Required

**Permission:** `view_lead`

**Description:** List all territories for the organization.

**Success Response (200 OK):**
```json
{
  "count": 4,
  "next": null,
  "previous": null,
  "results": [
    {
      "territory_id": 1,
      "name": "North America",
      "parent_territory": null,
      "is_group": true,
      "lft": 1,
      "rgt": 10,
      "organization": 1,
      "created_at": "2026-01-15T10:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

### 2.13 Create Territory

**Endpoint:** `POST /api/v1/crm/territories/`

**Authentication:** Required

**Permission:** `create_lead`

**Request Body:**
```json
{
  "name": "West Coast",
  "parent_territory": 1,
  "is_group": false
}
```

**Success Response (201 Created):** Same structure as list response

**Note:** All meta resources (Lead Statuses, Lead Sources, Territories) support full CRUD operations:
- `GET /api/v1/crm/{resource}/` - List all
- `GET /api/v1/crm/{resource}/{id}/` - Get single
- `POST /api/v1/crm/{resource}/` - Create
- `PUT /api/v1/crm/{resource}/{id}/` - Full update
- `PATCH /api/v1/crm/{resource}/{id}/` - Partial update
- `DELETE /api/v1/crm/{resource}/{id}/` - Delete

---

### 2.14 List Products

**Endpoint:** `GET /api/v1/crm/products/`

**Authentication:** Required

**Permission:** `view_product`

**Description:** List all products in the organization's catalog.

**Success Response (200 OK):**
```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "product_id": 1,
      "product_name": "Enterprise Plan",
      "product_code": "ENT-001",
      "product_type": 1,
      "description": "Full-featured enterprise solution",
      "is_active": true,
      "price": "999.000000",
      "currency": "USD",
      "image": "https://example.com/products/enterprise.png",
      "organization": 1,
      "created_at": "2026-01-15T10:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

### 2.15 Create Product

**Endpoint:** `POST /api/v1/crm/products/`

**Authentication:** Required

**Permission:** `create_product`

**Request Body:**
```json
{
  "product_name": "Professional Plan",
  "product_code": "PRO-001",
  "product_type": 1,
  "description": "Professional tier with advanced features",
  "is_active": true,
  "price": "499.00",
  "currency": "USD",
  "image": "https://example.com/products/professional.png"
}
```

**Success Response (201 Created):** Same structure as list response

---

### 2.16 List Product Types

**Endpoint:** `GET /api/v1/crm/product-types/`

**Authentication:** Required

**Permission:** `view_product`

**Description:** List all product types/categories.

**Success Response (200 OK):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "product_type_id": 1,
      "type_name": "Software",
      "description": "Software products and subscriptions",
      "organization": 1,
      "created_at": "2026-01-15T10:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

### 2.17 Create Product Type

**Endpoint:** `POST /api/v1/crm/product-types/`

**Authentication:** Required

**Permission:** `create_product`

**Request Body:**
```json
{
  "type_name": "Hardware",
  "description": "Physical hardware products"
}
```

**Success Response (201 Created):** Same structure as list response

**Note:** Products and Product Types also support full CRUD operations (GET, POST, PUT, PATCH, DELETE).

---

## Permission Model

### Staff Users

**Important:** Users with `is_staff=True` have **full access to all endpoints** and bypass all permission checks. This includes:
- All module-level permissions (view, create, update, delete, export, import)
- All record-level permissions (owned, team, hierarchy, etc.)
- Organization-scoped data access

Staff users are intended for system administrators and support personnel who need unrestricted access.

### Organization Isolation

**All CRM data is organization-scoped:**
- Every meta table (Lead Status, Lead Source, Territory, Product Type, etc.) is filtered by organization
- Users can only access data belonging to their organization
- The `organization` field is automatically set on creation
- Staff users (`is_staff=True`) can access all organizations

### Permission Hierarchy

1. **Superuser** (`is_superuser=True`): Full access to everything
2. **Staff** (`is_staff=True`): Full access to all endpoints and all organizations
3. **Role-based**: Permissions controlled by role assignments and module/record permissions

---

## CRM - Customers

### 3.1 List Customers

**Endpoint:** `GET /api/v1/crm/customers/`

**Authentication:** Required

**Permission:** `view_contact`

**Description:** List all customers with pagination. Applies record-level permissions.

**Query Parameters:**
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page
- `search` (string, optional): Search in name, email
- `status` (string, optional): Filter by status

**Success Response (200 OK):**
```json
{
  "count": 75,
  "next": "http://156.67.104.149:8013/api/v1/crm/customers/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Jane Smith",
      "email": "jane.smith@example.com",
      "phone": "+1234567891",
      "address": "456 Oak Ave, Los Angeles, CA 90001",
      "status": "Active",
      "organization": 1,
      "created_at": "2026-01-10T14:20:00Z",
      "updated_at": "2026-01-25T09:15:00Z"
    }
  ]
}
```

---

### 3.2 Get Customer

**Endpoint:** `GET /api/v1/crm/customers/{id}/`

**Authentication:** Required

**Permission:** `view_contact`

**Success Response (200 OK):**
```json
{
  "id": 1,
  "name": "Jane Smith",
  "email": "jane.smith@example.com",
  "phone": "+1234567891",
  "address": "456 Oak Ave, Los Angeles, CA 90001",
  "status": "Active",
  "organization": 1,
  "created_at": "2026-01-10T14:20:00Z",
  "updated_at": "2026-01-25T09:15:00Z"
}
```

---

### 3.3 Create Customer

**Endpoint:** `POST /api/v1/crm/customers/`

**Authentication:** Required

**Permission:** `create_contact`

**Request Body:**
```json
{
  "name": "Jane Smith",
  "email": "jane.smith@example.com",
  "phone": "+1234567891",
  "address": "456 Oak Ave, Los Angeles, CA 90001",
  "status": "Active"
}
```

**Required Fields:**
- `name` (string): Customer name

**Success Response (201 Created):** Same structure as Get Customer

---

### 3.4 Update Customer

**Endpoint:** `PUT /api/v1/crm/customers/{id}/`

**Authentication:** Required

**Permission:** `update_contact`

**Request Body:** Same as Create Customer

**Success Response (200 OK):** Same structure as Get Customer

---

### 3.5 Partial Update Customer

**Endpoint:** `PATCH /api/v1/crm/customers/{id}/`

**Authentication:** Required

**Permission:** `update_contact`

**Request Body:**
```json
{
  "phone": "+1234567999",
  "status": "Project Completed"
}
```

**Success Response (200 OK):** Same structure as Get Customer

---

### 3.6 Delete Customer

**Endpoint:** `DELETE /api/v1/crm/customers/{id}/`

**Authentication:** Required

**Permission:** `delete_contact`

**Success Response (204 No Content):** Empty response

---

## CRM - Tasks

### 4.1 List Tasks

**Endpoint:** `GET /api/v1/crm/tasks/`

**Authentication:** Required

**Description:** List all tasks with pagination and filtering.

**Query Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Items per page (default: 20)
- `search` (string, optional): Search in title, description
- `status` (string, optional): Filter by status (open, in_progress, completed, cancelled)
- `priority` (string, optional): Filter by priority (low, medium, high, critical)
- `assigned_to` (integer, optional): Filter by assigned user ID
- `task_type` (string, optional): Filter by task type

**Success Response (200 OK):**
```json
{
  "count": 50,
  "next": "http://156.67.104.149:8013/api/v1/crm/tasks/?page=2",
  "previous": null,
  "results": [
    {
      "task_id": 1,
      "title": "Follow up with client",
      "description": "Discuss project requirements",
      "task_type": "Call",
      "priority": "high",
      "status": "open",
      "assigned_to": 2,
      "assigned_by": 1,
      "due_date": "2026-02-15",
      "due_time": "14:00:00",
      "completed_on": null,
      "duration": 30,
      "content_type": 15,
      "object_id": 5,
      "meeting_from": null,
      "meeting_to": null,
      "location": null,
      "send_reminder": 1,
      "reminder_before": 15,
      "google_calendar_event_id": null,
      "is_followup": false,
      "organization": 1,
      "created_at": "2026-02-10T10:00:00Z",
      "updated_at": "2026-02-10T10:00:00Z"
    }
  ]
}
```

---

### 4.2 Get Task

**Endpoint:** `GET /api/v1/crm/tasks/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as list item

---

### 4.3 Create Task

**Endpoint:** `POST /api/v1/crm/tasks/`

**Authentication:** Required

**Request Body:**
```json
{
  "title": "Follow up with client",
  "description": "Discuss project requirements",
  "task_type": "Call",
  "priority": "high",
  "status": "open",
  "assigned_to": 2,
  "due_date": "2026-02-15",
  "due_time": "14:00:00",
  "duration": 30,
  "is_followup": true,
  "meeting_from": "2026-02-15T14:00:00Z",
  "meeting_to": "2026-02-15T14:30:00Z",
  "location": "Conference Room A",
  "send_reminder": 1,
  "reminder_before": 15
}
```

**Request Fields:**
- `title` (string, required): Task title (max 280 characters)
- `description` (text, optional): Detailed description
- `task_type` (string, required): Task, Call, Meeting, Email, Deadline
- `priority` (string, optional): low, medium, high, critical (default: medium)
- `status` (string, optional): open, in_progress, completed, cancelled (default: open)
- `assigned_to` (integer, optional): User ID to assign task to
- `due_date` (date, optional): YYYY-MM-DD format
- `due_time` (time, optional): HH:MM:SS format
- `duration` (integer, optional): Duration in minutes
- `content_type` (integer, optional): ContentType ID for polymorphic reference
- `object_id` (integer, optional): ID of related object (Lead, Customer, etc.)
- `is_followup` (boolean, optional): If true, syncs to Google Calendar
- `meeting_from` (datetime, optional): For meeting tasks
- `meeting_to` (datetime, optional): For meeting tasks
- `location` (string, optional): Meeting location (max 280 chars)
- `send_reminder` (integer, optional): 1 to enable reminders
- `reminder_before` (integer, optional): Minutes before due time to remind

**Success Response (201 Created):** Same structure as Get Task

**Note:** When `is_followup` is true and task is assigned to a user with Google Calendar sync enabled, the task is automatically added to their calendar.

---

### 4.4 Update Task

**Endpoint:** `PUT /api/v1/crm/tasks/{id}/`

**Authentication:** Required

**Request Body:** Same as Create Task (all fields required)

**Success Response (200 OK):** Same structure as Get Task

---

### 4.5 Partial Update Task

**Endpoint:** `PATCH /api/v1/crm/tasks/{id}/`

**Authentication:** Required

**Request Body:**
```json
{
  "status": "completed",
  "completed_on": "2026-02-10T16:00:00Z"
}
```

**Success Response (200 OK):** Same structure as Get Task

---

### 4.6 Delete Task

**Endpoint:** `DELETE /api/v1/crm/tasks/{id}/`

**Authentication:** Required

**Success Response (204 No Content):** Empty response

---

## CRM - Communications

### 5.1 Communication Statuses

#### 5.1.1 List Communication Statuses

**Endpoint:** `GET /api/v1/crm/communication-statuses/`

**Authentication:** Required

**Description:** List all communication statuses (for calls, emails, etc.)

**Success Response (200 OK):**
```json
{
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "crm_communication_status_id": 1,
      "status": "Completed",
      "organization": 1,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

#### 5.1.2 Create Communication Status

**Endpoint:** `POST /api/v1/crm/communication-statuses/`

**Authentication:** Required

**Request Body:**
```json
{
  "status": "No Answer"
}
```

**Success Response (201 Created):** Same structure as list item

---

### 5.2 Call Logs

#### 5.2.1 List Call Logs

**Endpoint:** `GET /api/v1/crm/call-logs/`

**Authentication:** Required

**Description:** List all call logs with pagination.

**Success Response (200 OK):**
```json
{
  "count": 120,
  "next": "http://156.67.104.149:8013/api/v1/crm/call-logs/?page=2",
  "previous": null,
  "results": [
    {
      "crm_call_log_id": 1,
      "from_number": "+1234567890",
      "to_number": "+0987654321",
      "status": 1,
      "call_type": "Outgoing",
      "duration": "00:05:30",
      "start_time": "2026-02-10T10:00:00Z",
      "end_time": "2026-02-10T10:05:30Z",
      "recording_url": "https://example.com/recording.mp3",
      "telephony_medium": "Twilio",
      "receiver": 2,
      "caller": 1,
      "content_type": 15,
      "object_id": 5,
      "organization": 1,
      "created_at": "2026-02-10T10:00:00Z",
      "updated_at": "2026-02-10T10:05:30Z"
    }
  ]
}
```

---

#### 5.2.2 Create Call Log

**Endpoint:** `POST /api/v1/crm/call-logs/`

**Authentication:** Required

**Request Body:**
```json
{
  "from_number": "+1234567890",
  "to_number": "+0987654321",
  "call_type": "Outgoing",
  "status": 1,
  "duration": "00:05:30",
  "start_time": "2026-02-10T10:00:00Z",
  "end_time": "2026-02-10T10:05:30Z",
  "telephony_medium": "Twilio",
  "receiver": 2,
  "caller": 1,
  "content_type": 15,
  "object_id": 5
}
```

**Request Fields:**
- `from_number` (string, required): Caller phone number (max 140 chars)
- `to_number` (string, required): Receiver phone number (max 140 chars)
- `call_type` (string, required): Incoming or Outgoing
- `status` (integer, optional): Communication status ID
- `duration` (duration, optional): Call duration (HH:MM:SS format)
- `start_time` (datetime, optional): Call start time
- `end_time` (datetime, optional): Call end time
- `recording_url` (url, optional): URL to call recording (max 140 chars)
- `telephony_medium` (string, optional): Manual, Twilio, or Exotel
- `receiver` (integer, optional): User ID of receiver
- `caller` (integer, optional): User ID of caller
- `content_type` (integer, optional): ContentType ID for linked entity
- `object_id` (integer, optional): ID of linked entity (Lead, Customer, etc.)

**Success Response (201 Created):** Same structure as list item

---

### 5.3 Notifications

#### 5.3.1 List Notifications

**Endpoint:** `GET /api/v1/crm/notifications/`

**Authentication:** Required

**Description:** List notifications for the current user (ordered by newest first).

**Success Response (200 OK):**
```json
{
  "count": 25,
  "next": null,
  "previous": null,
  "results": [
    {
      "crm_notification_id": 1,
      "notification_type": "Task",
      "from_user": 1,
      "to_user": 2,
      "notification_text": "New task assigned",
      "message": "<p>You have been assigned a new task: Follow up with client</p>",
      "is_read": false,
      "notification_source_content_type": 18,
      "comment": null,
      "organization": 1,
      "created_at": "2026-02-10T10:00:00Z",
      "updated_at": "2026-02-10T10:00:00Z"
    }
  ]
}
```

---

#### 5.3.2 Create Notification

**Endpoint:** `POST /api/v1/crm/notifications/`

**Authentication:** Required

**Request Body:**
```json
{
  "notification_type": "Task",
  "to_user": 2,
  "notification_text": "New task assigned",
  "message": "<p>You have been assigned a new task</p>"
}
```

**Request Fields:**
- `notification_type` (string, required): Mention, Task, Assignment, or WhatsApp
- `to_user` (integer, required): User ID to notify
- `from_user` (integer, optional): User ID sending notification
- `notification_text` (text, optional): Plain text notification
- `message` (text, optional): HTML formatted message
- `is_read` (boolean, optional): Mark as read (default: false)
- `notification_source_content_type` (integer, optional): ContentType ID
- `comment` (string, optional): Additional comment (max 140 chars)

**Success Response (201 Created):** Same structure as list item

---

#### 5.3.3 Mark Notification as Read

**Endpoint:** `PATCH /api/v1/crm/notifications/{id}/`

**Authentication:** Required

**Request Body:**
```json
{
  "is_read": true
}
```

**Success Response (200 OK):** Same structure as list item

---

## CRM - Issues

### 6.1 List Issues

**Endpoint:** `GET /api/v1/crm/issues/`

**Authentication:** Required

**Description:** List all issues with filtering, search, and ordering.

**Query Parameters:**
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page
- `search` (string, optional): Search in subject, description
- `status` (string, optional): Filter by status
- `priority` (string, optional): Filter by priority
- `issue_type` (integer, optional): Filter by issue type ID
- `assigned_to` (integer, optional): Filter by assigned user ID
- `raised_by` (integer, optional): Filter by raised by user ID
- `ordering` (string, optional): Sort field (opening_date, priority, status, first_response_time)

**Success Response (200 OK):**
```json
{
  "count": 45,
  "next": null,
  "previous": null,
  "results": [
    {
      "issue_id": 1,
      "subject": "Unable to login",
      "description": "User experiencing login issues",
      "issue_type": 1,
      "type_name": "Technical",
      "status": "Open",
      "priority": "High",
      "raised_by": 5,
      "raised_by_name": "John Doe",
      "assigned_to": 2,
      "assigned_to_name": "Support Team",
      "content_type": 15,
      "object_id": 10,
      "opening_date": "2026-02-10",
      "opening_time": "10:00:00",
      "resolution_date": null,
      "resolution_time": null,
      "first_response_time": null,
      "resolution_details": null,
      "parent_issue": null,
      "organization": 1,
      "created_at": "2026-02-10T10:00:00Z",
      "updated_at": "2026-02-10T10:00:00Z"
    }
  ]
}
```

---

### 6.2 Create Issue

**Endpoint:** `POST /api/v1/crm/issues/`

**Authentication:** Required

**Request Body:**
```json
{
  "subject": "Unable to login",
  "description": "User experiencing login issues after password reset",
  "issue_type": 1,
  "priority": "High",
  "status": "Open",
  "assigned_to": 2,
  "content_type": 15,
  "object_id": 10
}
```

**Request Fields:**
- `subject` (string, required): Issue subject (max 280 chars)
- `description` (text, optional): Detailed description
- `issue_type` (integer, optional): Issue type ID
- `status` (string, optional): Open, In Progress, On Hold, Resolved, Closed, Duplicate (default: Open)
- `priority` (string, optional): Low, Medium, High, Critical (default: Medium)
- `assigned_to` (integer, optional): User ID to assign to
- `content_type` (integer, required): ContentType ID for related entity
- `object_id` (integer, required): ID of related entity
- `parent_issue` (integer, optional): Parent issue ID for hierarchy
- `resolution_details` (text, optional): Details of resolution

**Success Response (201 Created):** Same structure as list item

**Note:** When status changes to "Closed", `first_response_time` is automatically calculated.

---

### 6.3 Issue Types

#### 6.3.1 List Issue Types

**Endpoint:** `GET /api/v1/crm/issue-types/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "issue_type_id": 1,
      "name": "Technical",
      "description": "Technical support issues",
      "default_priority": "Medium",
      "organization": 1,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

#### 6.3.2 Create Issue Type

**Endpoint:** `POST /api/v1/crm/issue-types/`

**Authentication:** Required

**Request Body:**
```json
{
  "name": "Billing",
  "description": "Billing and payment related issues",
  "default_priority": "High"
}
```

**Success Response (201 Created):** Same structure as list item

---

## CRM - Organizations

### 7.1 List CRM Organizations

**Endpoint:** `GET /api/v1/crm/crm-organizations/`

**Authentication:** Required

**Description:** List all CRM organizations (external companies/accounts).

**Success Response (200 OK):**
```json
{
  "count": 30,
  "next": null,
  "previous": null,
  "results": [
    {
      "crm_organization_id": 1,
      "organization_name": "Acme Corp",
      "website": "https://acmecorp.com",
      "organization_logo": "https://example.com/logo.png",
      "no_of_employees": 500,
      "annual_revenue": "5000000.00",
      "currency": "USD",
      "exchange_rate": 1.0,
      "industry": "Technology",
      "territory": "North America",
      "address": "123 Business St, San Francisco, CA 94103",
      "created_at": "2026-01-15T00:00:00Z",
      "updated_at": "2026-02-10T00:00:00Z"
    }
  ]
}
```

---

### 7.2 Create CRM Organization

**Endpoint:** `POST /api/v1/crm/crm-organizations/`

**Authentication:** Required

**Request Body:**
```json
{
  "organization_name": "Acme Corp",
  "website": "https://acmecorp.com",
  "no_of_employees": 500,
  "annual_revenue": "5000000.00",
  "currency": "USD",
  "industry": "Technology",
  "territory": "North America",
  "address": "123 Business St, San Francisco, CA 94103"
}
```

**Request Fields:**
- `organization_name` (string, required): Organization name (max 140 chars, unique)
- `website` (url, optional): Organization website (max 140 chars)
- `organization_logo` (url, optional): Logo URL
- `no_of_employees` (integer, optional): Number of employees
- `annual_revenue` (decimal, optional): Annual revenue (18 digits, 2 decimal places)
- `currency` (string, optional): Currency code (max 140 chars)
- `exchange_rate` (float, optional): Exchange rate (default: 1.0)
- `industry` (string, optional): Industry (max 140 chars)
- `territory` (string, optional): Territory/region (max 140 chars)
- `address` (text, optional): Full address

**Success Response (201 Created):** Same structure as list item

---

## CRM - Chat AI Assistant

### 8.1 Chat with AI

**Endpoint:** `POST /api/v1/crm/chat/`

**Authentication:** Required

**Description:** Send natural language queries to the AI assistant to retrieve CRM data.

**Request Body:**
```json
{
  "message": "Show me leads from the last 7 days"
}
```

**Request Fields:**
- `message` (string, required): Natural language query (1-2000 characters)

**Success Response (200 OK):**
```json
{
  "response": "Here are the leads from the last 7 days:\n\n| Lead Name | Email | Status | Created At |\n|-----------|-------|--------|------------|\n| John Doe | john@example.com | New | 2026-02-09 |\n| Jane Smith | jane@example.com | Qualified | 2026-02-08 |"
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Message is required"
}
```

**Error Response (500 Internal Server Error):**
```json
{
  "error": "AI service temporarily unavailable"
}
```

**Example Queries:**
- "Show me leads from the last 7 days"
- "Get leads created in the past month"
- "List recent leads from last 2 weeks"
- "Show customers with high priority"
- "Get open tasks assigned to me"

**Features:**
- **AI-Powered**: Uses Google Gemini with automatic function calling
- **Smart Formatting**: Returns tables for lists, formatted text for other responses
- **Context-Aware**: Automatically uses your organization context
- **Multi-tenancy**: Queries are scoped to your organization

---

## Projects Management

### 9.1 Project Types

#### 9.1.1 List Project Types

**Endpoint:** `GET /api/v1/projects/project-types/`

**Authentication:** Required

**Description:** List all project types.

**Success Response (200 OK):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "project_type_id": 1,
      "project_type": "Internal",
      "description": "Internal company projects",
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

#### 9.1.2 Create Project Type

**Endpoint:** `POST /api/v1/projects/project-types/`

**Request Body:**
```json
{
  "project_type": "Internal",
  "description": "Internal company projects"
}
```

**Success Response (201 Created):** Same structure as list item

---

### 9.2 Projects

#### 9.2.1 List Projects

**Endpoint:** `GET /api/v1/projects/projects/`

**Authentication:** Required

**Description:** List all projects with comprehensive details.

**Success Response (200 OK):**
```json
{
  "count": 20,
  "next": null,
  "previous": null,
  "results": [
    {
      "project_id": 1,
      "project_name": "Website Redesign",
      "naming_series": "PROJ-.####",
      "status": "Open",
      "project_type": 1,
      "is_active": "Yes",
      "percent_complete_method": "Task Completion",
      "percent_complete": "45.50",
      "priority": "High",
      "expected_start_date": "2026-02-01",
      "expected_end_date": "2026-04-30",
      "actual_start_date": "2026-02-01",
      "actual_end_date": null,
      "customer": 5,
      "company": 1,
      "estimated_costing": "50000.00",
      "total_costing_amount": "0.00",
      "created_at": "2026-02-01T00:00:00Z",
      "updated_at": "2026-02-10T10:00:00Z"
    }
  ]
}
```

#### 9.2.2 Create Project

**Endpoint:** `POST /api/v1/projects/projects/`

**Request Body:**
```json
{
  "project_name": "Website Redesign",
  "status": "Open",
  "project_type": 1,
  "priority": "High",
  "expected_start_date": "2026-02-01",
  "expected_end_date": "2026-04-30",
  "customer": 5,
  "estimated_costing": "50000.00",
  "percent_complete_method": "Task Completion"
}
```

**Request Fields:**
- `project_name` (string, required): Unique project name (max 140 chars)
- `naming_series` (string, optional): Default: "PROJ-.####"
- `status` (string, optional): Open, Completed, Cancelled (default: Open)
- `project_type` (integer, optional): Project type ID
- `is_active` (string, optional): Yes or No (default: Yes)
- `percent_complete_method` (string, optional): Manual, Task Completion, Task Progress, Task Weight
- `priority` (string, optional): Low, Medium, High
- `expected_start_date` (date, optional): YYYY-MM-DD format
- `expected_end_date` (date, optional): YYYY-MM-DD format
- `customer` (integer, optional): Customer ID
- `estimated_costing` (decimal, optional): Estimated cost
- `collect_progress` (boolean, optional): Enable progress collection
- `frequency` (string, optional): Hourly, Twice Daily, Daily, Weekly
- `subject` (string, optional): Email subject for progress reports (max 140 chars)
- `message` (text, optional): Email message for progress reports

**Success Response (201 Created):** Same structure as list item

**Read-only Fields:**
- `project_id`, `company`, `percent_complete`, `actual_start_date`, `actual_end_date`, `actual_time`, `total_costing_amount`, `total_purchase_cost`, `total_sales_amount`, `total_billable_amount`, `total_billed_amount`, `total_consumed_material_cost`, `gross_margin`, `per_gross_margin`

---

### 9.3 Task Groups

#### 9.3.1 List Task Groups

**Endpoint:** `GET /api/v1/projects/task-groups/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "task_group_id": 1,
      "title": "Development Phase",
      "project": 1,
      "created_at": "2026-02-01T00:00:00Z",
      "updated_at": "2026-02-01T00:00:00Z"
    }
  ]
}
```

#### 9.3.2 Create Task Group

**Endpoint:** `POST /api/v1/projects/task-groups/`

**Request Body:**
```json
{
  "title": "Development Phase",
  "project": 1
}
```

**Success Response (201 Created):** Same structure as list item

---

### 9.4 Project Task Statuses

#### 9.4.1 List Project Task Statuses

**Endpoint:** `GET /api/v1/projects/project-task-statuses/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 6,
  "next": null,
  "previous": null,
  "results": [
    {
      "project_task_status_id": 1,
      "status": "Open",
      "description": "Task is open and ready to work",
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### 9.5 Task Types

#### 9.5.1 List Task Types

**Endpoint:** `GET /api/v1/projects/task-types/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "task_type_id": 1,
      "name": "Development",
      "weight": "1.00",
      "description": "Development tasks",
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### 9.6 Project Tasks

#### 9.6.1 List Tasks

**Endpoint:** `GET /api/v1/projects/tasks/`

**Authentication:** Required

**Description:** List all project tasks with hierarchical structure.

**Success Response (200 OK):**
```json
{
  "count": 50,
  "next": null,
  "previous": null,
  "results": [
    {
      "task_id": 1,
      "subject": "Design Database Schema",
      "project": 1,
      "task_group": 1,
      "type": 1,
      "status": 2,
      "priority": "High",
      "task_weight": "2.00",
      "parent_task": null,
      "is_group": false,
      "is_template": false,
      "is_milestone": false,
      "description": "Design the complete database schema",
      "exp_start_date": "2026-02-01T00:00:00Z",
      "exp_end_date": "2026-02-05T00:00:00Z",
      "expected_time": "40.00",
      "progress": "75.00",
      "children": [
        {
          "task_id": 2,
          "subject": "Create ER Diagram",
          "parent_task": 1,
          "progress": "100.00",
          "children": []
        }
      ],
      "created_at": "2026-02-01T00:00:00Z",
      "updated_at": "2026-02-10T00:00:00Z"
    }
  ]
}
```

#### 9.6.2 Create Task

**Endpoint:** `POST /api/v1/projects/tasks/`

**Request Body:**
```json
{
  "subject": "Design Database Schema",
  "project": 1,
  "task_group": 1,
  "type": 1,
  "status": 1,
  "priority": "High",
  "parent_task": null,
  "exp_start_date": "2026-02-01T00:00:00Z",
  "exp_end_date": "2026-02-05T00:00:00Z",
  "expected_time": "40.00",
  "description": "Design the complete database schema"
}
```

**Request Fields:**
- `subject` (string, required): Task subject (max 140 chars)
- `project` (integer, optional): Project ID
- `task_group` (integer, optional): Task group ID
- `type` (integer, optional): Task type ID
- `status` (integer, optional): Task status ID
- `priority` (string, optional): Low, Medium, High, Urgent (default: Medium)
- `task_weight` (decimal, optional): Weight for calculations
- `parent_task` (integer, optional): Parent task ID for hierarchy
- `is_group` (boolean, optional): Is this a group task
- `is_template` (boolean, optional): Is this a template
- `is_milestone` (boolean, optional): Is this a milestone
- `description` (text, optional): Detailed description
- `exp_start_date` (datetime, optional): Expected start date/time
- `exp_end_date` (datetime, optional): Expected end date/time
- `expected_time` (decimal, optional): Expected hours
- `progress` (decimal, optional): Progress percentage (0-100)

**Success Response (201 Created):** Same structure as list item (includes children)

---

### 9.7 Task Dependencies

#### 9.7.1 List Task Dependencies

**Endpoint:** `GET /api/v1/projects/task-dependencies/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 15,
  "next": null,
  "previous": null,
  "results": [
    {
      "task_depends_on_id": 1,
      "parent": 3,
      "task": 1,
      "subject": "Design Database Schema",
      "project": 1,
      "created_at": "2026-02-01T00:00:00Z",
      "updated_at": "2026-02-01T00:00:00Z"
    }
  ]
}
```

**Note:** `parent` is the task that depends on `task`. Meaning: "parent depends on task" or "task must be completed before parent can start".

#### 9.7.2 Create Task Dependency

**Endpoint:** `POST /api/v1/projects/task-dependencies/`

**Request Body:**
```json
{
  "parent": 3,
  "task": 1
}
```

**Request Fields:**
- `parent` (integer, required): Task ID that has the dependency
- `task` (integer, required): Task ID that must be completed first

**Success Response (201 Created):** Same structure as list item

---

### 9.8 Project Updates

#### 9.8.1 List Project Updates

**Endpoint:** `GET /api/v1/projects/project-updates/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "project_update_id": 1,
      "naming_series": "PROJ-UPD-.2026.-",
      "project": 1,
      "user": 2,
      "docstatus": 1,
      "sent": true,
      "date": "2026-02-10",
      "time": "10:00:00",
      "created_at": "2026-02-10T10:00:00Z",
      "updated_at": "2026-02-10T10:00:00Z"
    }
  ]
}
```

#### 9.8.2 Create Project Update

**Endpoint:** `POST /api/v1/projects/project-updates/`

**Request Body:**
```json
{
  "project": 1,
  "user": 2,
  "docstatus": 0
}
```

**Request Fields:**
- `project` (integer, required): Project ID
- `user` (integer, optional): User ID to send update to
- `docstatus` (integer, optional): 0=Draft, 1=Submitted, 2=Cancelled (default: 0)

**Success Response (201 Created):** Same structure as list item

---

## Email Management

### 10.1 Email Configurations

#### 10.1.1 List Email Configurations

**Endpoint:** `GET /api/v1/emails/configs/`

**Authentication:** Required

**Description:** List all email configurations accessible to the current user (personal and organizational).

**Success Response (200 OK):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "label": "My Work Email",
      "email_address": "user@example.com",
      "provider": "gmail",
      "is_active": true,
      "is_default": false,
      "owner_type": "PERSONAL",
      "is_token_valid": true,
      "last_used_at": "2026-02-10T09:30:00Z",
      "created_at": "2026-02-01T00:00:00Z",
      "updated_at": "2026-02-10T09:30:00Z"
    }
  ]
}
```

---

#### 10.1.2 Get Email Configuration

**Endpoint:** `GET /api/v1/emails/configs/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as list item

---

#### 10.1.3 Create Email Configuration

**Endpoint:** `POST /api/v1/emails/configs/`

**Authentication:** Required

**Description:** Manually create an email configuration (for non-OAuth providers like SMTP).

**Request Body:**
```json
{
  "label": "Company SMTP",
  "email_address": "noreply@company.com",
  "provider": "smtp",
  "owner_type": "organization",
  "is_active": true,
  "is_default": false
}
```

**Request Fields:**
- `label` (string, required): Friendly label for this configuration (max 100 chars)
- `email_address` (email, required): Email address
- `provider` (string, required): gmail, outlook, smtp
- `owner_type` (string, required): user or organization
- `is_active` (boolean, optional): Enable/disable configuration (default: true)
- `is_default` (boolean, optional): Set as default (default: false)

**Success Response (201 Created):** Same structure as list item

---

#### 10.1.4 Update Email Configuration

**Endpoint:** `PATCH /api/v1/emails/configs/{id}/`

**Authentication:** Required

**Request Body:**
```json
{
  "label": "Updated Label",
  "is_active": false
}
```

**Success Response (200 OK):** Same structure as list item

---

#### 10.1.5 Delete Email Configuration

**Endpoint:** `DELETE /api/v1/emails/configs/{id}/`

**Authentication:** Required

**Success Response (204 No Content):** Empty response

---

### 10.2 OAuth Flow

#### 10.2.1 Initiate OAuth

**Endpoint:** `POST /api/v1/emails/connect/`

**Authentication:** Required

**Description:** Start OAuth flow to connect a Gmail account.

**Request Body:**
```json
{
  "owner_type": "user",
  "label": "My Gmail"
}
```

**Request Fields:**
- `owner_type` (string, required): user or organization
- `label` (string, optional): Label for the configuration (default: "My Email", max 100 chars)

**Success Response (200 OK):**
```json
{
  "authorization_url": "https://accounts.google.com/o/oauth2/v2/auth?client_id=..."
}
```

**Usage:** Redirect the user to the `authorization_url` to complete Google OAuth consent.

---

#### 10.2.2 OAuth Callback

**Endpoint:** `GET /api/v1/emails/callback/`

**Authentication:** None (Public callback)

**Description:** Handle OAuth callback from Google. This endpoint is called by Google after user authorization.

**Query Parameters:**
- `code` (string, required): Authorization code from Google
- `state` (string, required): State parameter for CSRF validation
- `error` (string, optional): Error if authorization failed

**Success Response (200 OK):**
```json
{
  "message": "Email connected successfully",
  "email": "user@gmail.com",
  "config_id": 5
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Authorization failed: access_denied"
}
```

---

### 10.3 Available Email Accounts

**Endpoint:** `GET /api/v1/emails/available/`

**Authentication:** Required

**Description:** Get all email accounts available to the current user for sending emails.

**Success Response (200 OK):**
```json
[
  {
    "id": 1,
    "label": "My Work Email",
    "email_address": "user@example.com",
    "type": "PERSONAL",
    "owner_name": "John Doe"
  },
  {
    "id": 2,
    "label": "Company Support",
    "email_address": "support@company.com",
    "type": "ORGANIZATION",
    "owner_name": "Demo Organization"
  }
]
```

---

### 10.4 Send Email

**Endpoint:** `POST /api/v1/emails/send/`

**Authentication:** Required

**Description:** Send an email using a connected Gmail account.

**Request Body:**
```json
{
  "config_id": 1,
  "to": ["recipient@example.com"],
  "cc": ["cc@example.com"],
  "bcc": ["bcc@example.com"],
  "subject": "Project Update",
  "body_text": "Here is the project update...",
  "body_html": "<p>Here is the <strong>project update</strong>...</p>"
}
```

**Request Fields:**
- `config_id` (integer, required): Email configuration ID to use
- `to` (array, required): List of recipient email addresses (min 1)
- `cc` (array, optional): List of CC email addresses
- `bcc` (array, optional): List of BCC email addresses
- `subject` (string, required): Email subject (max 500 chars)
- `body_text` (string, required): Plain text email body
- `body_html` (string, optional): HTML email body

**Success Response (200 OK):**
```json
{
  "message": "Email sent successfully",
  "message_id": "18d5c8f7a1b2c3d4"
}
```

**Error Responses:**

**404 Not Found:**
```json
{
  "error": "Email configuration not found"
}
```

**400 Bad Request (Token Expired):**
```json
{
  "error": "Access token expired and no refresh token available. Please reconnect the email."
}
```

**500 Internal Server Error:**
```json
{
  "error": "Failed to send email"
}
```

**Features:**
- **Automatic Token Refresh**: Tokens are automatically refreshed if expired
- **Multi-tenancy**: Access control based on user and organization
- **Usage Tracking**: `last_used_at` timestamp updated on successful send
- **Validation**: Ensures user has access to the selected email configuration

---

## User Management

### 11.1 List Users

**Endpoint:** `GET /api/v1/management/users/`

**Authentication:** Required

**Permission:** `view_user_management`

**Description:** List all users in the organization.

**Query Parameters:**
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page
- `search` (string, optional): Search by username, email, first name, last name, user_id
- `is_active` (boolean, optional): Filter by active status
- `is_staff` (boolean, optional): Filter by staff status
- `ordering` (string, optional): Sort field

**Success Response (200 OK):**
```json
{
  "count": 25,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "user_id": "admin@demoorg.com",
      "email": "admin@demoorg.com",
      "username": "admin@demoorg.com",
      "first_name": "Admin",
      "last_name": "User",
      "full_name": "Admin User",
      "salutation": "Mr.",
      "is_active": true,
      "is_staff": false,
      "is_superuser": false,
      "last_login": "2026-01-26T08:00:00Z",
      "date_joined": "2026-01-01T00:00:00Z",
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-26T08:00:00Z",
      "org_id": 1,
      "profile": {
        "phone": "+1234567890",
        "mobile": "+1234567890",
        "user_image": null,
        "timezone": "UTC",
        "language": "en",
        "gender": null,
        "birth_date": null,
        "location": null,
        "bio": null,
        "preferences": null
      }
    }
  ]
}
```

---

### 11.2 Get User

**Endpoint:** `GET /api/v1/management/users/{id}/`

**Authentication:** Required

**Permission:** `view_user_management`

**Success Response (200 OK):** Same structure as list item

---

### 11.3 Create User

**Endpoint:** `POST /api/v1/management/users/`

**Authentication:** Required

**Permission:** `create_user_management`

**Request Body:**
```json
{
  "email": "newuser@example.com",
  "username": "newuser@example.com",
  "first_name": "New",
  "last_name": "User",
  "password": "SecurePassword123!",
  "salutation": "Ms.",
  "is_active": true,
  "is_staff": false
}
```

**Success Response (201 Created):** Same structure as Get User

---

### 11.4 Update User

**Endpoint:** `PUT /api/v1/management/users/{id}/`

**Authentication:** Required

**Permission:** `update_user_management`

**Success Response (200 OK):** Same structure as Get User

---

### 11.5 Partial Update User

**Endpoint:** `PATCH /api/v1/management/users/{id}/`

**Authentication:** Required

**Permission:** `update_user_management`

**Request Body:**
```json
{
  "is_active": false,
  "last_name": "Updated"
}
```

**Success Response (200 OK):** Same structure as Get User

---

### 11.6 Delete User

**Endpoint:** `DELETE /api/v1/management/users/{id}/`

**Authentication:** Required

**Permission:** `delete_user_management`

**Success Response (204 No Content):** Empty response

---

### 11.7 Assign User Roles

**Endpoint:** `POST /api/v1/management/users/{user_id}/roles/`

**Authentication:** Required

**Permission:** `manage_roles`

**Description:** Assign roles to a user with priorities.

**Request Body:**
```json
{
  "role_ids": [1, 2, 3],
  "priorities": [1, 2, 3]
}
```

**Request Fields:**
- `role_ids` (array, required): Array of role IDs
- `priorities` (array, optional): Array of priorities (same length as role_ids)

**Success Response (200 OK):**
```json
{
  "message": "Roles assigned successfully",
  "user_id": 5,
  "roles": [
    {
      "role_id": 1,
      "role_name": "Sales Manager",
      "priority": 1
    },
    {
      "role_id": 2,
      "role_name": "Team Lead",
      "priority": 2
    }
  ]
}
```

---

### 11.8 Get User Effective Permissions

**Endpoint:** `GET /api/v1/management/users/{user_id}/permissions/`

**Authentication:** Required

**Permission:** `view_user_management`

**Description:** Get all effective permissions for a user (aggregated from all roles).

**Success Response (200 OK):**
```json
{
  "user_id": 5,
  "roles": [
    {
      "role_id": 1,
      "role_name": "Sales Manager",
      "priority": 1
    }
  ],
  "module_permissions": {
    "lead": {
      "can_read": true,
      "can_create": true,
      "can_update": true,
      "can_delete": true,
      "can_export": true,
      "can_import": false
    },
    "contact": {
      "can_read": true,
      "can_create": true,
      "can_update": true,
      "can_delete": false,
      "can_export": true,
      "can_import": false
    }
  },
  "record_permissions": {
    "lead": {
      "access_level": "all",
      "can_view": true,
      "can_edit": true,
      "can_delete": true
    }
  }
}
```

---

## Role Management

### 12.1 List Roles

**Endpoint:** `GET /api/v1/management/roles/`

**Authentication:** Required

**Permission:** `view_role`

**Query Parameters:**
- `page` (integer, optional): Page number
- `is_active` (boolean, optional): Filter by active status
- `is_custom` (boolean, optional): Filter by custom roles
- `permission_level` (integer, optional): Filter by permission level
- `search` (string, optional): Search by name, description

**Success Response (200 OK):**
```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "role_id": 1,
      "org_id": 1,
      "name": "Sales Manager",
      "description": "Manages sales team and operations",
      "is_custom": true,
      "is_active": true,
      "permission_level": 50,
      "created_by": 1,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z",
      "updated_by": 1,
      "module_permissions": [
        {
          "id": 1,
          "module_name": "lead",
          "can_read": true,
          "can_create": true,
          "can_update": true,
          "can_delete": false,
          "can_export": true,
          "can_import": false
        }
      ],
      "record_permissions": [
        {
          "id": 1,
          "module_name": "lead",
          "access_level": "team",
          "can_view": true,
          "can_edit": true,
          "can_delete": false
        }
      ]
    }
  ]
}
```

---

### 12.2 Get Role

**Endpoint:** `GET /api/v1/management/roles/{id}/`

**Authentication:** Required

**Permission:** `view_role`

**Success Response (200 OK):** Same structure as list item

---

### 12.3 Create Role

**Endpoint:** `POST /api/v1/management/roles/`

**Authentication:** Required

**Permission:** `create_role`

**Request Body:**
```json
{
  "name": "Custom Sales Role",
  "description": "Custom role for sales representatives",
  "is_custom": true,
  "is_active": true,
  "permission_level": 40
}
```

**Request Fields:**
- `name` (string, required): Role name (unique within organization)
- `description` (string, optional): Role description
- `is_custom` (boolean, default: true): Whether role is custom
- `is_active` (boolean, default: true): Whether role is active
- `permission_level` (integer, default: 0): Permission hierarchy level (0-100)

**Success Response (201 Created):** Same structure as Get Role

---

### 12.4 Update Role

**Endpoint:** `PUT /api/v1/management/roles/{id}/`

**Authentication:** Required

**Permission:** `update_role`

**Success Response (200 OK):** Same structure as Get Role

---

### 12.5 Delete Role

**Endpoint:** `DELETE /api/v1/management/roles/{id}/`

**Authentication:** Required

**Permission:** `delete_role`

**Success Response (204 No Content):** Empty response

---

## Team Management

### 13.1 List Teams

**Endpoint:** `GET /api/v1/management/teams/`

**Authentication:** Required

**Permission:** `view_team`

**Query Parameters:**
- `page` (integer, optional): Page number
- `search` (string, optional): Search by name, description
- `manager` (integer, optional): Filter by manager ID

**Success Response (200 OK):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "team_id": 1,
      "org_id": 1,
      "name": "Sales Team",
      "description": "Primary sales team",
      "manager_id": 1,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z",
      "deleted_at": null,
      "members": [
        {
          "user_id": 2,
          "full_name": "John Doe",
          "email": "john@example.com",
          "joined_at": "2026-01-05T00:00:00Z"
        }
      ]
    }
  ]
}
```

---

### 13.2 Get Team

**Endpoint:** `GET /api/v1/management/teams/{id}/`

**Authentication:** Required

**Permission:** `view_team`

**Success Response (200 OK):** Same structure as list item

---

### 13.3 Create Team

**Endpoint:** `POST /api/v1/management/teams/`

**Authentication:** Required

**Permission:** `create_team`

**Request Body:**
```json
{
  "name": "Marketing Team",
  "description": "Marketing and communications",
  "manager_id": 1
}
```

**Success Response (201 Created):** Same structure as Get Team

---

### 13.4 Update Team

**Endpoint:** `PUT /api/v1/management/teams/{id}/`

**Authentication:** Required

**Permission:** `update_team`

**Success Response (200 OK):** Same structure as Get Team

---

### 13.5 Delete Team

**Endpoint:** `DELETE /api/v1/management/teams/{id}/`

**Authentication:** Required

**Permission:** `delete_team`

**Success Response (204 No Content):** Empty response

---

### 13.6 List Team Members

**Endpoint:** `GET /api/v1/management/teams/{team_id}/members/`

**Authentication:** Required

**Permission:** `view_team_members`

**Success Response (200 OK):**
```json
[
  {
    "user_id": 2,
    "full_name": "John Doe",
    "email": "john@example.com",
    "team_id": 1,
    "joined_at": "2026-01-05T00:00:00Z"
  }
]
```

---

### 13.7 Add Team Member

**Endpoint:** `POST /api/v1/management/teams/{team_id}/members/`

**Authentication:** Required

**Permission:** `manage_team_members`

**Request Body:**
```json
{
  "user_id": 5
}
```

**Success Response (201 Created):**
```json
{
  "message": "User added to team successfully",
  "user_id": 5,
  "team_id": 1,
  "joined_at": "2026-01-26T10:30:00Z"
}
```

---

### 13.8 Remove Team Member

**Endpoint:** `DELETE /api/v1/management/teams/{team_id}/members/{user_id}/`

**Authentication:** Required

**Permission:** `manage_team_members`

**Success Response (204 No Content):** Empty response

---

## Hierarchy Management

### 14.1 List Hierarchies

**Endpoint:** `GET /api/v1/management/hierarchy/`

**Authentication:** Required

**Permission:** `view_hierarchy`

**Query Parameters:**
- `user` (integer, optional): Filter by user ID
- `manager` (integer, optional): Filter by manager ID
- `level` (integer, optional): Filter by hierarchy level
- `effective_from` (date, optional): Filter by effective from date
- `effective_to` (date, optional): Filter by effective to date

**Success Response (200 OK):**
```json
{
  "count": 20,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": 2,
      "manager": 1,
      "org_id": 1,
      "level": 2,
      "effective_from": "2026-01-01",
      "effective_to": null,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### 14.2 Get Hierarchy Tree

**Endpoint:** `GET /api/v1/management/hierarchy/tree/`

**Authentication:** Required

**Permission:** `view_hierarchy`

**Query Parameters:**
- `user_id` (integer, optional): Get tree for specific user (defaults to entire organization)

**Success Response (200 OK):**
```json
{
  "tree": [
    {
      "user_id": 1,
      "full_name": "Admin User",
      "email": "admin@example.com",
      "level": 1,
      "subordinates": [
        {
          "user_id": 2,
          "full_name": "Manager One",
          "email": "manager1@example.com",
          "level": 2,
          "subordinates": [
            {
              "user_id": 5,
              "full_name": "Team Member",
              "email": "member@example.com",
              "level": 3,
              "subordinates": []
            }
          ]
        }
      ]
    }
  ]
}
```

---

### 14.3 Get User Subordinates

**Endpoint:** `GET /api/v1/management/hierarchy/subordinates/`

**Authentication:** Required

**Permission:** `view_hierarchy`

**Query Parameters:**
- `user_id` (integer, required): User ID to get subordinates for

**Success Response (200 OK):**
```json
{
  "user_id": 1,
  "subordinates": [
    {
      "user_id": 2,
      "full_name": "Manager One",
      "email": "manager1@example.com",
      "level": 2,
      "direct_report": true
    },
    {
      "user_id": 5,
      "full_name": "Team Member",
      "email": "member@example.com",
      "level": 3,
      "direct_report": false
    }
  ]
}
```

---

### 14.4 Set User Manager

**Endpoint:** `PATCH /api/v1/management/users/{user_id}/manager/`

**Authentication:** Required

**Permission:** `manage_hierarchy`

**Description:** Set or update the manager for a user. Prevents circular hierarchies and enforces max depth of 10 levels.

**Request Body:**
```json
{
  "manager_id": 1
}
```

**Success Response (200 OK):**
```json
{
  "message": "Manager for user John Doe updated successfully.",
  "new_hierarchy": {
    "id": 5,
    "user": 2,
    "manager": 1,
    "org_id": 1,
    "level": 2,
    "effective_from": "2026-01-26",
    "effective_to": null
  }
}
```

**Error Responses:**

**400 Bad Request (Circular Reference):**
```json
{
  "error": "This would create a circular hierarchy"
}
```

**400 Bad Request (Max Depth):**
```json
{
  "error": "Hierarchy depth cannot exceed 10 levels"
}
```

**400 Bad Request (Self Management):**
```json
{
  "error": "User cannot be their own manager"
}
```

---

### 14.5 Update Hierarchy

**Endpoint:** `PUT /api/v1/management/hierarchy/{id}/`

**Authentication:** Required

**Permission:** `manage_hierarchy`

**Request Body:**
```json
{
  "user": 2,
  "manager": 1,
  "level": 2,
  "effective_from": "2026-01-01",
  "effective_to": null
}
```

**Success Response (200 OK):** Same structure as Get Hierarchy

---

## Permissions Management

### 15.1 Module Permissions

#### 15.1.1 List Module Permissions

**Endpoint:** `GET /api/v1/management/permissions/modules/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Query Parameters:**
- `role` (integer, optional): Filter by role ID
- `module_name` (string, optional): Filter by module name
- `page` (integer, optional): Page number

**Success Response (200 OK):**
```json
{
  "count": 15,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "role": 1,
      "org_id": 1,
      "module_name": "lead",
      "can_read": true,
      "can_create": true,
      "can_update": true,
      "can_delete": false,
      "can_export": true,
      "can_import": false,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

#### 15.1.2 Get Module Permission

**Endpoint:** `GET /api/v1/management/permissions/modules/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (200 OK):** Same structure as list item

---

#### 15.1.3 Create Module Permission

**Endpoint:** `POST /api/v1/management/permissions/modules/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Request Body:**
```json
{
  "role": 1,
  "module_name": "lead",
  "can_read": true,
  "can_create": true,
  "can_update": true,
  "can_delete": false,
  "can_export": true,
  "can_import": false
}
```

**Module Name Options:**
- `lead`, `deal`, `contact`, `organization_contact`, `product`, `report`, `user_management`, `role`, `organization`, `hierarchy`, `team`, `settings`

**Success Response (201 Created):** Same structure as Get Module Permission

---

#### 15.1.4 Update Module Permission

**Endpoint:** `PUT /api/v1/management/permissions/modules/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (200 OK):** Same structure as Get Module Permission

---

#### 15.1.5 Delete Module Permission

**Endpoint:** `DELETE /api/v1/management/permissions/modules/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (204 No Content):** Empty response

---

#### 15.1.6 Get Permissions By Role

**Endpoint:** `GET /api/v1/management/permissions/modules/by_role/?role_id={role_id}`

**Authentication:** Required

**Permission:** `manage_permissions`

**Query Parameters:**
- `role_id` (integer, required): Role ID

**Success Response (200 OK):**
```json
[
  {
    "id": 1,
    "role": 1,
    "module_name": "lead",
    "can_read": true,
    "can_create": true,
    "can_update": true,
    "can_delete": false,
    "can_export": true,
    "can_import": false
  }
]
```

---

### 15.2 Record Permissions

#### 15.2.1 List Record Permissions

**Endpoint:** `GET /api/v1/management/permissions/records/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (200 OK):**
```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "role": 1,
      "org_id": 1,
      "module_name": "lead",
      "access_level": "team",
      "can_view": true,
      "can_edit": true,
      "can_delete": false,
      "filter_conditions": {
        "status": "open"
      },
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

#### 15.2.2 Get Record Permission

**Endpoint:** `GET /api/v1/management/permissions/records/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (200 OK):** Same structure as list item

---

#### 15.2.3 Create Record Permission

**Endpoint:** `POST /api/v1/management/permissions/records/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Request Body:**
```json
{
  "role": 1,
  "module_name": "lead",
  "access_level": "team",
  "can_view": true,
  "can_edit": true,
  "can_delete": false,
  "filter_conditions": {
    "status": "open",
    "territory": "north"
  }
}
```

**Access Level Options:**
- `own`: Only records owned by the user
- `team`: Records owned by user's team members
- `hierarchy`: Records owned by user and their subordinates
- `all`: All records in organization
- `custom`: Custom filter conditions

**Success Response (201 Created):** Same structure as Get Record Permission

---

#### 15.2.4 Update Record Permission

**Endpoint:** `PUT /api/v1/management/permissions/records/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (200 OK):** Same structure as Get Record Permission

---

#### 15.2.5 Delete Record Permission

**Endpoint:** `DELETE /api/v1/management/permissions/records/{id}/`

**Authentication:** Required

**Permission:** `manage_permissions`

**Success Response (204 No Content):** Empty response

---

## Organization Management

### 16.1 List Organizations

**Endpoint:** `GET /api/v1/organizations/manage/`

**Authentication:** Required

**Permission:** Admin only

**Success Response (200 OK):**
```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Demo Organization",
      "subdomain": "demo-org",
      "logo": "https://example.com/logo.png",
      "website": "https://demoorg.com",
      "industry": "Technology",
      "employee_count": "50-100",
      "annual_revenue": "5000000.00",
      "currency": "USD",
      "address": "123 Business St",
      "city": "San Francisco",
      "state": "CA",
      "country": "USA",
      "postal_code": "94103",
      "settings": {
        "timezone": "America/Los_Angeles",
        "date_format": "MM/DD/YYYY"
      },
      "is_active": true,
      "plan": "Enterprise",
      "plan_end_date": "2027-01-01T00:00:00Z",
      "license_count": 100,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-20T10:00:00Z"
    }
  ]
}
```

---

### 16.2 Get Organization

**Endpoint:** `GET /api/v1/organizations/manage/{id}/`

**Authentication:** Required

**Permission:** Admin or organization member

**Success Response (200 OK):** Same structure as list item

---

### 16.3 Update Organization

**Endpoint:** `PATCH /api/v1/organizations/manage/{id}/`

**Authentication:** Required

**Permission:** `update_organization`

**Request Body:**
```json
{
  "name": "Updated Organization Name",
  "website": "https://newwebsite.com",
  "industry": "SaaS",
  "employee_count": "100-500",
  "settings": {
    "timezone": "UTC",
    "date_format": "YYYY-MM-DD"
  }
}
```

**Success Response (200 OK):** Same structure as Get Organization

---

## Access Control

### 17.1 User Data Filters

#### 17.1.1 List User Filters

**Endpoint:** `GET /api/v1/access-control/user-filters/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": 1,
      "org_id": 1,
      "module_name": "lead",
      "name": "My Open Leads",
      "filter_conditions": {
        "status": "open",
        "owner": "me"
      },
      "is_default": true,
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

#### 17.1.2 Get User Filter

**Endpoint:** `GET /api/v1/access-control/user-filters/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as list item

---

#### 17.1.3 Create User Filter

**Endpoint:** `POST /api/v1/access-control/user-filters/`

**Authentication:** Required

**Request Body:**
```json
{
  "module_name": "lead",
  "name": "High Priority Leads",
  "filter_conditions": {
    "lead_score": {"gte": 75},
    "status": "qualified"
  },
  "is_default": false
}
```

**Success Response (201 Created):** Same structure as Get User Filter

---

#### 17.1.4 Update User Filter

**Endpoint:** `PUT /api/v1/access-control/user-filters/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as Get User Filter

---

#### 17.1.5 Delete User Filter

**Endpoint:** `DELETE /api/v1/access-control/user-filters/{id}/`

**Authentication:** Required

**Success Response (204 No Content):** Empty response

---

### 17.2 Shared Records

#### 17.2.1 List Shared Records

**Endpoint:** `GET /api/v1/management/shared-records/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "org_id": 1,
      "module_name": "lead",
      "record_id": 5,
      "shared_by": 1,
      "shared_with_user": 2,
      "shared_with_team": null,
      "permission_level": "view",
      "can_reshare": false,
      "created_at": "2026-01-20T10:00:00Z"
    }
  ]
}
```

---

#### 17.2.2 Create Shared Record

**Endpoint:** `POST /api/v1/management/shared-records/`

**Authentication:** Required

**Request Body:**
```json
{
  "module_name": "lead",
  "record_id": 5,
  "shared_with_user": 2,
  "permission_level": "edit",
  "can_reshare": false
}
```

**Permission Levels:**
- `view`: Can only view the record
- `edit`: Can view and edit the record
- `full`: Can view, edit, and delete the record

**Success Response (201 Created):**
```json
{
  "id": 2,
  "org_id": 1,
  "module_name": "lead",
  "record_id": 5,
  "shared_by": 1,
  "shared_with_user": 2,
  "shared_with_team": null,
  "permission_level": "edit",
  "can_reshare": false,
  "created_at": "2026-01-26T10:30:00Z"
}
```

---

#### 17.2.3 Delete Shared Record

**Endpoint:** `DELETE /api/v1/management/shared-records/{id}/`

**Authentication:** Required

**Description:** Remove shared access to a record.

**Success Response (204 No Content):** Empty response

---

### 17.3 User View Settings

#### 17.3.1 List View Settings

**Endpoint:** `GET /api/v1/management/user-view-settings/`

**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": 1,
      "org_id": 1,
      "module_name": "lead",
      "view_name": "list",
      "settings": {
        "columns": [
          {
            "name": "name",
            "visible": true,
            "width": 200,
            "order": 1
          },
          {
            "name": "email",
            "visible": true,
            "width": 250,
            "order": 2
          },
          {
            "name": "status",
            "visible": true,
            "width": 150,
            "order": 3
          }
        ],
        "sort_by": "created_at",
        "sort_order": "desc",
        "page_size": 25
      },
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-01-20T15:00:00Z"
    }
  ]
}
```

---

#### 17.3.2 Get View Setting

**Endpoint:** `GET /api/v1/management/user-view-settings/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as list item

---

#### 17.3.3 Create View Setting

**Endpoint:** `POST /api/v1/management/user-view-settings/`

**Authentication:** Required

**Request Body:**
```json
{
  "module_name": "lead",
  "view_name": "list",
  "settings": {
    "columns": [
      {
        "name": "name",
        "visible": true,
        "width": 200,
        "order": 1
      },
      {
        "name": "email",
        "visible": true,
        "width": 250,
        "order": 2
      }
    ],
    "sort_by": "created_at",
    "sort_order": "desc",
    "page_size": 50
  }
}
```

**Success Response (201 Created):** Same structure as Get View Setting

---

#### 17.3.4 Update View Setting

**Endpoint:** `PUT /api/v1/management/user-view-settings/{id}/`

**Authentication:** Required

**Success Response (200 OK):** Same structure as Get View Setting

---

#### 17.3.5 Delete View Setting

**Endpoint:** `DELETE /api/v1/management/user-view-settings/{id}/`

**Authentication:** Required

**Success Response (204 No Content):** Empty response

---

### 17.4 Audit Logs

#### 17.4.1 List Audit Logs

**Endpoint:** `GET /api/v1/access-control/audit-logs/`

**Authentication:** Required

**Permission:** Admin or audit permission

**Description:** List all audit log entries for tracking system changes and user actions.

**Query Parameters:**
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page
- `action` (string, optional): Filter by action type (CREATE, UPDATE, DELETE, VIEW, LOGIN, LOGOUT)
- `user` (integer, optional): Filter by user ID
- `content_type` (integer, optional): Filter by content type
- `timestamp__gte` (datetime, optional): Filter by start date
- `timestamp__lte` (datetime, optional): Filter by end date

**Success Response (200 OK):**
```json
{
  "count": 500,
  "next": "http://156.67.104.149:8013/api/v1/access-control/audit-logs/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": 2,
      "user_email": "user@example.com",
      "action": "UPDATE",
      "content_type": 15,
      "object_id": "5",
      "object_repr": "Lead: John Doe",
      "changes": {
        "status": {"old": "New", "new": "Qualified"},
        "lead_score": {"old": 50, "new": 75}
      },
      "ip_address": "192.168.1.100",
      "user_agent": "Mozilla/5.0...",
      "timestamp": "2026-02-10T10:30:00Z",
      "organization": 1
    }
  ]
}
```

**Response Fields:**
- `id` (integer): Audit log entry ID
- `user` (integer): User ID who performed the action
- `user_email` (string): Email of the user
- `action` (string): Action type (CREATE, UPDATE, DELETE, VIEW, LOGIN, LOGOUT)
- `content_type` (integer): ContentType ID of the affected model
- `object_id` (string): ID of the affected object
- `object_repr` (string): String representation of the object
- `changes` (json): Object containing changed fields with old and new values
- `ip_address` (string): IP address of the request
- `user_agent` (string): Browser/client user agent
- `timestamp` (datetime): When the action occurred
- `organization` (integer): Organization ID

**Usage Example:**
```bash
# Get all audit logs for a specific user
GET /api/v1/access-control/audit-logs/?user=2

# Get all update actions in the last 24 hours
GET /api/v1/access-control/audit-logs/?action=UPDATE&timestamp__gte=2026-02-09T00:00:00Z

# Get all changes to leads
GET /api/v1/access-control/audit-logs/?content_type=15
```

**Note:** Audit logs are automatically created for:
- All CRM operations (Leads, Customers, Tasks, Issues, etc.)
- User management actions
- Role and permission changes
- Authentication events (login/logout)
- Access control modifications

---

#### 17.4.2 Get Audit Log Entry

**Endpoint:** `GET /api/v1/access-control/audit-logs/{id}/`

**Authentication:** Required

**Permission:** Admin or audit permission

**Success Response (200 OK):** Same structure as list item

---

## Error Codes

### Standard HTTP Status Codes

| Code | Description | Usage |
|------|-------------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request data, validation errors |
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### Error Response Format

```json
{
  "error": "Error message description",
  "field_name": ["Field-specific error message"],
  "detail": "Detailed error information"
}
```

### Common Error Examples

**Validation Error (400):**
```json
{
  "email": ["This field may not be blank."],
  "password": ["This field is required."]
}
```

**Authentication Error (401):**
```json
{
  "detail": "Authentication credentials were not provided."
}
```

**Permission Error (403):**
```json
{
  "detail": "You do not have permission to perform this action."
}
```

**Not Found (404):**
```json
{
  "detail": "Not found."
}
```

**Rate Limit (429):**
```json
{
  "error": "Request was throttled. Expected available in 45 seconds."
}
```

---

## Rate Limiting

### Default Rate Limits

| Endpoint Type | Rate Limit | Key |
|--------------|------------|-----|
| Login/Logout | 10 requests/minute | IP Address |
| Registration | 10 requests/minute | IP Address |
| Password Reset | 10 requests/minute | IP Address |
| Authenticated APIs | 60 requests/minute | User |

### Rate Limit Headers

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1706270400
```

### Rate Limit Response

When rate limit is exceeded:

```json
{
  "error": "Request was throttled. Expected available in 45 seconds."
}
```

**Status Code:** 429 Too Many Requests

---

## Additional Features

### 1. Caching

- **List endpoints**: Cached for 5 minutes (300 seconds)
- **Detail endpoints**: Cached for 5 minutes (300 seconds)
- **Hierarchy endpoints**: Cached for 15 minutes (900 seconds)
- Cache varies by user and organization
- Cache automatically invalidated on updates

### 2. Pagination

All list endpoints support pagination:

**Query Parameters:**
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)

**Response Format:**
```json
{
  "count": 150,
  "next": "http://156.67.104.149:8013/api/v1/crm/leads/?page=2",
  "previous": null,
  "results": []
}
```

### 3. Filtering

Most list endpoints support filtering via query parameters:

```
GET /api/v1/crm/leads/?status=1&is_converted=false&lead_owner=5
```

### 4. Searching

Most list endpoints support full-text search:

```
GET /api/v1/crm/leads/?search=john
```

### 5. Ordering

Most list endpoints support ordering:

```
GET /api/v1/crm/leads/?ordering=-created_at
GET /api/v1/crm/leads/?ordering=lead_name
```

Use `-` prefix for descending order.

### 6. API Documentation

Access interactive API documentation:

- **Swagger UI:** `http://156.67.104.149:8013/api/docs/`
- **ReDoc:** `http://156.67.104.149:8013/api/redoc/`
- **OpenAPI Schema:** `http://156.67.104.149:8013/api/schema/`

---

## Security Features

### 1. Account Lockout
- Account locked after 5 failed login attempts within 1 hour
- Automatic unlock after 1 hour
- Can be unlocked via password reset

### 2. JWT Token Security
- Access token expires after 1 hour
- Refresh token expires after 30 days
- Token includes user_id and organization_id in payload
- Token blacklisting on logout

### 3. Password Requirements
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

### 4. Organization Isolation
- All data strictly isolated by organization
- Users can only access data within their organization
- Organization ID automatically applied to all queries

### 5. Permission-Based Access Control
- Module-level permissions (CRUD operations)
- Record-level permissions (Own, Team, Hierarchy, All)
- Role-based access with priority hierarchy
- Dynamic permission evaluation

---

## Best Practices

### 1. Authentication
- Always store tokens securely (not in localStorage)
- Refresh tokens before they expire
- Implement automatic token refresh
- Handle 401 errors by redirecting to login

### 2. Error Handling
- Always check response status codes
- Parse error messages for user feedback
- Implement retry logic for 429 errors
- Log errors for debugging

### 3. Performance
- Use pagination for large datasets
- Implement client-side caching
- Use filters to reduce response size
- Batch operations when possible

### 4. Data Validation
- Validate data on client before sending
- Handle validation errors gracefully
- Show field-specific error messages
- Implement real-time validation where appropriate

---

## Support & Contact

For API support, bug reports, or feature requests:
- Email: support@nexocrm.com
- Documentation: https://docs.nexocrm.com
- GitHub Issues: https://github.com/sreekesh-nexotech/nexocrm-django-api/issues

---

**Document Version:** 2.0.0
**Last Updated:** February 10, 2026
**API Version:** v1

---

## Changelog

### Version 2.0.0 (February 10, 2026)
- **Added CRM - Tasks Module**: Complete documentation for task management including follow-up tasks and Google Calendar sync
- **Added CRM - Communications Module**: Communication statuses, call logs, and notifications endpoints
- **Added CRM - Issues Module**: Issue tracking and issue types management
- **Added CRM - Organizations Module**: External organization/company management
- **Added CRM - Chat AI Assistant**: Natural language query interface for CRM data
- **Added Projects Management Module**: Complete project management suite including:
  - Project types, projects, task groups
  - Project task statuses and types
  - Hierarchical project tasks with MPPT support
  - Task dependencies tracking
  - Project updates
- **Added Email Management Module**: OAuth-based email integration with:
  - Email configuration management
  - Google OAuth flow
  - Email sending with automatic token refresh
- **Added Audit Logs**: Comprehensive audit trail for all system actions
- Updated section numbering to accommodate new modules
- Enhanced field documentation with detailed data types and constraints
- Added more request/response examples
- Documented special features like Google Calendar sync, token refresh, and hierarchical structures

### Version 1.0.0 (January 26, 2026)
- Initial documentation release
- Authentication, Leads, Customers, User Management, Roles, Teams, Hierarchy, Permissions, Organizations, Access Control

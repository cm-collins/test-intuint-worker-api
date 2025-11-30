<!-- 41e4c76c-7f1f-461e-8c35-40514ed68582 b3e00783-4f93-461b-9741-b46fcd7936e3 -->
# Intuit Invoice & Credit Note API System

## Project Overview

Build a lightweight ASP.NET Core Minimal API that integrates with QuickBooks Online API to:

- Create invoices programmatically
- Create credit notes (credit memos) to cancel/adjust invoices
- Handle full OAuth 2.0 authentication flow with Intuit
- Provide simple REST endpoints for invoice and credit note operations

## Architecture

### Core Components

1. **Minimal API Endpoints** - Lightweight HTTP endpoints for invoice/credit note operations
2. **OAuth 2.0 Service** - Handles Intuit OAuth flow (authorization, token exchange, refresh)
3. **QuickBooks API Client** - HTTP client for interacting with QuickBooks Online API
4. **Models/DTOs** - Request/response models for invoices and credit memos
5. **Configuration** - App settings for Intuit credentials and API endpoints

### Technology Stack

- ASP.NET Core 9.0 (Minimal API)
- HttpClient for QuickBooks API calls
- System.Text.Json for JSON serialization
- OAuth 2.0 authorization code flow

## Implementation Plan

### 1. Project Setup & Dependencies

- Create ASP.NET Core Minimal API project structure
- Add necessary NuGet packages (if any SDK available, otherwise use HttpClient)
- Configure appsettings.json for Intuit credentials
- Set up development environment configuration

### 2. OAuth 2.0 Authentication Service

**File: `Services/IntuitOAuthService.cs`**

- Implement OAuth 2.0 authorization code flow
- Handle authorization URL generation
- Exchange authorization code for access/refresh tokens
- Token refresh mechanism
- Token storage (in-memory or configuration-based for simplicity)

**Endpoints:**

- `GET /auth/authorize` - Redirect to Intuit authorization page
- `GET /auth/callback` - Handle OAuth callback and exchange code for tokens
- `GET /auth/refresh` - Refresh access token using refresh token

### 3. QuickBooks API Client

**File: `Services/QuickBooksApiClient.cs`**

- HTTP client wrapper for QuickBooks Online API v3
- Base URL: `https://sandbox-quickbooks.api.intuit.com/v3/company/{companyId}/`
- Methods:
  - `CreateInvoiceAsync(InvoiceRequest request)` - POST to `/invoice`
  - `CreateCreditMemoAsync(CreditMemoRequest request)` - POST to `/creditmemo`
  - `GetInvoiceAsync(string invoiceId)` - GET `/invoice/{id}`
  - `ApplyCreditMemoToInvoiceAsync(string creditMemoId, string invoiceId)` - Create payment linking both

### 4. Data Models

**File: `Models/InvoiceModels.cs`**

- `InvoiceRequest` - DTO for creating invoices
- `InvoiceResponse` - Response from QuickBooks
- `CreditMemoRequest` - DTO for creating credit memos
- `CreditMemoResponse` - Response from QuickBooks
- `LineItem` - Invoice/credit memo line items
- `CustomerRef`, `ItemRef` - Reference objects

### 5. API Endpoints (Minimal API)

**File: `Program.cs`**

- `POST /api/invoices` - Create a new invoice
  - Request body: InvoiceRequest
  - Returns: Created invoice with ID
- `POST /api/invoices/{invoiceId}/credit-note` - Create credit note for an invoice
  - Creates credit memo matching invoice amount
  - Optionally applies credit to invoice
- `GET /api/invoices/{invoiceId}` - Get invoice details
- `GET /api/credit-notes/{creditNoteId}` - Get credit note details

### 6. Configuration

**File: `appsettings.json`**

```json
{
  "Intuit": {
    "ClientId": "",
    "ClientSecret": "",
    "RedirectUri": "https://localhost:5001/auth/callback",
    "Environment": "sandbox",
    "BaseUrl": "https://sandbox-quickbooks.api.intuit.com"
  }
}
```

### 7. Error Handling & Validation

- Global exception handler middleware
- Validation for invoice/credit memo requests
- Proper HTTP status codes
- Error response models

### 8. Documentation

- Update README.md with:
  - Setup instructions
  - OAuth configuration steps
  - API endpoint documentation
  - Example requests/responses

## Key Implementation Details

### Invoice Creation Flow

1. Client calls `POST /api/invoices` with invoice data
2. System validates request
3. Retrieves stored OAuth access token
4. Makes POST request to QuickBooks API `/v3/company/{companyId}/invoice`
5. Returns created invoice with QuickBooks ID

### Credit Note Creation Flow

1. Client calls `POST /api/invoices/{invoiceId}/credit-note`
2. System fetches original invoice from QuickBooks
3. Creates credit memo with matching line items (negative amounts)
4. Optionally creates payment to apply credit memo to invoice
5. Returns credit memo details

### OAuth Flow

1. User navigates to `/auth/authorize`
2. Redirected to Intuit authorization page
3. User authorizes application
4. Intuit redirects to `/auth/callback` with authorization code
5. System exchanges code for access/refresh tokens
6. Tokens stored for subsequent API calls

## File Structure

```
test-intuint-invoicing-api/
├── Program.cs                 # Minimal API setup & endpoints
├── appsettings.json           # Configuration
├── appsettings.Development.json
├── Services/
│   ├── IntuitOAuthService.cs  # OAuth 2.0 handling
│   └── QuickBooksApiClient.cs # QuickBooks API client
├── Models/
│   ├── InvoiceModels.cs       # Invoice/CreditMemo DTOs
│   └── ApiResponse.cs         # Standard API response models
└── README.md                  # Documentation
```

## Testing Considerations

- Use Intuit sandbox environment for testing
- Test OAuth flow end-to-end
- Validate invoice creation with sample data
- Test credit memo creation and application
- Handle error scenarios (invalid tokens, API errors)
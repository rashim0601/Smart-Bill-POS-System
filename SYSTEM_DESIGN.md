# SmartBill POS - Advanced System Design Document

**Version:** 1.0.0 | **Last Updated:** May 2026

---

## 📑 Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [High-Level Architecture](#high-level-architecture)
4. [Detailed Component Architecture](#detailed-component-architecture)
5. [Data Flow Architecture](#data-flow-architecture)
6. [Database Design](#database-design)
7. [Security Architecture](#security-architecture)
8. [API Architecture](#api-architecture)
9. [Integration Architecture](#integration-architecture)
10. [Scalability & Performance](#scalability--performance)
11. [Deployment Architecture](#deployment-architecture)
12. [System Reliability](#system-reliability)
13. [Monitoring & Logging](#monitoring--logging)

---

## 🎯 Executive Summary

**SmartBill POS** is a modern, cloud-ready Point of Sale system built with:
- **Frontend:** React 18 with Vite for rapid development
- **Backend:** Express.js with Node.js runtime
- **Database:** MongoDB for flexible document storage
- **Caching:** Redis for performance optimization
- **AI:** Google Gemini for intelligent product recognition

The system is designed for:
- **Multi-user support** with role-based access control
- **Real-time inventory** tracking across multiple locations
- **Scalable architecture** supporting horizontal scaling
- **Production-ready** with comprehensive error handling
- **Secure** with JWT authentication and encrypted sessions

---

## 📐 System Overview

### **System Boundaries**

```
┌─────────────────────────────────────────────────────────────┐
│                    SMARTBILL POS SYSTEM                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐    │
│  │   Frontend   │  │   Backend    │  │   Database    │    │
│  │   (React)    │  │  (Express)   │  │   (MongoDB)   │    │
│  └──────────────┘  └──────────────┘  └───────────────┘    │
│        ↓                   ↓                   ↑             │
│  REST API (HTTPS)          │            ← Mongoose ORM   │
│        ↓                   ↓                              │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │    Socket    │  │   Service    │  │    Cache      │   │
│  │   Events     │  │    Layer     │  │   (Redis)     │   │
│  └──────────────┘  └──────────────┘  └───────────────┘   │
│                                                             │
│  External Services:                                        │
│  ├─ Google Gemini API (AI)                                │
│  ├─ WhatsApp API (Notifications)                          │
│  └─ Email Service (Communication)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### **User Types & Access Levels**

```
┌────────────────────────────────────────────────┐
│            ROLE-BASED ACCESS CONTROL           │
├────────────────────────────────────────────────┤
│                                                 │
│ ADMIN                                          │
│ ├─ User management                            │
│ ├─ System configuration                       │
│ ├─ All reports                                │
│ └─ Full system access                         │
│                                                 │
│ MANAGER                                        │
│ ├─ Inventory management                       │
│ ├─ Customer management                        │
│ ├─ Sales reports                              │
│ └─ Staff oversight                            │
│                                                 │
│ CASHIER                                        │
│ ├─ Create bills                               │
│ ├─ View products                              │
│ ├─ Customer lookup                            │
│ └─ Own bill history                           │
│                                                 │
└────────────────────────────────────────────────┘
```

---

## 🏗️ High-Level Architecture

### **3-Tier Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION TIER                        │
│  React Component Layer | State Management | UI Rendering   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │   Billing    │ │  Inventory   │ │ Customer     │       │
│  │   Components │ │  Components  │ │ Components   │       │
│  └──────────────┘ └──────────────┘ └──────────────┘       │
└────────────────┬──────────────────────────────────────────┘
                 │ REST API (HTTP/HTTPS)
                 │ JWT Token Authentication
┌────────────────▼──────────────────────────────────────────┐
│                   APPLICATION TIER                        │
│  Express.js Request Handler | Route Controllers           │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│  │   Auth API   │ │  Product API │ │  Bill API    │      │
│  └──────────────┘ └──────────────┘ └──────────────┘      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│  │  Inventory   │ │ Customer API │ │ Reports API  │      │
│  │   API        │ │              │ │              │      │
│  └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                             │
│  Middleware Stack:                                        │
│  ├─ CORS Handler                                         │
│  ├─ Authentication Middleware                           │
│  ├─ Session Validation                                  │
│  ├─ Error Handler                                       │
│  └─ Request Logger                                      │
└────────────────┬──────────────────────────────────────────┘
                 │ Mongoose ODM | Query Optimization
┌────────────────▼──────────────────────────────────────────┐
│                    DATA ACCESS TIER                       │
│  MongoDB Connection | Query Builder | Caching Layer      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │   Product    │ │   Bill       │ │  Customer    │     │
│  │   Service    │ │   Service    │ │  Service     │     │
│  └──────────────┘ └──────────────┘ └──────────────┘     │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  Inventory   │ │  Cache       │ │  Session     │     │
│  │  Service     │ │  Service     │ │  Service     │     │
│  └──────────────┘ └──────────────┘ └──────────────┘     │
└────────────────┬──────────────────────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼─────┐  ┌──▼────────┐  ┌─▼─────────┐
│ MongoDB  │  │ Redis     │  │ External  │
│ (Data)   │  │ (Cache)   │  │ APIs      │
└──────────┘  └───────────┘  └───────────┘
```

---

## 🔧 Detailed Component Architecture

### **Frontend Architecture**

```
React Application (Vite)
│
├── Entry Point (main.jsx)
│   └── App.jsx (Root Component)
│
├── Context Layer (State Management)
│   └── AuthContext
│       ├── user: { id, username, email, role, shop details }
│       ├── token: JWT Token
│       ├── sessionId: Session ID
│       └── Methods: login(), logout(), updateUser()
│
├── Page Components
│   ├── Login/Signup Pages
│   ├── BillingSystem (Main Dashboard)
│   ├── ProfilePage
│   └── SalesReports
│
├── Feature Components
│   ├── ProductScanner
│   │   └── Quagga.js Integration
│   ├── BillingSystem
│   │   ├── Product Search
│   │   ├── Cart Management
│   │   ├── Bill Display
│   │   └── Payment Modal
│   ├── InventoryManager
│   │   ├── Stock Tracking
│   │   ├── Stock Alerts
│   │   └── Reorder Management
│   └── CustomerManager
│       ├── Customer CRUD
│       ├── Purchase History
│       └── Customer Analytics
│
├── UI Component Library (Layout/)
│   ├── Alert, Badge, Button
│   ├── Card, Form, FormField
│   ├── Table, Navbar, Footer
│   └── Toast (Notifications)
│
├── Service Layer
│   ├── quaggaService (Barcode scanning)
│   ├── API Client (Fetch wrapper)
│   └── Auth Service (Token management)
│
├── Styling
│   ├── Tailwind CSS
│   ├── Component-level CSS
│   └── Utility classes
│
└── Configuration
    ├── vite.config.js
    ├── tailwind.config.js
    └── .env (API endpoints)
```

### **Backend Architecture**

```
Express.js Server
│
├── Server Initialization (server.js)
│   ├── Mongoose Connection
│   ├── Middleware Setup
│   └── Route Registration
│
├── Middleware Stack
│   ├── CORS Middleware
│   ├── JSON Parser (50MB limit)
│   ├── URL Encoded Parser
│   ├── Session Middleware
│   └── Request Logger
│
├── Route Layer
│   ├── /api/auth (Authentication)
│   │   ├── POST /signup
│   │   ├── POST /login
│   │   ├── GET /user
│   │   └── PUT /user
│   │
│   ├── /api/products (Product CRUD)
│   │   ├── GET / (List with filters)
│   │   ├── GET /barcode/:barcode
│   │   ├── POST / (Create)
│   │   ├── PUT /:id (Update)
│   │   └── DELETE /:id
│   │
│   ├── /api/bills (Bill Management)
│   │   ├── POST / (Create bill)
│   │   ├── GET / (List bills)
│   │   ├── GET /:id (Get bill)
│   │   ├── GET /daily-stats
│   │   └── DELETE /:id
│   │
│   ├── /api/inventory (Stock Management)
│   │   ├── GET / (List inventory)
│   │   ├── POST / (Add stock)
│   │   ├── PUT /:id (Update stock)
│   │   └── GET /alerts/low-stock
│   │
│   └── /api/customers (Customer CRUD)
│       ├── GET / (List customers)
│       ├── POST / (Create customer)
│       ├── PUT /:id (Update)
│       └── DELETE /:id
│
├── Service Layer (Business Logic)
│   ├── billService
│   │   ├── calculateTax()
│   │   ├── generateBillNumber()
│   │   ├── calculateChange()
│   │   └── generatePDF()
│   │
│   ├── inventoryService
│   │   ├── updateStock()
│   │   ├── checkLowStock()
│   │   ├── calculateReorderQuantity()
│   │   └── generateStockAlert()
│   │
│   ├── cacheService
│   │   ├── getCachedProduct()
│   │   ├── setCachedProduct()
│   │   ├── invalidateCache()
│   │   └── getCachedInventory()
│   │
│   ├── sessionService
│   │   ├── createSession()
│   │   ├── getSession()
│   │   ├── updateSessionActivity()
│   │   └── destroySession()
│   │
│   └── whatsappService
│       ├── sendBillViaWhatsApp()
│       ├── sendLowStockAlert()
│       └── formatMessage()
│
├── Middleware (Custom)
│   ├── sessionMiddleware
│   │   ├── validateSession()
│   │   └── optionalSession()
│   │
│   └── authMiddleware (in auth.js)
│       └── verifyToken()
│
├── Models (Mongoose Schemas)
│   ├── User
│   ├── Product
│   ├── Bill
│   ├── Customer
│   └── Inventory
│
├── Configuration
│   ├── redis.js (Redis config)
│   └── environment variables
│
└── Error Handling
    ├── Global error handler
    ├── Validation errors
    └── Database errors
```

---

## 🔄 Data Flow Architecture

### **Bill Creation Flow**

```
┌─────────────────────────────────────────────────────────────┐
│                    BILL CREATION FLOW                       │
└─────────────────────────────────────────────────────────────┘

User Actions                   System Processing
     │                              │
     ├─ Scan/Add Product            │
     │  └──────────────────────────▶│ Fetch Product from Cache/DB
     │                              │ │
     │                              ├─ Verify Stock Availability
     │                              │ │
     │                              └─ Return Product Details
     │  ◀────────────────────────────┤
     │  (Display in Cart)            │
     │                               │
     ├─ Set Quantity                │
     │  (Update Bill Items Array)    │
     │                               │
     ├─ Calculate Totals            │
     │  ├─ Subtotal                 │
     │  ├─ Tax (Subtotal × GST%)    │
     │  └─ Total (Subtotal + Tax)   │
     │                               │
     ├─ Apply Discount              │
     │  └─ Recalculate Total        │
     │                               │
     ├─ Enter Customer (Optional)   │
     │  └──────────────────────────▶│ Lookup Customer
     │                              │ │
     │                              └─ Return Customer Details
     │  ◀────────────────────────────┤
     │                               │
     ├─ Select Payment Method       │
     │  └─ Cash/Card/UPI/Check      │
     │                               │
     ├─ Enter Amount Received       │
     │  └─ Calculate Change         │
     │                               │
     ├─ Confirm & Submit            │
     │  └──────────────────────────▶│ Validate Bill Data
     │                              │ │
     │                              ├─ Generate Bill Number
     │                              │ │
     │                              ├─ Deduct from Inventory
     │                              │ │
     │                              ├─ Update Customer Stats
     │                              │ │
     │                              ├─ Save Bill to Database
     │                              │ │
     │                              ├─ Cache Bill Data
     │                              │ │
     │                              └─ Generate PDF
     │  ◀────────────────────────────┤
     │                               │
     └─ Display Bill Summary         │
        (Print/Download/Email)       │
```

### **Product Barcode Scan Flow**

```
┌─────────────────────────────────────────────────────────────┐
│                BARCODE SCAN FLOW                            │
└─────────────────────────────────────────────────────────────┘

Browser Barcode Scanner
       │
       ├─ Capture Camera Feed
       │
       ├─ Process with Quagga.js
       │
       ├─ Extract Barcode Digits
       │  └──────────────────────────────────────┐
       │                                         │
       └─ Send API Request                       │
          └─ GET /api/products/barcode/:barcode │
             │                                   │
             ├─ Check Redis Cache ───────────────┤
             │  ├─ HIT: Return cached product   │
             │  └─ MISS: Query MongoDB          │
             │          │                       │
             │          ├─ Find by barcode     │
             │          │                       │
             │          ├─ Fetch associated    │
             │          │  inventory            │
             │          │                       │
             │          ├─ Cache in Redis      │
             │          │                       │
             │          └─ Return response     │
             │                                  │
             ◀──────────────────────────────────┘
             │
       ◀─────┘
       │
       └─ Validate Product
          └─ Check Stock Status
             │
             ├─ In Stock ──────┐
             ├─ Low Stock ─────┼─ Show Toast
             └─ Out of Stock ──┤
                               │
                               └─ Add to Cart
                                  (or show error)
```

### **Authentication & Session Flow**

```
┌─────────────────────────────────────────────────────────────┐
│              AUTHENTICATION & SESSION FLOW                  │
└─────────────────────────────────────────────────────────────┘

User Login
   │
   ├─ POST /api/auth/login
   │  └─ { username, password }
   │
   ├─ Validate Credentials
   │  ├─ Find User in DB
   │  ├─ Compare Password Hash
   │  └─ Verify Active Status
   │
   ├─ Generate Tokens
   │  ├─ JWT Token (stored in localStorage)
   │  └─ Create Session in Redis
   │
   ├─ Response to Client
   │  └─ { token, user, sessionId }
   │
   ├─ Client Storage
   │  ├─ localStorage.authToken = JWT
   │  ├─ localStorage.user = User Object
   │  └─ localStorage.sessionId = Session ID
   │
   └─ Subsequent Requests
      ├─ Include Auth Header
      │  └─ Authorization: Bearer <JWT>
      │
      ├─ Verify Token
      │  ├─ Decode JWT
      │  ├─ Check Signature
      │  └─ Validate Expiry
      │
      └─ Validate Session
         ├─ Get Session from Redis
         ├─ Update Last Activity
         └─ Attach to Request
```

---

## 📊 Database Design

### **Relational Schema Diagram**

```
┌─────────────────────────────────────────────────────────────┐
│                   DATABASE SCHEMA                           │
└─────────────────────────────────────────────────────────────┘

          ┌────────────────┐
          │     User       │
          ├────────────────┤
          │ _id (PK)       │
          │ username       │◄────────┐
          │ email          │         │
          │ password       │         │
          │ role           │         │
          │ shopName       │         │
          │ shopAddress    │         │
          │ phone          │         │
          │ gstRate        │         │
          │ lastLogin      │         │
          └────────────────┘         │
                 ▲                    │
                 │                    │
        ┌────────┴────────────────────┴──────────┐
        │                                         │
        │                                         │
   ┌────▼─────────┐  ┌──────────────┐  ┌────────▼──┐
   │   Product    │  │    Bill      │  │  Customer │
   ├──────────────┤  ├──────────────┤  ├───────────┤
   │ _id (PK)     │  │ _id (PK)     │  │ _id (PK)  │
   │ name         │  │ userId (FK)  │◄─┤ userId    │
   │ price        │  │ billNumber   │  │ (FK)      │
   │ category     │  │ items[]      │  │ name      │
   │ barcode      │  │  -productId  │  │ mobile    │
   │ sku          │  │  -quantity   │  │ address   │
   │ image        │  │  -unitPrice  │  │ gstNumber │
   │ active       │  │ subtotal     │  │ totalBills│
   │              │  │ tax          │  │ totalSpent│
   │              │  │ discount     │  │ isActive  │
   │              │  │ total        │  └───────────┘
   │              │  │ paymentMethod│
   │              │  │ customer info│
   │              │  │ notes        │
   └──────────────┘  └──────────────┘
        ▲                    │
        │                    │
        │                    │
   ┌────┴──────────────────────────┐
   │                               │
┌──▼─────────────┐                │
│  Inventory     │                │
├────────────────┤                │
│ _id (PK)       │                │
│ productId (FK) │◄───────────────┘
│ barcode        │
│ quantity       │
│ minStock       │
│ maxStock       │
│ reorderLevel   │
│ location       │
│ warehouse      │
│ lastRestocked  │
│ status         │
└────────────────┘

Foreign Keys:
- Bill.userId → User._id
- Bill.items.productId → Product._id
- Customer.userId → User._id
- Inventory.productId → Product._id

Indices:
- User: username (unique), email (unique)
- Product: name, barcode, category
- Bill: userId, createdAt, billNumber
- Customer: userId + mobileNumber, userId + name
- Inventory: productId, barcode, warehouse
```

### **Data Consistency Strategy**

```
┌─────────────────────────────────────────────────────────────┐
│            DATA CONSISTENCY & INTEGRITY                     │
└─────────────────────────────────────────────────────────────┘

ACID Compliance:
─────────────────

Atomicity:
├─ Bill creation is atomic transaction
│  ├─ Save bill document
│  ├─ Deduct inventory
│  └─ Update customer stats
│  (All or nothing principle)

Consistency:
├─ Inventory quantity must be non-negative
├─ Subtotal + Tax = Total (calculated)
├─ Customer.totalBills increments on bill save
├─ Product.active determines visibility

Isolation:
├─ User sessions isolated
├─ Concurrent bill creation supported
├─ Redis locks for critical operations

Durability:
├─ MongoDB replica set for redundancy
├─ Backup strategy implemented
└─ Transaction logs maintained

Referential Integrity:
──────────────────────
├─ Bill.userId must exist in User
├─ Bill.items[].productId must exist in Product
├─ Inventory.productId must exist in Product
└─ Customer.userId must exist in User

Cascade Operations:
──────────────────
├─ Delete User → (soft delete bills)
├─ Delete Product → Keep inventory records
└─ Delete Customer → Keep bill history
```

---

## 🔐 Security Architecture

### **Authentication & Authorization**

```
┌─────────────────────────────────────────────────────────────┐
│            SECURITY LAYERS & MECHANISMS                     │
└─────────────────────────────────────────────────────────────┘

Layer 1: Transport Security
──────────────────────────
├─ HTTPS/TLS for all communications
├─ Certificate pinning (production)
├─ HSTS headers
└─ Secure cookies (httpOnly, Secure flags)

Layer 2: Authentication
──────────────────────
├─ Username/Password with bcryptjs
│  └─ Salt rounds: 10
│  └─ Hash algorithm: bcrypt
│
├─ JWT Token Based
│  ├─ Signature: HS256 (HMAC-SHA256)
│  ├─ Expiry: 7 days (configurable)
│  ├─ Stored in: localStorage (frontend)
│  └─ Sent in: Authorization header
│
└─ Session Management
   ├─ Redis session store
   ├─ Session expiry: 24 hours
   ├─ Activity timestamp tracking
   └─ Concurrent session limit

Layer 3: Authorization
─────────────────────
├─ Role-Based Access Control (RBAC)
│  ├─ Admin: Full system access
│  ├─ Manager: Inventory & customer access
│  └─ Cashier: Billing operations only
│
└─ Resource-level Authorization
   ├─ Users can only access own data
   ├─ Bills filtered by userId
   ├─ Customers filtered by userId
   └─ Inventory scoped to user

Layer 4: Input Validation
────────────────────────
├─ Frontend validation
│  ├─ Type checking
│  ├─ Length validation
│  └─ Format validation (email, phone)
│
└─ Backend validation
   ├─ express-validator middleware
   ├─ Mongoose schema validation
   ├─ Type coercion prevention
   └─ SQL injection prevention (MongoDB)

Layer 5: API Security
────────────────────
├─ CORS policy enforcement
│  └─ Whitelist allowed origins
│
├─ Rate limiting (recommended)
│  ├─ Per user/IP limits
│  ├─ Exponential backoff
│  └─ Distributed rate limiting
│
├─ Request size limits
│  └─ 50MB max payload
│
└─ Content-Type validation
   └─ application/json enforcement

Layer 6: Data Security
────────────────────
├─ Encryption at rest
│  ├─ MongoDB encryption
│  ├─ Sensitive field encryption
│  └─ Password hashing
│
├─ Encryption in transit
│  ├─ TLS 1.2+
│  └─ Secure headers
│
└─ Data masking
   ├─ Customer phone digits masked
   └─ Credit card last 4 digits only

Layer 7: Audit & Logging
───────────────────────
├─ Request logging
│  ├─ Timestamp, method, path
│  ├─ User ID, status code
│  └─ Response time
│
├─ Error logging
│  ├─ Stack traces
│  ├─ Context information
│  └─ Severity levels
│
└─ Audit trail
   ├─ Price changes logged
   ├─ Stock adjustments tracked
   └─ User actions recorded
```

### **Data Protection**

```
Sensitive Data Handling:
──────────────────────

1. Passwords
   ├─ Never stored in plaintext
   ├─ Hashed with bcrypt (10 rounds)
   ├─ Unique salt per password
   └─ Never logged or exposed

2. API Keys/Secrets
   ├─ Stored in .env (not versioned)
   ├─ Environment-specific values
   ├─ Rotated regularly
   └─ Never logged

3. User Tokens
   ├─ JWT payload validated
   ├─ Signature verification
   ├─ Expiry checking
   └─ Token refresh strategy

4. Personal Information
   ├─ Phone numbers validated
   ├─ Email addresses verified
   ├─ Address information encrypted
   └─ PII access logged
```

---

## 🔌 API Architecture

### **Request-Response Pattern**

```
┌─────────────────────────────────────────────────────────────┐
│              API REQUEST-RESPONSE FLOW                      │
└─────────────────────────────────────────────────────────────┘

CLIENT REQUEST:
───────────────
POST /api/bills
Header:
  Authorization: Bearer <JWT_TOKEN>
  Content-Type: application/json
  X-Session-ID: <SESSION_ID> (optional)

Body:
{
  items: [
    {
      productId: "60f7b3c3d8c4e1a2b3c4d5e6",
      productName: "Product A",
      quantity: 2,
      unitPrice: 100,
      totalPrice: 200
    }
  ],
  subtotal: 200,
  tax: 10,
  taxPercentage: 5,
  discount: 0,
  total: 210,
  paymentMethod: "Cash",
  amountReceived: 250,
  customerName: "John Doe",
  customerMobile: "9876543210",
  shopName: "My Shop",
  shopAddress: "123 Main St"
}

SERVER PROCESSING:
──────────────────
1. Extract token from Authorization header
2. Verify JWT signature & expiry
3. Validate session if provided
4. Extract user ID from token
5. Validate request body
   ├─ Check required fields
   ├─ Validate data types
   ├─ Validate product exists
   └─ Check inventory
6. Process business logic
   ├─ Generate bill number
   ├─ Calculate totals
   ├─ Deduct from inventory
   └─ Save to database
7. Generate response
8. Send to client

SUCCESS RESPONSE:
────────────────
HTTP 200 OK
{
  success: true,
  message: "Bill created successfully",
  data: {
    _id: "60f7b3c3d8c4e1a2b3c4d5e7",
    billNumber: "BILL-1609459200000",
    userId: "60f7b3c3d8c4e1a2b3c4d5e6",
    items: [...],
    subtotal: 200,
    tax: 10,
    total: 210,
    paymentStatus: "Completed",
    createdAt: "2024-05-02T10:30:00Z"
  }
}

ERROR RESPONSE:
──────────────
HTTP 400 Bad Request
{
  success: false,
  error: "Invalid product ID",
  details: {
    field: "items[0].productId",
    message: "Product not found"
  }
}

OR

HTTP 401 Unauthorized
{
  error: "Invalid token. Access denied."
}

OR

HTTP 500 Internal Server Error
{
  error: "Internal server error",
  message: "Database connection failed"
}
```

### **API Response Standardization**

```
Standard Success Response:
{
  success: true,
  message: "Operation successful",
  data: { ... },
  timestamp: "2024-05-02T10:30:00Z"
}

Standard Error Response:
{
  success: false,
  error: "Error type",
  message: "Human-readable message",
  code: "ERROR_CODE",
  details: { ... },
  timestamp: "2024-05-02T10:30:00Z"
}

Pagination Response:
{
  success: true,
  data: [ ... ],
  pagination: {
    page: 1,
    limit: 10,
    total: 100,
    pages: 10
  }
}
```

---

## 🔗 Integration Architecture

### **External Service Integrations**

```
┌─────────────────────────────────────────────────────────────┐
│           EXTERNAL SERVICE INTEGRATIONS                     │
└─────────────────────────────────────────────────────────────┘

1. Google Gemini AI
   ├─ Purpose: Product image recognition
   ├─ API Endpoint: https://generativelanguage.googleapis.com
   ├─ Authentication: API Key in headers
   ├─ Use Cases:
   │  ├─ Extract product details from images
   │  ├─ Parse receipt/bill images
   │  ├─ Auto-categorize products
   │  └─ Batch product import
   └─ Flow:
      ├─ Upload image to endpoint
      ├─ Process with Gemini model
      ├─ Extract: name, price, category, brand
      ├─ Return confidence scores
      └─ Save to database

2. WhatsApp Business API
   ├─ Purpose: Bill delivery & notifications
   ├─ API Endpoint: https://api.whatsapp.com
   ├─ Authentication: Account SID + API Token
   ├─ Use Cases:
   │  ├─ Send bill to customer
   │  ├─ Low stock alerts
   │  ├─ Payment reminders
   │  └─ Order confirmations
   └─ Flow:
      ├─ Format message
      ├─ Send to WhatsApp API
      ├─ Track delivery status
      └─ Log in audit trail

3. Email Service (Optional)
   ├─ Purpose: Email notifications
   ├─ Provider: SMTP (Gmail, SendGrid, etc.)
   ├─ Use Cases:
   │  ├─ Send bill receipts
   │  ├─ Password reset
   │  └─ System notifications
   └─ Flow:
      ├─ Prepare email content
      ├─ Connect to SMTP server
      ├─ Send with MIME formatting
      └─ Track delivery

4. Payment Gateway (Future)
   ├─ Purpose: Online payment processing
   ├─ Providers: Razorpay, Stripe, PayU
   ├─ Use Cases:
   │  ├─ Process card payments
   │  ├─ UPI payments
   │  └─ Digital wallets
   └─ Flow:
      ├─ Create payment order
      ├─ Get payment link
      ├─ Verify payment status
      └─ Update bill status

5. SMS Service (Optional)
   ├─ Purpose: SMS notifications
   ├─ Providers: Twilio, AWS SNS
   ├─ Use Cases:
   │  ├─ Low stock alerts
   │  ├─ Order confirmations
   │  └─ Customer notifications
   └─ Flow:
      ├─ Format SMS message
      ├─ Send to SMS API
      └─ Track delivery
```

### **Integration Error Handling**

```
Retry Strategy:
───────────────
├─ Immediate retry (1st attempt)
├─ Exponential backoff
│  ├─ 1st retry: 1 second
│  ├─ 2nd retry: 2 seconds
│  ├─ 3rd retry: 4 seconds
│  └─ Max retries: 3
│
└─ Fallback mechanism
   ├─ Log error
   ├─ Queue for later processing
   └─ Alert administrator

Circuit Breaker:
────────────────
├─ Monitor integration health
├─ Open circuit if failures > threshold
├─ Graceful degradation
└─ Automatic recovery

Timeout Management:
───────────────────
├─ Connection timeout: 5 seconds
├─ Read timeout: 10 seconds
├─ Total timeout: 15 seconds
└─ Fallback on timeout
```

---

## 📈 Scalability & Performance

### **Horizontal Scaling Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│          HORIZONTAL SCALING DESIGN                          │
└─────────────────────────────────────────────────────────────┘

Load Balancing:
───────────────
┌────────────────────┐
│   Client Requests  │
└────────┬───────────┘
         │
         ▼
┌────────────────────────────────────┐
│    Load Balancer (Nginx/HAProxy)   │
│  ├─ Round-robin                    │
│  ├─ Sticky sessions (optional)     │
│  └─ Health checks                  │
└────────┬─────────────┬─────────────┘
         │             │
    ┌────▼─┐       ┌───▼────┐      ┌────────┐
    │Server│       │Server  │      │Server  │
    │ Node │       │ Node   │      │ Node N │
    │  1   │       │  2     │      │        │
    └────┬─┘       └───┬────┘      └───┬────┘
         │             │               │
         └─────────────┼───────────────┘
                       │
                  ┌────▼──────────┐
                  │ Shared MongoDB│
                  │  Replica Set  │
                  │  (Cluster)    │
                  └───────────────┘

Connection Pooling:
───────────────────
├─ MongoDB: Connection pool per server
│  ├─ Min connections: 5
│  ├─ Max connections: 20
│  └─ Connection timeout: 10s
│
└─ Redis: Connection pool
   ├─ Max pool size: 10
   └─ Reconnection strategy

Database Optimization:
──────────────────────
├─ Query optimization
│  ├─ Index tuning
│  ├─ Query profiling
│  └─ Explain analysis
│
├─ Sharding strategy
│  ├─ Shard by userId
│  ├─ Even distribution
│  └─ Future scaling
│
└─ Archival strategy
   ├─ Archive bills older than 1 year
   ├─ Keep metadata in main DB
   └─ Separate archive collection

Caching Strategy:
─────────────────
├─ L1: Application cache (Memory)
│  ├─ Product catalog
│  ├─ User sessions
│  └─ TTL: 1 hour
│
├─ L2: Redis cache
│  ├─ Frequently accessed data
│  ├─ Session store
│  ├─ Cache warming
│  └─ TTL: 5 minutes
│
└─ L3: CDN (Static assets)
   ├─ Product images
   ├─ JS bundles
   └─ CSS files
```

### **Performance Metrics**

```
Target Metrics:
───────────────
├─ Page Load Time: < 2 seconds
├─ API Response Time: < 500ms
├─ 95th percentile latency: < 1 second
├─ Database query: < 100ms
├─ Cache hit rate: > 80%
├─ Throughput: > 1000 requests/sec
└─ Availability: 99.9% uptime

Bottleneck Analysis:
────────────────────
├─ Network I/O
│  ├─ Compression (gzip)
│  ├─ HTTP/2 multiplexing
│  └─ Request batching
│
├─ Database I/O
│  ├─ Connection pooling
│  ├─ Query optimization
│  └─ Caching layer
│
├─ CPU Usage
│  ├─ Async operations
│  ├─ Worker threads
│  └─ Load distribution
│
└─ Memory Management
   ├─ Garbage collection tuning
   ├─ Memory limits
   └─ Leak detection
```

---

## 🚀 Deployment Architecture

### **CI/CD Pipeline**

```
┌─────────────────────────────────────────────────────────────┐
│            CI/CD DEPLOYMENT PIPELINE                        │
└─────────────────────────────────────────────────────────────┘

Git Flow:
─────────
Developer
    │
    ├─ Create feature branch
    │
    ├─ Commit & push
    │
    └─ Create Pull Request
          │
          ▼
      ┌─────────────────┐
      │  GitHub Actions │
      └─────────────────┘
          │
          ├─ Run Tests
          │  ├─ Unit tests
          │  ├─ Integration tests
          │  └─ E2E tests
          │
          ├─ Code Quality
          │  ├─ ESLint
          │  ├─ SonarQube
          │  └─ Code coverage
          │
          ├─ Build Artifacts
          │  ├─ Frontend build
          │  └─ Backend bundle
          │
          └─ Deploy to Staging
             │
             ├─ Run smoke tests
             ├─ Performance tests
             └─ Security scan

PR Review & Merge
    │
    ├─ Code review required
    ├─ Approvals
    │
    └─ Merge to main branch
          │
          ▼
      ┌─────────────────┐
      │  CD Pipeline    │
      └─────────────────┘
          │
          ├─ Build Docker image
          │
          ├─ Push to registry
          │
          ├─ Deploy to Production
          │  ├─ Blue-green deployment
          │  ├─ Canary rollout (10% → 50% → 100%)
          │  └─ Health checks
          │
          ├─ Database migrations
          │
          ├─ Cache invalidation
          │
          └─ Monitor metrics
             ├─ Error rates
             ├─ Latency
             └─ Resource usage

Rollback Strategy:
──────────────────
├─ Automatic rollback on failure
├─ Manual rollback option
├─ Version pinning
└─ Database migration rollback
```

### **Infrastructure as Code (IaC)**

```
Kubernetes Deployment:
──────────────────────

apiVersion: apps/v1
kind: Deployment
metadata:
  name: smartbill-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: smartbill-api
  template:
    metadata:
      labels:
        app: smartbill-api
    spec:
      containers:
      - name: api
        image: smartbill-api:v1.0.0
        ports:
        - containerPort: 5001
        env:
        - name: MONGODB_URI
          valueFrom:
            secretKeyRef:
              name: smartbill-secrets
              key: mongodb-uri
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: smartbill-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api
            port: 5001
          initialDelaySeconds: 10
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: smartbill-api-service
spec:
  selector:
    app: smartbill-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5001
  type: LoadBalancer

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongodb-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### **Cloud Deployment Options**

```
Option 1: AWS (Recommended)
───────────────────────────
├─ EC2 for compute
├─ RDS for MongoDB (AWS Document DB)
├─ ElastiCache for Redis
├─ S3 for file storage
├─ CloudFront for CDN
├─ Route53 for DNS
└─ CloudWatch for monitoring

Option 2: Google Cloud
──────────────────────
├─ Cloud Run for containerized apps
├─ Cloud Firestore for database
├─ Memorystore for Redis
├─ Cloud Storage for files
├─ Cloud CDN
└─ Cloud Monitoring

Option 3: Microsoft Azure
──────────────────────────
├─ App Service for backend
├─ Azure Cosmos DB for MongoDB
├─ Azure Cache for Redis
├─ Blob Storage for files
├─ CDN
└─ Application Insights

Option 4: Heroku (Simple)
────────────────────────
├─ Heroku Dynos for Node.js
├─ MongoDB Atlas
├─ Heroku Redis
├─ Built-in logging
└─ Easy deployment via Git
```

---

## 🛡️ System Reliability

### **Disaster Recovery & Backup**

```
┌─────────────────────────────────────────────────────────────┐
│          DISASTER RECOVERY STRATEGY                         │
└─────────────────────────────────────────────────────────────┘

Backup Strategy:
────────────────
├─ Frequency
│  ├─ Daily automated backups
│  ├─ Hourly incremental backups
│  └─ Real-time replication
│
├─ Storage
│  ├─ Primary: Same region
│  ├─ Secondary: Different region
│  └─ Archive: Long-term storage
│
├─ Retention
│  ├─ Daily: 7 days
│  ├─ Weekly: 4 weeks
│  ├─ Monthly: 12 months
│  └─ Yearly: 7 years (compliance)
│
└─ Verification
   ├─ Test restore monthly
   ├─ Validate data integrity
   └─ Document procedures

Disaster Recovery Plan:
───────────────────────
├─ RTO (Recovery Time Objective): 1 hour
├─ RPO (Recovery Point Objective): 15 minutes
│
├─ Failure Scenarios
│  ├─ Database failure
│  │  ├─ Failover to replica
│  │  ├─ Promote secondary
│  │  └─ Restore from backup
│  │
│  ├─ Application crash
│  │  ├─ Auto-restart
│  │  ├─ Load balancer redirect
│  │  └─ Multi-instance deployment
│  │
│  ├─ Network outage
│  │  ├─ Failover to secondary region
│  │  ├─ DNS update
│  │  └─ CDN caching
│  │
│  └─ Data corruption
│     ├─ Detect via integrity checks
│     ├─ Isolate affected data
│     ├─ Restore from backup
│     └─ Investigate root cause
│
└─ Communication Plan
   ├─ Notify stakeholders
   ├─ Update status page
   ├─ Post-incident review
   └─ Improve processes

MongoDB Replica Set:
────────────────────
┌──────────┐
│ Primary  │ (Reads & Writes)
└────┬─────┘
     │
     ├─ Oplog replication
     │
     ├─────────────────┬──────────────────┐
     │                 │                  │
  ┌──▼──┐         ┌──▼──┐          ┌──▼──┐
  │Sec.1│         │Sec.2│          │Sec.3│
  └─────┘         └─────┘          └─────┘
  (Replica)       (Replica)        (Arbiter)

Automatic failover if Primary fails
```

### **High Availability Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│              HIGH AVAILABILITY SETUP                        │
└─────────────────────────────────────────────────────────────┘

Multi-Region Deployment:
────────────────────────
     Region A (Primary)               Region B (Secondary)
    ┌──────────────────┐            ┌──────────────────┐
    │ Load Balancer    │            │ Load Balancer    │
    └────────┬─────────┘            └────────┬─────────┘
             │                               │
    ┌────────┴──────────┐         ┌──────────┴────────┐
    │                   │         │                   │
 ┌──▼──┐ ┌──▼──┐ ┌──▼──┐     ┌──▼──┐ ┌──▼──┐
 │API-1│ │API-2│ │API-3│     │API-4│ │API-5│
 └─────┘ └─────┘ └─────┘     └─────┘ └─────┘
    │                   │         │                   │
    └────────┬──────────┘         └──────────┬────────┘
             │                               │
      ┌──────▼──────────────────────────────▼────────┐
      │  MongoDB Replica Set (Multi-Region)         │
      │  Primary in Region A, Secondaries in B & C   │
      └────────────────────────────────────────────┘
             │
      ┌──────▼───────────┐
      │ Global CDN       │
      │ (CloudFront/etc) │
      └──────────────────┘

Failover Mechanism:
───────────────────
├─ Health Check (every 10 seconds)
│  ├─ API endpoint latency
│  ├─ Database connectivity
│  └─ Cache availability
│
├─ Automatic Failover
│  ├─ Region A down
│  │  └─ Route traffic to Region B
│  │
│  ├─ Database Primary down
│  │  └─ Promote secondary as primary
│  │
│  └─ Service crash
│     └─ Restart via orchestration
│
└─ Manual Intervention
   ├─ Alert operations team
   ├─ Run diagnostics
   └─ Execute recovery procedures
```

---

## 📊 Monitoring & Logging

### **Observability Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│         MONITORING & OBSERVABILITY STACK                    │
└─────────────────────────────────────────────────────────────┘

Application Metrics:
────────────────────
┌─────────────────────────────────────┐
│  Prometheus/StatsD                  │
│  (Metrics Collection)               │
└────────────┬────────────────────────┘
             │
        ┌────▼──────┐
        │            │
    ┌───▼──┐    ┌──▼────┐
    │Grafana│    │DataDog│
    │ (UI) │    │(SaaS) │
    └──────┘    └───────┘

Metrics Collected:
├─ Request metrics
│  ├─ Request rate (req/sec)
│  ├─ Response time (p50, p95, p99)
│  ├─ Error rate (5xx, 4xx)
│  └─ Status code distribution
│
├─ Application metrics
│  ├─ Active connections
│  ├─ JWT validations
│  ├─ Cache hit rate
│  ├─ Database query times
│  └─ Business metrics (bills created, revenue)
│
├─ System metrics
│  ├─ CPU usage
│  ├─ Memory usage
│  ├─ Disk I/O
│  ├─ Network I/O
│  └─ File descriptors
│
└─ Database metrics
   ├─ Query count
   ├─ Query latency
   ├─ Connections
   ├─ Replication lag
   └─ Index usage

Logging Architecture:
────────────────────
┌──────────────────────────────────┐
│ Application Logs (server.js)     │
│ - Request logs                   │
│ - Error logs                     │
│ - Auth logs                      │
│ - Business logic logs            │
└────────┬─────────────────────────┘
         │
    ┌────▼──────────────┐
    │                   │
  ┌─▼─────────┐    ┌───▼────────┐
  │ ELK Stack │    │ Cloudwatch │
  │ or        │    │ or         │
  │ Splunk    │    │ Datadog    │
  └───────────┘    └────────────┘
     Logs          (Aggregation)
     Query
     Analysis
     Alerts

Log Levels:
├─ DEBUG: Development debugging
├─ INFO: General information
├─ WARN: Warning conditions
├─ ERROR: Error conditions
└─ FATAL: Fatal errors

Log Format (JSON):
{
  "timestamp": "2024-05-02T10:30:00Z",
  "level": "INFO",
  "service": "smartbill-api",
  "message": "Bill created successfully",
  "userId": "60f7b3c3d8c4e1a2b3c4d5e6",
  "billId": "60f7b3c3d8c4e1a2b3c4d5e7",
  "duration_ms": 245,
  "requestId": "req-123456"
}

Alerting:
────────
├─ Alert Conditions
│  ├─ Error rate > 5%
│  ├─ Response time p99 > 1s
│  ├─ CPU > 80%
│  ├─ Memory > 85%
│  ├─ Disk > 90%
│  ├─ Database replication lag > 5s
│  └─ Cache down
│
├─ Notification Channels
│  ├─ Email
│  ├─ Slack
│  ├─ PagerDuty
│  ├─ SMS (critical)
│  └─ Dashboard

Dashboards:
──────────
├─ Business Dashboard
│  ├─ Bills created
│  ├─ Revenue
│  ├─ Top products
│  └─ Customer metrics
│
├─ Operational Dashboard
│  ├─ System health
│  ├─ API latency
│  ├─ Error rates
│  ├─ Database performance
│  └─ Cache statistics
│
└─ Security Dashboard
   ├─ Login attempts
   ├─ Failed auth
   ├─ Suspicious activities
   └─ Access patterns
```

---

## 🔄 Data Integrity & Consistency

### **Transaction Management**

```
Database Transactions:
──────────────────────

Bill Creation Transaction:
├─ BEGIN TRANSACTION
│
├─ Save Bill Document
│  ├─ Validate bill data
│  ├─ Insert into bills collection
│  └─ Get bill ID
│
├─ Update Inventory
│  ├─ For each item in bill:
│  │  ├─ Find inventory record
│  │  ├─ Deduct quantity
│  │  └─ Update stock status
│
├─ Update Customer Stats
│  ├─ Find customer
│  ├─ Increment totalBills
│  ├─ Add to totalSpent
│  └─ Update lastPurchase
│
├─ Create Audit Log
│  ├─ Log transaction
│  ├─ Record changes
│  └─ Store timestamp
│
├─ IF any step fails
│  └─ ROLLBACK (undo all changes)
│
└─ ELSE
   ├─ COMMIT (apply all changes)
   └─ Return success
```

---

## 🎯 Future Enhancements

```
Planned Features:
─────────────────

Phase 2:
├─ Mobile app (React Native)
├─ Offline support with sync
├─ Advanced analytics & dashboards
├─ Multi-branch management
├─ Staff performance metrics
└─ Supplier management

Phase 3:
├─ Payment gateway integration
├─ Online store integration
├─ Customer loyalty program
├─ SMS/Email marketing
├─ Predictive analytics
└─ ML-based inventory optimization

Phase 4:
├─ Blockchain for bill authenticity
├─ IoT integration (smart shelves)
├─ Augmented reality catalog
├─ Voice-controlled checkout
└─ Automated supplier ordering
```

---

## 📚 References & Standards

```
Compliance & Standards:
───────────────────────
├─ GST Compliance (India)
├─ GDPR (EU)
├─ CCPA (California)
├─ SOC 2 Type II
├─ ISO 27001 (Security)
├─ REST API Best Practices
└─ Node.js Best Practices

Technologies & Frameworks:
──────────────────────────
├─ OWASP Guidelines
├─ Twelve-Factor App Methodology
├─ Microservices Patterns
└─ Cloud Native Computing
```

---

**System Design Document v1.0** | **Last Updated:** May 2026 | **Maintained By:** SmartBill Architecture Team

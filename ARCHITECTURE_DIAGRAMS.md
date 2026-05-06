# SmartBill POS - Visual Architecture Guide

**Version:** 1.0.0 | **Last Updated:** May 2026

---

## 📐 Complete System Architecture Diagram

```
╔════════════════════════════════════════════════════════════════════════════╗
║                       SMARTBILL POS SYSTEM OVERVIEW                       ║
╚════════════════════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────────────────────┐
│                          PRESENTATION LAYER                                │
│                         (Client-Side / React)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│   │   Billing    │  │  Inventory   │  │  Customers   │  │  Reports &   │   │
│   │   Interface  │  │  Management  │  │  Management  │  │  Analytics   │   │
│   └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│   │   Product    │  │   Payment    │  │   Profile &  │                     │
│   │   Scanner    │  │   Modal      │  │   Settings   │                     │
│   └──────────────┘  └──────────────┘  └──────────────┘                     │
│                                                                              │
│   State Management: AuthContext (Redux alternative)                        │
│   Styling: Tailwind CSS + Component-level CSS                             │
│   HTTP Client: Fetch API                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │  REST API (HTTP/HTTPS)        │
                    │  JWT Token Authentication      │
                    │  CORS Enabled                  │
                    └───────────────┬───────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                                    │
│                      (Backend / Express.js)                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                        MIDDLEWARE STACK                            │   │
│   │  ├─ CORS Handler                                                   │   │
│   │  ├─ JSON Body Parser (50MB limit)                                │   │
│   │  ├─ Session Validator                                            │   │
│   │  ├─ Request Logger                                               │   │
│   │  └─ Error Handler                                                │   │
│   └────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                        ROUTE HANDLERS                              │  │
│   │                                                                      │  │
│   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │  │
│   │  │  /api/auth   │  │ /api/products│  │  /api/bills  │             │  │
│   │  │              │  │              │  │              │             │  │
│   │  │ - signup     │  │ - list all   │  │ - create     │             │  │
│   │  │ - login      │  │ - get by id  │  │ - get list   │             │  │
│   │  │ - verify JWT │  │ - search     │  │ - get by id  │             │  │
│   │  │ - user info  │  │ - by barcode │  │ - daily stat │             │  │
│   │  │ - logout     │  │ - CRUD       │  │ - reports    │             │  │
│   │  └──────────────┘  └──────────────┘  └──────────────┘             │  │
│   │                                                                      │  │
│   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │  │
│   │  │ /api/inventory│ │/api/customers│  │   /health   │             │  │
│   │  │              │  │              │  │   /api      │             │  │
│   │  │ - list       │  │ - list       │  │             │             │  │
│   │  │ - get by id  │  │ - get by id  │  │ Status info │             │  │
│   │  │ - add stock  │  │ - create     │  │             │             │  │
│   │  │ - update qty │  │ - update     │  │             │             │  │
│   │  │ - alerts     │  │ - delete     │  │             │             │  │
│   │  └──────────────┘  └──────────────┘  └──────────────┘             │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────┐    │
│   │                       SERVICE LAYER                              │    │
│   │              (Business Logic & Data Processing)                  │    │
│   │                                                                   │    │
│   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │    │
│   │  │  Bill        │  │  Inventory   │  │  Customer    │           │    │
│   │  │  Service     │  │  Service     │  │  Service     │           │    │
│   │  │              │  │              │  │              │           │    │
│   │  │- Calculate   │  │- Check stock │  │- Find/Add    │           │    │
│   │  │  totals      │  │- Low alerts  │  │  customers   │           │    │
│   │  │- Generate    │  │- Reorder qty │  │- Update      │           │    │
│   │  │  bill number │  │- Status      │  │  purchase    │           │    │
│   │  │- Tax calc    │  │  update      │  │  history     │           │    │
│   │  │- PDF gen     │  │- Warehouse   │  │- Total spent │           │    │
│   │  └──────────────┘  └──────────────┘  └──────────────┘           │    │
│   │                                                                   │    │
│   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │    │
│   │  │  Cache       │  │  Session     │  │  WhatsApp    │           │    │
│   │  │  Service     │  │  Service     │  │  Service     │           │    │
│   │  │              │  │              │  │              │           │    │
│   │  │- Get cached  │  │- Create      │  │- Send bill   │           │    │
│   │  │  data        │  │  session     │  │- Send alerts │           │    │
│   │  │- Set cache   │  │- Validate    │  │- Format msg  │           │    │
│   │  │- Invalidate  │  │  session     │  │              │           │    │
│   │  │- TTL mgmt    │  │- Update      │  │              │           │    │
│   │  │              │  │  activity    │  │              │           │    │
│   │  └──────────────┘  └──────────────┘  └──────────────┘           │    │
│   └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │  Mongoose ODM                 │
                    │  Query Builder & Validation   │
                    │  Connection Pooling           │
                    └───────────────┬───────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                     DATA ACCESS & STORAGE LAYER                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────┐    │
│   │                    DATABASE COLLECTIONS                         │    │
│   │                                                                   │    │
│   │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐       │    │
│   │  │  Users        │  │  Products     │  │  Bills        │       │    │
│   │  │  (Accounts)   │  │  (Catalog)    │  │  (Invoices)   │       │    │
│   │  ├───────────────┤  ├───────────────┤  ├───────────────┤       │    │
│   │  │ username      │  │ name          │  │ billNumber    │       │    │
│   │  │ email         │  │ price         │  │ items[]       │       │    │
│   │  │ password      │  │ category      │  │ subtotal      │       │    │
│   │  │ role          │  │ barcode       │  │ tax           │       │    │
│   │  │ shopName      │  │ priceHistory[]│  │ discount      │       │    │
│   │  │ gstRate       │  │ image         │  │ total         │       │    │
│   │  │ lastLogin     │  │ active        │  │ paymentMethod │       │    │
│   │  │ timestamps    │  │ timestamps    │  │ customerName  │       │    │
│   │  │ ...           │  │ ...           │  │ ...           │       │    │
│   │  └───────────────┘  └───────────────┘  └───────────────┘       │    │
│   │                                                                   │    │
│   │  ┌───────────────┐  ┌───────────────┐                          │    │
│   │  │  Customers    │  │  Inventory    │                          │    │
│   │  │  (CRM Data)   │  │  (Stock)      │                          │    │
│   │  ├───────────────┤  ├───────────────┤                          │    │
│   │  │ name          │  │ productId     │                          │    │
│   │  │ mobileNumber  │  │ barcode       │                          │    │
│   │  │ address       │  │ quantity      │                          │    │
│   │  │ city/state    │  │ minStock      │                          │    │
│   │  │ gstNumber     │  │ maxStock      │                          │    │
│   │  │ totalBills    │  │ location      │                          │    │
│   │  │ totalSpent    │  │ warehouse     │                          │    │
│   │  │ isActive      │  │ status        │                          │    │
│   │  │ ...           │  │ ...           │                          │    │
│   │  └───────────────┘  └───────────────┘                          │    │
│   └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────┐    │
│   │                    MONGODB FEATURES USED                         │    │
│   │                                                                   │    │
│   │  ├─ Schema Validation (Mongoose)                               │    │
│   │  ├─ Compound Indexing for queries                              │    │
│   │  ├─ Foreign Key relationships (via ObjectId)                   │    │
│   │  ├─ Atomic operations on documents                             │    │
│   │  ├─ TTL Indices (for session expiry)                           │    │
│   │  ├─ Aggregation Pipeline (for reports)                         │    │
│   │  └─ Transactions (multi-document ACID)                         │    │
│   └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────┐    │
│   │                   CACHE LAYER (Redis)                           │    │
│   │                                                                   │    │
│   │  ├─ Session Store (User sessions)                              │    │
│   │  ├─ Product Cache (Frequently accessed products)               │    │
│   │  ├─ Inventory Cache (Stock levels)                             │    │
│   │  ├─ Bill Cache (Recent bills)                                  │    │
│   │  └─ Query Result Cache (Analytics data)                        │    │
│   └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────┐    │
│   │               EXTERNAL INTEGRATIONS                             │    │
│   │                                                                   │    │
│   │  ├─ Google Gemini API (AI for product recognition)             │    │
│   │  ├─ WhatsApp Business API (Notifications)                      │    │
│   │  ├─ SMTP Server (Email notifications)                          │    │
│   │  └─ File Storage (S3 or local for images/PDFs)                 │    │
│   └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
```

---

## 🔄 Bill Creation Workflow Diagram

```
╔══════════════════════════════════════════════════════════════════════════╗
║                     BILL CREATION WORKFLOW                              ║
╚══════════════════════════════════════════════════════════════════════════╝

START
  │
  ├─→ User scans/selects product
  │   └─→ Frontend → GET /api/products/barcode/:barcode
  │       └─→ Check Redis cache (HIT/MISS)
  │           └─→ If MISS: Query MongoDB
  │               └─→ Cache result in Redis
  │               └─→ Return product details
  │
  ├─→ System validates stock availability
  │   ├─ Check Inventory collection
  │   └─ If quantity > 0: Display product
  │
  ├─→ User adds quantity and item to cart
  │   └─→ Frontend state updates (React)
  │
  ├─→ User selects customer (optional)
  │   └─→ Frontend → GET /api/customers?search=...
  │
  ├─→ System calculates bill totals
  │   ├─ Subtotal = Σ(item.price × item.quantity)
  │   ├─ Tax = Subtotal × (gstRate / 100)
  │   ├─ Total = Subtotal + Tax - Discount
  │   └─→ Display in real-time
  │
  ├─→ User selects payment method
  │   ├─ Cash
  │   ├─ Card
  │   ├─ UPI
  │   └─ Check
  │
  ├─→ User enters amount received
  │   └─→ System calculates change
  │
  ├─→ User confirms payment
  │   │
  │   └─→ Frontend → POST /api/bills
  │       {
  │         items: [...],
  │         subtotal, tax, discount, total,
  │         paymentMethod, amountReceived,
  │         customerName, customerMobile,
  │         shopName, shopAddress, gstNumber
  │       }
  │       │
  │       └─→ Backend processing:
  │           │
  │           ├─→ Validate request (express-validator)
  │           │
  │           ├─→ Verify JWT token
  │           │
  │           ├─→ Validate inventory for each item
  │           │   └─→ Query Inventory collection
  │           │
  │           ├─→ Generate unique bill number
  │           │   └─→ Format: BILL-<timestamp>
  │           │
  │           ├─→ START TRANSACTION
  │           │   ├─→ Save Bill document
  │           │   │   └─→ Insert with userId
  │           │   │
  │           │   ├─→ Deduct inventory for each item
  │           │   │   └─→ Inventory.quantity -= item.quantity
  │           │   │
  │           │   ├─→ Update inventory status
  │           │   │   └─→ If quantity ≤ minStock: "Low Stock"
  │           │   │   └─→ If quantity = 0: "Out of Stock"
  │           │   │
  │           │   ├─→ Update customer (if provided)
  │           │   │   ├─→ Customer.totalBills++
  │           │   │   ├─→ Customer.totalSpent += bill.total
  │           │   │   └─→ Customer.lastPurchase = now
  │           │   │
  │           │   ├─→ Cache bill data in Redis
  │           │   │   └─→ TTL: 10 minutes
  │           │   │
  │           │   └─→ Generate PDF
  │           │       └─→ Use html2pdf library
  │           │
  │           ├─→ END TRANSACTION (COMMIT)
  │           │
  │           └─→ Return response with bill details
  │               {
  │                 success: true,
  │                 data: {
  │                   _id, billNumber, total,
  │                   pdfUrl, ...
  │                 }
  │               }
  │
  ├─→ Frontend displays bill summary
  │   ├─ Bill number
  │   ├─ Items and amounts
  │   ├─ Total and change
  │   └─ Print/Download/Email buttons
  │
  ├─→ User actions
  │   ├─ Print bill → Print dialog opens
  │   ├─ Download → Save as PDF
  │   ├─ Send via WhatsApp → API call to WhatsApp service
  │   └─ Email → Send via email service
  │
  └─→ END - Bill successfully created

═══════════════════════════════════════════════════════════════════════════════
```

---

## 🔐 Authentication & Authorization Flow

```
╔══════════════════════════════════════════════════════════════════════════╗
║                  AUTHENTICATION & AUTHORIZATION FLOW                     ║
╚══════════════════════════════════════════════════════════════════════════╝

LOGIN PROCESS:
──────────────

User Enters Credentials
    │
    └─→ Frontend: POST /api/auth/login
        {
          username: "john_doe",
          password: "mypassword"
        }
        │
        └─→ Backend validates:
            │
            ├─→ Find user in DB by username
            │   └─→ User.findOne({ username: "john_doe" })
            │
            ├─→ Compare provided password with hash
            │   └─→ bcryptjs.compare(password, user.password)
            │       ├─ Match? Continue
            │       └─ No match? Return 401 Unauthorized
            │
            ├─→ Verify user is active
            │   └─→ user.active === true ?
            │       ├─ Yes? Continue
            │       └─ No? Return 403 Forbidden
            │
            ├─→ Generate JWT Token
            │   {
            │     Header: { alg: "HS256", typ: "JWT" }
            │     Payload: { id: userId, iat, exp: now + 7d }
            │     Signature: HMACSHA256(header.payload, JWT_SECRET)
            │   }
            │
            ├─→ Create Redis Session
            │   {
            │     sessionId: "unique-id",
            │     userId: "60f7b3c3d8c4e1a2b3c4d5e6",
            │     username: "john_doe",
            │     role: "Cashier",
            │     loginTime: now,
            │     lastActivity: now,
            │     expiresAt: now + 24h
            │   }
            │
            └─→ Return Response
                {
                  success: true,
                  token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
                  sessionId: "session-12345",
                  user: {
                    _id: "60f7b3c3d8c4e1a2b3c4d5e6",
                    username: "john_doe",
                    email: "john@example.com",
                    role: "Cashier",
                    shopName: "My Shop"
                  }
                }

Frontend Stores:
├─ localStorage['authToken'] = token
├─ localStorage['user'] = user object
└─ localStorage['sessionId'] = sessionId


SUBSEQUENT REQUESTS:
────────────────────

User Makes API Request (e.g., POST /api/bills)
    │
    ├─→ Frontend includes headers:
    │   {
    │     "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    │     "X-Session-ID": "session-12345",
    │     "Content-Type": "application/json"
    │   }
    │
    └─→ Backend processes:
        │
        ├─→ Extract token from Authorization header
        │   └─→ Split "Bearer <token>" → get token
        │
        ├─→ Verify JWT signature
        │   ├─→ Decode token using JWT_SECRET
        │   ├─→ Check signature validity
        │   ├─ Valid? Continue
        │   └─ Invalid? Return 401 "Invalid token"
        │
        ├─→ Check token expiry
        │   ├─→ exp > currentTime ?
        │   ├─ Yes? Continue
        │   └─ No? Return 401 "Token expired"
        │
        ├─→ Extract user ID from token
        │   └─→ req.user = decoded payload
        │       └─→ { id: "60f7b3c3d8c4e1a2b3c4d5e6", iat, exp }
        │
        ├─→ Validate session (optional)
        │   ├─→ Get session from Redis using sessionId
        │   ├─→ Update lastActivity: now
        │   └─→ Check if still valid
        │
        ├─→ Check user role & permissions
        │   └─→ if (user.role === "Cashier" && isCreateBillEndpoint) {
        │       ├─ Allow? Continue
        │       └─ No? Return 403 "Forbidden"
        │
        ├─→ Process request
        │   └─→ Execute business logic with req.user.id
        │
        └─→ Return response


ROLE-BASED ACCESS CONTROL:
──────────────────────────

Roles & Permissions Matrix:

                      │ Cashier │ Manager │ Admin
────────────────────────────────────────────────────
Create Bills          │   ✓     │   ✓     │   ✓
View Own Bills        │   ✓     │   ✓     │   ✓
View All Bills        │   ✗     │   ✓     │   ✓
Manage Inventory      │   ✗     │   ✓     │   ✓
Manage Customers      │   ✗     │   ✓     │   ✓
View Reports          │   ✗     │   ✓     │   ✓
Manage Users          │   ✗     │   ✗     │   ✓
System Settings       │   ✗     │   ✗     │   ✓
────────────────────────────────────────────────────


LOGOUT PROCESS:
───────────────

User clicks Logout
    │
    ├─→ Frontend: POST /api/auth/logout
    │   └─→ Send token + sessionId
    │
    └─→ Backend:
        ├─→ Delete session from Redis
        ├─→ Verify token is invalidated
        └─→ Return success

Frontend cleanup:
├─ Remove localStorage['authToken']
├─ Remove localStorage['user']
├─ Remove localStorage['sessionId']
└─ Redirect to login page

═══════════════════════════════════════════════════════════════════════════════
```

---

## 📊 Data Model Relationships

```
╔══════════════════════════════════════════════════════════════════════════╗
║                       DATABASE RELATIONSHIPS                            ║
╚══════════════════════════════════════════════════════════════════════════╝

                              USER
                           ┌──────┐
                           │  _id │ (PK)
                           │ name │
                           │ role │
                           │  ... │
                           └───┬──┘
                               │ (1:N) owns/creates
                    ┌──────────┼──────────┬──────────┐
                    │          │          │          │
                    │          │          │          │
        ┌───────────▼──┐  ┌────▼────┐  ┌─▼────────┐ │
        │    BILL      │  │INVENTORY │  │CUSTOMER  │ │
        ├──────────────┤  ├──────────┤  ├──────────┤ │
        │    _id (PK)  │  │   _id    │  │   _id    │ │
        │  billNumber  │  │productId │  │  name    │ │
        │ userId (FK) ─┼─→│(FK)──────┼─→│mobile    │ │
        │   items[]    │  │quantity  │  │address   │ │
        │   total      │  │location  │  │gstNumber │ │
        │   ...        │  │   ...    │  │totalBills│ │
        └────────┬─────┘  └──────────┘  │totalSpent│ │
                 │                       │ isActive │ │
                 │                       └──────────┘ │
                 │                                     │
        Bill.items[]                                   │
        references                                     │
        Product._id ──────────────────────────────────┘
                 │
                 │
        ┌────────▼──────────┐
        │    PRODUCT        │
        ├───────────────────┤
        │      _id (PK)     │
        │      name         │
        │      price        │
        │      category     │
        │      barcode      │
        │      priceHistory │
        │      image        │
        │      active       │
        │      ...          │
        └──────────────────┘


Key Relationships:
──────────────────

1. User → Bill (1:N)
   └─ One user can have many bills
   └─ Bill.userId references User._id

2. User → Customer (1:N)
   └─ One user can have many customers
   └─ Customer.userId references User._id

3. User → Inventory (Implicit via Product)
   └─ One user's products have inventory

4. Product → Bill.items (1:N)
   └─ One product can be in many bills
   └─ Bill.items[].productId references Product._id

5. Product → Inventory (1:1)
   └─ Each product has one inventory record
   └─ Inventory.productId references Product._id

6. Customer → Bill (via customer info)
   └─ Customer can have many bills
   └─ Bill.customerName and customerMobile link to Customer


Access Patterns:
────────────────

Query 1: Get user's bills
    db.bills.find({ userId: ObjectId(...) })

Query 2: Get product by barcode
    db.products.findOne({ barcode: "123456" })

Query 3: Check inventory level
    db.inventory.find({ productId: ObjectId(...) })

Query 4: Get customer's purchase history
    db.bills.find({ customerMobile: "9876543210" })

Query 5: Low stock alert
    db.inventory.find({
      $expr: { $lt: ["$quantity", "$minStock"] }
    })

═══════════════════════════════════════════════════════════════════════════════
```

---

## 🔌 Component Dependency Graph

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    COMPONENT DEPENDENCY HIERARCHY                       ║
╚══════════════════════════════════════════════════════════════════════════╝

                            App.jsx
                              │
                ┌─────────────┼──────────────┐
                │             │              │
                ▼             ▼              ▼
           ┌─────────┐   ┌──────────┐   ┌─────────┐
           │ Login   │   │ Signup   │   │ Main    │
           │ Page    │   │ Page     │   │ App     │
           └─────────┘   └──────────┘   │(Auth'd) │
                                         └────┬────┘
                                              │
                        ┌─────────────────────┼─────────────────────┐
                        │                     │                     │
                        ▼                     ▼                     ▼
                  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
                  │ AppNavbar    │      │ Tabs Router  │      │ ProfilePage  │
                  └──────────────┘      └──────┬───────┘      └──────────────┘
                                               │
                  ┌────────────────────────────┼────────────────────────┐
                  │                            │                        │
                  ▼                            ▼                        ▼
            ┌────────────────┐       ┌──────────────────┐     ┌────────────────┐
            │  Billing Tab   │       │  Inventory Tab   │     │  Customers Tab │
            ├────────────────┤       ├──────────────────┤     ├────────────────┤
            │                │       │                  │     │                │
            │ ┌────────────┐ │       │ ┌──────────────┐ │     │ ┌────────────┐ │
            │ │ Scanner    │ │       │ │ Manager      │ │     │ │ Manager    │ │
            │ └────────────┘ │       │ └──────────────┘ │     │ └────────────┘ │
            │                │       │                  │     │                │
            │ ┌────────────┐ │       │ ┌──────────────┐ │     └────────────────┘
            │ │ Bill Items │ │       │ │ Stock List   │ │
            │ └────────────┘ │       │ └──────────────┘ │
            │                │       │                  │
            │ ┌────────────┐ │       │ ┌──────────────┐ │
            │ │ Bill       │ │       │ │ Low Stock    │ │
            │ │ Display    │ │       │ │ Alerts       │ │
            │ └────────────┘ │       │ └──────────────┘ │
            │                │       │                  │
            │ ┌────────────┐ │       └──────────────────┘
            │ │ Payment    │ │
            │ │ Modal      │ │
            │ └────────────┘ │
            └────────────────┘
                  ▲
                  │
                  └──────── Uses ────────┐
                                          │
                          ┌───────────────▼──────────────┐
                          │   Shared UI Components       │
                          │ (Layout folder)              │
                          ├──────────────────────────────┤
                          │ • Button.jsx                 │
                          │ • Card.jsx                   │
                          │ • Form.jsx                   │
                          │ • Table.jsx                  │
                          │ • Alert.jsx                  │
                          │ • Badge.jsx                  │
                          │ • Toast.jsx                  │
                          └──────────────────────────────┘
                                    ▲
                                    │
                                    │ Uses
                                    │
                          ┌─────────┴──────────┐
                          │                    │
                   ┌──────▼──────┐      ┌─────▼──────┐
                   │ AuthContext │      │ Services   │
                   │             │      │            │
                   │ • user      │      │ • quaggaS. │
                   │ • token     │      │ • api      │
                   │ • login()   │      │ • cache    │
                   │ • logout()  │      └────────────┘
                   └─────────────┘

═══════════════════════════════════════════════════════════════════════════════
```

---

## 🚀 Request-Response Flow Diagram

```
╔══════════════════════════════════════════════════════════════════════════╗
║              REQUEST-RESPONSE FLOW FOR BILL CREATION                    ║
╚══════════════════════════════════════════════════════════════════════════╝

CLIENT                                    SERVER
──────────────────────────────────────────────────────────────────────────

User Form Submit
       │
       ├─→ Validate form data (client-side)
       │   ├─ Check required fields
       │   ├─ Validate data types
       │   └─ Check constraints
       │
       ├─→ POST /api/bills
       │   {
       │     "Authorization": "Bearer <JWT>",
       │     "Content-Type": "application/json"
       │   }
       │   Body: {
       │     items: [...],
       │     subtotal: 1000,
       │     tax: 50,
       │     discount: 0,
       │     total: 1050,
       │     paymentMethod: "Cash",
       │     amountReceived: 1100,
       │     customerName: "John Doe",
       │     ...
       │   }
       │
       │────────────────────────────────────────→ Middleware:
       │                                          ├─ Parse JSON
       │                                          ├─ CORS check
       │                                          └─ Log request
       │
       │                                          Auth Middleware:
       │                                          ├─ Extract token
       │                                          ├─ Verify JWT
       │                                          └─ Extract user ID
       │
       │                                          Request Handler:
       │                                          ├─ body validation
       │                                          └─ inventory check
       │
       │                                          Service Layer:
       │                                          ├─ generateBillNumber()
       │                                          ├─ calculateTotals()
       │                                          ├─ updateInventory()
       │                                          ├─ updateCustomer()
       │                                          └─ cacheBill()
       │
       │                                          Database:
       │                                          ├─ INSERT bills
       │                                          ├─ UPDATE inventory
       │                                          ├─ UPDATE customers
       │                                          └─ COMMIT transaction
       │
       │◀────────────────────────────────────────
       │
       │ HTTP 200 OK
       │ {
       │   success: true,
       │   message: "Bill created successfully",
       │   data: {
       │     _id: "60f7b3c3d8c4e1a2b3c4d5e7",
       │     billNumber: "BILL-1609459200000",
       │     total: 1050,
       │     paymentStatus: "Completed",
       │     pdfUrl: "blob:http://...",
       │     createdAt: "2024-05-02T10:30:00Z"
       │   }
       │ }
       │
       │
       ├─ Update UI with success
       │
       ├─ Show bill preview
       │
       └─ Offer print/download/email options


ERROR SCENARIOS:
────────────────

1. Invalid Token (401)
   └─ Response: { error: "Invalid token" }
      Client: Redirect to login page

2. Insufficient Stock (400)
   └─ Response: { error: "Out of stock for item X" }
      Client: Show alert, allow quantity adjustment

3. Database Error (500)
   └─ Response: { error: "Internal server error" }
      Client: Retry with exponential backoff

4. Validation Error (400)
   └─ Response: {
        success: false,
        errors: [
          { field: "items[0].quantity", message: "must be > 0" }
        ]
      }
      Client: Show form errors, allow correction

═══════════════════════════════════════════════════════════════════════════════
```

---

## 📈 Scalability Architecture

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    HORIZONTAL SCALING DESIGN                            ║
╚══════════════════════════════════════════════════════════════════════════╝

WITHOUT SCALING (Single Instance):
──────────────────────────────────

┌──────────────┐
│ Client       │
│ Requests     │
└───────┬──────┘
        │
        ▼
┌──────────────────────────┐
│  Node.js Server          │
│  ├─ Express.js           │
│  ├─ Routes               │
│  └─ Services             │
└────────────┬─────────────┘
             │
        ┌────▼──────┐
        │ MongoDB    │
        │ (Single)   │
        └────────────┘

Issues: Single point of failure, limited throughput


WITH SCALING (Multi-Instance):
──────────────────────────────

                         ┌─────────────────────┐
                         │  Client Requests    │
                         │  (10K+ req/sec)     │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │    DNS Load Balancer           │
                    │  (Route53, CloudFront, etc)    │
                    └───────────────┬───────────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                │                   │                   │
                ▼                   ▼                   ▼
        ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
        │   Server 1   │    │   Server 2   │    │   Server N   │
        │  (Zone A)    │    │  (Zone B)    │    │  (Zone C)    │
        ├──────────────┤    ├──────────────┤    ├──────────────┤
        │  Node.js     │    │  Node.js     │    │  Node.js     │
        │  Port 5001   │    │  Port 5001   │    │  Port 5001   │
        │              │    │              │    │              │
        │ ┌──────────┐ │    │ ┌──────────┐ │    │ ┌──────────┐ │
        │ │Connection│ │    │ │Connection│ │    │ │Connection│ │
        │ │Pool      │ │    │ │Pool      │ │    │ │Pool      │ │
        │ └──────────┘ │    │ └──────────┘ │    │ └──────────┘ │
        └──────┬───────┘    └──────┬───────┘    └──────┬───────┘
               │                   │                   │
               └───────────────────┼───────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
         ▼                         ▼                         ▼
    ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
    │ Redis Cache  │          │   MongoDB    │          │   CDN Edge   │
    │ (In-Memory)  │          │  Replica Set │          │  (Static)    │
    │              │          │              │          │              │
    │ Primary:     │          │ Primary:     │          │ Images, CSS, │
    │ In memory    │          │ Region A     │          │ JS bundles   │
    │              │          │              │          │              │
    │ Secondary:   │          │ Secondaries: │          │ TTL: 30 days │
    │ For failover │          │ Regions B,C  │          │              │
    └──────────────┘          └──────────────┘          └──────────────┘

Advantages:
├─ No single point of failure
├─ Horizontal load balancing
├─ Geographic distribution
├─ Database replication
├─ Caching layer
├─ CDN for static assets
└─ Automatic failover


METRICS & MONITORING:
─────────────────────

Per Instance:
├─ CPU usage: < 70%
├─ Memory: < 80%
├─ Active connections: monitored
├─ Request latency: p99 < 1s
└─ Error rate: < 1%

Cluster Level:
├─ Total throughput: > 10K req/sec
├─ Database replication lag: < 1s
├─ Cache hit rate: > 80%
├─ API availability: 99.9%
└─ Network latency: < 100ms

═══════════════════════════════════════════════════════════════════════════════
```

---

## 🔐 Security Layers Overview

```
╔══════════════════════════════════════════════════════════════════════════╗
║                        SECURITY ARCHITECTURE                            ║
╚══════════════════════════════════════════════════════════════════════════╝

                          EXTERNAL THREATS
                                 │
                                 ▼
                    ┌────────────────────────────┐
                    │  Layer 1: Network Security │
                    ├────────────────────────────┤
                    │ • HTTPS/TLS 1.2+           │
                    │ • WAF (Web App Firewall)   │
                    │ • DDoS Protection          │
                    │ • Rate Limiting            │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 2: Authentication    │
                    ├────────────────────────────┤
                    │ • Username/Password auth   │
                    │ • bcryptjs hashing         │
                    │ • JWT tokens               │
                    │ • Session management       │
                    │ • MFA (Optional)           │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 3: Authorization     │
                    ├────────────────────────────┤
                    │ • Role-based access (RBAC) │
                    │ • Resource ownership check │
                    │ • Permission validation    │
                    │ • API key validation       │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 4: Input Validation  │
                    ├────────────────────────────┤
                    │ • Client-side validation   │
                    │ • Server-side validation   │
                    │ • Type checking            │
                    │ • XSS prevention           │
                    │ • SQL injection prevention │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 5: Data Protection   │
                    ├────────────────────────────┤
                    │ • Encrypted passwords      │
                    │ • Encrypted sensitive data │
                    │ • TLS in transit           │
                    │ • Field-level encryption   │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 6: Audit & Logging   │
                    ├────────────────────────────┤
                    │ • Access logging           │
                    │ • Error logging            │
                    │ • Change tracking          │
                    │ • Compliance audits        │
                    └──────────┬─────────────────┘
                               │
                               ▼
                    ┌────────────────────────────┐
                    │ Layer 7: Infrastructure    │
                    ├────────────────────────────┤
                    │ • Network segmentation     │
                    │ • Database encryption      │
                    │ • Secrets management       │
                    │ • Secure backups           │
                    │ • VPC/Private networks     │
                    └────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
```

---

## 📊 Technology Stack Visualization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMPLETE TECH STACK                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   FRONTEND                           BACKEND                                │
│   ┌─────────────────────┐            ┌──────────────────────┐              │
│   │ React 18 + Vite     │            │ Node.js + Express.js │              │
│   │                     │            │                      │              │
│   ├─────────────────────┤            ├──────────────────────┤              │
│   │ Components:         │            │ Routing:             │              │
│   │ ├─ Functional       │            │ ├─ Express Router    │              │
│   │ ├─ Hooks            │            │ ├─ Middleware        │              │
│   │ └─ Context API      │            │ └─ Error handler     │              │
│   │                     │            │                      │              │
│   ├─────────────────────┤            ├──────────────────────┤              │
│   │ Styling:            │            │ Database:            │              │
│   │ ├─ Tailwind CSS     │            │ ├─ MongoDB 5.0+      │              │
│   │ ├─ Component CSS    │            │ ├─ Mongoose ODM      │              │
│   │ └─ Utility classes  │            │ └─ Replica Set       │              │
│   │                     │            │                      │              │
│   ├─────────────────────┤            ├──────────────────────┤              │
│   │ Libraries:          │            │ Caching:             │              │
│   │ ├─ Lucide React     │            │ ├─ Redis             │              │
│   │ ├─ html2pdf         │            │ ├─ Session store     │              │
│   │ ├─ ZXing (Barcode)  │            │ └─ Query cache       │              │
│   │ └─ Quagga (QR)      │            │                      │              │
│   │                     │            ├──────────────────────┤              │
│   ├─────────────────────┤            │ Security:            │              │
│   │ Build:              │            │ ├─ JWT (jsonwebtoken)│              │
│   │ ├─ Vite            │            │ ├─ bcryptjs          │              │
│   │ ├─ ESLint          │            │ ├─ CORS              │              │
│   │ └─ npm             │            │ └─ Validation        │              │
│   │                     │            │                      │              │
│   └─────────────────────┘            ├──────────────────────┤              │
│                                      │ APIs:                │              │
│   FRONTEND (localhost:5173)           │ ├─ Google Gemini     │              │
│                                      │ ├─ WhatsApp API      │              │
│                                      │ ├─ Email SMTP        │              │
│                                      │ └─ Payment Gateway    │              │
│                                      │                      │              │
│                                      └──────────────────────┘              │
│                                                                              │
│                            BACKEND (localhost:5001)                        │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                        DEPLOYMENT & INFRASTRUCTURE                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Development:                 Production:                                  │
│  ├─ Local machines            ├─ Cloud Platform (AWS/GCP/Azure)           │
│  ├─ npm dev servers           ├─ Kubernetes/Docker                        │
│  ├─ MongoDB Atlas              ├─ Managed Database                        │
│  └─ ngrok for testing         ├─ CDN (CloudFront/Cloudflare)             │
│                               ├─ Load Balancer                            │
│                               ├─ CI/CD (GitHub Actions/GitLab)            │
│                               ├─ Monitoring (Prometheus/Datadog)          │
│                               └─ Logging (ELK/CloudWatch)                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

**Visual Architecture Guide v1.0** | **Last Updated:** May 2026 | **SmartBill Documentation Team**

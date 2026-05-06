# SmartBill POS System - Comprehensive Documentation

**Version:** 1.0.0 | **Status:** Production Ready | **Last Updated:** May 2026

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Core Features](#core-features)
4. [Technology Stack](#technology-stack)
5. [Project Structure](#project-structure)
6. [Database Models](#database-models)
7. [API Endpoints](#api-endpoints)
8. [Frontend Components](#frontend-components)
9. [Installation & Setup](#installation--setup)
10. [Configuration](#configuration)
11. [Usage Guide](#usage-guide)
12. [Deployment](#deployment)
13. [Troubleshooting](#troubleshooting)

---

## 🎯 Project Overview

**SmartBill POS** is an enterprise-grade Point of Sale (POS) system designed for retail businesses, shops, and small enterprises. The system provides a complete billing and inventory management solution with AI-powered features, barcode scanning, and real-time inventory tracking.

### Key Objectives
- ✅ Streamline retail billing operations
- ✅ Provide real-time inventory management
- ✅ Enable multi-user access with role-based controls
- ✅ Integrate AI for intelligent product recognition
- ✅ Maintain comprehensive sales records and analytics
- ✅ Support multiple payment methods
- ✅ Generate professional bills with GST compliance

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT TIER (Frontend)                      │
│  React 18 + Vite | Tailwind CSS | Barcode Scanner | Gemini AI  │
├─────────────────────────────────────────────────────────────────┤
│                   Network Layer (HTTP/REST)                     │
│               CORS Enabled | JWT Token Auth                     │
├─────────────────────────────────────────────────────────────────┤
│                  SERVER TIER (Backend Layer)                    │
│  Express.js Server | Route Controllers | Service Layer         │
│         JWT Verification | Session Validation                   │
├─────────────────────────────────────────────────────────────────┤
│              DATA ACCESS LAYER (Business Logic)                 │
│   Bill Service | Inventory Service | Cache Service             │
│   Gemini AI Service | WhatsApp Service                          │
├─────────────────────────────────────────────────────────────────┤
│              PERSISTENCE LAYER (Database)                       │
│     MongoDB (Mongoose) | Redis Cache Layer                      │
│   Users | Products | Bills | Customers | Inventory             │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Flow

```
User Request
    ↓
[Browser/React App]
    ↓
API Call with JWT Token
    ↓
[Express Server] → Session Middleware → Token Verification
    ↓
Route Handler (auth/products/bills/inventory/customers)
    ↓
Service Layer (Business Logic)
    ↓
MongoDB Query ← (Redis Cache Layer)
    ↓
Response JSON
    ↓
[React Component] → State Update → UI Render
```

---

## ✨ Core Features

### 1. **Authentication & User Management**
- User registration and login
- JWT-based token authentication
- Session tracking via Redis
- Role-based access control (Cashier, Manager, Admin)
- Shop profile management with GST details
- Password encryption using bcryptjs

### 2. **Billing System**
- Create new bills with multiple items
- Real-time cart management
- Dynamic tax calculation (GST support)
- Discount application
- Multiple payment methods (Cash, Card, UPI, Check)
- Automatic bill numbering
- Bill history and retrieval
- PDF bill generation and printing

### 3. **Product Management**
- CRUD operations for products
- Product categorization
- Price history tracking with effective dates
- Barcode and SKU management
- Product images and metadata
- Search and filtering by category, name, or barcode
- Price change audit trail

### 4. **Inventory System**
- Real-time stock tracking
- Minimum and maximum stock levels
- Stock status alerts (In Stock, Low Stock, Out of Stock)
- Multi-warehouse support
- Location-based organization
- Reorder level management
- Stock movement history
- Inventory reports and analytics

### 5. **Customer Management**
- Customer database with contact details
- Purchase history per customer
- Total spent tracking
- Customer segmentation
- Active/inactive customer status
- Duplicate phone number prevention

### 6. **AI-Powered Features**
- **Gemini AI Integration**
  - Automatic product recognition from images
  - Extract product details (name, price, category, brand)
  - Confidence level assessment
  - Receipt/bill image parsing
  - Batch product import from images

### 7. **Reporting & Analytics**
- Daily sales reports
- Revenue analytics
- Customer purchase patterns
- Product performance metrics
- Payment method distribution
- Inventory status reports

### 8. **Additional Features**
- WhatsApp bill delivery integration
- Session-based user activity tracking
- Redis caching for performance optimization
- Health check endpoints
- Comprehensive API documentation
- Error handling and logging

---

## 💻 Technology Stack

### **Frontend**
| Technology | Purpose | Version |
|-----------|---------|---------|
| React | UI Framework | 18.2.0 |
| Vite | Build Tool & Dev Server | 5.4.21 |
| Tailwind CSS | Styling Framework | 4.x |
| Lucide React | Icon Library | 0.395.0 |
| @zxing/browser | Barcode Scanner | 0.1.5 |
| @google/generative-ai | Gemini AI Integration | 0.24.1 |
| html2pdf.js | PDF Generation | 0.14.0 |

### **Backend**
| Technology | Purpose | Version |
|-----------|---------|---------|
| Node.js | Runtime Environment | 16+ |
| Express.js | Web Framework | 4.22.1 |
| MongoDB | NoSQL Database | 8.21.1 |
| Mongoose | ODM | 8.21.1 |
| JWT | Authentication | 9.0.2 |
| bcryptjs | Password Hashing | 2.4.3 |
| Redis | Caching & Sessions | 4.6.12 |
| @google/generative-ai | Gemini AI API | 0.24.1 |
| axios | HTTP Client | 1.13.4 |
| CORS | Cross-Origin Support | 2.8.5 |
| dotenv | Environment Variables | 16.6.1 |

### **Development Tools**
| Tool | Purpose |
|------|---------|
| Nodemon | Auto-reload during development |
| ESLint | Code quality & linting |
| npm | Package manager |

---

## 📁 Project Structure

```
StartUp/
├── cliet-side/                          # Frontend Application
│   ├── src/
│   │   ├── components/                  # React Components
│   │   │   ├── AppNavbar.jsx            # Top navigation bar
│   │   │   ├── BillingSystem.jsx        # Main POS interface
│   │   │   ├── BillDisplay.jsx          # Bill preview
│   │   │   ├── BillItems.jsx            # Cart items list
│   │   │   ├── BillSidebar.jsx          # Quick actions sidebar
│   │   │   ├── Bills.jsx                # Bill history
│   │   │   ├── CustomerManager.jsx      # Customer CRUD
│   │   │   ├── InventoryManager.jsx     # Stock management
│   │   │   ├── Login.jsx                # Login page
│   │   │   ├── Signup.jsx               # Registration page
│   │   │   ├── ProfilePage.jsx          # User profile management
│   │   │   ├── ProductScanner.jsx       # Barcode scanner
│   │   │   ├── SalesReport.jsx          # Reports & analytics
│   │   │   ├── ShopSetupModal.jsx       # Shop details form
│   │   │   ├── PaymentModal.jsx         # Payment processing
│   │   │   ├── Toast.jsx                # Notifications
│   │   │   ├── Layout/                  # Reusable UI components
│   │   │   │   ├── Alert.jsx
│   │   │   │   ├── Badge.jsx
│   │   │   │   ├── Button.jsx
│   │   │   │   ├── Card.jsx
│   │   │   │   ├── Form.jsx
│   │   │   │   ├── Table.jsx
│   │   │   │   └── index.js
│   │   │   └── styles/                  # Component CSS
│   │   ├── context/
│   │   │   └── AuthContext.jsx          # Global auth state
│   │   ├── services/
│   │   │   └── quaggaService.js         # Barcode scanning logic
│   │   ├── pages/                       # Page components
│   │   │   ├── ComponentShowcase.jsx
│   │   │   └── ExampleHomePage.jsx
│   │   ├── App.jsx                      # Root component
│   │   ├── main.jsx                     # Entry point
│   │   ├── App.css
│   │   └── index.css
│   ├── public/                          # Static assets
│   ├── package.json
│   ├── vite.config.js                   # Vite configuration
│   ├── tailwind.config.js               # Tailwind setup
│   └── eslint.config.js                 # ESLint configuration
│
└── server-side/                         # Backend Application
    ├── models/                          # Database Schemas
    │   ├── User.js                      # User model
    │   ├── Product.js                   # Product model
    │   ├── Bill.js                      # Bill/Invoice model
    │   ├── Customer.js                  # Customer model
    │   └── Inventory.js                 # Stock model
    ├── routes/                          # API Endpoints
    │   ├── auth.js                      # Authentication routes
    │   ├── products.js                  # Product CRUD routes
    │   ├── bills.js                     # Bill management routes
    │   ├── inventory.js                 # Inventory routes
    │   └── customers.js                 # Customer routes
    ├── services/                        # Business Logic
    │   ├── billService.js               # Bill processing
    │   ├── cacheService.js              # Redis caching
    │   ├── sessionService.js            # Session management
    │   └── whatsappService.js           # WhatsApp integration
    ├── middleware/
    │   └── sessionMiddleware.js         # Session validation
    ├── config/
    │   └── redis.js                     # Redis configuration
    ├── server.js                        # Main server file
    ├── package.json
    ├── README.md
    └── .env                             # Environment variables
```

---

## 🗄️ Database Models

### **User Model**
```javascript
{
  _id: ObjectId,
  username: String (unique, required),
  email: String (unique, sparse),
  password: String (hashed, required),
  role: String (enum: ['Cashier', 'Manager', 'Admin'], default: 'Cashier'),
  
  // Shop Information
  shopName: String,
  shopAddress: String,
  phone: String,
  gstNumber: String,
  gstRate: Number (0-100, default: 5),
  shopEmail: String,
  
  // User Status
  active: Boolean (default: true),
  lastLogin: Date,
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

**Indices:** 
- Primary: username (unique)
- Secondary: email (unique)

---

### **Product Model**
```javascript
{
  _id: ObjectId,
  name: String (required, indexed),
  price: Number (required),
  
  // Price History Tracking
  priceHistory: [{
    price: Number,
    effectiveFrom: Date,
    updatedBy: String,
    reason: String,
    updatedAt: Date
  }],
  
  category: String (required),
  description: String,
  brand: String,
  
  // Identifiers
  sku: String (unique, sparse),
  barcode: String (unique, sparse, indexed),
  
  // Image & Media
  image: String (URL),
  images: [String],
  
  // Stock Info
  stock: Number (default: 0),
  
  // Status
  active: Boolean (default: true),
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

**Indices:**
- name, barcode, sku, active

---

### **Bill Model**
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: User, required),
  billNumber: String (unique, required),
  
  // Shop Details
  shopName: String,
  shopAddress: String,
  shopPhone: String,
  gstNumber: String,
  
  // Customer Details
  customerName: String,
  customerMobile: String,
  
  // Bill Items
  items: [{
    productId: ObjectId,
    productName: String,
    quantity: Number,
    unitPrice: Number,
    totalPrice: Number
  }],
  
  // Financial Details
  subtotal: Number,
  tax: Number,
  taxPercentage: Number (default: 5),
  discount: Number (default: 0),
  total: Number,
  
  // Payment Info
  paymentMethod: String (enum: ['Cash', 'Card', 'UPI', 'Check']),
  paymentStatus: String (enum: ['Pending', 'Completed', 'Failed', 'Refunded']),
  amountReceived: Number,
  change: Number,
  
  // Additional
  notes: String,
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

**Indices:**
- userId + billNumber
- createdAt

---

### **Customer Model**
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: User, required),
  name: String (required, trimmed),
  mobileNumber: String (required, regex validated: 10 digits),
  
  // Address
  address: String,
  city: String,
  state: String,
  zipCode: String,
  
  // Additional Info
  gstNumber: String,
  notes: String,
  
  // Purchase Metrics
  totalBills: Number (default: 0),
  totalSpent: Number (default: 0),
  
  // Status
  isActive: Boolean (default: true),
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

**Indices:**
- userId + mobileNumber (unique)
- userId + name

---

### **Inventory Model**
```javascript
{
  _id: ObjectId,
  productId: ObjectId (ref: Product, required),
  
  // Stock Tracking
  barcode: String (unique, sparse, indexed),
  quantity: Number (required, default: 0),
  minStock: Number (default: 10),
  maxStock: Number (default: 100),
  reorderLevel: Number (default: 20),
  
  // Location
  location: String (default: 'Main Store'),
  warehouse: String (default: 'Default'),
  
  // Status
  status: String (enum: ['In Stock', 'Low Stock', 'Out of Stock']),
  
  // Management
  lastRestocked: Date (default: now),
  notes: String,
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

**Indices:**
- productId
- barcode
- warehouse + location

---

## 🔌 API Endpoints

### **Authentication Routes** (`/api/auth`)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/signup` | Register new user | ✗ |
| POST | `/login` | User login | ✗ |
| GET | `/user` | Get current user | ✓ |
| PUT | `/user` | Update user profile | ✓ |
| POST | `/logout` | Logout user | ✓ |

### **Product Routes** (`/api/products`)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/` | Get all products (with filters) | ✗ |
| GET | `/barcode/:barcode` | Get product by barcode | ✗ |
| GET | `/:id` | Get product by ID | ✗ |
| POST | `/` | Create new product | ✓ |
| PUT | `/:id` | Update product | ✓ |
| DELETE | `/:id` | Delete product | ✓ |
| POST | `/batch` | Batch import products | ✓ |

**Query Parameters:**
- `category`: Filter by category
- `search`: Search by name, brand, SKU, or barcode
- `limit`: Number of results

### **Bill Routes** (`/api/bills`)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/` | Get user's bills | ✓ |
| GET | `/:id` | Get bill by ID | ✓ |
| GET | `/number/:billNumber` | Get bill by bill number | ✓ |
| POST | `/` | Create new bill | ✓ |
| PUT | `/:id` | Update bill | ✓ |
| DELETE | `/:id` | Delete bill | ✓ |
| GET | `/daily-stats` | Get daily statistics | ✓ |
| GET | `/reports/summary` | Get sales summary | ✓ |

**Filters:**
- `startDate`: Date range filter
- `endDate`: Date range filter
- `paymentMethod`: Filter by payment type

### **Inventory Routes** (`/api/inventory`)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/` | Get all inventory | ✓ |
| GET | `/:id` | Get inventory by ID | ✓ |
| POST | `/` | Create inventory entry | ✓ |
| PUT | `/:id` | Update stock quantity | ✓ |
| DELETE | `/:id` | Delete inventory entry | ✓ |
| POST | `/restock` | Restock products | ✓ |
| GET | `/alerts/low-stock` | Get low stock items | ✓ |

**Query Parameters:**
- `warehouse`: Filter by warehouse
- `status`: Filter by status (In Stock, Low Stock, Out of Stock)
- `search`: Search products

### **Customer Routes** (`/api/customers`)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/` | Get all customers | ✓ |
| GET | `/:id` | Get customer by ID | ✓ |
| POST | `/` | Create new customer | ✓ |
| PUT | `/:id` | Update customer | ✓ |
| DELETE | `/:id` | Delete customer | ✓ |
| GET | `/:id/bills` | Get customer's bills | ✓ |

**Query Parameters:**
- `search`: Search by name or phone
- `sortBy`: Sort field (createdAt, totalSpent)
- `order`: asc or desc

---

## 🎨 Frontend Components

### **Main Components**

1. **BillingSystem.jsx** - Core POS interface
   - Product selection and scanning
   - Shopping cart management
   - Real-time calculations
   - Payment processing
   - Multiple tab management (Billing, Bills, Inventory)

2. **ProductScanner.jsx** - Barcode input
   - QR/Barcode scanning integration
   - Manual barcode entry
   - Quagga.js integration

3. **BillDisplay.jsx** - Bill preview
   - Professional bill layout
   - Tax calculations
   - Payment summary
   - Print preparation

4. **PaymentModal.jsx** - Payment processing
   - Multiple payment method support
   - Change calculation
   - Payment confirmation

5. **CustomerManager.jsx** - Customer CRUD
   - Add/Edit customer information
   - Search and filter
   - Purchase history

6. **InventoryManager.jsx** - Stock management
   - Real-time stock levels
   - Stock alerts
   - Reorder management
   - Stock history

7. **SalesReport.jsx** - Analytics
   - Daily/monthly reports
   - Revenue charts
   - Payment method breakdown
   - Top products analysis

8. **Login.jsx / Signup.jsx** - Authentication
   - User registration
   - Login with validation
   - Session creation

9. **ProfilePage.jsx** - User management
   - Profile editing
   - Shop details
   - GST configuration
   - Role management

### **Layout Components** (`/Layout`)
- Alert.jsx - Alert notifications
- Badge.jsx - Status badges
- Button.jsx - Reusable buttons
- Card.jsx - Card containers
- Form.jsx - Form wrapper
- Table.jsx - Data tables

---

## 🚀 Installation & Setup

### **Prerequisites**
- Node.js (v16 or higher)
- MongoDB (local or Atlas cloud)
- npm or yarn
- Git

### **Backend Setup**

```bash
# 1. Navigate to server directory
cd server-side

# 2. Install dependencies
npm install

# 3. Create .env file
cp .env.example .env

# 4. Configure environment variables (see Configuration section)

# 5. Start development server
npm run dev

# Server runs on: http://localhost:5001 (or PORT from .env)
```

### **Frontend Setup**

```bash
# 1. Navigate to client directory
cd cliet-side

# 2. Install dependencies
npm install

# 3. Create .env file for Vite
cp .env.example .env.local

# 4. Configure environment variables
# VITE_API_BASE_URL=http://localhost:5001

# 5. Start development server
npm run dev

# Client runs on: http://localhost:5173 (or port from output)
```

### **Database Setup**

#### MongoDB Atlas (Cloud)
1. Create account at mongodb.com
2. Create a new cluster
3. Get connection string
4. Add to .env: `MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/smartbill`

#### Local MongoDB
1. Install MongoDB Community Edition
2. Start MongoDB service
3. Add to .env: `MONGODB_URI=mongodb://localhost:27017/smartbill`

---

## ⚙️ Configuration

### **Backend .env**
```env
# Server Configuration
PORT=5001
NODE_ENV=development

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/smartbill
DATABASE_NAME=smartbill

# Authentication
JWT_SECRET=your-secure-jwt-secret-key-change-in-production
JWT_EXPIRE=7d

# CORS
CORS_ORIGIN=http://localhost:5173

# Gemini AI
GEMINI_API_KEY=your-google-generative-ai-key

# Redis (Optional)
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=
REDIS_DB=0

# WhatsApp Integration (Optional)
WHATSAPP_API_KEY=
WHATSAPP_ACCOUNT_SID=
WHATSAPP_PHONE_NUMBER=

# Email (Optional)
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASSWORD=
```

### **Frontend .env.local**
```env
VITE_API_BASE_URL=http://localhost:5001
VITE_ENVIRONMENT=development
```

---

## 📖 Usage Guide

### **User Registration & Login**

1. **Sign Up**
   - Click "Sign Up" button
   - Enter username, email, password
   - Submit form
   - Account created successfully

2. **Login**
   - Enter username/email and password
   - JWT token stored in localStorage
   - Session created in Redis

3. **Shop Setup**
   - Complete shop profile (name, address, phone, GST)
   - Configure GST rate (default: 5%)
   - Save details for bill generation

### **Creating a Bill**

1. **Add Products**
   - Scan barcode or search product
   - Or enter product details manually
   - Add to cart with quantity

2. **Configure Bill**
   - Customer details (optional)
   - Apply discounts if needed
   - Select payment method

3. **Payment Processing**
   - Enter amount received
   - System calculates change
   - Confirm payment

4. **Generate Bill**
   - Preview bill
   - Print or download as PDF
   - Bill saved to database

### **Inventory Management**

1. **Add Stock**
   - Navigate to Inventory Manager
   - Add new product with barcode
   - Set min/max stock levels

2. **Update Stock**
   - Search product
   - Adjust quantity
   - System updates automatically

3. **Monitor Alerts**
   - View low stock alerts
   - Set reorder levels
   - Track stock history

### **Customer Management**

1. **Add Customer**
   - Click "Add Customer"
   - Enter name, phone, address
   - Optional: GST number, notes

2. **Search Customer**
   - Search by name or phone
   - Add to bill
   - Track purchase history

### **Reports & Analytics**

1. **Daily Reports**
   - View daily sales
   - Revenue breakdown
   - Payment method distribution

2. **Advanced Reports**
   - Date range filtering
   - Product performance
   - Customer insights
   - Export functionality

---

## 🌐 Deployment

### **Backend Deployment (Heroku/Railway)**

```bash
# 1. Build for production
npm run build

# 2. Set environment variables on platform
# - MongoDB Atlas URI
# - JWT_SECRET (strong key)
# - GEMINI_API_KEY
# - CORS_ORIGIN (production URL)

# 3. Deploy
git push heroku main
# or
railway deploy
```

### **Frontend Deployment (Vercel/Netlify)**

```bash
# 1. Build production bundle
npm run build

# 2. Environment variables
VITE_API_BASE_URL=https://api.yourdomain.com

# 3. Deploy
vercel deploy
# or
netlify deploy --prod --dir=dist
```

### **Production Checklist**

- [ ] Set strong JWT_SECRET
- [ ] Configure MongoDB Atlas production cluster
- [ ] Set CORS_ORIGIN to production domain
- [ ] Enable HTTPS/SSL
- [ ] Configure backup strategy
- [ ] Set up monitoring/logging
- [ ] Configure CDN for static assets
- [ ] Set up automated backups
- [ ] Configure error tracking (Sentry)
- [ ] Enable API rate limiting
- [ ] Set up security headers

---

## 🐛 Troubleshooting

### **Common Issues**

**Issue: CORS Error**
```
Solution: Check CORS_ORIGIN in .env matches frontend URL
```

**Issue: MongoDB Connection Failed**
```
Solution: 
1. Verify MONGODB_URI in .env
2. Check IP whitelist in MongoDB Atlas
3. Verify database user credentials
```

**Issue: Barcode Scanner Not Working**
```
Solution:
1. Ensure camera permissions granted
2. Use HTTPS in production
3. Test with different barcode formats
```

**Issue: Token Expired**
```
Solution:
1. Clear localStorage
2. Login again
3. Check JWT_EXPIRE setting in .env
```

**Issue: Slow Performance**
```
Solution:
1. Enable Redis caching
2. Add database indices
3. Implement pagination
4. Optimize product images
```

### **Debug Mode**

Enable verbose logging:
```javascript
// server.js
process.env.DEBUG = 'smartbill:*'

// Frontend (main.jsx)
window.__DEBUG__ = true
```

---

## 📊 Performance Optimization

### **Frontend**
- Code splitting with Vite
- Lazy loading components
- Image optimization
- Caching strategy with Service Workers
- Minified production bundle

### **Backend**
- Database query optimization
- Redis caching layer
- Database indices on frequently queried fields
- Pagination for large datasets
- Connection pooling for MongoDB

### **Database**
- Indexed fields for quick queries
- Archival of old bills
- Data aggregation for reports
- Sharding strategy for scaling

---

## 🔒 Security Features

- ✅ Password hashing with bcryptjs
- ✅ JWT token authentication
- ✅ CORS protection
- ✅ Input validation and sanitization
- ✅ SQL injection prevention (MongoDB)
- ✅ XSS protection via React
- ✅ Session timeout
- ✅ Role-based access control
- ✅ Rate limiting (recommended)
- ✅ HTTPS enforcement (production)

---

## 📞 Support & Contact

For issues, feature requests, or questions:
- Create an issue in the repository
- Contact development team
- Check API documentation
- Review logs for errors

---

## 📜 License

This project is licensed under the MIT License - see LICENSE file for details.

---

## 👥 Contributors

- Development Team
- Quality Assurance
- UI/UX Design

---

**Last Updated:** May 2026 | **Version:** 1.0.0 | **Maintained By:** SmartBill Team

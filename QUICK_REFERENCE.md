# SmartBill POS - Quick Reference Guide

**Version:** 1.0.0 | **Last Updated:** May 2026

---

## 🚀 Quick Start (5 minutes)

### Backend Setup
```bash
cd server-side
npm install
# Create .env with MONGODB_URI, JWT_SECRET, GEMINI_API_KEY
npm run dev
# Server: http://localhost:5001
```

### Frontend Setup
```bash
cd cliet-side
npm install
# Create .env with VITE_API_BASE_URL=http://localhost:5001
npm run dev
# Client: http://localhost:5173
```

---

## 📁 Project Structure at a Glance

```
cliet-side/
├── src/
│   ├── components/          # React components
│   ├── context/AuthContext  # Global auth state
│   ├── services/            # API calls
│   └── App.jsx              # Root component

server-side/
├── models/                  # MongoDB schemas
├── routes/                  # API endpoints
├── services/                # Business logic
├── middleware/              # Auth, validation
└── config/                  # Database config
```

---

## 🔑 Key Concepts

### User Roles
- **Admin**: Full access
- **Manager**: Inventory & customers
- **Cashier**: Billing only

### Main Features
1. **Billing** - Create invoices, calculate tax, process payments
2. **Inventory** - Track stock, low stock alerts
3. **Customers** - Manage customer data, purchase history
4. **Reports** - Sales analytics, revenue tracking
5. **AI** - Gemini for product recognition from images

---

## 🔌 Essential API Endpoints

| Feature | Endpoint | Method | Auth |
|---------|----------|--------|------|
| **Login** | `/api/auth/login` | POST | ✗ |
| **Get Products** | `/api/products` | GET | ✗ |
| **Create Bill** | `/api/bills` | POST | ✓ |
| **Get Bills** | `/api/bills` | GET | ✓ |
| **Manage Inventory** | `/api/inventory` | GET/POST/PUT | ✓ |
| **Manage Customers** | `/api/customers` | GET/POST/PUT | ✓ |
| **Health Check** | `/health` | GET | ✗ |

**Note:** `✓` = Requires JWT token in header

---

## 🗄️ Database Models Summary

### User
```javascript
{ username, email, password, role, shopName, shopAddress, phone, gstRate }
```

### Product
```javascript
{ name, price, category, barcode, sku, image, stock, active }
```

### Bill
```javascript
{ billNumber, items[], subtotal, tax, discount, total, paymentMethod, customerName }
```

### Customer
```javascript
{ name, mobileNumber, address, city, state, gstNumber, totalBills, totalSpent }
```

### Inventory
```javascript
{ productId, barcode, quantity, minStock, maxStock, location, warehouse, status }
```

---

## 🔐 Authentication Flow

```
1. User Login (POST /api/auth/login)
2. Server validates credentials
3. Returns: { token, user, sessionId }
4. Client stores token in localStorage
5. Future requests include: Authorization: Bearer <token>
6. Server verifies token on each request
```

**Token Expiry:** 7 days (configurable in .env)

---

## 💾 Environment Variables

### Backend (.env)
```env
MONGODB_URI=mongodb+srv://...
JWT_SECRET=your-secret-key
PORT=5001
CORS_ORIGIN=http://localhost:5173
GEMINI_API_KEY=your-api-key
```

### Frontend (.env.local)
```env
VITE_API_BASE_URL=http://localhost:5001
```

---

## 🐛 Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| CORS Error | Check CORS_ORIGIN in .env matches frontend URL |
| MongoDB connection fails | Verify MONGODB_URI and IP whitelist in Atlas |
| Token expired | User needs to login again |
| Barcode scanner not working | Ensure HTTPS in production, camera permissions |
| Slow performance | Enable Redis, check database indices |

---

## 🔍 Debugging Tips

### Check Server Logs
```bash
# Backend logs are printed to console
npm run dev
```

### Check Network Requests
```javascript
// Open DevTools (F12) → Network tab
// See API calls and responses
```

### Enable Debug Mode
```javascript
// In browser console
window.__DEBUG__ = true
```

### Test API Directly
```bash
curl -X GET http://localhost:5001/api/products \
  -H "Authorization: Bearer <your-token>"
```

---

## 📊 Key API Responses

### Successful Bill Creation
```json
{
  "success": true,
  "data": {
    "_id": "60f7b3c3d8c4e1a2b3c4d5e7",
    "billNumber": "BILL-1609459200000",
    "total": 210,
    "paymentStatus": "Completed",
    "createdAt": "2024-05-02T10:30:00Z"
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": "Unauthorized",
  "message": "Invalid token"
}
```

### Product List
```json
{
  "success": true,
  "count": 25,
  "data": [
    { "_id": "...", "name": "Product A", "price": 100, "barcode": "123456" },
    ...
  ]
}
```

---

## 📈 Frontend Component Hierarchy

```
App
├── Login / Signup (if not authenticated)
└── [After Auth]
    ├── AppNavbar
    ├── BillingSystem (Main Tab)
    │   ├── ProductScanner
    │   ├── BillItems (Cart)
    │   ├── BillDisplay (Preview)
    │   └── PaymentModal
    ├── Bills Tab (History)
    ├── Inventory Tab
    ├── Customers Tab
    ├── Reports Tab
    └── Profile Page
```

---

## 🔄 Data Flow Example: Creating a Bill

```
1. User scans product barcode
   └→ ProductScanner sends GET /api/products/barcode/:barcode

2. Product fetched and added to cart
   └→ Frontend updates billItems state

3. User confirms bill
   └→ Sends POST /api/bills with items, customer, payment details

4. Backend processes:
   ├─ Validates inventory
   ├─ Calculates totals (subtotal + tax - discount)
   ├─ Deducts stock
   ├─ Updates customer stats
   └─ Saves bill document

5. Returns bill details and PDF link
   └→ Frontend displays bill, offers print/download
```

---

## 🛠️ Development Commands

### Backend
```bash
npm run dev          # Start development server with auto-reload
npm start            # Start production server
npm test             # Run tests
```

### Frontend
```bash
npm run dev          # Start Vite dev server
npm run build        # Create production build
npm run preview      # Preview production build
npm run lint         # Run ESLint
```

---

## 📝 Code Patterns

### Create a Component
```jsx
import React, { useState } from 'react'
import './ComponentName.css'

function ComponentName() {
  const [state, setState] = useState(null)
  
  return (
    <div className="container">
      {/* Content here */}
    </div>
  )
}

export default ComponentName
```

### Create an API Endpoint
```javascript
// server-side/routes/example.js
import express from 'express'
const router = express.Router()

router.get('/endpoint', verifyToken, async (req, res) => {
  try {
    const data = await Model.find()
    res.json({ success: true, data })
  } catch (error) {
    res.status(500).json({ error: error.message })
  }
})

export default router
```

### Call API from Frontend
```javascript
const response = await fetch(
  `${import.meta.env.VITE_API_BASE_URL}/api/endpoint`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  }
)
const result = await response.json()
```

---

## 🚀 Deployment Checklist

- [ ] Set strong JWT_SECRET
- [ ] Configure MONGODB_URI for production
- [ ] Set CORS_ORIGIN to production domain
- [ ] Enable HTTPS/SSL
- [ ] Set NODE_ENV=production
- [ ] Configure error tracking (Sentry)
- [ ] Set up monitoring (Datadog/NewRelic)
- [ ] Configure automated backups
- [ ] Run security scan
- [ ] Performance test
- [ ] Load test
- [ ] Smoke test production

---

## 📞 Useful Links

- **MongoDB Documentation**: https://docs.mongodb.com
- **Express.js Guide**: https://expressjs.com
- **React Documentation**: https://react.dev
- **Mongoose ODM**: https://mongoosejs.com
- **JWT Introduction**: https://jwt.io

---

## 👥 Team Contact

- **Frontend Lead**: [name]
- **Backend Lead**: [name]
- **DevOps/Infrastructure**: [name]
- **Database Admin**: [name]

---

## 📊 Performance Guidelines

- API response time: < 500ms
- Page load: < 2 seconds
- Cache hit rate: > 80%
- Database queries: < 100ms

---

## 🔐 Security Checklist

- [ ] No hardcoded secrets
- [ ] All passwords hashed
- [ ] JWT tokens validated
- [ ] Input validation on backend
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] CORS properly configured
- [ ] HTTPS in production
- [ ] Rate limiting enabled
- [ ] Audit logging in place

---

## 📚 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | May 2026 | Initial release |

---

## 💡 Tips & Tricks

### Speed up development
```bash
# Use nodemon for auto-reload
npm install -D nodemon

# Use Vite for instant HMR (Hot Module Reload)
npm run dev
```

### Debug database queries
```javascript
// In Mongoose, enable debug mode
mongoose.set('debug', true)
```

### Test endpoints quickly
```bash
# Use REST Client extension in VS Code
# Or use Postman/Insomnia
```

### Check React component renders
```javascript
// Add this in component
console.log('ComponentName rendered')
```

---

## 🎯 Next Steps

1. **Read README.md** for comprehensive documentation
2. **Read SYSTEM_DESIGN.md** for architecture details
3. **Explore code** - Start with App.jsx and server.js
4. **Set up development** - Follow Quick Start section
5. **Try creating a bill** - End-to-end flow
6. **Review database** - Check MongoDB Atlas

---

**Quick Reference v1.0** | Last Updated: May 2026 | SmartBill Team

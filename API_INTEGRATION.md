# Backend API Integration Guide

This guide explains how to integrate the frontend with the new Express backend API.

## 🔄 Migration from localStorage to API

Previously, all data was stored in `localStorage` via `bank-core-v2.js`. Now we're using a proper backend API with JWT authentication.

## 📋 Key Changes

### 1. Authentication Flow

**Before (localStorage):**
```javascript
VanstraBank.login(email, password);
// Data stored in localStorage
```

**After (API):**
```javascript
const response = await fetch('http://localhost:5000/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});

const { token, user } = await response.json();
localStorage.setItem('authToken', token);
```

### 2. Token Management

Store JWT token:
```javascript
localStorage.setItem('authToken', token);
```

Include token in all requests:
```javascript
const headers = {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${localStorage.getItem('authToken')}`
};
```

### 3. API Base URL

Set in a config file:
```javascript
const API_BASE_URL = 'http://localhost:5000/api';
```

## 🛣️ Updated Endpoints for Frontend

### Authentication

**Login**
```javascript
POST /api/auth/login
{ email, password }
→ { token, user }
```

**Signup**
```javascript
POST /api/auth/signup
{ fullName, email, password, phone }
→ { token, user }
```

**Get Current User**
```javascript
GET /api/auth/me
(requires Authorization header)
→ { user }
```

**Logout**
```javascript
POST /api/auth/logout
(requires Authorization header)
→ { success: true }
```

### User Profile

**Get Profile**
```javascript
GET /api/users/profile
→ { user }
```

**Update Profile**
```javascript
PUT /api/users/profile
{ fullName, phone, avatar }
→ { user }
```

### Transactions

**Get Transactions**
```javascript
GET /api/users/transactions
→ { transactions: [] }
```

**Create Transaction**
```javascript
POST /api/users/transaction
{ type, amount, description, recipientId }
→ { transaction, newBalance }
```

### Admin Functions

**Get All Users** ⚙️
```javascript
GET /api/admin/users
(requires admin role)
→ { users: [] }
```

**Get User Details** ⚙️
```javascript
GET /api/admin/users/:id
→ { user }
```

**Update User Balance** ⚙️
```javascript
PUT /api/admin/users/:id/balance
{ balance }
→ { user }
```

**Update Account Status** ⚙️
```javascript
PUT /api/admin/users/:id/status
{ status: 'active'|'frozen'|'blocked' }
→ { user }
```

**Get Admin Stats** ⚙️
```javascript
GET /api/admin/stats
→ { stats: {...} }
```

**Delete User** ⚙️
```javascript
DELETE /api/admin/users/:id
```

## 🔑 Headers Required

```javascript
// All authenticated requests need:
{
  'Content-Type': 'application/json',
  'Authorization': 'Bearer <JWT_TOKEN>'
}
```

## ⚙️ Frontend Implementation Plan

### Step 1: Create API Service Module
Create `bank-api-client.js` to handle all API calls with token management.

### Step 2: Update Authentication Pages
- `login.html` → Use `/api/auth/login`
- `signup.html` → Use `/api/auth/signup`

### Step 3: Update Dashboard Pages
- `dashboard-v2.html` → Use `/api/users/profile` and `/api/users/transactions`
- `profile-v2.html` → Use `/api/users/profile` for GET/PUT
- `cards-v2.html` → (Keep as-is or link to user data from API)
- `wealth-v2.html` → (Keep as-is or link to investment data)

### Step 4: Update Admin Pages
- `admin-v2.html` → Use `/api/admin/users`, `/api/admin/stats`, `/api/admin/users/:id/balance`, `/api/admin/users/:id/status`

## 🔍 Error Handling

```javascript
fetch(url, options)
  .then(res => {
    if (!res.ok) {
      throw new Error(`HTTP ${res.status}: ${res.statusText}`);
    }
    return res.json();
  })
  .catch(error => {
    console.error('API Error:', error);
    // Show user-friendly error message
  });
```

## 🧪 Testing the API

### Using cURL

**Login:**
```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@vanstra.bank","password":"Test1234!"}'
```

**Get User Profile:**
```bash
curl http://localhost:5000/api/users/profile \
  -H "Authorization: Bearer <TOKEN>"
```

**Admin: Get All Users:**
```bash
curl http://localhost:5000/api/admin/users \
  -H "Authorization: Bearer <ADMIN_TOKEN>"
```

## 📝 Token Refresh Strategy

Current implementation: Tokens expire after 7 days.

For better UX, implement token refresh:
```javascript
const token = localStorage.getItem('authToken');
const decoded = jwt_decode(token);
if (Date.now() >= decoded.exp * 1000) {
  // Token expired - redirect to login
  window.location.href = '/login.html';
}
```

## 🚀 Parallel Operation

During transition, you CAN keep both systems running:
- New features use the API
- Old features still use localStorage
- Gradually migrate pages one by one

## ✅ Validation Checklist

Before deploying to production:
- [ ] All API endpoints tested with valid data
- [ ] Error responses handled gracefully
- [ ] Token properly stored and sent
- [ ] Admin routes require admin token
- [ ] User cannot edit other users' data
- [ ] Passwords hashed on backend
- [ ] CORS configured correctly
- [ ] Database backups implemented

## 🔐 Security Checklist

- [ ] JWT_SECRET is strong and unique
- [ ] Passwords never sent in logs
- [ ] HTTPS enabled in production
- [ ] Rate limiting on auth endpoints
- [ ] Admin routes verified on backend
- [ ] Input validation on all routes
- [ ] SQL/NoSQL injection prevented

---

**Next Step**: Create `bank-api-client.js` to abstract all API calls with proper error handling and token management.

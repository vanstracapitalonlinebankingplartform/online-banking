# Login/Logout Testing & Debugging Guide

## Overview
This guide helps you test the complete login/logout cycle and understand what's happening at each step using browser console logs.

## Quick Start Test

### Step 1: Open Console
1. Press **F12** or right-click → **Inspect** to open DevTools
2. Click the **Console** tab
3. Keep this open throughout the test

### Step 2: Sign Up a Test User (if needed)
Go to [signup.html](signup.html) and create a test account:
- **Name**: Test User
- **Email**: test@example.com
- **Password**: Test1234!
- **PIN**: 1234

### Step 3: Login
Go to [login.html](login.html) and enter:
- **Email**: test@example.com
- **Password**: Test1234!
- Click **Sign In**

### Expected Console Output (Login Success):

```
📝 LOGIN FORM SUBMITTED: {email: "test@example.com"}
🔐 Calling VanstraBank.login()
🔐 LOGIN ATTEMPT: test@example.com
📦 Users data from localStorage: exists
👥 All users found: 1
📋 User list: [{id: "...", email: "test@example.com"}]
🔍 User found: ID=..., Email=test@example.com
🔐 Password verified
✅ User saved to localStorage
🔑 createSession for user: ...
📍 Loaded existing sessions: 0
✅ Sessions saved to localStorage
✅ currentSession token set: SES-1708618800000...
✅ LOGIN SUCCESS: {email: "test@example.com", sessionToken: "SES-..."}
📋 Login result: {success: true, hasSessionToken: true}
✅ Login successful, setting currentSession
💾 currentSession saved: SES-1708618800000...
✓ Verification: currentSession in localStorage: YES
🚀 Redirecting to dashboard...
```

**Then on Dashboard (page load):**
```
📱 Dashboard Loading - loadUserData() called
🔍 getCurrentUser - SessionToken: SES-1708618800000...
📍 Session lookup: Found, UserID=...
👤 User lookup: Found - test@example.com
✅ User loaded: {email: "test@example.com", name: "Test User"}
```

### Step 4: Logout
Click the **Logout** button in the dashboard (usually in settings or top-right corner).

### Expected Console Output (Logout):

```
🚀 Setting up Suspension Interceptor...
📍 Suspension Interceptor Status: {isSuspended: false}
✅ Account is active, no suspension needed
🔍 getCurrentUser - SessionToken: null
❌ No session token found
(redirects to login.html)
```

## Troubleshooting

### Issue 1: "User not found" error on login

**Possible Causes:**
- Email doesn't exist in localStorage
- User data was cleared
- Email is case-sensitive (use lowercase)

**Debug Steps:**
1. In console, type:
   ```javascript
   JSON.parse(localStorage.getItem('vanstraUsers'))
   ```
2. Check if your email is in the list
3. If empty, create a new account

### Issue 2: "Invalid password" error

**Possible Causes:**
- Password is wrong
- Password was changed
- Hash function mismatch

**Debug Steps:**
1. Look for this in console:
   ```
   🔍 User found: ID=..., Email=...
   ```
2. If user is found but password fails, try resetting password
3. Check that you're entering password correctly (case-sensitive)

### Issue 3: Login succeeds but redirects to login page

**This is the bug you're reporting!**

**Debugging steps:**

1. **Check Step 1**: After login succeeds, verify currentSession was saved:
   ```javascript
   console.log(localStorage.getItem('currentSession'))
   // Should show: SES-1708618800000-XXXXXX
   ```

2. **Check Step 2**: Verify sessions object was saved:
   ```javascript
   console.log(localStorage.getItem('vanstraSessions'))
   // Should show: {"SES-...": {"userId": "...", "createdAt": "...", "expiresAt": "..."}}
   ```

3. **Check Step 3**: Verify user exists in vanstraUsers:
   ```javascript
   const users = JSON.parse(localStorage.getItem('vanstraUsers'));
   const sessions = JSON.parse(localStorage.getItem('vanstraSessions'));
   const token = localStorage.getItem('currentSession');
   const session = sessions[token];
   const user = users[session.userId];
   console.log('User should exist:', user);
   ```

4. **Check Step 4**: Manually test getCurrentUser:
   ```javascript
   const result = VanstraBank.getCurrentUser();
   console.log('Should return user object:', result);
   ```

## Testing Sequence

### Complete Login/Logout Cycle

```
1. Clear localStorage (optional):
   localStorage.clear()

2. Sign up at signup.html
   
3. Login at login.html
   → Check console for all login logs
   → Verify dashboard loads
   
4. Click some features on dashboard
   → Transfer, Deposit, Pay Bills, etc.
   
5. Logout
   → Check console for logout logs
   → Should redirect to login.html
   
6. Try to access dashboard directly
   → Should redirect to login.html
   
7. Login again
   → Should load dashboard successfully
   
8. REPEAT 5-7 multiple times
   → This is where the bug manifests
```

## What Each Log Means

| Log | Meaning |
|-----|---------|
| `📝 LOGIN FORM SUBMITTED` | User clicked login button |
| `🔐 Calling VanstraBank.login()` | Login JS function called |
| `📦 Users data from localStorage` | Checking if users exist in storage |
| `👥 All users found: X` | Number of users in system |
| `🔍 User found: ID=...` | User email matched in system |
| `❌ User not found in system` | Email doesn't exist |
| `✅ User saved to localStorage` | User last-login updated |
| `🔑 createSession for user` | Creating new session |
| `✅ Sessions saved to localStorage` | Session token stored |
| `✅ currentSession token set` | Token marked as current |
| `✅ LOGIN SUCCESS` | Login completed |
| `💾 currentSession saved` | Token saved successfully |
| `✓ Verification: currentSession in localStorage: YES` | Token is accessible |
| `🚀 Redirecting to dashboard` | About to load dashboard |
| `🔍 getCurrentUser - SessionToken: SES-...` | Dashboard checking for session |
| `📍 Session lookup: Found, UserID=...` | Session token found |
| `👤 User lookup: Found` | User found in localStorage |
| `❌ No session token found` | No currentSession in localStorage |

## Browser Console Tricks

### See all localStorage:
```javascript
Object.keys(localStorage).forEach(key => {
  console.log(key, ':', localStorage.getItem(key).substring(0, 100))
})
```

### Check if localStorage is being saved:
```javascript
// Before logout
console.log('Before:', localStorage.getItem('currentSession'))

// After logout
console.log('After:', localStorage.getItem('currentSession'))
```

### Monitor localStorage changes in real-time:
```javascript
// Put this in console to see localStorage changes
setInterval(() => {
  console.log('currentSession:', localStorage.getItem('currentSession').substring(0, 20) || 'NONE')
}, 1000)
```

## If the Bug Still Occurs

If you follow all these steps and the bug still happens, collect this information:

1. **Screenshot of console logs** for the complete cycle
2. **What error message** appears when redirected to login
3. **localStorage dump**:
   ```javascript
   console.log(localStorage)
   ```
4. **sessionStorage dump** (for suspension system):
   ```javascript
   console.log(sessionStorage)
   ```

## Related Files

- [bank-core-v2.js](bank-core-v2.js) - Core auth functions (login, logout, getCurrentUser)
- [login.html](login.html) - Login form
- [dashboard-v2.html](dashboard-v2.html) - Dashboard (calls loadUserData)
- [signup.html](signup.html) - Account creation

## Quick Access Links

- [Login Page](login.html)
- [Signup Page](signup.html)
- [Dashboard](dashboard-v2.html)
- [Admin Panel](admin-v3.html)

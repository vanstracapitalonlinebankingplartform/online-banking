# Account Suspension Verification System

## Overview
When an admin freezes or suspends a user account, the system now requires the user to verify their identity by entering **5 sequential verification codes** before they can access any platform features.

## Verification Codes

Each code represents a different banking abbreviation:

| Step | Code Type | Code | Full Name |
|------|-----------|------|-----------|
| 1 | **COT** | `962101` | Customer Offset Token |
| 2 | **AFD** | `385247` | Account Freeze Directive |
| 3 | **SVR** | `704856` | Suspension Verification Request |
| 4 | **ACE** | `521690` | Account Clearance Encryption |
| 5 | **FVP** | `813429` | Final Verification Protocol |

## How It Works

### For Users
1. When a user logs in with a **frozen**, **suspended**, or **locked** account:
   - The dashboard loads
   - A verification modal immediately appears
   - All page interaction is blocked (overlaid)
   
2. The user enters the codes one at a time:
   - The modal displays which code is required (e.g., "Enter COT Code")
   - The user enters the 6-digit code from their admin notification
   - Clicking **Verify Code** checks the entry
   
3. Progress tracking:
   - A progress bar shows completion (e.g., "Step 1 of 5")
   - After each correct code: "Code verified! 4 more code(s) required"
   - Incorrect codes show: "Invalid code. Please check your verification letter from admin."
   
4. After all 5 codes are verified:
   - Success message: "Account verification complete! Your account is now active."
   - Account status is changed from `frozen`/`suspended` to `active`
   - Page reloads automatically
   - User can now access all features

### For Admins
To freeze an account and force verification:

1. Go to the **Admin Dashboard** ([admin-v3.html](admin-v3.html))
2. Find the user in the users table
3. Click **change status** or use the admin API to update:
   ```javascript
   // Change user status
   VanstraBank.updateUser(userId, { status: 'frozen' });
   // or
   VanstraBank.updateUser(userId, { status: 'suspended' });
   // or
   VanstraBank.updateUser(userId, { status: 'locked' });
   ```
4. Send the user a notification with the verification codes

## Intercepted Features
When an account is suspended, clicking any of these quickly triggers the verification modal:
- **Transfer** - Send money
- **Deposit** - Mobile check deposit
- **Pay Bills** - Bill payment
- **Support** - Get help

The user cannot bypass these interceptions even if they close the modal.

## Backend Implementation

### Core Functions in `bank-core-v2.js`

#### Check Suspension Status
```javascript
const status = VanstraBank.getAccountSuspensionStatus();
// Returns: { isSuspended: bool, status: string, userId: string }
```

#### Initialize Challenge
```javascript
VanstraBank.initializeSuspensionChallenge(userId);
// Creates a new verification session
```

#### Get Current Challenge
```javascript
const challenge = VanstraBank.getCurrentSuspensionChallenge();
// Returns: { userId, codesRequired: [], codesEntered: [], currentStep: 0, ... }
```

#### Verify Code
```javascript
const result = VanstraBank.verifySuspensionCode('962101');
// Returns: {
//   success: bool,
//   completed: bool,
//   message: string,
//   currentStep: number,
//   totalSteps: number,
//   progress: number (0-100)
// }
```

#### Clear Challenge
```javascript
VanstraBank.clearSuspensionChallenge();
// Removes the active verification session
```

## Frontend Implementation

### Modal Component
The suspension verification modal is located in [dashboard-v2.html](dashboard-v2.html) with ID `#suspensionVerificationModal`.

**Elements:**
- Progress bar with percentage
- Current code type display (e.g., "COT")
- 6-digit code input field
- Verify Code button
- Cancel button (logs user out)

### Interceptor Setup
On page load, `setupSuspensionInterceptor()` checks if the account is suspended and shows the modal if needed.

### Click Interceptor
Each quick action calls:
```javascript
if(interceptQuickAction('transfer')) openTransferModal()
```

This prevents the transfer modal from opening until all codes are verified.

## Security Features

1. **Session-based challenges**: Each session gets a unique `sessionId`
2. **Code validation**: Codes are checked in order (must complete Step 1 before Step 2)
3. **Auto-unblock**: Once all codes are verified, the account status changes to `active`
4. **Override protection**: Closing the modal forces logout
5. **Progress tracking**: Users see exactly which step they're on

## Customization

To change the verification codes, edit `bank-core-v2.js` lines ~906-912:

```javascript
const SUSPENSION_CODES = {
    COT: '962101',    // Customer Offset Token
    AFD: '385247',    // Account Freeze Directive
    SVR: '704856',    // Suspension Verification Request
    ACE: '521690',    // Account Clearance Encryption
    FVP: '813429'     // Final Verification Protocol
};
```

## Testing

### Test as User
1. Go to [admin-v3.html](admin-v3.html)
2. Find a test user
3. Change their status to `frozen`
4. Log in as that user
5. Verify codes in order:
   - 962101 (COT)
   - 385247 (AFD)
   - 704856 (SVR)
   - 521690 (ACE)
   - 813429 (FVP)

### Test Cancel
1. On the verification modal, click **Cancel**
2. Expected: User is logged out and redirected to login page

## Data Storage

Verification challenge data is stored in `sessionStorage`:
```javascript
sessionStorage.getItem('suspensionChallenge')
// Returns JSON with current verification state
```

After successful verification, the account's `status` field is updated in `localStorage`:
```javascript
user.status = 'active';
user.suspensionVerifiedAt = '2026-02-23T12:34:56.789Z';
```

## Error Handling

| Error | Behavior |
|-------|----------|
| No challenge active | Show "No active verification challenge" |
| Wrong code | Shake input, show error message, allow retry |
| Code too short | Prevent submission |
| Browser refresh | Challenge persists (stored in sessionStorage) |
| Close modal without finishing | Must log in again |

## Events Emitted

```javascript
// When account is unsuspended
emit('account_unsuspended', { 
  userId: '...', 
  timestamp: '2026-02-23T12:34:56.789Z' 
})
```

This event can be monitored by admins for audit trails.

## Future Enhancements

- [ ] SMS/Email code delivery instead of manual admin notification
- [ ] Rate limiting after 3 failed attempts
- [ ] OTP (One-Time Password) generation
- [ ] Biometric unlock option
- [ ] Time-limited codes (expire after 1 hour)
- [ ] Admin dashboard showing verification attempts

# Reward System Improvements

## Summary of Changes

This document outlines the improvements made to the reward system, focusing on account synchronization, reward usage in cart, and reward expiration.

---

## 1. Rewards Tab Account Creation

### What Changed
- **Before**: Rewards tab only asked for phone number
- **After**: Rewards tab now asks for both name and phone number

### Why This Matters
When a customer enters their information in the Rewards tab, it now:
1. Creates a full customer account in the database
2. Saves both name and phone to localStorage
3. Automatically syncs with Cart and Account tabs
4. Claims any pending reward transfers sent to their phone number

### Implementation Details
- File: `src/components/bakery/WalletTab.tsx`
- Added `registerCustomerFn` and `claimPendingTransfersFn` imports
- Modified `handlePhoneSubmit` to:
  - Save both phone and name to localStorage
  - Call `registerCustomer` to create/update customer in database
  - Call `claimPendingTransfers` to claim any pending rewards
  - This ensures the customer is fully registered across all systems

### User Experience
- Customer enters name + phone in Rewards tab → Account is created
- They can immediately use Cart tab without re-entering information
- Account tab shows their profile automatically
- Any rewards sent to their phone before registration are automatically claimed

---

## 2. Reward Usage in Cart

### What Changed
- **Before**: Customers could not use their rewards when placing orders
- **After**: Customers can now use their reward balance to reduce order total

### Features
1. **Automatic Balance Loading**: When customer enters phone (8+ digits), wallet balance loads automatically
2. **Flexible Usage**: Customer can choose how much reward to use via:
   - Slider control (increments of 10)
   - Manual number input
   - "Use Max" button to apply maximum available
3. **Smart Limits**: Maximum reward usage is the lesser of:
   - Available wallet balance
   - Order total amount
4. **Real-time Calculation**: Final price updates instantly as reward amount changes
5. **Visual Feedback**: Shows both original total and final price after rewards

### Implementation Details
- File: `src/components/bakery/CartTab.tsx`
- Added reward selection UI with Slider component
- Integrated `getWalletBalanceFn` to fetch balance
- Modified `createCustomerOrderFn` to accept `rewardUsed` parameter
- Server validates reward balance before processing order
- Wallet balance is deducted and transaction is recorded

### User Experience
```
Order Total: 2000 millimes
Wallet Balance: 200 millimes
Customer uses: 200 millimes
Final Price: 1800 millimes
```

The 200 millimes used disappear from wallet and are recorded in transaction history.

---

## 3. Reward Expiration (3 Months)

### What Changed
- **Before**: Rewards never expired
- **After**: Earned rewards expire after 3 months

### How It Works
1. **When Rewards Are Earned**:
   - System sets `expiresAt` to 3 months from earning date
   - Stored in `wallet_transactions` table

2. **When Balance Is Checked**:
   - System checks for expired rewards
   - Marks expired transactions with `isExpired: true`
   - Deducts expired amount from wallet balance
   - Returns updated balance to customer

3. **Transaction Types Affected**:
   - Only `earn` type transactions expire
   - Transfers, spends, and claims do not expire

### Implementation Details
- File: `src/server/functions/wallet.ts`
- Modified `recordTransaction` to set `expiresAt` for earn transactions
- Modified `getWalletBalanceFn` to:
  - Query all non-expired earn transactions
  - Check if any have passed expiration date
  - Mark expired transactions
  - Update wallet balance accordingly

### User Experience
- Customer earns 50 millimes on January 1st
- Reward expires on April 1st (90 days later)
- On April 2nd, when customer checks balance:
  - System detects expiration
  - Deducts 50 millimes from balance
  - Customer sees updated balance

---

## 4. Sign-In Redirect Change

### What Changed
- **Before**: After staff sign-in, redirected to `/dashboard`
- **After**: After staff sign-in, redirected to `/` (menu/home page)

### Why This Matters
- Staff can see the customer-facing menu immediately
- They can navigate to dashboard manually if needed
- More intuitive flow for staff who want to browse products

### Implementation Details
- File: `src/routes/_auth/sign-in.tsx`
- Changed `beforeLoad` redirect from `/dashboard` to `/`
- Changed default `validateSearch` redirect from `/dashboard` to `/`

---

## Technical Architecture

### Data Flow: Rewards Tab → Cart → Account

```
1. Customer enters name + phone in Rewards Tab
   ↓
2. registerCustomerFn creates customer record
   ↓
3. Data saved to localStorage (CUSTOMER_NAME, CUSTOMER_PHONE)
   ↓
4. Custom event 'customer-info-updated' dispatched
   ↓
5. Cart Tab listens to event and loads customer info
   ↓
6. Account Tab listens to event and loads customer info
   ↓
7. All tabs now have synchronized customer data
```

### Data Flow: Reward Usage in Order

```
1. Customer selects reward amount in Cart
   ↓
2. createCustomerOrderFn receives rewardUsed parameter
   ↓
3. Server validates wallet balance >= rewardUsed
   ↓
4. Order total = original total - rewardUsed
   ↓
5. Wallet balance deducted
   ↓
6. Transaction recorded with type 'spend'
   ↓
7. Customer totalSpent updated with final amount
```

### Database Schema Changes

No schema changes required! The existing `wallet_transactions` table already has:
- `expiresAt` (datetime, nullable)
- `isExpired` (boolean)

These fields are now being utilized for the 3-month expiration feature.

---

## Testing Checklist

### Rewards Tab Account Creation
- [ ] Enter name + phone in Rewards tab
- [ ] Check localStorage has both CUSTOMER_NAME and CUSTOMER_PHONE
- [ ] Navigate to Cart tab - info should be pre-filled
- [ ] Navigate to Account tab - profile should show name + phone
- [ ] Send rewards to a phone number, then register with that phone - rewards should be claimed

### Reward Usage in Cart
- [ ] Add items to cart (total > 1000 millimes)
- [ ] Enter phone number with wallet balance
- [ ] Verify wallet balance loads automatically
- [ ] Use slider to select reward amount
- [ ] Verify final price updates correctly
- [ ] Click "Use Max" button - should use maximum available
- [ ] Submit order - verify balance is deducted
- [ ] Check transaction history - should show 'spend' transaction

### Reward Expiration
- [ ] Create a test transaction with expiresAt in the past
- [ ] Call getWalletBalanceFn
- [ ] Verify transaction is marked as expired
- [ ] Verify wallet balance is reduced
- [ ] Check that expired rewards don't count toward available balance

### Sign-In Redirect
- [ ] Sign in as staff
- [ ] Verify redirect goes to menu (/) not dashboard
- [ ] Manually navigate to /dashboard - should work

---

## Future Enhancements

1. **Expiration Notifications**
   - Send notification 1 week before rewards expire
   - Show expiring rewards in wallet UI

2. **Reward History Filtering**
   - Filter by transaction type (earn, spend, transfer)
   - Show expired vs active rewards separately

3. **Bulk Reward Operations**
   - Extend expiration for all customers
   - Gift rewards with custom expiration dates

4. **Analytics**
   - Track reward redemption rates
   - Monitor expiration patterns
   - Identify high-value customers

---

## Files Modified

1. `src/components/bakery/WalletTab.tsx`
   - Added customer registration on phone submit
   - Added pending transfer claiming

2. `src/components/bakery/CartTab.tsx`
   - Added reward balance loading
   - Added reward selection UI (slider + input)
   - Added reward usage in order submission

3. `src/server/functions/wallet.ts`
   - Added 3-month expiration logic
   - Modified recordTransaction to set expiresAt
   - Modified getWalletBalanceFn to handle expiration

4. `src/server/functions/orders.ts`
   - Added rewardUsed validation
   - Added wallet deduction logic
   - Added transaction recording for reward usage

5. `src/routes/_auth/sign-in.tsx`
   - Changed redirect destination from /dashboard to /

---

## Conclusion

These improvements create a seamless reward experience:
- **One-time registration**: Enter info once in any tab, use everywhere
- **Flexible redemption**: Use as much or as little reward as desired
- **Automatic expiration**: Keeps reward system healthy and encourages usage
- **Better UX**: Staff see menu first, customers have unified experience

# Performance & Real-Time Updates Implementation

## Problem
Components were slow to update and required manual page refreshes to see changes after:
- Creating orders
- Using rewards
- Transferring wallet points
- Updating customer information

## Solution

### 1. **Query Invalidation After Mutations**
Added proper cache invalidation in all mutation success handlers:

#### FloatingCartButton.tsx
- Invalidates: `customerOrders`, `customerByPhone`, `wallet`, `walletHistory`, `breads`, `orders`
- Dispatches: `customer-info-updated`, `order-created` events

#### CartTab.tsx
- Invalidates: `wallet`, `walletHistory`, `customerByPhone`, `customerOrders`, `breads`, `orders`
- Dispatches: `customer-info-updated`, `order-created` events

#### WalletTab.tsx
- After registration: Invalidates `wallet`, `walletHistory`, `customerByPhone`
- After transfer: Invalidates wallet data for both sender and recipient
- Dispatches: `customer-info-updated` event

#### AccountTab.tsx
- After profile update: Invalidates `customer`, `customerByPhone`, `wallet`, `walletHistory`, `customerOrders`
- Dispatches: `customer-info-updated` event

### 2. **Event-Driven Updates**
Added window event listeners to automatically refresh data across components:

#### WalletTab
```typescript
useEffect(() => {
  const handleOrderCreated = () => {
    void queryClient.invalidateQueries({ queryKey: ['wallet', customerPhone] })
    void queryClient.invalidateQueries({ queryKey: ['walletHistory', customerPhone] })
  }
  window.addEventListener('order-created', handleOrderCreated)
  return () => window.removeEventListener('order-created', handleOrderCreated)
}, [customerPhone, queryClient])
```

#### AccountTab
```typescript
useEffect(() => {
  const handleDataUpdate = () => {
    void queryClient.invalidateQueries({ queryKey: ['customer', customerPhone] })
    void queryClient.invalidateQueries({ queryKey: ['customerOrders', customerPhone] })
  }
  window.addEventListener('order-created', handleDataUpdate)
  window.addEventListener('customer-info-updated', handleDataUpdate)
  return () => {
    window.removeEventListener('order-created', handleDataUpdate)
    window.removeEventListener('customer-info-updated', handleDataUpdate)
  }
}, [customerPhone, queryClient])
```

### 3. **Reduced Stale Times**
Dramatically reduced query stale times for faster updates:

| Query | Before | After |
|-------|--------|-------|
| Wallet balance | 10s | 1s |
| Wallet history | 10s | 1s |
| Customer data | 30s | 5s |
| Customer orders | 30s | 5s |
| Order refetch interval | 30s | 10s |

### 4. **Immediate Wallet Balance Loading**
Updated wallet balance loading to fetch immediately when:
- Drawer/form opens
- Phone number changes (≥8 digits)
- Order form is shown

### 5. **Fixed Missing Variables**
Added missing calculations in FloatingCartButton:
```typescript
const maxRewardsToUse = Math.min(walletBalance, totalPrice)
const finalPrice = totalPrice - rewardsToUse
const formattedFinal = formatPrice(finalPrice, language)
```

### 6. **Added Missing Imports**
- `Wallet`, `Sparkles` icons from lucide-react
- `Slider` component from ui
- `getWalletBalanceFn` from wallet functions
- `useQueryClient` from react-query

## Results

✅ **Instant Updates**: Changes appear immediately without page refresh
✅ **Cross-Component Sync**: All tabs update when data changes anywhere
✅ **Real-Time Order Tracking**: Orders refresh every 10 seconds
✅ **Wallet Balance**: Updates within 1 second after transactions
✅ **Customer Info**: Syncs across Cart, Wallet, and Account tabs

## Technical Details

### Query Key Structure
All queries now use consistent key patterns with parameters:
- `['wallet', phoneNumber]`
- `['walletHistory', phoneNumber]`
- `['customer', phoneNumber]`
- `['customerOrders', phoneNumber]`

This allows targeted invalidation per customer.

### Event System
Two custom events coordinate updates:
1. `customer-info-updated` - Fired when name/phone changes
2. `order-created` - Fired when new order is placed

Components listen for these events and invalidate their queries accordingly.

### Performance Impact
- Reduced unnecessary API calls through targeted invalidation
- Faster perceived performance (1-5s vs 30s+ before)
- Better UX with immediate feedback
- No manual refresh needed

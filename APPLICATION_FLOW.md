# 🥖 Semsem Bakery - Complete Application Flow

**Visual guide to understand how everything works together**

---

## 🎯 User Journeys

### 1. **First-Time Customer Journey**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. DISCOVER                                                 │
│    Customer opens app → Sees beautiful bread selection     │
│    ↓                                                        │
│ 2. BROWSE                                                   │
│    Views breads in grid/list → Sees live status & prices   │
│    ↓                                                        │
│ 3. SELECT                                                   │
│    Adds items to cart → Sees cart badge update             │
│    ↓                                                        │
│ 4. CHECKOUT                                                 │
│    Opens cart → Enters phone & name → Places order         │
│    ↓                                                        │
│ 5. TRACK                                                    │
│    Goes to Account tab → Sees order status with countdown  │
│    ↓                                                        │
│ 6. PICKUP                                                   │
│    Gets notification → Picks up order → Earns rewards      │
│    ↓                                                        │
│ 7. ENGAGE                                                   │
│    Joins community → Shares review → Becomes loyal         │
└─────────────────────────────────────────────────────────────┘
```

### 2. **Returning Customer Journey**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. QUICK ACCESS                                             │
│    Opens PWA from home screen → Instant load               │
│    ↓                                                        │
│ 2. CHECK STATUS                                             │
│    Sees favorite bread is ready → Adds to cart             │
│    ↓                                                        │
│ 3. USE REWARDS                                              │
│    Opens cart → Applies wallet balance → Saves money       │
│    ↓                                                        │
│ 4. FAST CHECKOUT                                            │
│    Info pre-filled → One-tap order → Confirmation          │
│    ↓                                                        │
│ 5. COMMUNITY                                                │
│    Shares photo → Gets likes → Earns VIP status            │
└─────────────────────────────────────────────────────────────┘
```

### 3. **Staff Member Journey**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. MORNING SETUP                                            │
│    Signs in → Opens bakery → Sets announcement             │
│    ↓                                                        │
│ 2. INVENTORY UPDATE                                         │
│    Adds new breads → Updates quantities → Sets timers      │
│    ↓                                                        │
│ 3. ORDER PROCESSING                                         │
│    Receives orders → Confirms → Updates to preparing       │
│    ↓                                                        │
│ 4. READY NOTIFICATION                                       │
│    Marks ready → System notifies customer → Countdown      │
│    ↓                                                        │
│ 5. CUSTOMER SERVICE                                         │
│    Approves verifications → Replies to posts → Manages     │
│    ↓                                                        │
│ 6. END OF DAY                                               │
│    Reviews analytics → Plans tomorrow → Closes bakery      │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Data Flow

### Order Creation Flow

```
Customer Side                Server Side                Database
─────────────                ───────────                ────────

[Add to Cart]
     │
     ├─> Store in state
     │
[Checkout]
     │
     ├─> Validate form
     │        │
     │        ├─> Check phone (8+ digits)
     │        ├─> Check name (required)
     │        └─> Check cart (not empty)
     │
     ├─> Call createCustomerOrderFn()
     │                │
     │                ├─> Authenticate
     │                ├─> Register customer ──> [customers table]
     │                ├─> Create order ──────> [orders table]
     │                ├─> Create order items ─> [order_items table]
     │                ├─> Update bread stock ─> [breads table]
     │                ├─> Record transaction ─> [wallet_transactions]
     │                └─> Return order ID
     │
     ├─> Show success
     ├─> Clear cart
     ├─> Invalidate queries
     └─> Navigate to orders

Staff Side
──────────

[Dashboard]
     │
     ├─> Query orders (real-time)
     │        │
     │        └─> Filter by status
     │
[Update Status]
     │
     ├─> Call updateOrderStatusFn()
     │        │
     │        ├─> Validate permission
     │        ├─> Update order ──────> [orders table]
     │        ├─> Set ready timestamp
     │        ├─> Calculate deadline
     │        └─> Notify customer
     │
     └─> Refresh order list
```

### Rewards System Flow

```
Order Completed
     │
     ├─> Calculate reward (5% of total)
     │
     ├─> Create transaction ──────> [wallet_transactions]
     │        │
     │        ├─> type: 'earn'
     │        ├─> amount: calculated
     │        ├─> expiresAt: +90 days
     │        └─> isExpired: false
     │
     ├─> Update wallet balance ───> [wallets table]
     │        │
     │        ├─> balance += amount
     │        └─> totalEarned += amount
     │
     └─> Show notification to customer

Customer Uses Rewards
     │
     ├─> Select amount in cart
     │
     ├─> Validate balance
     │        │
     │        └─> Check if sufficient
     │
     ├─> Create order with rewardUsed
     │        │
     │        ├─> Create transaction ──> [wallet_transactions]
     │        │        │
     │        │        ├─> type: 'spend'
     │        │        └─> amount: -used
     │        │
     │        └─> Update wallet ──────> [wallets table]
     │                 │
     │                 ├─> balance -= used
     │                 └─> totalSpent += used
     │
     └─> Complete order with discount

Expiration Check (Daily)
     │
     ├─> Query expired transactions
     │        │
     │        └─> WHERE expiresAt < now AND !isExpired
     │
     ├─> Mark as expired ──────────> [wallet_transactions]
     │        │
     │        └─> isExpired: true
     │
     └─> Update wallet balance ────> [wallets table]
              │
              └─> balance -= expired amount
```

### Phone Verification Flow

```
Customer Side                Server Side                Database
─────────────                ───────────                ────────

[Request Verification]
     │
     ├─> Enter phone & name
     │
     ├─> Call requestPhoneVerificationFn()
     │                │
     │                ├─> Validate input
     │                ├─> Check for duplicates
     │                ├─> Create verification ──> [phone_verifications]
     │                │        │
     │                │        ├─> status: 'pending'
     │                │        ├─> customerPhone
     │                │        └─> customerName
     │                │
     │                └─> Return success
     │
     └─> Show confirmation

Staff Side
──────────

[Dashboard → Verifications Tab]
     │
     ├─> Query pending verifications
     │        │
     │        └─> WHERE status = 'pending'
     │
[Approve/Reject]
     │
     ├─> Call approvePhoneVerificationFn()
     │        │
     │        ├─> Validate permission
     │        ├─> Update verification ──> [phone_verifications]
     │        │        │
     │        │        ├─> status: 'verified'/'rejected'
     │        │        ├─> verifiedBy: staff ID
     │        │        ├─> verifiedAt: timestamp
     │        │        └─> rejectedReason: if rejected
     │        │
     │        └─> Return success
     │
     └─> Refresh list

Customer Side (Real-time)
─────────────

[Account Tab]
     │
     ├─> Query verification status (5s staleTime)
     │        │
     │        └─> Show badge if verified
     │
     └─> Update UI based on status
```

---

## 🎨 Component Hierarchy

### Customer App

```
App
├── LanguageProvider
│   └── QueryClientProvider
│       └── Router
│           ├── _public (Layout)
│           │   └── index (Home)
│           │       ├── BakeryHeader
│           │       │   ├── Logo
│           │       │   ├── StatusBadge
│           │       │   └── Announcement
│           │       ├── BottomTabBar
│           │       │   ├── HomeTab
│           │       │   ├── WalletTab
│           │       │   ├── CommunityTab
│           │       │   └── AccountTab
│           │       ├── BreadGrid
│           │       │   └── BreadCard[]
│           │       │       ├── Image
│           │       │       ├── StatusBadge
│           │       │       ├── CountdownTimer
│           │       │       ├── PriceDisplay
│           │       │       ├── ActionButtons
│           │       │       └── ReminderButton
│           │       ├── FloatingCartButton
│           │       │   └── CartDrawer
│           │       │       ├── CustomerInfoForm
│           │       │       ├── RewardsSlider
│           │       │       ├── CartItemList
│           │       │       └── CheckoutButton
│           │       ├── WalletTab
│           │       │   ├── BalanceCard
│           │       │   ├── TransferForm
│           │       │   ├── TransactionHistory
│           │       │   └── PendingTransfers
│           │       ├── CommunityTab
│           │       │   ├── PostList
│           │       │   ├── PostCard
│           │       │   ├── CommentSection
│           │       │   └── CreatePostButton
│           │       └── AccountTab
│           │           ├── ProfileHeader
│           │           ├── OrdersList
│           │           ├── SettingsGrid
│           │           ├── LanguageDialog
│           │           ├── VerificationButton
│           │           └── SupportModal
│           └── _protected (Layout)
│               └── dashboard
│                   ├── Header
│                   ├── TabsList
│                   ├── InventoryTab
│                   │   ├── StatsCards
│                   │   ├── BreadTable
│                   │   └── BreadForm
│                   ├── OrdersTab
│                   │   ├── OrdersList
│                   │   ├── OrderCard
│                   │   └── StatusButtons
│                   ├── CustomersTab
│                   │   ├── CustomerList
│                   │   ├── CustomerCard
│                   │   └── LoyaltyBadge
│                   ├── ReservationsTab
│                   │   ├── ReservationList
│                   │   └── ReservationForm
│                   ├── VerificationsTab
│                   │   ├── PendingList
│                   │   └── ApproveRejectButtons
│                   ├── CommunityTab
│                   │   ├── PostModeration
│                   │   └── StaffReply
│                   ├── SupportTab
│                   │   ├── TicketList
│                   │   └── ReplyForm
│                   └── SettingsTab
│                       ├── BakeryControls
│                       ├── AppSettings
│                       ├── RewardsConfig
│                       └── PasswordChange
```

---

## 🔐 Permission Model

### Row-Level Security

```
Table: breads
─────────────
CREATE: Staff only (createdBy = currentUser.$id)
READ:   Everyone (public)
UPDATE: Owner only (createdBy = currentUser.$id)
DELETE: Owner only (createdBy = currentUser.$id)

Table: orders
─────────────
CREATE: Anyone (customer or staff)
READ:   Owner or staff (createdBy = currentUser.$id)
UPDATE: Staff only (createdBy = staff.$id)
DELETE: Staff only (createdBy = staff.$id)

Table: customers
────────────────
CREATE: Anyone (self-registration)
READ:   Owner or staff (createdBy = currentUser.$id)
UPDATE: Owner or staff (createdBy = currentUser.$id)
DELETE: Staff only

Table: wallets
──────────────
CREATE: System (on customer registration)
READ:   Owner only (createdBy = currentUser.$id)
UPDATE: System only (via server functions)
DELETE: Never

Table: phone_verifications
──────────────────────────
CREATE: Anyone (customer request)
READ:   Owner or staff
UPDATE: Staff only (approval/rejection)
DELETE: Staff only

Table: community_posts
──────────────────────
CREATE: Anyone (with moderation)
READ:   Everyone (if approved)
UPDATE: Owner or staff
DELETE: Owner or staff
```

---

## 📊 State Management

### Client State (React Query)

```
Query Keys:
───────────
['breads']                          → All breads
['bakeryStatus']                    → Open/closed status
['customer', phone]                 → Customer by phone
['customerOrders', phone]           → Orders for customer
['wallet', phone]                   → Wallet balance
['walletHistory', phone]            → Transaction history
['verificationStatus', phone]       → Verification status
['communityPosts']                  → All posts
['orders']                          → All orders (staff)
['customers']                       → All customers (staff)
['reservations']                    → All reservations (staff)
['pendingVerifications']            → Pending verifications (staff)

Mutations:
──────────
createBread                         → Add new bread
updateBread                         → Edit bread
deleteBread                         → Remove bread
createOrder                         → Place order
updateOrderStatus                   → Change order status
registerCustomer                    → Create/update customer
transferRewards                     → Send rewards
claimTransfer                       → Receive rewards
createPost                          → New community post
requestVerification                 → Request phone verification
approveVerification                 → Approve verification
rejectVerification                  → Reject verification

Invalidation Strategy:
──────────────────────
After mutation → Invalidate related queries
After order → Invalidate ['orders'], ['customerOrders'], ['wallet']
After bread update → Invalidate ['breads']
After verification → Invalidate ['verificationStatus'], ['pendingVerifications']
```

### Local State (localStorage)

```
Keys:
─────
CUSTOMER_PHONE                      → User's phone number
CUSTOMER_NAME                       → User's name
CUSTOMER_ADDRESS                    → User's address
USER_LANGUAGE                       → Selected language (en/ar/fr)
LANGUAGE_SELECTED                   → Has user chosen language
CART_ITEMS                          → Shopping cart contents
VIEW_MODE                           → Grid or list view
COACH_MARKS_SHOWN                   → Tutorial completion

Sync Strategy:
──────────────
Write → localStorage.setItem()
Read → localStorage.getItem()
Clear → localStorage.removeItem()
Event → window.dispatchEvent('storage')
Listen → window.addEventListener('storage')
```

---

## 🎯 Real-Time Features

### Live Updates

```
Feature: Bread Status
─────────────────────
Polling: Every 10 seconds
Query: ['breads'] with refetchInterval
Update: Automatic UI refresh
Countdown: Client-side timer with server sync

Feature: Order Tracking
───────────────────────
Polling: Every 10 seconds
Query: ['customerOrders', phone] with refetchInterval
Countdown: Pickup deadline timer
Notification: When status changes

Feature: Wallet Balance
───────────────────────
Polling: On drawer open
Query: ['wallet', phone]
Update: After every transaction
Expiration: Daily check for expired rewards

Feature: Verification Status
────────────────────────────
Polling: Every 5 seconds
Query: ['verificationStatus', phone]
Update: When staff approves/rejects
Badge: Shows verified status
```

---

## 🚀 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         VERCEL EDGE                         │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                    CDN Layer                          │ │
│  │  • Static assets (images, fonts, icons)              │ │
│  │  • Cached responses                                   │ │
│  │  • Edge functions                                     │ │
│  └───────────────────────────────────────────────────────┘ │
│                            │                                │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                 TanStack Start App                    │ │
│  │  • SSR rendering                                      │ │
│  │  • Server functions                                   │ │
│  │  • API routes                                         │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      APPWRITE CLOUD                         │
│                                                             │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  │
│  │   Database    │  │    Storage    │  │     Auth      │  │
│  │  • TablesDB   │  │  • Files      │  │  • Sessions   │  │
│  │  • Queries    │  │  • Images     │  │  • Users      │  │
│  │  • Indexes    │  │  • Uploads    │  │  • Tokens     │  │
│  └───────────────┘  └───────────────┘  └───────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT DEVICES                         │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Mobile  │  │  Tablet  │  │ Desktop  │  │   PWA    │  │
│  │  Safari  │  │  Chrome  │  │ Firefox  │  │  Installed│  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎉 Success!

Your bakery app is now fully documented and ready for production deployment!

**Key Takeaways:**
- ✅ Complete user journeys mapped
- ✅ Data flows documented
- ✅ Component hierarchy clear
- ✅ Permission model defined
- ✅ State management explained
- ✅ Real-time features outlined
- ✅ Deployment architecture shown

**Deploy with confidence! 🚀🥖**

---

**Last Updated:** February 8, 2026  
**Version:** 1.0.0  
**Status:** 🟢 Production Ready

# CTA Button Combinations - Test Matrix

## Test Scenarios

### Scenario 1: All CTAs Enabled (Default)
**Config**: Cart ✅ | Phone ✅ | WhatsApp ✅ | Reserve ✅

**Expected Result**:
- **Primary Action**: 🛒 Cart (full-width button with "Add to Cart" label)
- **Secondary Actions**: 📞 Phone + 💬 WhatsApp + 📅 Reserve (icon-only buttons below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  🛒  Add to Cart            │  ← Primary (orange gradient)
├─────────────────────────────┤
│  📞  │  💬  │  📅           │  ← Secondary (gray buttons)
└─────────────────────────────┘
```

---

### Scenario 2: Cart Disabled, All Others Enabled
**Config**: Cart ❌ | Phone ✅ | WhatsApp ✅ | Reserve ✅

**Expected Result**:
- **Primary Action**: 📅 Reserve (full-width button with "Reserve by Phone" label)
- **Secondary Actions**: 📞 Phone + 💬 WhatsApp (icon-only buttons below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  📅  Reserve by Phone       │  ← Primary (green gradient)
├─────────────────────────────┤
│  📞  │  💬                  │  ← Secondary (gray buttons)
└─────────────────────────────┘
```

---

### Scenario 3: Only Phone & WhatsApp Enabled
**Config**: Cart ❌ | Phone ✅ | WhatsApp ✅ | Reserve ❌

**Expected Result**:
- **Primary Action**: 📞 Phone (full-width button with "Call Us" label)
- **Secondary Actions**: 💬 WhatsApp (icon-only button below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  📞  Call Us                │  ← Primary (green gradient)
├─────────────────────────────┤
│  💬                         │  ← Secondary (gray button)
└─────────────────────────────┘
```

---

### Scenario 4: Only WhatsApp Enabled
**Config**: Cart ❌ | Phone ❌ | WhatsApp ✅ | Reserve ❌

**Expected Result**:
- **Primary Action**: 💬 WhatsApp (full-width button with "WhatsApp" label)
- **Secondary Actions**: None

**Visual Layout**:
```
┌─────────────────────────────┐
│  💬  WhatsApp               │  ← Primary (green gradient)
└─────────────────────────────┘
```

---

### Scenario 5: Only Phone Enabled
**Config**: Cart ❌ | Phone ✅ | WhatsApp ❌ | Reserve ❌

**Expected Result**:
- **Primary Action**: 📞 Phone (full-width button with "Call Us" label)
- **Secondary Actions**: None

**Visual Layout**:
```
┌─────────────────────────────┐
│  📞  Call Us                │  ← Primary (green gradient)
└─────────────────────────────┘
```

---

### Scenario 6: Only Reserve Enabled
**Config**: Cart ❌ | Phone ❌ | WhatsApp ❌ | Reserve ✅

**Expected Result**:
- **Primary Action**: 📅 Reserve (full-width button with "Reserve by Phone" label)
- **Secondary Actions**: None

**Visual Layout**:
```
┌─────────────────────────────┐
│  📅  Reserve by Phone       │  ← Primary (green gradient)
└─────────────────────────────┘
```

---

### Scenario 7: Cart + Phone Only
**Config**: Cart ✅ | Phone ✅ | WhatsApp ❌ | Reserve ❌

**Expected Result**:
- **Primary Action**: 🛒 Cart (full-width button)
- **Secondary Actions**: 📞 Phone (icon-only button below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  🛒  Add to Cart            │  ← Primary (orange gradient)
├─────────────────────────────┤
│  📞                         │  ← Secondary (gray button)
└─────────────────────────────┘
```

---

### Scenario 8: Cart + WhatsApp Only
**Config**: Cart ✅ | Phone ❌ | WhatsApp ✅ | Reserve ❌

**Expected Result**:
- **Primary Action**: 🛒 Cart (full-width button)
- **Secondary Actions**: 💬 WhatsApp (icon-only button below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  🛒  Add to Cart            │  ← Primary (orange gradient)
├─────────────────────────────┤
│  💬                         │  ← Secondary (gray button)
└─────────────────────────────┘
```

---

### Scenario 9: Cart + Reserve Only
**Config**: Cart ✅ | Phone ❌ | WhatsApp ❌ | Reserve ✅

**Expected Result**:
- **Primary Action**: 🛒 Cart (full-width button)
- **Secondary Actions**: 📅 Reserve (icon-only button below)

**Visual Layout**:
```
┌─────────────────────────────┐
│  🛒  Add to Cart            │  ← Primary (orange gradient)
├─────────────────────────────┤
│  📅                         │  ← Secondary (gray button)
└─────────────────────────────┘
```

---

### Scenario 10: All CTAs Disabled
**Config**: Cart ❌ | Phone ❌ | WhatsApp ❌ | Reserve ❌

**Expected Result**:
- **No action buttons shown** (bread card shows info only)
- Sold out items show "Notify Me" button if phone number is configured

---

## Priority Order

The system follows this priority when selecting the primary action:

1. **Cart** (if enabled)
2. **Reserve** (if cart disabled)
3. **Phone** (if cart & reserve disabled)
4. **WhatsApp** (if only WhatsApp enabled)

All other enabled actions become secondary (icon-only) buttons.

---

## How to Test

### In Dashboard → App Settings:

1. Go to **CTA Buttons** section
2. Toggle each button on/off to test combinations
3. Save settings
4. Open PWA in new tab/incognito to see changes
5. Verify primary and secondary buttons match expected layout

### Visual Checks:

- ✅ Primary button has full width + label + gradient background
- ✅ Secondary buttons are icon-only + gray background
- ✅ Secondary buttons don't duplicate the primary action
- ✅ Buttons work correctly (cart adds item, phone calls, etc.)
- ✅ Sold out items show "Notify Me" instead of action buttons

---

## Common Issues Fixed

### ❌ Before (Broken):
- Disabling cart would hide ALL buttons
- Secondary actions only showed when cart was enabled
- Phone/WhatsApp buttons disappeared when cart was off

### ✅ After (Fixed):
- Disabling cart promotes next action to primary
- Secondary actions show regardless of cart state
- All enabled CTAs are always visible
- No duplicate buttons (primary never appears in secondary)

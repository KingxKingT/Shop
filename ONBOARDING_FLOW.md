# Onboarding Flow - PWA Install → Coach Marks

## ✅ Sequential Onboarding Experience

The app now has a proper first-time user onboarding flow that shows prompts in the correct order:

### Flow Sequence:

1. **Language Selection** (First time only)
   - User selects Arabic or English
   - Stored in localStorage
   - Required before any other onboarding

2. **PWA Install Prompt** (First time only)
   - Shows after language is selected
   - Centered modal overlay with gradient design
   - Two options:
     - **"Later"** - Dismisses for 7 days
     - **"Install"** - Triggers browser install prompt
   
3. **Coach Marks** (After PWA prompt is dismissed)
   - Only shows AFTER user responds to PWA install
   - Highlights the grid/list view toggle button
   - Spotlight effect with glowing ring
   - Tooltip explains the feature
   - Tap anywhere to dismiss

## 🎯 Implementation Details

### State Management

```typescript
// Track PWA install completion
const [pwaInstallComplete, setPwaInstallComplete] = useState(false)

// Ref to the view mode toggle button
const viewModeToggleRef = useRef<HTMLButtonElement>(null)
```

### Component Communication

**InstallBanner** → Notifies parent when dismissed:
```typescript
<InstallBanner onComplete={() => setPwaInstallComplete(true)} />
```

**CoachMarks** → Waits for trigger:
```typescript
<CoachMarks 
  targetRef={viewModeToggleRef} 
  trigger={pwaInstallComplete}
/>
```

**BreadGrid** → Receives ref for the toggle button:
```typescript
<BreadGrid
  viewModeToggleRef={viewModeToggleRef}
  // ... other props
/>
```

## 📱 User Experience

### First Visit:
1. User opens app
2. Language overlay appears
3. User selects language
4. PWA install prompt appears (centered modal)
5. User clicks "Install" or "Later"
6. **800ms delay**
7. Coach marks appear highlighting grid/list toggle
8. User taps to dismiss
9. Normal app experience begins

### Subsequent Visits:
- No language selection (already saved)
- No PWA prompt (dismissed for 7 days or already installed)
- No coach marks (already shown once)
- Direct to app content

## 🎨 Visual Design

### PWA Install Prompt:
- **Gradient background**: Amber to Orange
- **White icon container** with Download icon
- **Two-button layout**: "Later" (ghost) + "Install" (solid white)
- **Backdrop blur** for focus
- **Smooth animations**: fade-in + zoom-in

### Coach Marks:
- **Dark overlay** with spotlight cutout
- **Glowing ring** around target (pulsing animation)
- **White tooltip bubble** with arrow
- **Close button** in corner
- **"Tap to dismiss"** hint at bottom

## 🔧 Technical Features

### Persistence:
- **Language**: `localStorage.getItem('language-selected')`
- **PWA Install**: `localStorage.getItem('bakery-install-dismissed')`
- **Coach Marks**: `localStorage.getItem('bakery-coach-marks-shown')`

### Timing:
- **PWA prompt**: Shows immediately after language selection
- **Coach marks delay**: 800ms after PWA prompt dismissed
- **PWA re-prompt**: 7 days after dismissal

### Conditional Rendering:
```typescript
// PWA Install - shows if not dismissed and not installed
if (isInstalled || dismissed) return null

// Coach Marks - shows if:
// 1. Language selected
// 2. Not shown before
// 3. PWA install complete (trigger = true)
if (!trigger || coachMarksShown) return null
```

## 🚀 Benefits

### For Users:
- ✅ Clear, sequential onboarding
- ✅ No overwhelming multiple prompts at once
- ✅ Can dismiss and continue using app
- ✅ Helpful hints about features
- ✅ Non-intrusive (shows once)

### For Developers:
- ✅ Clean state management
- ✅ Reusable components
- ✅ Easy to extend with more steps
- ✅ Proper TypeScript typing
- ✅ Callback-based coordination

## 📝 Code Structure

```
src/
├── components/bakery/
│   ├── InstallBanner.tsx      # PWA install prompt
│   ├── CoachMarks.tsx         # Feature hints
│   ├── BreadGrid.tsx          # Accepts viewModeToggleRef
│   └── LanguageOverlay.tsx    # Language selection
└── routes/_public/
    └── index.tsx              # Coordinates onboarding flow
```

## 🎯 Future Enhancements

Potential additions to the onboarding flow:

1. **Welcome tour** - Multi-step walkthrough
2. **Feature highlights** - Show new features after updates
3. **Contextual tips** - Show hints based on user actions
4. **Onboarding progress** - Track completion percentage
5. **Skip option** - Let users skip entire onboarding

## ✨ Result

The app now provides a professional, non-intrusive onboarding experience that:
- Guides new users through key features
- Respects user choices (can dismiss)
- Doesn't repeat unnecessarily
- Maintains smooth app performance
- Works seamlessly with offline support

All onboarding elements are properly coordinated and only show when appropriate, creating a polished first-time user experience.

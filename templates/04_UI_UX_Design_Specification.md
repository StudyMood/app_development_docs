# 🎨 UI / UX Design Specification & Design System

> **Document Owner:** Lead UI/UX Product Designer  
> **Target Audience:** Frontend / Mobile Engineers, QA, Product Managers  
> **Tools Used:** Figma / Adobe XD / Tokens Studio  
> **Figma Workspace Link:** `https://figma.com/@project-workspace`  
> **Version:** 1.0  

---

## 1. Information Architecture & Navigation Flow

App ke screen hierarchy aur flow ka overview:

```
[Splash Screen]
      │
      ▼
[Onboarding / Phone OTP Login]
      │
      ▼
[Main Bottom Navigation Bar]
 ├── 🏠 Tab 1: Home (Hyperlocal Restaurants, Top Deals, Search Bar)
 ├── 🔍 Tab 2: Explore / Categories (Cuisines, Healthy Bowls, Quick Snacks)
 ├── 🛒 Tab 3: Cart & Active Orders (Live status ticker)
 └── 👤 Tab 4: Profile & Settings (Saved Addresses, Corporate Invoices, Help)
```

---

## 2. Design Tokens & Visual Style Guide

### 2.1 Color Palette

| Token Name | Hex Code | Purpose / Usage |
|------------|----------|-----------------|
| `--color-primary` | `#FF4B2B` | Brand color, Primary Call-To-Action buttons, active tabs |
| `--color-primary-hover` | `#E03E20` | Pressed / Hover state for primary buttons |
| `--color-secondary` | `#0D1B2A` | Primary text headings, dark navbar, footer |
| `--color-accent-green` | `#10B981` | Success states, Pure Veg indicator, discounts badge |
| `--color-accent-red` | `#EF4444` | Non-Veg indicator, Error alerts, destructive actions |
| `--color-surface-bg` | `#F8FAFC` | App screen background |
| `--color-surface-card` | `#FFFFFF` | Restaurant cards, modal dialogs, bottom sheets |
| `--color-text-muted` | `#64748B` | Subtitles, timestamps, placeholder text |

### 2.2 Typography Scale (Font: Inter / Outfit)

| Style | Font Size | Line Height | Font Weight | Usage |
|-------|-----------|-------------|-------------|-------|
| `Display / H1` | 28px | 34px | Bold (700) | Major screen headers (e.g. Welcome back) |
| `Heading / H2` | 20px | 26px | Semi-Bold (600) | Restaurant card names, Section titles |
| `Heading / H3` | 16px | 22px | Semi-Bold (600) | Food item titles, Cart total |
| `Body / Regular`| 14px | 20px | Regular (400) | Item descriptions, reviews |
| `Caption / Small`| 12px | 16px | Medium (500) | Badges, tags, ETA pill text |

### 2.3 Spacing & Radius System (8-Point Grid)

- **Spacing Tokens:**
  - `space-xs`: 4px
  - `space-sm`: 8px
  - `space-md`: 16px (Standard screen padding)
  - `space-lg`: 24px
  - `space-xl`: 32px
- **Border Radius:**
  - Buttons & Inputs: `12px` (Rounded modern corners)
  - Cards & Bottom Sheets: `16px`
  - Badges & Pills: `999px` (Pill / Fully rounded)

---

## 3. Core Component Specs & States

Har interactive element ke sabhi states defined hone chahiye taaki developer ko guess na karna pade:

### 3.1 Primary Button (`CTA Button`)
- **Height:** 52px (Mobile thumb friendly)
- **Width:** 100% full-width on bottom bar, auto on desktop
- **States:**
  - `Default`: Background `#FF4B2B`, White text, 12px radius, subtle shadow.
  - `Pressed`: Background `#E03E20`, scaled down 2% (`transform: scale(0.98)`).
  - `Disabled`: Background `#CBD5E1`, text `#94A3B8`, no ripple effect.
  - `Loading`: Spinner animation (Lottie / SVG) centered, text hidden, disabled to prevent double taps.

### 3.2 Skeleton / Shimmer Loading Screen
- Shimmer gradient animation running left-to-right (Duration: 1.2s, infinite loop).
- Placeholder shapes match exact layout of cards to prevent Layout Shift (CLS = 0).

---

## 4. Micro-Interactions & Animations

1. **Add to Cart Bounce:**
   - Jab user "+ Add" button dabaye, button animate hokar "+ / - counter" pill ban jaye.
   - Haptic Feedback: Light click vibration on mobile device.
2. **Pull-to-Refresh:**
   - Custom food-themed spinner pulls down from top, spins smoothly, and bounces back with ease-out curve.
3. **Screen Transition:**
   - Shared Element Transition for food image moving from list view to dish details bottom sheet (300ms cubic-bezier(0.4, 0.0, 0.2, 1)).

---

## 5. Accessibility & Device Adaptations

- **Contrast Ratio:** Text and interactive elements must satisfy minimum **WCAG 2.1 AA** (4.5:1 for body text, 3:1 for large text).
- **Dark Mode Support:** Automatic switching based on system preference; dark surface colors using `#121212` and `#1E1E1E` (avoiding pure `#000000` to prevent eye strain).
- **Safe Area Insets:** Top status bar and bottom gesture bar padding respected for iPhone notch and Android punch-hole displays.

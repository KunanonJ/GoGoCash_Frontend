# CURSOR.md — GoGoCash 1.1 Development Plan

This file is the **single source of truth** for Cursor AI when developing GoGoCash 1.1.
Read this entire file before writing any code. Follow every rule exactly.

---

## 0. Two-Repo Strategy (CRITICAL — Read First)

GoGoCash runs two repositories. You must understand both before touching any code.

| Repo | Purpose | Stack |
|---|---|---|
| [`mygogocash/gogocash_app`](https://github.com/mygogocash/gogocash_app) | **GoGoCash 1.0** — production app, live users | Next.js 16, MUI, next-intl, Firebase, Ethers.js, Crossmint |
| [`KunanonJ/GoGoCash_Frontend`](https://github.com/KunanonJ/GoGoCash_Frontend) | **GoGoCash 1.1** — THIS REPO, redesign target | Next.js 15.1, Tailwind, no MUI, Wagmi/Viem |

**Your job:** Rebuild every feature from `gogocash_app` (1.0) into this repo (1.1) using the new design system from [Figma node-id=2-908](https://www.figma.com/design/jFDx8MnbCtlCaTQxlhpJIp/GoGoCash-1.1?node-id=2-908). Do NOT copy-paste 1.0 code directly — re-implement it cleanly using the 1.1 architecture rules below.

---

## 1. What Exists in GoGoCash 1.0 (Your Source of Truth for Logic)

Use these 1.0 features as the **functional specification** for what to build in 1.1.

### 1.0 App Routes (`src/app/[locale]/`)

| 1.0 Route | Feature | 1.1 Target Route |
|---|---|---|
| `/` (home) | Home feed with merchant cards | `src/app/(page)/home/` |
| `/login` | Email + social login | `src/app/login/` |
| `/register` | Multi-step signup | `src/app/sign-up/` |
| `/auth/*` | Auth sub-pages | `src/app/login/` |
| `/shop` | Merchant directory | `src/app/shop/` |
| `/shop/[id]` | Merchant detail | `src/app/shop/[merchantId]/` |
| `/category` | Category browse | Merge into shop filter |
| `/quest` | Quest list | `src/app/(page)/quest/` |
| `/quest/[id]` | Quest detail | `src/app/(page)/quest/[questId]/` |
| `/(profile)/*` | Profile, settings | `src/app/profile/` |

### 1.0 Features to Port (`src/features/`)

| 1.0 Feature Dir | What It Does | 1.1 Feature Dir |
|---|---|---|
| `features/auth` | Login, register, OTP, social | `features/mobile/login` + `features/mobile/register` |
| `features/home` | Home feed, banner, merchant grid | `features/mobile/home` |
| `features/shop` | Merchant listing, detail page | `features/mobile/shop` |
| `features/category` | Category filter | Merge into shop feature |
| `features/quest` | Quest list + detail | New: `features/mobile/quest` |
| `features/wallet` | Balance, withdraw | `src/app/wallet/` |
| `features/transaction` | Tx history | `src/app/history/` |
| `features/referral` | Referral/affiliate | New: `features/mobile/affiliate` |
| `features/subscription` | Subscription plans | New: `features/mobile/subscription` |
| `features/profile` | Profile, settings | `src/app/profile/` |
| `features/search` | Search merchants | Merge into shop header |

### 1.0 Common Components to Upgrade (Do NOT copy — rebuild with new design)

| 1.0 Component | Location | 1.1 Action |
|---|---|---|
| `Button.tsx` | `components/common/Button.tsx` | Rebuild with Tailwind variants |
| `Input.tsx` | `components/common/Input.tsx` | Rebuild with Tailwind + react-hook-form |
| `OtpInput.tsx` | `components/common/OtpInput.tsx` | **REUSE LOGIC** — just restyle |
| `Title.tsx` | `components/common/Title.tsx` | Rebuild as typography system |
| `Step.tsx` | `components/common/Step.tsx` | Rebuild as `StepIndicator` |
| `ViewAll.tsx` | `components/common/ViewAll.tsx` | Rebuild as `SectionHeader` |
| `card/*` | `components/common/card/` | Rebuild as MerchantCard, QuestCard, etc. |

---

## 2. Tech Stack Delta: 1.0 → 1.1

| Concern | 1.0 (gogocash_app) | 1.1 (THIS REPO) | Migration Action |
|---|---|---|---|
| Framework | Next.js 16 | Next.js 15.1 | ✅ Already set |
| UI Library | MUI v7 + Emotion | **Tailwind CSS only** | Remove MUI entirely |
| i18n | next-intl v4 (locale routing) | next-intl v4 | ✅ Keep — add `src/messages/` + `src/i18n/` |
| Auth | NextAuth v4 + Firebase | NextAuth v4 + Firebase | Port `.env` Firebase keys |
| Web3/Wallet | Ethers.js v6 | **Wagmi v2 + Viem** | Replace Ethers calls |
| NFT/Wallet UI | Crossmint SDK | Crossmint SDK | ✅ Keep |
| State/Data | TanStack Query v5 | TanStack Query v5 | ✅ Keep |
| Analytics | GTM + GA4 + PostHog + Meta Pixel | Same | Port analytics components from 1.0 |
| Toast | react-hot-toast | react-hot-toast | ✅ Keep |
| Carousel | Swiper v12 | Swiper v12 | ✅ Keep |
| Phone validation | libphonenumber-js | libphonenumber-js | ✅ Keep |
| Testing | None in 1.0 | Jest + RTL | Write new tests |
| CSS approach | Tailwind v4 + MUI | **Tailwind v4 only** | No Emotion/styled-components |

---

## 3. Architecture Rules (Mandatory for 1.1)

### File Naming
- Components: `PascalCase.tsx`
- Utilities/hooks: `camelCase.ts`
- Interface file per component folder: `interface.ts` with props prefixed `I` (e.g., `IMerchantCardProps`)
- Barrel export per folder: `index.ts`

### Directory Structure
```
src/
  app/
    [locale]/              ← i18n routing wrapper (copy from 1.0)
      (page)/              ← authenticated pages
      auth/                ← login, register, OTP
      shop/
        [merchantId]/
      quest/
        [questId]/
      wallet/
      history/
      profile/
      notification/
      affiliate/
  features/
    mobile/
      [feature]/
        views/             ← page-level views
        components/        ← feature-local components
        hooks/             ← feature-local hooks
  components/
    common/
      [ComponentName]/
        ComponentName.tsx
        interface.ts
        index.ts
  hooks/                   ← global hooks
  lib/                     ← apiClient, utils
  constants/               ← enums, routes
  providers/               ← AuthProvider, WalletProvider
  i18n/                    ← next-intl config (port from 1.0)
  messages/
    en.json                ← English strings
    th.json                ← Thai strings
  interfaces/              ← global TypeScript interfaces
```

### Import Order
1. External libraries
2. Internal aliases (`@/components/...`)
3. Relative imports (`./Component`)

---

## 4. Environment Variables (Port from 1.0 `.env.example`)

Copy `.env.example` → `.env.local`. These vars come directly from `mygogocash/gogocash_app`:

```bash
# Core
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_FRONTEND_URL=http://localhost:3000
NEXTAUTH_SECRET=replace-with-a-long-random-string

# Analytics (all same as 1.0)
NEXT_PUBLIC_ANALYTICS_ENABLED=true
NEXT_PUBLIC_GTM_ID=GTM-WVGBK9HM
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-Q66JRSM0MB
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
NEXT_PUBLIC_META_PIXEL_ID=207487147928890
NEXT_PUBLIC_META_USER_SALT=replace-with-shared-hash-salt
NEXT_PUBLIC_ANALYTICS_DEBUG=false

# Firebase (same as 1.0)
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Crossmint (same as 1.0)
NEXT_PUBLIC_CROSSMINT_API_KEY=
NEXT_PUBLIC_CROSSMINT_COLLECTION_ID=

# Telegram (same as 1.0)
NEXT_PUBLIC_TELEGRAM_BOT_TOKEN=
NEXT_PUBLIC_TELEGRAM_BOT_USERNAME=

# Web3 — withdraw contracts (same as 1.0 but use Wagmi chain IDs)
NEXT_PUBLIC_CHAIN_ID_WITHDRAW_POLYGON=
NEXT_PUBLIC_CHAIN_ID_WITHDRAW_BNB=
NEXT_PUBLIC_CHAIN_ID_WITHDRAW_SONIC=
NEXT_PUBLIC_CHAIN_ID_WITHDRAW_CELO=
NEXT_PUBLIC_CONTRACT_WITHDRAW_ADDRESS_POLYGON=
NEXT_PUBLIC_CONTRACT_WITHDRAW_ADDRESS_BNB=
NEXT_PUBLIC_CONTRACT_WITHDRAW_ADDRESS_SONIC=
NEXT_PUBLIC_CONTRACT_WITHDRAW_ADDRESS_CELO=
```

---

## 5. i18n Setup (Port Directly from 1.0)

GoGoCash 1.0 uses `next-intl` with locale routing under `src/app/[locale]/`. **Replicate this exactly in 1.1.**

1. Copy `next-intl.config.ts` from 1.0 → 1.1 root
2. Copy `src/i18n/` directory from 1.0
3. Copy `src/messages/en.json` + `src/messages/th.json` from 1.0 as base
4. Wrap `src/app/layout.tsx` with `NextIntlClientProvider` (same as 1.0 `src/app/[locale]/layout.tsx`)
5. Update `middleware.ts` to include locale routing (merge with auth guard)

---

## 6. Analytics (Port Components from 1.0)

1.0 has a `src/components/analytics/` directory with GTM, GA4, PostHog, and Meta Pixel components.
- Copy the entire `src/components/analytics/` folder from 1.0 into 1.1
- These are environment-gated via `NEXT_PUBLIC_ANALYTICS_ENABLED`
- Do NOT rewrite — they work, just port them

---

## 7. Sprint Plan — 12 Weeks

### SPRINT 1 (Weeks 1–2): Foundation & Design System
**Gate:** Nothing else starts until Sprint 1 is merged.

- [ ] `T-01` Install missing 1.1 dependencies:
  ```bash
  yarn add next-intl wagmi viem @tanstack/react-query react-hot-toast swiper libphonenumber-js firebase next-auth @crossmint/client-sdk-react-ui ethers lodash react-error-boundary
  ```
- [ ] `T-02` Port `next-intl.config.ts`, `src/i18n/`, `src/messages/en.json`, `src/messages/th.json` from 1.0
- [ ] `T-03` Port `src/components/analytics/` from 1.0 unchanged
- [ ] `T-04` Extract all Figma design tokens (node-id=2-908) → populate `src/styles/globals.css` CSS variables
- [ ] `T-05` Extend `tailwind.config.js` with custom colors, fonts, spacing from Figma
- [ ] `T-06` Build atomic components in `src/components/common/` (each needs `Component.tsx` + `interface.ts` + `index.ts`):
  - `Button` — variants: primary, ghost, destructive, loading (restyle from 1.0 `Button.tsx` logic)
  - `Input` — text, password, search (restyle from 1.0 `Input.tsx`)
  - `OtpInput` — **port logic from 1.0 `OtpInput.tsx` directly**, restyle only
  - `Badge` — cashback-%, status, category
  - `MerchantCard` — merchant tile with logo, cashback %, CTA
  - `QuestCard` — progress bar, reward, expiry
  - `BottomSheet` — mobile drawer modal
  - `Skeleton` — loading placeholder
  - `StepIndicator` — restyle from 1.0 `Step.tsx`
  - `SectionHeader` — restyle from 1.0 `ViewAll.tsx`
  - `Avatar` — user/merchant image with fallback initials
- [ ] `T-07` Dark/light mode toggle via `data-theme` on `<html>`
- [ ] `T-08` Set up `src/lib/apiClient.ts` with Axios interceptors (auth header + 401 redirect)

**Branch:** `feature/1.1-design-system`

---

### SPRINT 2 (Weeks 3–5): Auth & i18n
**Goal:** Full auth flow in EN and TH, identical to 1.0 functionality.

**Port from 1.0 `src/features/auth/` — reimplement with Tailwind:**
- [ ] `T-09` Splash/Intro screen — `src/features/mobile/intro/`
- [ ] `T-10` Login page — `src/app/[locale]/auth/login/`
  - Email + password form
  - Google OAuth (NextAuth, same as 1.0)
  - Firebase phone auth option
  - "Forgot password" → reset flow
- [ ] `T-11` Register flow — `src/app/[locale]/auth/register/`
  - Step 1: Email + password (port validation from 1.0)
  - Step 2: Phone OTP — **use 1.0 `OtpInput.tsx` logic** with libphonenumber-js validation
  - Step 3: Profile setup + referral code input
- [ ] `T-12` Reset/new password pages — port from 1.0
- [ ] `T-13` Auth middleware — update `middleware.ts`:
  - Locale detection + routing (next-intl)
  - Protected route guard (NextAuth session check)
  - Protected: `/(page)/*`, `/wallet`, `/profile`, `/history`, `/notification`, `/affiliate`
  - Public: `/auth/*`, `/shop` (browse only)
- [ ] `T-14` `src/providers/AuthProvider.tsx` — wrap NextAuth SessionProvider + user context

**Branch:** `feature/1.1-auth`

---

### SPRINT 3 (Weeks 6–8): Home Feed & Shop (Critical Path)
**Goal:** Core cashback loop — user browses merchants and activates a deal.

**Port from 1.0 `src/features/home/` and `src/features/shop/`:**

#### Home Feed — `src/features/mobile/home/`
- [ ] `T-15` Header: balance pill (USDT), notification bell with unread dot
- [ ] `T-16` Hero banner: Swiper carousel (port Swiper config from 1.0, restyle)
- [ ] `T-17` Category filter bar: horizontal scroll chips — data from `/categories` API
- [ ] `T-18` Merchant grid: 2-col `MerchantCard` tiles
- [ ] `T-19` Quest section: horizontal Swiper with `QuestCard` components
- [ ] `T-20` "For You" personalized section: mock data in Sprint 3, real API post-launch

#### Shop — `src/features/mobile/shop/`
- [ ] `T-21` Search bar with `useDebounce` hook (300ms) — port search logic from 1.0 `features/search/`
- [ ] `T-22` Filter `BottomSheet`: category, cashback %, new
- [ ] `T-23` Merchant storefront: `src/app/[locale]/shop/[merchantId]/page.tsx`
  - Merchant logo, cover, cashback badge, deal tiles
  - "Activate Cashback" CTA → calls `/deals/:id/activate`
  - "Shop Now" → external URL with UTM tracking params

#### Quest — `src/features/mobile/quest/` (NEW in 1.1)
- [ ] `T-24` Quest list page: `src/app/[locale]/quest/page.tsx`
- [ ] `T-25` Quest detail: `src/app/[locale]/quest/[questId]/page.tsx`
  - Progress bar, milestone badges, streak counter
  - Reward unlock animation (CSS keyframes, no heavy lib)

**API hooks (`src/hooks/`):**
```typescript
useGetMerchants(params: IGetMerchantsParams)       // paginated
useGetMerchantById(merchantId: string)
useGetCategories()
useGetQuests()
useActivateDeal(dealId: string)
useSearchMerchants(query: string)                  // debounced
```

**Branch:** `feature/1.1-cashback-engine`

---

### SPRINT 4 (Weeks 9–10): Wallet & Transactions
**Goal:** Port wallet + multi-chain withdraw from 1.0, replace Ethers.js with Wagmi.

**Port from 1.0 `src/features/wallet/` and `src/features/transaction/`:**

- [ ] `T-26` Wallet home: `src/app/[locale]/wallet/page.tsx`
  - Balance display (USDT/USDC)
  - Pending vs confirmed cashback breakdown
  - Action buttons: Withdraw, History
- [ ] `T-27` Multi-chain withdraw flow (replaces Ethers.js with Wagmi v2):
  - Chain selector: Polygon, BNB, Sonic, Celo (use env contract addresses)
  - Wagmi `useWriteContract` for withdraw transaction
  - OTP confirmation step before on-chain tx
  - Success screen with tx hash link
- [ ] `T-28` Transaction history: `src/app/[locale]/history/page.tsx`
  - Port from 1.0 `features/transaction/`
  - Filter tabs: All / Cashback / Withdrawal
  - Date-grouped list, status badge
  - Infinite scroll (TanStack Query `useInfiniteQuery`)
- [ ] `T-29` `src/providers/WalletProvider.tsx` — Wagmi `WagmiProvider` + QueryClient
- [ ] `T-30` Saving Plus teaser: `src/app/[locale]/(page)/saving/page.tsx`
  - Locked UI, "Coming Soon" badge, email waitlist form → POST `/waitlist`

**Wagmi replacement map:**
| 1.0 Ethers.js | 1.1 Wagmi/Viem |
|---|---|
| `new ethers.BrowserProvider(window.ethereum)` | `useWalletClient()` |
| `contract.withdraw(amount)` | `useWriteContract({ abi, address, functionName: 'withdraw' })` |
| `provider.getBalance(address)` | `useBalance({ address })` |
| `ethers.parseUnits(amount, 6)` | `parseUnits(amount, 6)` from viem |

**Branch:** `feature/1.1-wallet`

---

### SPRINT 5 (Weeks 11–12): Profile, Affiliate, Subscription & QA

**Port from 1.0 `src/features/profile/`, `src/features/referral/`, `src/features/subscription/`:**

- [ ] `T-31` Profile page: `src/app/[locale]/profile/page.tsx`
  - Avatar, name, email, tier badge
  - Referral code with copy + share
  - Settings: language toggle (EN/TH via next-intl), notifications, security
- [ ] `T-32` Affiliate center (upgrade from 1.0 `features/referral/`): `src/app/[locale]/affiliate/page.tsx`
  - Referral link generator
  - Tier progress (Bronze → Silver → Gold)
  - Earnings table with payout status
- [ ] `T-33` Subscription page (port from 1.0 `features/subscription/`): `src/app/[locale]/(page)/subscription/page.tsx`
- [ ] `T-34` Notifications: `src/app/[locale]/notification/page.tsx`
  - Port notification types from 1.0
  - Mark all read, swipe-to-dismiss
- [ ] `T-35` Crossmint integration: port `CrossmintErrorBoundary.tsx` from 1.0 `components/common/`
- [ ] `T-36` QA pass:
  - Lighthouse mobile score ≥ 85
  - All screens have: loading state (Skeleton), empty state, error state
  - `yarn lint` 0 errors, `yarn test` all pass
  - No hardcoded colors, no console.log, no .env secrets in code

**Branch:** `feature/1.1-profile-affiliate`

---

## 8. API Client Pattern

```typescript
// src/lib/apiClient.ts
import axios from 'axios';
import { getSession } from 'next-auth/react';

const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: { 'Content-Type': 'application/json' },
});

apiClient.interceptors.request.use(async (config) => {
  const session = await getSession();
  if (session?.accessToken) {
    config.headers.Authorization = `Bearer ${session.accessToken}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  (res) => res,
  (err) => {
    if (err.response?.status === 401) {
      window.location.href = '/auth/login';
    }
    return Promise.reject(err);
  }
);

export default apiClient;
```

---

## 9. Component Rules

Every component MUST have:
1. Typed interface in `interface.ts`
2. Loading state using `<Skeleton />`
3. Empty state (custom UI, not null/undefined)
4. Error state with retry button where applicable
5. No hardcoded colors — only `var(--token-name)` or Tailwind classes

```typescript
// interface.ts
export interface IMerchantCardProps {
  merchantId: string;
  name: string;
  logoUrl: string;
  cashbackPercent: number;
  isNew?: boolean;
  isLoading?: boolean;
  onClick?: () => void;
}
```

---

## 10. Git Workflow

```
main        ← production (never commit directly)
develop     ← integration (all PRs target here)
feature/*   ← one branch per sprint task
```

### Commit Format (Conventional Commits)
```
feat(shop): add merchant storefront with deal activation
fix(wallet): correct Wagmi chain ID mismatch on BNB withdraw
chore(i18n): port en.json and th.json from 1.0
refactor(auth): replace Ethers provider with Wagmi useWalletClient
test(otp): add RTL tests for OtpInput component
```

### PR Checklist
- [ ] `yarn lint` passes (0 errors)
- [ ] `yarn test` passes
- [ ] Screenshots of all new/changed screens attached
- [ ] No `console.log` left in code
- [ ] No hardcoded hex colors or pixel values
- [ ] No `.env` values committed
- [ ] i18n: all user-facing strings added to both `en.json` and `th.json`

---

## 11. Performance Targets

| Metric | Target |
|---|---|
| First Contentful Paint | < 1.5s |
| Time to Interactive | < 3.0s |
| Lighthouse Mobile Score | ≥ 85 |
| Initial JS bundle | < 200kb gzipped |

**Rules:**
- `next/image` for all images — never `<img>`
- Dynamic import for heavy libs: `dynamic(() => import('./HeavyComponent'), { ssr: false })`
- Virtualize lists > 50 items using `react-virtual`
- Swiper: import only required modules (no full bundle)

---

## 12. Commands

```bash
yarn dev          # Start dev server
yarn build        # Production build
yarn lint         # ESLint
yarn format       # Prettier
yarn test         # Run all tests
yarn test:watch   # Watch mode
```

---

## 13. Figma Handoff Protocol

For every screen you implement:
1. Open [Figma GoGoCash 1.1 node-id=2-908](https://www.figma.com/design/jFDx8MnbCtlCaTQxlhpJIp/GoGoCash-1.1?node-id=2-908)
2. Select the exact frame → Inspect panel → extract padding, font-size, color → map to CSS variable
3. Export icons as SVG → `public/icons/`
4. Export images as WebP → `public/images/`
5. Figma auto-layout → Tailwind `flex` or `grid`
6. Figma corner radius → `rounded-{n}`
7. Every Figma component state (default/hover/disabled/loading) must be implemented

---

*Last updated: 2026-03-27 | GoGoCash CEO / Perplexity*
*Source repos: [1.0](https://github.com/mygogocash/gogocash_app) → [1.1](https://github.com/KunanonJ/GoGoCash_Frontend)*

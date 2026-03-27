# CURSOR.md — GoGoCash 1.1 Development Plan

This file is the single source of truth for Cursor AI when developing GoGoCash 1.1.
Read this entire file before writing any code. Follow every instruction exactly.

---

## 1. Project Context

**Product:** GoGoCash — a stablecoin cashback platform connecting users to 220+ partner merchants.
**Version:** 1.1 (Beta release)
**Design Source:** [Figma — GoGoCash 1.1 (node-id=2-908)](https://www.figma.com/design/jFDx8MnbCtlCaTQxlhpJIp/GoGoCash-1.1?node-id=2-908)
**Repo:** [KunanonJ/GoGoCash_Frontend](https://github.com/KunanonJ/GoGoCash_Frontend)
**Active Branch:** `develop` → feature branches → PR to `develop` → merge to `main` for releases

---

## 2. Tech Stack (Do Not Change)

| Layer | Technology |
|---|---|
| Framework | Next.js 15.1+ (App Router) |
| Language | TypeScript (strict mode) |
| UI | React 19.0 |
| Styling | TailwindCSS + CSS variables (`var(--primary-1)`) |
| State | React Context / hooks (no Redux) |
| Testing | Jest + React Testing Library |
| Linting | ESLint + Prettier (run `yarn format` before commit) |
| Package manager | Yarn |
| Containerization | Docker (existing Dockerfile) |

---

## 3. Architecture Rules (Mandatory)

### File Naming
- Components: `PascalCase.tsx` (e.g., `CashbackCard.tsx`)
- Utilities/hooks: `camelCase.ts` (e.g., `useCashback.ts`)
- Interfaces file per component: `interface.ts` with props prefixed `I` (e.g., `ICashbackCardProps`)
- Index barrel file per component folder: `index.ts`

### Directory Structure Pattern
```
src/
  app/
    (page)/              ← App Router pages (existing pattern)
    [new-route]/         ← Add new pages here
  features/
    mobile/
      [feature]/
        views/           ← Page-level view components
        components/      ← Feature-local components
        hooks/           ← Feature-local hooks
  components/
    common/
      [name]/
        Component.tsx
        interface.ts
        index.ts
  hooks/                 ← Global shared hooks
  lib/                   ← API clients, utilities
  constants/             ← Enums, static config
  providers/             ← Context providers
```

### Import Order (ESLint enforced)
1. External libraries (react, next, etc.)
2. Internal aliases (`@/components/...`)
3. Relative imports (`./Component`)

---

## 4. Design System Tokens

All visual values MUST come from CSS variables. Never hardcode hex colors or pixel values.

### Colors (define in `src/styles/globals.css`)
```css
:root {
  /* Brand */
  --primary-1: #[from Figma];      /* Primary brand color */
  --primary-2: #[from Figma];      /* Hover / active state */
  --cashback-green: #[from Figma]; /* Cashback value accent */
  --stablecoin-accent: #[from Figma];

  /* Neutrals */
  --bg-primary: #[from Figma];
  --bg-secondary: #[from Figma];
  --text-primary: #[from Figma];
  --text-secondary: #[from Figma];
  --border-default: #[from Figma];

  /* Semantic */
  --success: #[from Figma];
  --error: #[from Figma];
  --warning: #[from Figma];
}

[data-theme="dark"] {
  /* Override all tokens for dark mode */
}
```
> **ACTION FOR CURSOR:** Extract exact hex values from Figma node-id=2-908 Styles panel and populate the variables above.

### Typography Scale
- Font family: as specified in Figma (default: Inter or Satoshi)
- Scale: `text-xs` → `text-2xl` using Tailwind; map to Figma text styles
- Line heights and letter spacing from Figma → extend `tailwind.config.js`

### Spacing Grid
- Base unit: 4px
- All padding/margin use multiples of 4px (Tailwind p-1=4px, p-2=8px, etc.)

---

## 5. Sprint Plan — 12 Weeks

### SPRINT 1 (Weeks 1–2): Design System & Token Infrastructure
**Goal:** Zero UI work starts until this sprint is done.

**Tasks:**
- [ ] `T-01` Extract all design tokens from Figma node-id=2-908 → populate `globals.css`
- [ ] `T-02` Extend `tailwind.config.js` with custom colors, fonts, and spacing from Figma
- [ ] `T-03` Build atomic components in `src/components/common/`:
  - `Button` (variants: primary, ghost, destructive, loading state)
  - `Input` (variants: text, OTP, search, password)
  - `Badge` (variants: cashback-%, status, category)
  - `Card` (variants: merchant-tile, cashback-summary, quest-card)
  - `BottomSheet` (modal drawer for mobile)
  - `Avatar` (user/merchant profile image)
  - `Skeleton` (loading placeholder for all card types)
  - `Icon` (SVG sprite wrapper: `<Icon name="cashback" size={24} />`)
- [ ] `T-04` Set up dark/light mode toggle via `data-theme` attribute on `<html>`
- [ ] `T-05` Write Jest snapshot tests for every atomic component

**Branch:** `feature/1.1-design-system`

---

### SPRINT 2 (Weeks 3–5): Authentication & Onboarding
**Goal:** A new user can register, verify, and reach the home screen in <90 seconds.

**Existing routes to UPGRADE** (not replace):
- `src/app/login/` → `src/features/mobile/login/`
- `src/app/sign-up/` → `src/features/mobile/register/`
- `src/app/reset-password/` → `src/features/mobile/reset-password/`

**New screens to BUILD** (match Figma exactly):
- [ ] `T-06` Splash / intro screen — `src/features/mobile/intro/`
  - Animated logo entry, "Get Started" CTA
- [ ] `T-07` Sign Up flow — upgrade `src/features/mobile/register/`
  - Step 1: Email + password
  - Step 2: Phone OTP (6-digit `Input` OTP component, resend timer 60s)
  - Step 3: Profile setup (name, avatar upload optional)
  - Referral code field (maps affiliate attribution to user record)
- [ ] `T-08` Login — upgrade `src/features/mobile/login/`
  - Email/password + "Forgot password" link
  - Social login buttons (Google OAuth — use NextAuth.js)
- [ ] `T-09` Auth guard middleware — update `middleware.ts`
  - Protected routes: `/wallet`, `/profile`, `/history`, `/(page)/*`
  - Unprotected: `/login`, `/sign-up`, `/reset-password`, `/shop` (browse-only)

**Branch:** `feature/1.1-auth`

---

### SPRINT 3 (Weeks 6–8): Core Cashback Engine (Critical Path)
**Goal:** User can browse merchants, activate a deal, and see cashback credited to their wallet.

**Screens to BUILD:**

#### Home Feed — upgrade `src/features/mobile/home/`
- [ ] `T-10` Header: GoGoCash balance pill (USDT), notification bell with unread badge
- [ ] `T-11` Hero banner: rotating promotional carousel (swipeable, auto-advance 5s)
- [ ] `T-12` Category filter bar: horizontal scroll, icon+label chips (Food, Fashion, Travel, etc.)
- [ ] `T-13` Merchant grid: 2-column `Card` tiles with merchant logo, name, cashback %, "Shop Now" CTA
- [ ] `T-14` Quest section: horizontal scroll cards with progress bar, reward amount, expiry countdown
- [ ] `T-15` Personalized section: "For You" AI-driven offers (mock API in Sprint 3, real data post-launch)

#### Shop / Merchant Directory — upgrade `src/features/mobile/shop/`
- [ ] `T-16` Search bar with debounced API call (`useDebounce` hook, 300ms)
- [ ] `T-17` Filter/sort bottom sheet: by category, cashback %, new arrivals
- [ ] `T-18` Merchant storefront page: `src/app/shop/[merchantId]/page.tsx`
  - Logo, cover image, cashback rate badge
  - Deal tiles with terms
  - "Activate Cashback" CTA → triggers transaction tracking
  - "Shop Now" button → opens merchant URL with tracking param

#### Quest System
- [ ] `T-19` Quest detail page: `src/app/(page)/quest/[questId]/page.tsx`
  - Progress tracker, milestone badges, reward unlock animation (CSS keyframes)
  - Streak counter component

**API Hooks (create in `src/hooks/`):**
```typescript
// Required hooks
useGetMerchants(params: IGetMerchantsParams)
useGetMerchantById(merchantId: string)
useGetQuests()
useActivateDeal(dealId: string)
useSearchMerchants(query: string)
```

**Branch:** `feature/1.1-cashback-engine`

---

### SPRINT 4 (Weeks 9–10): Wallet & Rewards Hub
**Goal:** User can view cashback balance, transaction history, and initiate withdrawal.

**Screens to BUILD** — upgrade `src/app/wallet/`:
- [ ] `T-20` Wallet home: stablecoin balance (USDT/USDC), pending vs confirmed cashback, action buttons (Withdraw, History)
- [ ] `T-21` Transaction history: upgrade `src/app/history/` — filterable list (all/cashback/withdrawal), date grouping, status badge
- [ ] `T-22` Withdrawal flow:
  - Step 1: Enter amount (min/max validation)
  - Step 2: Select destination (bank account / crypto wallet)
  - Step 3: OTP confirmation
  - Step 4: Success screen with estimated arrival time
- [ ] `T-23` Saving Plus teaser screen: `src/app/(page)/saving/page.tsx`
  - Locked UI with "Coming Soon" badge
  - Email waitlist capture form → POST to `/api/waitlist`
- [ ] `T-24` Cashback Lending placeholder: design-visible, non-functional, "2026" badge

**API Hooks:**
```typescript
useGetWalletBalance(userId: string)
useGetTransactionHistory(userId: string, filters: ITransactionFilters)
useInitiateWithdrawal(payload: IWithdrawalPayload)
```

**Branch:** `feature/1.1-wallet`

---

### SPRINT 5 (Weeks 11–12): Profile, Affiliate & Polish
**Goal:** Complete all remaining screens, run QA, and prepare for beta launch.

**Screens to BUILD:**

#### Profile — upgrade `src/app/profile/`
- [ ] `T-25` Profile overview: avatar, name, email, tier badge (Bronze/Silver/Gold), referral code
- [ ] `T-26` Settings: notification preferences, language (EN/TH), security (change password, 2FA toggle)
- [ ] `T-27` KYC status card: verified/unverified state with "Verify Now" CTA

#### Affiliate / Referral Center
- [ ] `T-28` New route: `src/app/(page)/affiliate/page.tsx`
  - Referral link with copy-to-clipboard + share sheet
  - Tier progress bar (Bronze → Silver → Gold based on referrals)
  - Earnings table: referral count, earned cashback, payout status

#### Notifications — upgrade `src/app/notification/`
- [ ] `T-29` Notification list: icon type (cashback credited, quest complete, promo, system)
  - Mark all as read action
  - Swipe-to-dismiss (mobile gesture)

#### QA & Performance
- [ ] `T-30` Lighthouse score ≥ 85 on mobile (performance + accessibility)
- [ ] `T-31` All API error states handled: empty state component, retry button, toast notifications
- [ ] `T-32` Skeleton loaders on all data-fetching screens
- [ ] `T-33` Run `yarn test` — all tests passing, coverage ≥ 70% on new components

**Branch:** `feature/1.1-profile-affiliate`

---

## 6. API Contract

All API calls go through `src/lib/apiClient.ts`. Base URL from `process.env.NEXT_PUBLIC_API_URL`.

```typescript
// src/lib/apiClient.ts pattern
const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: { 'Content-Type': 'application/json' },
});

// Attach JWT from auth context on every request
apiClient.interceptors.request.use(attachAuthHeader);
// Handle 401 → redirect to /login
apiClient.interceptors.response.use(handleSuccess, handleAuthError);
```

### Endpoint Reference

| Method | Endpoint | Feature |
|---|---|---|
| POST | `/auth/login` | Login |
| POST | `/auth/register` | Sign Up |
| POST | `/auth/otp/verify` | OTP verification |
| GET | `/merchants` | Merchant list (paginated) |
| GET | `/merchants/:id` | Merchant detail |
| GET | `/merchants/search?q=` | Search |
| GET | `/quests` | Quest list |
| POST | `/deals/:id/activate` | Activate cashback deal |
| GET | `/wallet/balance` | Wallet balance |
| GET | `/wallet/transactions` | Transaction history |
| POST | `/wallet/withdraw` | Initiate withdrawal |
| GET | `/affiliate/stats` | Affiliate dashboard |
| POST | `/waitlist` | Saving Plus waitlist |

---

## 7. Component Implementation Rules

### Every component MUST have:
1. A typed interface in `interface.ts`
2. Default props defined
3. Loading state (use `Skeleton` component)
4. Empty state (custom empty UI, not null)
5. Error state (show error message + retry if applicable)

### Example pattern:
```typescript
// interface.ts
export interface IMerchantCardProps {
  merchantId: string;
  name: string;
  logoUrl: string;
  cashbackPercent: number;
  isNew?: boolean;
  onClick?: () => void;
}

// MerchantCard.tsx
export const MerchantCard: React.FC<IMerchantCardProps> = ({
  name,
  logoUrl,
  cashbackPercent,
  isNew = false,
  onClick,
}) => {
  return (
    <div
      className="rounded-2xl bg-[var(--bg-secondary)] p-4 cursor-pointer"
      onClick={onClick}
    >
      {/* ... */}
    </div>
  );
};
```

---

## 8. State Management Rules

- **Auth state:** `src/providers/AuthProvider.tsx` — user object, token, login/logout actions
- **Wallet state:** `src/providers/WalletProvider.tsx` — balance, pending cashback, refresh
- **UI state:** Local `useState` only — do NOT put transient UI state in providers
- **Server state:** React Query (`@tanstack/react-query`) for all API data fetching
  - Cache time: 5 minutes for merchant list, 30 seconds for wallet balance
  - Stale time: 1 minute for merchant data

---

## 9. Git Workflow

```
main           ← production releases only
develop        ← integration branch (PRs target here)
feature/1.1-*  ← one branch per sprint (see Sprint Plan above)
```

### Commit Message Format (Conventional Commits)
```
feat(wallet): add withdrawal flow with OTP confirmation
fix(auth): correct OTP resend timer not resetting
chore(deps): upgrade wagmi to v2.5
style(home): align merchant grid to 4px grid
test(cashback): add unit tests for useCashback hook
```

### PR Checklist
- [ ] `yarn lint` passes with 0 errors
- [ ] `yarn test` passes
- [ ] Screenshots of all new screens attached
- [ ] No `console.log` statements
- [ ] No hardcoded colors or pixel values (use CSS variables)
- [ ] No `.env` secrets committed

---

## 10. Environment Variables

Copy `.env.example` to `.env.local`. Never commit `.env.local`.

```bash
NEXT_PUBLIC_API_URL=https://api.gogocash.co/v1
NEXT_PUBLIC_CHAIN_ID=56                          # BNB Chain mainnet
NEXT_PUBLIC_STABLECOIN_CONTRACT=0x...            # USDT/USDC contract
NEXT_AUTH_SECRET=...                             # NextAuth secret
NEXT_PUBLIC_GOOGLE_CLIENT_ID=...                 # Google OAuth
```

---

## 11. Performance Targets

| Metric | Target |
|---|---|
| First Contentful Paint | < 1.5s |
| Time to Interactive | < 3.0s |
| Lighthouse Mobile Score | ≥ 85 |
| Bundle size (initial JS) | < 200kb gzipped |
| API response time (P95) | < 500ms |

**Techniques:**
- Use `next/image` for all images — never `<img>` tags
- Dynamic import for heavy components: `const QuestAnimation = dynamic(() => import('./QuestAnimation'), { ssr: false })`
- Virtualize long lists (merchant directory) using `react-virtual`

---

## 12. Commands Reference

```bash
yarn dev          # Start dev server
yarn build        # Production build
yarn lint         # ESLint check
yarn format       # Prettier format
yarn test         # Run all tests
yarn test:watch   # Watch mode
```

---

## 13. Figma Handoff Notes for Cursor

When implementing any screen:
1. Open Figma node-id=2-908
2. Select the exact frame/component
3. Use Figma's "Inspect" panel for: exact padding, font size, color hex → map to CSS variable
4. Export icons as SVGs → place in `public/icons/`
5. Export images as WebP → place in `public/images/`
6. Every Figma auto-layout → Tailwind `flex` or `grid`
7. Figma corner radius → Tailwind `rounded-{n}`

---

*Last updated: 2026-03-27 by Perplexity / GoGoCash CEO*

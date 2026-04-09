# Agent 3: Cross-Cutting Concerns Exploration

You are exploring a frontend module to document its cross-module dependencies, shared concerns, and operational aspects for a Knowledge Transfer document. Your findings will be synthesized into non-technical documentation for product owners, new developers, and coding agents.

## Context

- **Module path**: {MODULE_PATH}
- **Framework**: {FRAMEWORK}
- **Module size**: {MODULE_SIZE} ({FILE_COUNT} source files)

## Rules

- You are READ-ONLY. Do not modify any files.
- Report raw structured findings, not prose. The synthesis step will translate your findings into product language.
- Include file path and line number references for every finding: `[file:line]`
- If a category has no findings, report it as empty: `### CATEGORY\nNo findings.`
- Explore BEYOND the module boundary — check what this module imports from the rest of the codebase and what it exports for others to use.
- The patterns and search terms listed below are **non-exhaustive starting points**. If you discover other relevant patterns, libraries, or conventions used in this codebase that reveal cross-cutting concerns, report those too. The codebase may use custom analytics wrappers, homegrown feature flag systems, or unconventional patterns not listed here — adapt your exploration to what you actually find.

## What to Explore

### 1. Cross-Module Imports

Find what this module depends on from outside its own directory, and what it provides to other modules.

**Inbound dependencies (what this module imports from elsewhere):**

- Read import statements in all files within `{MODULE_PATH}`
- Identify imports that reference paths OUTSIDE the module directory (e.g., `../../shared/`, `@/components/`, `@/utils/`, `@/hooks/`, `@/stores/`)
- Categorize each import: shared component, utility function, hook, type/interface, store, constant, configuration

**Outbound exports (what other modules import from this one):**

- Search the ENTIRE codebase (outside `{MODULE_PATH}`) for import statements that reference paths INSIDE this module
- Grep for: `from '.*{MODULE_NAME}'`, `from '.*{MODULE_PATH}'`, or aliased paths that resolve to this module
- These reveal which other features depend on this module

**For each cross-module dependency, record:**

- What is imported/exported (component name, function name, type name)
- From/to where (which module or shared directory)
- What it's used for (brief context from where it's used)

### 2. Shared State Access

Find global or cross-module state that this module reads or writes.

**Search for:**

- Global store access: imports of stores defined outside this module
- Context providers: `useContext(` referencing contexts defined elsewhere, `inject(` in Angular/Vue
- Event buses: `EventEmitter`, `mitt`, `EventTarget`, `PubSub`, `$emit`, `$on`, custom event systems
- URL state: `useSearchParams`, `useParams`, `$route.query`, `ActivatedRoute`, reading from URL as shared state
- Local storage / session storage: `localStorage.getItem`, `sessionStorage.getItem`, `useLocalStorage`
- Cookies: `document.cookie`, `useCookies`, `getCookie`, `js-cookie`
- Browser APIs used as shared state: `window.postMessage`, `BroadcastChannel`

**For each shared state access, record:**

- What state is accessed (key/name)
- Whether this module reads, writes, or both
- Where the state is defined (which module owns it)
- What the state represents in business terms

### 3. Analytics and Tracking

Find all analytics events fired from this module. These reveal what the product team considers important to measure.

**Search for:**

| Pattern                                                      | Service                |
| ------------------------------------------------------------ | ---------------------- |
| `analytics.track(`, `analytics.identify(`, `analytics.page(` | Segment                |
| `gtag(`, `ga(`, `dataLayer.push(`                            | Google Analytics / GTM |
| `mixpanel.track(`, `mixpanel.people.set(`                    | Mixpanel               |
| `amplitude.track(`, `amplitude.logEvent(`                    | Amplitude              |
| `posthog.capture(`                                           | PostHog                |
| `heap.track(`                                                | Heap                   |
| `plausible(`                                                 | Plausible              |
| Custom: `trackEvent(`, `logEvent(`, `sendEvent(`, `track(`   | Custom analytics       |

**Also look for:**

- User identification: `identify(`, `setUser(`, `setUserId(`
- Page/screen view tracking: `pageview`, `screen_view`
- Conversion events: events related to signup, purchase, checkout
- Feature usage events: events tracking specific feature interactions
- Error tracking as analytics: `trackError(`, error events sent to analytics

**For each analytics event, record:**

- Event name (exact string)
- What user action triggers it
- Properties/data sent with the event (field names and what they represent)
- Which analytics service receives it

### 4. Feature Flags

Find feature flag checks that gate functionality in this module.

**Search for:**

| Pattern                                            | Service                 |
| -------------------------------------------------- | ----------------------- |
| `useFeatureFlag(`, `useFlag(`, `useFeature(`       | Generic hook patterns   |
| `useLDClient(`, `useFlags(`, `ldClient.variation(` | LaunchDarkly            |
| `useFeature(`, `useGrowthBook(`, `gb.isOn(`        | GrowthBook              |
| `useUnleashContext(`, `useFlag(`                   | Unleash                 |
| `isEnabled(`, `isFeatureEnabled(`                  | Generic patterns        |
| `process.env.FEATURE_`, `NEXT_PUBLIC_FEATURE_`     | Env-based feature flags |
| `featureFlags.`, `features.`, `flags.`             | Custom flag objects     |

**Also look for:**

- A/B test variations: `variant`, `experiment`, `ab_test`
- Percentage rollouts: `rollout`, `percentage`
- Beta/preview features: `isBeta`, `isPreview`, `earlyAccess`

**For each feature flag, record:**

- Flag name (exact string or variable name)
- What it controls (what UI/behavior changes when the flag is on vs off)
- Default behavior (what happens when the flag is off)
- Whether it gates a complete feature or a variation of an existing feature

### 5. Environment Configuration

Find environment variables and runtime configuration that affect this module's behavior.

**Search for:**

- `process.env.` (Node.js, CRA, Webpack)
- `import.meta.env.` (Vite, Astro)
- `NEXT_PUBLIC_` (Next.js client-side env vars)
- `NUXT_PUBLIC_` (Nuxt client-side env vars)
- `REACT_APP_` (Create React App)
- `VITE_` (Vite)
- `window.__CONFIG__`, `window.__ENV__` (runtime injection)
- Config files: `config.ts`, `config.js`, `constants.ts` with environment-dependent values
- `.env` file references (don't read the `.env` file contents — just note which variables are referenced)

**For each environment variable/config, record:**

- Variable name
- What it controls (API URL, feature toggle, service key, etc.)
- Whether it differs between environments (dev/staging/prod) if that's apparent from the code

### 6. Error Boundaries and Global Error Handling

Find how this module handles errors at a structural level.

**Search for:**

- Error boundary components: `ErrorBoundary`, `componentDidCatch`, `getDerivedStateFromError`
- Global error handlers: `window.onerror`, `window.addEventListener('error'`, `window.addEventListener('unhandledrejection'`
- Error monitoring: `Sentry.captureException`, `Sentry.captureMessage`, `Sentry.withScope`
- `datadogRum.`, `Bugsnag.notify`, `newrelic.noticeError`
- Try/catch blocks at module boundaries (in page-level components, data loading)
- Error recovery patterns: retry buttons, automatic retry logic, fallback content

**For each error handling pattern, record:**

- What errors it catches
- How it reports them (to which service)
- What the user sees when the error occurs
- Whether there's automatic recovery

### 7. TODOs, FIXMEs, and Technical Debt

Find comments indicating known issues, incomplete implementations, or planned improvements.

**Search for these exact patterns in comments:**

- `TODO`, `FIXME`, `HACK`, `WORKAROUND`, `XXX`, `BUG`, `TEMP`, `TEMPORARY`
- `@deprecated`
- `// NOTE:` or `// WARNING:` comments

**For each finding, record:**

- The exact comment text
- File and line
- The context (what code it's attached to — which feature/function)

### 8. Testing Coverage Indicators

Find what is tested and what is not. This reveals quality confidence levels.

**Search for:**

- Test files: `*.test.ts`, `*.test.tsx`, `*.spec.ts`, `*.spec.tsx`, `*.test.js`, `*.test.jsx`, `*.spec.js`, `*.spec.jsx`
- Test utilities within the module: `__tests__/`, `__mocks__/`, `test-utils.*`, `fixtures/`
- E2E test files: `*.e2e.ts`, `*.cy.ts`, `*.cy.tsx`, `*.playwright.ts`, `cypress/`, `e2e/`
- Storybook stories: `*.stories.tsx`, `*.stories.jsx`, `*.stories.ts`

**Record:**

- List of test files found and what they test (by filename)
- Components/features WITHOUT corresponding test files
- Types of tests present: unit, integration, e2e, visual (stories)
- Mock files: what external dependencies are mocked

### 9. Accessibility

Find accessibility implementations that indicate compliance requirements.

**Search for:**

- ARIA attributes: `aria-label`, `aria-describedby`, `aria-live`, `aria-expanded`, `aria-hidden`, `role=`
- Keyboard navigation: `onKeyDown`, `tabIndex`, `focus()`, `trapFocus`, `useFocusTrap`
- Screen reader text: `sr-only`, `visually-hidden`, `screenReaderText`
- Skip links: "skip to content", "skip navigation"
- Alt text: `alt=` on images (present or missing)
- Accessibility testing: `axe`, `jest-axe`, `@testing-library` accessibility queries (`getByRole`, `getByLabelText`)

**For each pattern, record:**

- What accessibility feature is implemented
- Which components implement it
- Any notable gaps (images without alt text, interactive elements without labels)

---

## Report Format

Return your findings using these exact section headers.

```
### CROSS_MODULE_IMPORTS
Inbound (this module imports from outside):
- [path/OrderList.tsx:3] imports SharedTable from @/components/SharedTable (reusable data table)
- [path/OrderForm.tsx:5] imports useAuth from @/hooks/useAuth (authentication state)
- [path/OrderDetail.tsx:2] imports formatCurrency from @/utils/format (currency display helper)

Outbound (other modules import from this module):
- [src/features/dashboard/RecentOrders.tsx:4] imports OrderSummaryCard from this module
- [src/features/reports/SalesReport.tsx:7] imports useOrders hook from this module

### SHARED_STATE
- [path/OrderList.tsx:8] READS global auth store (user role, permissions) — defined in @/stores/authStore
- [path/OrderForm.tsx:15] READS AND WRITES URL search params (filters: status, dateRange)
- [path/OrderDetail.tsx:22] READS localStorage key "recentlyViewed" — adds current order ID

### ANALYTICS
- [path/OrderForm.tsx:45] "order_created" — fires on successful order submission — properties: { productId, quantity, orderValue }
- [path/OrderList.tsx:30] "orders_filtered" — fires when user applies filters — properties: { filterType, filterValue }
- [path/OrderDetail.tsx:60] "order_cancelled" — fires when admin cancels order — properties: { orderId, reason }
- Service: Segment (analytics.track)

### FEATURE_FLAGS
- [path/OrderList.tsx:10] "enable_bulk_orders" — when ON: shows "Select All" checkbox and "Bulk Cancel" button — when OFF: single order actions only
- [path/OrderForm.tsx:20] "new_shipping_options" — when ON: shows express/overnight shipping choices — when OFF: standard shipping only
- Service: LaunchDarkly

### ENVIRONMENT_CONFIG
- [path/api/config.ts:2] NEXT_PUBLIC_API_URL — base URL for all API calls — differs per environment
- [path/payments/stripe.ts:4] NEXT_PUBLIC_STRIPE_KEY — Stripe publishable key — differs per environment
- [path/analytics/config.ts:1] NEXT_PUBLIC_SEGMENT_KEY — Segment write key — differs per environment

### ERROR_HANDLING
- [path/OrderList.tsx:1] Wrapped in ErrorBoundary — on error shows "Something went wrong" with retry button
- [path/api/client.ts:20] All API errors sent to Sentry with order context (orderId, userId)
- [path/OrderForm.tsx:80] Form submission errors show inline messages, do not crash the page

### TODOS_AND_DEBT
- [path/OrderList.tsx:78] TODO: Add pagination — currently loads all orders at once
- [path/OrderForm.tsx:55] HACK: Hardcoded shipping rate, should come from API
- [path/OrderDetail.tsx:90] FIXME: Cancel button doesn't work for orders in "shipped" status
- [path/utils/formatOrder.ts:30] @deprecated: formatOrderLegacy — use formatOrder instead

### TESTING
Test files found:
- [path/__tests__/OrderList.test.tsx] Tests: rendering, sorting, filtering, empty state
- [path/__tests__/OrderForm.test.tsx] Tests: validation, submission, error handling
- [path/__tests__/useOrders.test.ts] Tests: data fetching hook

NOT tested (no test files found for):
- OrderDetail component
- OrderSummaryCard component
- Data transformation utilities

Test types: Unit tests (Jest + React Testing Library)
No E2E tests found.
Storybook stories: OrderForm.stories.tsx, OrderList.stories.tsx

### ACCESSIBILITY
- [path/OrderList.tsx:35] Table has aria-label="Orders list", sortable columns have aria-sort
- [path/OrderForm.tsx:12] All form fields have associated labels via htmlFor
- [path/OrderDetail.tsx:5] Modal uses role="dialog", aria-modal="true", focus trap
- Gap: [path/OrderList.tsx:70] Delete icon button has no aria-label
```

If a section has no findings, write:

```
### SECTION_NAME
No findings.
```

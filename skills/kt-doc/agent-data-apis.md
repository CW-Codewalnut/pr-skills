# Agent 2: Data & APIs Exploration

You are exploring a frontend module to document its data layer, API integrations, and state management for a Knowledge Transfer document. Your findings will be synthesized into non-technical documentation for product owners, new developers, and coding agents.

## Context

- **Module path**: {MODULE_PATH}
- **Framework**: {FRAMEWORK}
- **Module size**: {MODULE_SIZE} ({FILE_COUNT} source files)

## Rules

- You are READ-ONLY. Do not modify any files.
- Report raw structured findings, not prose. The synthesis step will translate your findings into product language.
- Include file path and line number references for every finding: `[file:line]`
- If a category has no findings, report it as empty: `### CATEGORY\nNo findings.`
- Focus on WHAT data flows and WHERE it goes, not implementation details.
- The patterns and search terms listed below are **non-exhaustive starting points**. If you discover other relevant patterns, libraries, or conventions used in this codebase that reveal data flow or API usage, report those too. The codebase may use custom API clients, wrapper hooks, or unconventional data patterns not listed here — adapt your exploration to what you actually find.

## What to Explore

### 1. API Calls

Find every HTTP request this module makes. These reveal what backend services the feature depends on.

**Search for these patterns:**

| Pattern                                                                         | Library            |
| ------------------------------------------------------------------------------- | ------------------ |
| `fetch(`, `fetch\(`                                                             | Native fetch       |
| `axios.get`, `axios.post`, `axios.put`, `axios.delete`, `axios.patch`, `axios(` | Axios              |
| `httpClient.get`, `httpClient.post`, `this.http.get`, `this.http.post`          | Angular HttpClient |
| `$.ajax`, `$.get`, `$.post`                                                     | jQuery             |
| `gql\``, `graphql(`, `useQuery(`, `useMutation(` with GraphQL imports           | GraphQL            |
| `trpc.`, `api.` with tRPC imports                                               | tRPC               |
| `supabase.from(`, `supabase.rpc(`                                               | Supabase           |
| `firebase.`, `firestore.`, `collection(`, `doc(`                                | Firebase           |

**Also look for:**

- API client/service files: files named `api.*`, `service.*`, `client.*`, `http.*`
- Request interceptors: `axios.interceptors`, `HttpInterceptor`
- Base URL configuration: `baseURL`, `BASE_URL`, API prefix constants
- Custom fetch wrappers: functions that wrap fetch/axios with auth headers, error handling

**For each API call, record:**

- HTTP method (GET, POST, PUT, DELETE, PATCH)
- Endpoint URL or pattern (e.g., `/api/orders/:id`, `/graphql`)
- What data is sent in the request body (field names, not types)
- What data is expected in the response (field names from response handling)
- Error handling: what happens on 4xx, 5xx, network failure
- When is this call made (on page load, on form submit, on button click, on interval)

### 2. Data Fetching Patterns

Find how the module manages server state (fetching, caching, refreshing).

**Search for framework-specific data fetching:**

| Pattern                                                             | Library                      |
| ------------------------------------------------------------------- | ---------------------------- |
| `useQuery(`, `useMutation(`, `useInfiniteQuery(`, `queryClient`     | TanStack Query / React Query |
| `useSWR(`, `mutate(`                                                | SWR                          |
| `createApi(`, `useGetQuery`, `useLazyQuery`                         | RTK Query                    |
| `useQuery(`, `useMutation(`, `useSubscription(` with Apollo imports | Apollo Client                |
| `useFetch(`, `useAsyncData(`, `$fetch(`                             | Nuxt                         |
| `load(` function in `+page.server.ts` or `+page.ts`                 | SvelteKit                    |
| `loader(`, `action(`                                                | Remix                        |
| `getServerSideProps`, `getStaticProps`, `getServerAction`           | Next.js                      |
| `resolve(` in services with `Observable` return                     | Angular                      |

**For each data fetching pattern, record:**

- What data is being fetched (query key or description)
- Cache/refetch strategy: stale time, refetch on window focus, polling interval
- Optimistic updates: does the UI update before the server confirms?
- Pagination: infinite scroll, page-based, cursor-based
- Dependencies: does this query depend on another query's result?

### 3. State Management

Find how the module manages client-side state (UI state, form state, shared state).

**Search for:**

| Pattern                                                                                | Library                                                   |
| -------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `createSlice(`, `configureStore(`, `useSelector(`, `useDispatch(`, `createAsyncThunk(` | Redux Toolkit                                             |
| `createStore(`, `connect(`, `mapStateToProps`                                          | Redux (legacy)                                            |
| `create(` with `set`/`get` callbacks                                                   | Zustand                                                   |
| `makeAutoObservable(`, `observer(`, `observable(`                                      | MobX                                                      |
| `defineStore(`, `storeToRefs(`                                                         | Pinia (Vue)                                               |
| `createStore(` with `mutations`/`actions`/`getters`                                    | Vuex                                                      |
| `createAction(`, `createReducer(`, `createEffect(`                                     | NgRx                                                      |
| `createContext(`, `useContext(`, `Provider`                                            | React Context                                             |
| `writable(`, `readable(`, `derived(`                                                   | Svelte stores                                             |
| `signal(`, `computed(`, `effect(`                                                      | Signals (Angular/Solid/Preact)                            |
| `atom(`, `selector(`                                                                   | Recoil / Jotai                                            |
| `useState(`, `useReducer(`                                                             | React local state (only if managing complex shared state) |

**For each store/state, record:**

- What data it holds (the shape — field names and what they represent)
- What actions/mutations can modify it
- Which components read from it
- Whether it's local to this module or shared globally

### 4. Type Definitions and Data Models

Find interfaces, types, and schemas that reveal the business data model.

**Search for:**

- TypeScript interfaces and types: `interface`, `type`, especially those matching entity names (User, Order, Product, etc.)
- Prop types: `PropTypes.`, `Props`, interfaces ending in `Props`
- API response types: interfaces matching `*Response`, `*Result`, `*Data`
- Validation schemas: `z.object(`, `yup.object(`, `Joi.object(`
- Database/ORM models if frontend accesses them directly

**For each type/interface, record:**

- The entity name
- Key fields and what they represent (in business terms)
- Enums and their values — these often represent statuses, roles, categories
- Required vs optional fields

### 5. Data Transformations

Find functions that transform data between API responses and UI display.

**Search for:**

- `.map(`, `.filter(`, `.reduce(` on data arrays
- Formatter functions: `format`, `transform`, `normalize`, `parse`, `serialize`
- Date formatting: `formatDate`, `dayjs`, `moment`, `date-fns`, `Intl.DateTimeFormat`
- Currency/number formatting: `formatCurrency`, `Intl.NumberFormat`, `toFixed`
- Status mapping: functions that convert codes to labels (e.g., `1 → "Active"`)
- Sorting/grouping logic: `sort`, `groupBy`, `orderBy`

**For each transformation, record:**

- What goes in (source format)
- What comes out (display format)
- Why (what business purpose — e.g., "converts cents to dollar display", "groups orders by date")

### 6. Authentication and Authorization

Find how the module handles auth concerns.

**Search for:**

- Auth headers: `Authorization`, `Bearer`, `token`, `accessToken`, `refreshToken`
- Auth hooks/services: `useAuth`, `useSession`, `AuthService`, `getSession`
- Protected routes: `PrivateRoute`, `AuthGuard`, `canActivate`, redirect to login
- Role/permission checks in the data layer: `user.role`, `hasPermission`, `can(`
- Token refresh: `refreshToken`, `interceptor` that handles 401

**Record:**

- What auth method is used (JWT, session, OAuth, API key)
- What happens when auth fails (redirect, show login modal, retry with refresh)
- What roles/permissions are checked and where

### 7. External Service Integrations

Find third-party services this module communicates with beyond the primary backend API.

**Search for:**

- Payment: `stripe`, `Stripe(`, `paypal`, `braintree`, `loadStripe`, `@stripe/`
- Analytics: `analytics.track`, `gtag(`, `mixpanel`, `amplitude`, `posthog`, `segment`
- Auth providers: `Auth0`, `firebase.auth`, `supabase.auth`, `clerk`, `nextAuth`
- Storage: `S3`, `uploadFile`, `cloudinary`, `firebase.storage`
- Real-time: `WebSocket`, `socket.io`, `pusher`, `ably`, `supabase.channel`
- Maps: `google.maps`, `mapbox`, `leaflet`
- Email: `sendgrid`, `mailgun`, `resend`
- Search: `algolia`, `elasticsearch`, `meilisearch`, `typesense`
- Monitoring: `Sentry`, `datadog`, `newrelic`, `bugsnag`
- CDN: image URLs with CDN domains, `next/image` loader configs
- AI/ML: `openai`, `anthropic`, `huggingface`, `replicate`

**For each integration, record:**

- What service
- What it's used for (in business terms — "handles payment processing", "stores user-uploaded images")
- How it's initialized/configured

---

## Report Format

Return your findings using these exact section headers.

```
### API_CALLS
- [path/orderService.ts:12] GET /api/orders -> fetches list of user's orders (response: id, product, quantity, status, createdAt)
- [path/orderService.ts:28] POST /api/orders -> creates new order (sends: productId, quantity, shippingAddress, notes) (on error: shows toast)
- [path/orderService.ts:45] DELETE /api/orders/:id -> cancels an order (on 403: shows "not authorized" message)
- [path/orderService.ts:58] GET /api/products -> fetches available products for order form

### DATA_FETCHING
- [path/useOrders.ts:5] useQuery('orders') -> fetches orders, refetches on window focus, staleTime 5 minutes
- [path/useCreateOrder.ts:8] useMutation -> creates order, invalidates 'orders' cache on success, optimistic update adds to list

### STATE_MANAGEMENT
- [path/orderStore.ts:3] Zustand store 'orderStore': holds { orders[], selectedOrder, filters: { status, dateRange, search } }
  - Actions: setFilters, clearFilters, setSelectedOrder
  - Read by: OrderList, OrderFilters, OrderDetail components

### TYPE_DEFINITIONS
- [path/types.ts:1] Order: { id, productId, productName, quantity, status (enum: draft|pending|confirmed|shipped|delivered|cancelled), shippingAddress, notes, createdAt, updatedAt }
- [path/types.ts:15] OrderStatus enum: draft, pending, confirmed, shipped, delivered, cancelled
- [path/types.ts:20] CreateOrderRequest: { productId (required), quantity (required), shippingAddress (required), notes (optional) }

### DATA_TRANSFORMATIONS
- [path/utils/formatOrder.ts:5] formatOrderDate: converts ISO timestamp to "Jan 5, 2025" display format
- [path/utils/formatOrder.ts:12] formatCurrency: converts cents (integer) to "$12.50" display
- [path/utils/formatOrder.ts:20] getStatusLabel: maps status codes to display labels ("pending" -> "Awaiting Confirmation")
- [path/hooks/useGroupedOrders.ts:3] groupOrdersByDate: groups orders array by creation date for timeline display

### AUTHENTICATION
- [path/api/client.ts:8] JWT Bearer token in Authorization header for all API calls
- [path/api/client.ts:15] On 401 response: attempts token refresh, if fails redirects to /login
- [path/hooks/useAuth.ts:3] useAuth hook provides: user, isAuthenticated, login, logout, hasRole

### EXTERNAL_SERVICES
- [path/payments/stripe.ts:1] Stripe: handles payment processing for orders, initialized with publishable key from env
- [path/analytics/tracking.ts:5] Segment: tracks order events (order_created, order_cancelled, checkout_started)
- [path/uploads/s3.ts:1] AWS S3: stores product images uploaded by admins
```

If a section has no findings, write:

```
### SECTION_NAME
No findings.
```

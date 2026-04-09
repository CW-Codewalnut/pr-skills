# KT Document Output Template

Use this template when writing the final KT document. Replace all placeholders in square brackets. Follow the writing guidance in comments (lines starting with `<!--`) — do not include the comments in the output.

---

# {MODULE_NAME} — Knowledge Transfer Document

> **Generated**: {DATE}
> **Module Path**: `{MODULE_PATH}`
> **Framework**: {FRAMEWORK}

## Table of Contents

- [KT Document Output Template](#kt-document-output-template)
- [{MODULE_NAME} — Knowledge Transfer Document](#module_name--knowledge-transfer-document)
  - [Table of Contents](#table-of-contents)
  - [1. Module Overview](#1-module-overview)
  - [2. User-Facing Functionality](#2-user-facing-functionality)
  - [3. User Flows \& Journeys](#3-user-flows--journeys)
  - [4. Business Rules \& Logic](#4-business-rules--logic)
  - [5. Edge Cases \& Error Handling](#5-edge-cases--error-handling)
  - [6. API \& Service Dependencies](#6-api--service-dependencies)
  - [7. Cross-Module Interactions](#7-cross-module-interactions)
  - [8. Data Flow](#8-data-flow)
  - [9. Configuration \& Feature Flags](#9-configuration--feature-flags)
  - [10. Key Metrics \& Analytics](#10-key-metrics--analytics)
  - [11. Known Limitations \& Tech Debt](#11-known-limitations--tech-debt)
  - [12. Glossary](#12-glossary)

---

<!--
LANGUAGE RULES — apply to EVERY section below:

- Write in simple, plain English. No cognitive overload.
- Short sentences: aim for 10–20 words. One idea per sentence.
- Use common, everyday words only. If a simpler word exists, use it:
  - "use" not "utilize"
  - "start" not "initiate"
  - "show" not "render" or "display"
  - "get" not "retrieve" or "fetch"
  - "check" not "validate"
  - "send" not "transmit"
  - "save" not "persist"
  - "remove" not "eliminate"
  - "need" not "require"
  - "set up" not "configure"
- Active voice: "The system loads orders" not "Orders are loaded by the system"
- No idioms, metaphors, or cultural references (e.g., avoid "out of the box", "under the hood", "falls through the cracks")
- No technical jargon. Translate everything to business language.
- Concrete over abstract: "The user clicks the Save button" not "The user initiates the persistence action"
- A non-native English speaker should understand every sentence without a dictionary.
-->

## 1. Module Overview

<!--
SOURCE: All 3 agents (aggregate findings) + framework detection
GUIDANCE:
- 1-2 paragraphs explaining what this module IS and WHY it exists
- Write as if the reader has never seen the codebase
- Answer: What problem does this solve for users? Where does it fit in the product?
- Mention the detected framework naturally (e.g., "Built as a React application" or "Part of a Next.js application")
- NO code references, NO file paths, NO technical jargon
-->

## 2. User-Facing Functionality

<!--
SOURCE: Agent 1 (USER_FACING_TEXT, FORMS, USER_INTERACTIONS)
GUIDANCE:
- Bulleted list of capabilities from the USER's perspective
- Each bullet = one thing a user can DO
- Use active voice: "Users can...", "The system allows...", "Users are able to..."
- Be specific: "Users can filter orders by date range, status, and product category" not "Users can filter orders"
- Include all interactive capabilities: search, sort, filter, create, edit, delete, export, etc.
-->

## 3. User Flows & Journeys

<!--
SOURCE: Agent 1 (USER_FLOWS, ROUTES, NAVIGATION) + Agent 2 (API calls made during flows)
GUIDANCE:
- Numbered step-by-step walkthroughs showing how a user moves through the feature
- Cover the HAPPY PATH first, then key alternate paths
- Format: "1. User [action] → 2. System [response] → 3. User [next action] → ... → N. Outcome"
- Include entry points: how does a user GET to this feature?
- Include exit points: where does the user GO after completing the flow?
- Separate multiple distinct flows with subheadings (e.g., "### Creating an Order", "### Cancelling an Order")
- Do NOT reference component names or route paths — describe in user terms
-->

## 4. Business Rules & Logic

<!--
SOURCE: Agent 1 (CONDITIONAL_RENDERING) + Agent 2 (data validation, type enums)
GUIDANCE:
- List of rules the code enforces, written as "When X, then Y" statements
- These come from: conditional rendering, role checks, permission gates, validation rules, status-based behavior
- Group related rules together
- Examples:
  - "When a user is on the free plan, the export feature is not available"
  - "Orders can only be cancelled before they are shipped"
  - "A minimum of 1 item is required to place an order; maximum is 100"
  - "Only users with the admin role can view the management dashboard"
- Include validation rules from forms written in business terms
-->

## 5. Edge Cases & Error Handling

<!--
SOURCE: Agent 1 (LOADING_EMPTY_ERROR_STATES) + Agent 2 (API error handling, auth failure handling)
GUIDANCE:
- What happens when things go wrong or hit boundaries
- Format: "When [condition], the system [behavior]"
- Cover these categories:
  - EMPTY STATES: What does the user see when there's no data?
  - LOADING STATES: What does the user see while data is loading?
  - ERROR STATES: What does the user see when something fails?
  - VALIDATION ERRORS: What happens when the user enters invalid data?
  - NETWORK ERRORS: What happens when the API is unreachable?
  - AUTH ERRORS: What happens when the user's session expires?
  - BOUNDARY CONDITIONS: Max limits, min limits, character limits
  - RECOVERY: Can the user retry? Does the system auto-retry?
-->

## 6. API & Service Dependencies

<!--
SOURCE: Agent 2 (API_CALLS, EXTERNAL_SERVICES)
GUIDANCE:
- List every backend API endpoint this module calls
- For each endpoint:
  - What it does (in business terms): "Retrieves the user's order history"
  - What data it needs: "Requires the order ID"
  - What data it returns: "Returns order details including items, status, and shipping info"
  - When it's called: "On page load" / "When user submits the form" / "Every 30 seconds"
- ALSO list external services (separate subsection):
  - Payment processors, analytics services, auth providers, storage services, etc.
  - What each service is used for in business terms
- Do NOT include raw URL paths — describe the purpose
-->

## 7. Cross-Module Interactions

<!--
SOURCE: Agent 3 (CROSS_MODULE_IMPORTS, SHARED_STATE)
GUIDANCE:
- How this module relates to other parts of the application
- Organize as:
  - "This module USES from other modules:" — shared components, utilities, data it reads
  - "Other modules USE from this module:" — what this module provides to the rest of the app
  - "Shared data:" — state/data that flows between this module and others
- Write in product terms: "The orders module uses the shared authentication system to verify user identity" not "imports useAuth from @/hooks/useAuth"
- Note navigation connections: "Users can navigate from the dashboard to this module's order list"
-->

## 8. Data Flow

<!--
SOURCE: Agent 2 (STATE_MANAGEMENT, DATA_TRANSFORMATIONS, DATA_FETCHING) + Agent 3 (SHARED_STATE)
GUIDANCE:
- Explain how data moves through this module in PLAIN ENGLISH
- Structure as: Where data ENTERS → What HAPPENS to it → Where it's DISPLAYED or STORED
- Cover the main data paths (don't enumerate every variable)
- Example:
  "Order data enters the module when the page loads via an API request to the backend.
   The raw data is transformed to add display-friendly dates and currency formatting,
   then stored in a local cache. The order list component reads from this cache and
   displays it in a table. When the user applies filters, the cache is filtered
   client-side without a new API call. Changes (new orders, cancellations) trigger
   a cache refresh to stay in sync with the backend."
- Mention caching behavior if present
- Mention real-time updates if present (WebSockets, polling)
-->

## 9. Configuration & Feature Flags

<!--
SOURCE: Agent 3 (FEATURE_FLAGS, ENVIRONMENT_CONFIG)
GUIDANCE:
- Two subsections: Configuration and Feature Flags

For Configuration:
- What settings affect this module's behavior without code changes
- Format: "[Setting] — [what it controls]"
- Note if different environments (dev/staging/production) have different values

For Feature Flags:
- Format: "[Flag name] — [what it controls] — When enabled: [behavior]. When disabled: [behavior]"
- Include A/B tests and gradual rollouts if found
- These are valuable for POs — they show what's being tested or gradually released
-->

## 10. Key Metrics & Analytics

<!--
SOURCE: Agent 3 (ANALYTICS) + Agent 1 (tracked user interactions)
GUIDANCE:
- List every analytics event tracked in this module
- Format: "**[Event name]** — Tracked when [user action]. Captures: [what data is sent]"
- Group by user journey if possible (e.g., all checkout-related events together)
- These tell the product team what's being measured — this section is especially valuable for POs
- Note which analytics service receives the events
- If no analytics events found, state: "No analytics events are currently tracked in this module."
-->

## 11. Known Limitations & Tech Debt

<!--
SOURCE: Agent 3 (TODOS_AND_DEBT, TESTING) + all agents (concerns noted)
GUIDANCE:
- What doesn't work well, what's fragile, what's incomplete
- Include:
  - TODOs and FIXMEs from the code (paraphrased in business terms)
  - Missing test coverage (which areas are not tested)
  - Deprecated features/code that should be replaced
  - Known bugs or workarounds
  - Performance concerns (e.g., "loads all data at once, no pagination")
- Write in terms stakeholders understand: "The order list currently loads all orders at once, which may be slow for users with many orders. Pagination has not yet been implemented."
- This helps POs understand what's realistic to promise and what needs investment
-->

## 12. Glossary

<!--
SOURCE: All 3 agents (domain terms encountered)
GUIDANCE:
- Domain-specific terms used in this module
- Format: **Term** — plain English definition
- Only include terms that wouldn't be obvious to a non-technical reader
- Include:
  - Business domain terms (e.g., "SKU", "fulfillment", "churn")
  - Status/state names (e.g., "Pending — order has been placed but not yet confirmed by the system")
  - Role names (e.g., "Admin — a user with full management access")
  - Feature names that are specific to this product
- Do NOT include technical terms (API, database, component) unless they appear in user-facing text
- If no specialized terms found, state: "No specialized terminology is used in this module."
-->

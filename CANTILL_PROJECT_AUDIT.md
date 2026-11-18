# CANTILL_PROJECT_AUDIT.md — Comprehensive Project Audit & Functionality Verification

**Audit Date:** 2026-11-18  
**Project:** Cantill.ai — Enterprise Procurement Automation & Deterministic Decision Platform  
**Scope:** Phase 7A Inspection-Only Audit across Phases 1 through 6 (Architecture, Backend APIs, MongoDB Schemas, Deterministic Procurement Engine, Security/RBAC, Test Suites, and UI/UX).

---

## A. Overall Status

| Dimension | Status | Summary |
| :--- | :--- | :--- |
| **Project Build & Static Analysis** | **PASSING** | `npm run type-check` (`tsc --noEmit`) passes with **0 errors**. `npm run lint` (`eslint .`) passes with **0 errors/warnings**. Production Next.js build (`next build --webpack`) compiles all **23 routes** cleanly. |
| **Functional Readiness** | **HIGH (Core Complete, Minor Gaps in Legacy Pages)** | Core conversational PR intake (Phase 3), deterministic Policy v5.0 decision engine & PO consolidation (Phase 4A–4D), versioned policy management & PDF extraction (Phase 5), and preferred supplier intelligence (Phase 6) are fully implemented and verified (`213 / 213` total assertions passing across `phase2`–`phase6` test suites). A few secondary Phase 1 pages (`/reports`, `/settings`) still have placeholder/toast-only actions. |
| **Database & Offline Resilience** | **PARTIAL (Fallback Active; Live MongoDB Unreachable in Local Env + 2 Schema/Import Bugs)** | Local MongoDB (`localhost:27017`) is currently offline in this environment; all services transparently fall back to Task Kit CSVs and in-memory stores (`lib/db/fallbackStore.ts`). However, inspecting `models/Supplier.ts` and `services/importService.ts` revealed two concrete MongoDB schema/import bugs when importing suppliers with null performance metrics (`on_time_pct` / `quality_rating`) and reading `categories: string[]` from MongoDB. |
| **Security & RBAC Readiness** | **STRONG (With 2 Minor Hardening Items)** | Auth.js v5 JWT authentication, bcrypt password hashing, server-side `requireRole()` / `getCurrentUser()` guards, separation-of-duties on policy approval, and requester data isolation are working. Minor gaps: `middleware.ts` only checks cookie presence on a subset of routes (`/chat`, `/requisitions`, `/suppliers`, `/contracts`, `/analytics` rely on page/API-level guards rather than middleware redirect), and `/settings` / `/reports` lack server-side role checks. |
| **UI/UX Readiness** | **PARTIAL (Core Redesigned; Legacy Pages Visually Inconsistent)** | The Public Landing Page (`/`), Sign-In (`/auth/signin`), Authenticated Overview (`/`), Sidebar/Header, Buyer Decision Workspace (`/decisions`, `/purchase-orders`, `/sourcing-requests`), Policy Management (`/policies`), Analytics (`/analytics`), and Supplier Intelligence (`/suppliers`, `/suppliers/[id]`) use the new white + violet (`#6938EF`) + pastel (`#FFF2CC`, `#DDF3FF`, `#FCE3EF`, `#D9FAF5`) design system. However, older pages (`/items`, `/contracts`, `/plants`, `/import`, `/admin/users`, `/reports`, `/settings`, `/requisitions`) still use the older Phase 1/2 `rounded-md` / `text-xs` dark-slate styling, creating visual inconsistency across navigation. |

---

## B. Phase Completion Matrix

| Phase | Subphase / Module | Status | Evidence & Verification Details |
| :--- | :--- | :--- | :--- |
| **Phase 1** | Project Foundation, Next.js 16, TypeScript, Mongoose, Layout | **COMPLETE** | `package.json`, `tsconfig.json`, `lib/db/mongoose.ts`, `config/env.ts`, `components/layout/app-layout.tsx` verified. `tsc --noEmit` and `eslint .` pass with 0 errors. |
| **Phase 2** | Authentication, RBAC & Master Data CSV Imports | **PARTIAL** | Auth.js v5 (`lib/auth/`), bcrypt hashing, role guards (`lib/auth/guard.ts`), and 6 CSV import APIs (`/api/import/*`) implemented (`15/15` tests in `tests/phase2.test.ts` pass). Marked **PARTIAL** because `importSuppliers()` in `services/importService.ts` rejects rows with empty `on_time_pct` or `quality_rating` (which exist in `supplier_master.csv` for suppliers with missing metrics), and `models/Supplier.ts` marks `onTimePct` / `qualityRating` as `required: true` instead of nullable. |
| **Phase 3** | Conversational Procurement Chat & PR Management | **COMPLETE** | Deterministic rule-based NLP parser (`services/chat-intake/`), item matcher, clarification state machine, PR number generator, `/api/chat/*`, `/api/requisitions/*`, `/chat`, and `/requisitions` implemented and verified (`36/36` tests in `tests/phase3.test.ts` pass). |
| **Phase 4A** | Item Resolution, Clubbing, Valuation, Contracts & Ceilings | **COMPLETE** | 4-tier deterministic item resolver (`resolver.ts`), 14d/$\pm 7$d non-transitive clubber (`clubber.ts`), 3-tier valuation (`valuator.ts`), and chronological contract ceiling ledger + PDF Annex contracts `RC-2026-031` / `RC-2026-033` (`contractEvaluator.ts`) verified against all worked examples. |
| **Phase 4B** | Supplier Eligibility, Ranking & Procurement Decision Rules | **COMPLETE** | 548-day/contract eligibility, 5-factor weighted scoring (`35/25/15/15/10`), category top-up (`50/50`), and priority rule evaluation (`1.2` New Item, `5.1` Capex, `5.2` Contract PO, `5.3` Emergency PO, `5.4` Spot PO, `5.5` RFQ, `5.6` RFP, `5.7` Blocked/Inactive exclusion) verified across all **14 worked decisions** and **3 worked supplier rankings**. |
| **Phase 4C** | PO Consolidation, Batch Processing API, Idempotency & CSV Export | **COMPLETE** | 24.0-hour non-transitive PO consolidator (`PO-0001`..`PO-0081`, `status: "PROPOSED"`), SHA-256 idempotent `POST /api/process-batch`, RFC 4180 12-column CSV export (`/api/process-batch/[id]/export`), and `NOT_PROCESSED` data-quality exclusions (`PR-26-00172/1`, `PR-26-00173/1`) verified (`101/101` tests in `tests/phase4.test.ts` pass). |
| **Phase 4D** | Buyer Decision Workspace, Explainability & Docs | **COMPLETE** | `components/decisions/buyer-decision-workspace.tsx`, `/decisions`, `/purchase-orders`, `/sourcing-requests`, `DECISIONS.md`, and `APPROACH.md` implemented and verified. |
| **Phase 5** | Policy Management, PDF Extraction, Versioning & Comparison | **COMPLETE** | `PolicyVersion` model, Zod validation (`policyValidation.ts`), pure-TS deterministic PDF extractor (`pdfPolicyExtractor.ts`), side-by-side comparator (`policyComparator.ts`), separation-of-duties approval workflow (`policyService.ts`), scheduled auto-activation, and historical batch snapshot preservation verified (`37/37` tests in `tests/phase5.test.ts` pass). |
| **Phase 6** | Preferred Supplier Engine, Supplier Intelligence & Landing/Dashboard Redesign | **COMPLETE** | `supplierIntelligenceService.ts`, `/api/suppliers/intelligence`, `/api/suppliers/recommendations`, `/api/suppliers/[id]`, `/suppliers`, `/suppliers/[id]`, `PublicLandingPage`, and `AuthenticatedDashboard` implemented and verified (`24/24` tests in `tests/phase6.test.ts` pass). |

---

## C. Critical Issues

1. **MongoDB `Supplier` Schema & `importSuppliers()` Reject Legitimate Null Performance Metrics (`models/Supplier.ts`, `services/importService.ts`)**
   - **Evidence**: Corvo Procurement Policy v5.0 and `supplier_master.csv` include suppliers with missing `on_time_pct` or `quality_rating` (which must be preserved as `null` / `"Not Available"` and never fabricated). However:
     - [`models/Supplier.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/models/Supplier.ts#L51-L62) defines `onTimePct: { type: Number, required: true }` and `qualityRating: { type: Number, required: true }`.
     - [`services/importService.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/services/importService.ts#L204-L230) parses `row.on_time_pct?.trim() ?? "0"`—when the CSV cell is empty `""`, `parseFloat("")` returns `NaN` and the row is rejected as an import error instead of storing `null`.
2. **Type Mismatch on `Supplier.categories` When Loading Master Data from MongoDB (`services/decision-engine/utils/masterDataLoader.ts`)**
   - **Evidence**: In [`models/Supplier.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/models/Supplier.ts#L39-L43), `categories` is stored in MongoDB as `string[]` (`type: [String]`). However, in [`services/decision-engine/utils/masterDataLoader.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/services/decision-engine/utils/masterDataLoader.ts#L238), `buildContextFromCollections()` assigns `categories: supp.categories || ""` without converting `string[]` to a semicolon/comma-separated `string`. Downstream callers (`preferredSupplierEngine.ts` line 302 `supp.categories.toLowerCase()`) expect `categories` to be a `string` and will throw `TypeError: supp.categories.toLowerCase is not a function` when MongoDB is active and seeded.

---

## D. Functional Issues & Incomplete / Non-Functional Controls

1. **Unused / Legacy Dashboard Components Left Behind (`components/dashboard/`)**
   - [`components/dashboard/metric-card.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/dashboard/metric-card.tsx), [`components/dashboard/workflow-stepper.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/dashboard/workflow-stepper.tsx), [`components/dashboard/recent-activity.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/dashboard/recent-activity.tsx), and [`components/dashboard/quick-actions.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/dashboard/quick-actions.tsx) are dead Phase 1 placeholder files superseded by `authenticated-dashboard.tsx`.
2. **Unused Hardcoded `NotificationsPopover` Component (`components/layout/notifications-popover.tsx`)**
   - Contains hardcoded Phase 1 strings (`"Cantill.ai Phase 1 foundation initialized successfully"`). Currently removed from `Header`, leaving the file unused dead code.
3. **Outdated Route List in `GlobalSearchDialog` (`components/layout/global-search-dialog.tsx`)**
   - [`components/layout/global-search-dialog.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/layout/global-search-dialog.tsx#L26-L97) hardcodes `SEARCH_ITEMS` without role filtering (so a `REQUESTER` pressing `⌘K` sees Buyer/Admin links like `/purchase-orders`, `/suppliers`, `/contracts`, `/policies`, `/analytics`) and is missing `/decisions` (*Processing History*), `/sourcing-requests` (*Sourcing Requests*), `/items` (*Master Data*), `/import` (*Data Import*), and `/admin/users` (*User Management*).
4. **Placeholder-Only `/reports` and `/settings` Pages (`app/reports/page.tsx`, `app/settings/page.tsx`)**
   - [`app/reports/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/reports/page.tsx) has buttons (*"Schedule Export"*, *"Generate Report"*) that only fire a toast message (`"Scheduled digest will execute and export once transaction data exists"`) and do not export actual data.
   - [`app/settings/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/settings/page.tsx) has a *"Save Changes"* form that only shows a toast without persisting anything, though its Database Health Check tab (`/api/health?checkDb=true`) is genuinely functional.
5. **`npm test` Script in `package.json` Only Runs Phase 3 Tests**
   - [`package.json`](file:///c:/Users/USER/Desktop/cantill_fullstack/package.json#L11-L14) defines `"test"` and `"test:all"` to only run `phase2.test.ts` and `phase3.test.ts`, omitting `tests/phase4.test.ts`, `tests/phase5.test.ts`, and `tests/phase6.test.ts`.

---

## E. Data & Security Issues

1. **Inconsistent Route Protection in `middleware.ts` and Client-Only Pages**
   - [`middleware.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/middleware.ts#L27-L38) only redirects unauthenticated users on `/admin`, `/import`, `/decisions`, `/purchase-orders`, `/sourcing-requests`, and `/policies`.
   - Client-rendered pages [`app/chat/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/chat/page.tsx), [`app/requisitions/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/requisitions/page.tsx), [`app/items/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/items/page.tsx), [`app/contracts/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/contracts/page.tsx), [`app/plants/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/plants/page.tsx), [`app/analytics/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/analytics/page.tsx), [`app/reports/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/reports/page.tsx), and [`app/settings/page.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/app/settings/page.tsx) do not have a server-side `getCurrentUser()` redirect or middleware guard—unauthenticated visitors can load the empty client shell before the underlying API calls return `401 Unauthorized`.
2. **Duplicate Supplier Lookup Logic Between Legacy `services/supplierService.ts` and New `supplierIntelligenceService.ts`**
   - [`services/supplierService.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/services/supplierService.ts) is still used by `GET /api/suppliers`, while `GET /api/suppliers/intelligence` and `GET /api/suppliers/[id]` use `supplierIntelligenceService.ts`. Furthermore, `services/supplierService.ts` does not apply the in-memory admin status overrides (`getFallbackSupplierOverrides()`) when reading from MongoDB.

---

## F. UI/UX Issues (Inspection for Upcoming Redesign)

1. **Visual Split Between Redesigned Pages and Legacy Phase 1/2/3 Pages**:
   - **Redesigned Pages** (`/`, `/auth/signin`, `/decisions`, `/purchase-orders`, `/sourcing-requests`, `/policies`, `/suppliers`, `/suppliers/[id]`, `/analytics`): Use the new white + `#6938EF` violet + pastel (`#FFF2CC`, `#DDF3FF`, `#FCE3EF`, `#D9FAF5`) `rounded-2xl` card aesthetic with balanced `font-medium`/`font-semibold` typography.
   - **Un-redesigned Legacy Pages** (`/requisitions`, `/chat`, `/items`, `/contracts`, `/plants`, `/import`, `/admin/users`, `/reports`, `/settings`): Still use the older Phase 1/2 `rounded-md` / `rounded-lg` cards, tiny `text-[10px]` / `text-[11px]` labels, `font-bold` headers, and muted gray backgrounds without pastel summary cards.
2. **Dense Modals & Tables on Buyer Decision Workspace (`components/decisions/buyer-decision-workspace.tsx`) and Policy Workspace (`components/policies/policy-management-workspace.tsx`)**:
   - While their top headers and KPI strips were updated to pastel cards, the inner filter bars, table badges, and modals inside `buyer-decision-workspace.tsx` (lines 620–1570) and `policy-management-workspace.tsx` (lines 840–2170) still contain dense `text-[10px]` / `text-[11px]` cells and multi-layered borders that can be simplified for better readability.
3. **Mobile Drawer for Chat Context Panel (`components/chat/chat-workspace.tsx`)**:
   - On screens narrower than `lg` (`1024px`), `ContextPanel` (`hidden lg:block` at line 350 of `chat-workspace.tsx`) is hidden even when the user clicks the top-right `PanelRight` toggle button! On mobile/tablet widths, toggling `showContext` should open `ContextPanel` as a slide-over/overlay panel so mobile users can view and edit the Requisition Summary.

---

## G. Recommended Fix Order

1. **Priority 1 — Database & Data Integrity Fixes**:
   - Update [`models/Supplier.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/models/Supplier.ts) so `onTimePct` and `qualityRating` are optional/nullable (`type: Number, default: null, required: false`).
   - Update [`services/importService.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/services/importService.ts) (`importSuppliers`) so empty `on_time_pct` and `quality_rating` cells in `supplier_master.csv` are stored as `null` rather than failing validation.
   - Update [`services/decision-engine/utils/masterDataLoader.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/services/decision-engine/utils/masterDataLoader.ts) (`buildContextFromCollections`) to normalize `supp.categories` (`Array.isArray(supp.categories) ? supp.categories.join("; ") : String(supp.categories || "")`).
2. **Priority 2 — Security & Route Protection Hardening**:
   - Update [`middleware.ts`](file:///c:/Users/USER/Desktop/cantill_fullstack/middleware.ts) to protect all workspace routes (everything except `/`, `/auth/signin`, `/api/auth`, `/api/health`, and static assets) when unauthenticated.
   - Filter `SEARCH_ITEMS` in [`components/layout/global-search-dialog.tsx`](file:///c:/Users/USER/Desktop/cantill_fullstack/components/layout/global-search-dialog.tsx) by the logged-in user's role and add missing routes (`/decisions`, `/sourcing-requests`, `/items`, `/import`, `/admin/users`).
3. **Priority 3 — Cleanup Dead Code & Unify Supplier Service**:
   - Remove unused Phase 1 dashboard placeholder files (`components/dashboard/metric-card.tsx`, `workflow-stepper.tsx`, `recent-activity.tsx`, `quick-actions.tsx`, `components/layout/notifications-popover.tsx`).
   - Unify `services/supplierService.ts` with `supplierIntelligenceService.ts` and update `package.json` test scripts to include `phase4`, `phase5`, and `phase6`.
4. **Priority 4 — Complete Platform-Wide UI/UX Harmonization**:
   - Bring `/requisitions`, `/chat` (including mobile `ContextPanel` drawer), `/items`, `/contracts`, `/plants`, `/import`, `/admin/users`, and `/settings` into the unified Cantill.ai pastel + white + `#6938EF` design language with consistent `rounded-2xl` cards, balanced `font-medium`/`font-semibold` typography, and clean tables.

---

## H. File-Level Findings Reference

| File Path | Issue / Finding | Severity |
| :--- | :--- | :--- |
| [models/Supplier.ts](file:///c:/Users/USER/Desktop/cantill_fullstack/models/Supplier.ts#L51-L62) | `onTimePct` and `qualityRating` are marked `required: true`, preventing MongoDB persistence of `null` supplier metrics. | **High (Data Integrity)** |
| [services/importService.ts](file:///c:/Users/USER/Desktop/cantill_fullstack/services/importService.ts#L204-L230) | `importSuppliers()` fails with `NaN` error when `on_time_pct` or `quality_rating` is blank in `supplier_master.csv`. | **High (Data Integrity)** |
| [services/decision-engine/utils/masterDataLoader.ts](file:///c:/Users/USER/Desktop/cantill_fullstack/services/decision-engine/utils/masterDataLoader.ts#L238) | `buildContextFromCollections()` passes `supp.categories` (`string[]` in MongoDB) directly where a `string` is expected by `.toLowerCase()`. | **High (DB Integration)** |
| [middleware.ts](file:///c:/Users/USER/Desktop/cantill_fullstack/middleware.ts#L27-L38) | Only checks auth cookie on 6 route prefixes; `/chat`, `/requisitions`, `/suppliers`, `/contracts`, `/items`, `/analytics` are not redirected at middleware level. | **Medium (Security/UX)** |
| [components/layout/global-search-dialog.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/components/layout/global-search-dialog.tsx#L26-L97) | Command palette shows Buyer/Admin routes to Requesters and lacks `/decisions`, `/sourcing-requests`, `/items`, `/import`, `/admin/users`. | **Medium (RBAC/UX)** |
| [components/chat/chat-workspace.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/components/chat/chat-workspace.tsx#L350) | `ContextPanel` wrapper has `hidden lg:block`, making the right panel toggle non-functional on tablet/mobile screens. | **Medium (Responsive UX)** |
| [app/requisitions/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/requisitions/page.tsx), [app/items/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/items/page.tsx), [app/contracts/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/contracts/page.tsx), [app/import/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/import/page.tsx), [app/admin/users/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/admin/users/page.tsx) | Legacy Phase 2/3 styling (`rounded-md`, tiny `text-[10px]`/`text-xs` labels, no pastel summary cards); visually inconsistent with the redesigned dashboard and supplier/policy workspaces. | **Medium (UI Consistency)** |
| [app/reports/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/reports/page.tsx), [app/settings/page.tsx](file:///c:/Users/USER/Desktop/cantill_fullstack/app/settings/page.tsx) | Phase 1 placeholder buttons (*"Schedule Export"*, *"Save Changes"*) only trigger toast notifications without backend persistence. | **Low (Legacy Placeholders)** |
| [package.json](file:///c:/Users/USER/Desktop/cantill_fullstack/package.json#L11-L14) | `test:all` script runs only `phase2.test.ts` and `phase3.test.ts`, omitting `phase4.test.ts`, `phase5.test.ts`, and `phase6.test.ts`. | **Low (Tooling)** |

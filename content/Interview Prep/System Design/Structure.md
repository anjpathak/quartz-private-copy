---
{"publish":true,"created":"2026-04-03T08:22:26.141Z","modified":"2026-08-17T03:02:40.797Z"}
---

## The Agoda Staff Frontend Platform Playbook (45-Min Execution)

### Step 1 — Clarify Requirements (5 min)

- **Functional:** What are the core user flows? _(e.g., searching, filtering, booking)._

- **Non-Functional:** Define scale (DAU), device constraints, network realities, and Core Web Vitals targets.

- **Define "Winning":** Explicitly establish what success looks like for this specific system—is the primary goal SEO _(requires SSR)_, rapid conversion _(requires heavy CSR/optimistic UI)_, or Developer Experience _(requires strict monorepo tooling)_?

### Step 2 — High-Level Architecture & Delivery (5–7 min)

- **The Pipeline:** Draw the flow: `Client` $\rightarrow$ `CDN` $\rightarrow$ `Edge` $\rightarrow$ `BFF (Backend for Frontend)`.

- **Rendering Strategy:** Justify SSR, CSR, SSG, or Streaming based on the "winning" criteria established in Step 1.

- **Organizational Scaling:** If applicable, discuss micro-frontend splitting (Module Federation) to allow concurrent teams to deploy independently.

### Step 3 — Component Tree & State Model (7–10 min)

- **The UI:** Draw the high-level component tree.

- **State Categorization:** Clearly define where state lives:

  - _Server State:_ Fetched data _(e.g., hotel details)._

  - _URL State (Crucial for Travel):_ Search filters, dates, and pagination MUST live in the URL for shareability and accurate hydration.

  - _Global State:_ User auth, theme, i18n.

  - _Local State:_ UI toggles _(e.g., modal open/close)._

### Step 4 — API Contracts & BFF (7–10 min)

- **Payloads:** Define the JSON contracts between the client and the BFF.

- **BFF Role:** Explain how the BFF aggregates underlying microservices and handles rate-limiting.

- **Caching & Invalidation (The Travel Tech Nuance):** Detail your caching strategy _(e.g., Redis at the BFF layer)_ and specifically address cache invalidation. For volatile travel data like room availability and live pricing, explicitly mention using **Stale-While-Revalidate** patterns or short TTLs to prevent users from booking unavailable rooms. _(Do not draw DB schemas unless explicitly prompted)._

### Step 5 — Deep Dives & Bottlenecks (10–15 min)

- **Proactive Surfacing:** Call out the inevitable frontend bottlenecks before the interviewer does _(e.g., massive bundle sizes, expensive hydration costs, XSS risks in user reviews, CLS from ad scripts)._

- **The Platform Guardrail Lens:** When solving these bottlenecks, frame your solution as a platform offering. State clearly who owns the fix. Don't just say _"I'd lazy-load this."_ Say: _"The Platform Team (me) will build a `<LazyWidget>` wrapper with baked-in Suspense boundaries. Product engineers will use this component, and our CI/CD pipeline will enforce its usage, ensuring they get lazy-loading out of the box without having to wire it up manually."_

### Step 6 — Verify & Summarize (3 min)

- **Map Back:** Loop back to Step 1 and verify that your architecture satisfies the original requirements _(e.g., "We needed strong SEO, which is why we committed to Streaming SSR.")_

- **Future Proofing:** State 1-2 things you would optimize if you had an extra month to build this.

- **Production Monitoring:** Name one specific production metric you would monitor on Datadog/Sentry to validate the design is working in the wild _(e.g., "I would strictly monitor our p75 INP on mobile devices to ensure our third-party marketing scripts aren't degrading the booking flow.")_

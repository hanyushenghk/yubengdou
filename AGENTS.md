# Codex implementation instructions

You are building a fully functional desktop-first trilingual Hong Kong children's growth companion services demo. Do not copy third-party logos, images or proprietary text. Default zh-HK, additional zh-CN and en. HKD only.

Develop real DB-backed functionality except the bank payment gateway, which must be simulated, prominently labeled MOCK / DEMO, with no real money movement. Use Next.js, TypeScript, PostgreSQL/Drizzle, Tailwind/shadcn, next-intl, Better Auth. Email/password registration with email verification and password reset, Google and Facebook OAuth if configured. Never fake OAuth success.

Roles CUSTOMER, TUTOR, ADMIN; enforce ownership and role checks in server endpoints, not only UI. Parents own child records, tutors view assigned jobs only. Minimize children's PII, guardian consent and deletion.

End-to-end: browse -> cart -> order -> mock payment -> paid entitlement/hour card -> service requirements -> appointment -> tutor assignment -> tutor plan -> parent approval -> service execution -> tutor feedback -> parent confirmation -> hours settlement.

Monetary amounts integer HKD cents. Transactions and idempotency for payment/refund/entitlement grants. Hour ledger must track grants, holds, releases, consumption, reversals and never depend on editing a single balance. Prevent double booking under concurrency. Use Zod validation, migrations, reproducible seed with synthetic data, .env.example without secrets.

Milestones: T01 app scaffold+tooling; T02 DB schema/migrations/seeds; T03 desktop trilingual layout; T04 auth/RBAC; T05 catalog; T06 cart/orders; T07 mock payment; T08 hour cards; T09 parent/child and requirements; T10 schedule/booking; T11 tutor dispatch/plans/feedback; T12 parent dashboard; T13 admin operations; T14 notifications; T15 tests/security; T16 deployment/demo documentation.

Each task: implement real code, typecheck, lint, test, update README, create PR with tested commands; no premature merging. Start T01. Keep module interfaces stable and do not invent real payment credentials.

# Yubengdou — complete project handoff context

Last updated: 2026-10-15. This file is the cross-chat context handoff for Codex. Read with AGENTS.md and docs/DEMO_SPEC.md.

## Client and goal
The client is a Hong Kong registered company with a Hong Kong corporate bank account. Build a **desktop browser-only**, production-shaped DEMO of a Hong Kong children / youth growth planning and companion-services commerce platform. Target customers predominantly live in Hong Kong. The reference product is a WeChat mini program named 鱼蹦豆儿童规划陪伴服务平台; the user supplied seventeen screenshots. The screenshots are references for functional flows, not a license to reuse imagery, people's photos, brand identity, service claims, or copyrighted wording. Use an original working product name/placeholder brand.

## Confirmed decisions (not open questions)
- **Desktop only in phase 1**: macOS/Windows Chrome, Edge, Safari, Firefox. Use desktop layouts, header navigation, left-side categories/dashboard navigation; no mobile-first design or mini-program.
- **Three languages**: zh-HK default Traditional Chinese (Hong Kong diction), zh-CN Simplified Chinese, en English. Locale prefix URLs; translate interface, products, plans, service-provider profiles, transactional email and SEO metadata. Shared accounts, products and orders across locale views.
- **Direct website commerce**: Browse and purchase online, not merely WhatsApp inquiry / lead forms.
- **Authentication**: Google OAuth, Facebook Login, and email/password registration/login. Email verification and password reset. **No SMS OTP / phone number registration or login**. Phone number might be collected optionally for actual service fulfilment, not account creation.
- **Hong Kong local payment priority**: FPS, AlipayHK, WeChat Pay HK, and Visa/Mastercard as supplementary methods. Client has HK company/bank account. Evaluate payment gateways Airwallex or AsiaPay with actual merchant review later. **No real bank/payment gateway in this DEMO**: implement secure mock provider & clear labels; production gateway interface must be pluggable later.
- **HKD**: Store price in integer HKD cents; localize formatting.
- **Three actor-facing experiences**: public + logged-in parent/customer site, tutor/consultant workbench, admin/operations back office. Server-side RBAC for CUSTOMER, TUTOR, ADMIN.
- **All core functions except real payment gateway MUST work**, persist in database and link through a working service fulfilment lifecycle. No static pretend app.

## Reference screenshots / visible modules
- Home: promotional banners, Hong Kong-focused geography rather than Shenzhen, notice strip, service categories, featured services, service search.
- Categories: children, teens, plus family-growth services (elderly companion appears in reference; keep taxonomy extensible, but not a committed launch product).
- Service catalog: trials, learning companion sessions, whole-subject packages, youth educational planning, growth evaluations; different durations/prices/specifications; use **synthetic sample HKD prices** as fixtures pending client approval.
- Product detail: hero/media, pricing, original-price display, sales count, SKU/specification choice, long-form sections, purchase button, favorites, support/contact, customer reviews (later).
- Tutor/experts directory: categories 規劃師 / 陪伴導師 / 評估師; profile cards, qualifications, regions, availability; do not claim any real person is endorsed/verified.
- Account: orders (pending / paid / after-sales), service-hour card purchased / used / available, wallet and top-up were visible in reference (wallet is NOT phase-one real-money feature), coupons, favorites, browsing, group purchase, loyalty, shipping address, invoice, contact support, my services (pending / scheduled / in progress / completed).
- Reference service lifecycle: trial card purchase -> submit household/child needs -> customer service coordination -> agree appointment -> match tutor -> tutor drafts plan -> parent approves plan -> tutor provides session -> tutor sends feedback -> parent confirms -> settlement / repeat purchase.
- Robot tab was visible, **but no proof of functioning AI**. AI assistant optional and not blocking core DEMO.

## Important architecture notes
The mini-program screenshots do NOT expose backend, database or exact APIs; all original implementation claims would be guesses. Build an independent product, not a reverse-engineering attempt. Avoid user-facing fake testimonials or forged professional credentials. No production financial wallet or real-time transfer simulation that looks real.

## Delivery expectations
1. Runnable customer website in 3 languages with products, catalog, cart, checkout, mock payment, orders, account/children records.
2. Real reservation scheduling, tutor availability/assignments, plan approval, session report, parent confirmation, immutable hour ledger.
3. Admin CRUD for catalog/localized content, tutors, users, orders, operations and audit.
4. Seed demo accounts and synthetic example data, migration, tests, documentation, and a deployable demo environment.
5. Deliver changes incrementally as PRs with end-to-end evidence; do not silently push unfinished / untested application into main.

## Current repository state at handoff
Initially empty. README.md and AGENTS.md previously created, GitHub issue #1 describes T01-T04. This handoff expands that context. If Codex project can read this repository, Codex should treat this GitHub repo as canonical shared requirements.

## Open decisions (use documented development defaults instead of blocking)
Merchant's chosen gateway and account credentials; final service catalog, real prices, subscription/refund/cancellation policies; actual trademark and licensed imagery; region-specific tutor licensing/verification; hosting region and personal-data transfer review; actual notification sender and domain. Demo should run with safe dummy config and clearly labeled placeholders.

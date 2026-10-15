# Detailed runnable DEMO specifications — v2.1

## 1. Definition of done
A demo means **real persistence and role-based end-to-end behavior**. Only external payment authorization is mocked. Ensure admin or the assigned tutor can complete actions from their own dashboards, that parent can see all resulting states, and that it survives restart. Local email delivery may use a test inbox, and OAuth requires real provider credentials but should be wired for integration and correctly feature-gated.

## 2. Desktop routes and pages
Prefix all public/role pages with /[locale] where locale in zh-HK, zh-CN, en.
- / : brand hero, banner, home categories, recommended services, notices.
- /services : keyword, category, service-kind, age and price filtering, pagination.
- /services/[slug] : translated title/description/SEO, images, packages/SKUs, price HKD, CTA.
- /tutors : directory filters by role, district, age expertise, subjects; /tutors/[slug] profile.
- /auth/sign-in, /auth/sign-up, /auth/forgot-password, /auth/verify-email : login methods and recoveries.
- /cart, /checkout, /checkout/return, /orders/[id] : genuine cart and checkout, statuses, payment result.
- /account : user dashboard; /account/children, /account/orders, /account/hour-cards, /account/appointments, /account/favorites, /account/profile; ownership enforced.
- /appointments/[id] : needs questionnaire, requested slots, confirmed appointment, approved service plan, tutor feedback, parent acceptance.
- /tutor : upcoming sessions, assignment detail, availability calendar, submit plans, upload post-session reports.
- /admin : KPIs and searchable queues; catalog, localized content, users, tutors, availability, assignments, orders, payments, entitlements/ledger, appointments, refunds, reports, audit.
- /privacy, /terms, /refund-policy, /contact : three-language localized legal/help pages (draft wording pending counsel/client review).
For desktop use top navbar, side nav for dashboards, sidebar category on catalog, 3-4 card grids; do not simply blow up mini-program narrow view.

## 3. Minimum data model (Postgres / Drizzle)
Use UUID PK, created_at/updated_at timestamp with timezone, indexes, normalized role enum, Zod schemas, and migrations.
- users(id,email,email_verified,name,role,status,preferred_locale)
- auth_accounts/provider_id, sessions, verification_tokens (Better Auth owned tables; map with supported adapter)
- child_profiles(id,guardian_user_id,nickname,birth_year_or_age_band,school_stage,region,goals,consent_at) — avoid unnecessary sensitive data.
- categories(id,slug,parent_id,sort_order,is_active)
- category_translations(category_id,locale,name,description)
- products(id,slug,kind,status,category_id,cover_asset_id,sort_order)
- product_translations(product_id,locale,title,short_description,long_description,seo_title,seo_description)
- product_skus(id,product_id,sku,price_cents,currency=HKD,hours_minutes?,session_count?,duration_minutes?,is_active)
- media_assets(id,object_key,public_url,mime,size_bytes,alt_per_locale)
- carts/cart_items(user_id,sku_id,quantity)
- orders(id,user_id,status,currency,total_cents,created_at,paid_at,cancelled_at,version)
- order_items(id,order_id,sku_id,quantity,unit_price_cents,title_snapshot,grant_type)
- payment_attempts(id,order_id,provider=mock,method_label,status,amount_cents,external_ref,idempotency_key,processed_at)
- refund_requests(id,order_id,reason,status,amount_cents,reviewed_by,processed_at)
- entitlements(id,user_id,order_item_id,type,hours_granted_minutes?,state)
- hour_ledger(id,entitlement_id,appointment_id?,event_type,delta_minutes,reference_key,created_by,created_at) immutable; unique reference_key
- tutor_profiles(id,user_id,slug,display_name,category,service_regions,approval_state,photo_asset_id)
- tutor_translations(tutor_id,locale,bio,credentials_description,subjects)
- tutor_availability(id,tutor_id,start_at,end_at,status)
- service_requests(id,user_id,child_id,order_id,appointment_id?,requirements,preferred_regions,preferences,status)
- appointments(id,user_id,tutor_id?,service_request_id,entitlement_id,start_at,end_at,status,version)
- assignments(id,appointment_id,tutor_id,assigned_by,status)
- service_plans(id,appointment_id,version,content,status,submitted_by,approved_by,approved_at)
- session_reports(id,appointment_id,content,actual_minutes,submitted_at,parent_confirmed_at,status)
- notifications(id,user_id,type,locale,template,payload,status,send_attempts)
- audit_events(id,actor_id,entity_type,entity_id,operation,before_data,after_data,created_at)
- site_content(id,key,status), site_content_translations(content_id,locale,title,body)
Can add favorite and coupon tables for extension. Use foreign keys, enum constraints and indexes on orders(user_id,created_at), appointments(tutor_id,start_at), ledger(entitlement_id,created_at).

## 4. Key state machines
Order: DRAFT -> PENDING_PAYMENT -> PAID | PAYMENT_FAILED | CANCELLED; PAID -> PARTIAL_REFUND | REFUNDED where policy permits. All transitions guarded and audited.
Payment mock: CREATED -> SUCCESS | FAILED | CANCELLED, and SUCCESS -> REFUNDED if entitled; separate immutable attempts. Simulate payment method FPS/AlipayHK/WeChat Pay HK/card but never render official live QR code.
Appointment: REQUESTED -> PENDING_SCHEDULE -> SCHEDULED -> ASSIGNED -> PLAN_SUBMITTED -> PLAN_APPROVED -> IN_PROGRESS -> REPORT_SUBMITTED -> COMPLETED. Support CANCELLED/RESCHEDULE_REQUIRED based on policy. No skipping authorization.
Hour card: purchase grant positive; booking places hold; cancellation releases hold; successful completion consumes actual minutes and releases any unused hold; refunds post negative adjustments. Reservation hold and actual consumption never double-deduct. Every movement atomic and idempotent.
Tutor scheduling: use transaction + conflict lock or DB exclusion constraint and unique logical reservations; race test concurrent bookings of same timeslot.
Refund may reverse unused entitlement; refuse refund if committed hours exceed refundable balance; require explicit admin approval and audit.

## 5. REST API contract (/api/v1)
Consistent response {data,meta?} or {error:{code,message,details?}}; HTTP status and Zod validation. Paginate lists; idempotency-key on payments and reservation mutations.
GET /catalog/categories; GET /catalog/products; GET /catalog/products/:slug; GET /tutors; GET /tutors/:slug;
GET/POST/DELETE /cart/items; GET /cart;
POST /orders; GET /orders; GET /orders/:id;
POST /payments/mock/attempt; POST /payments/mock/:id/succeed; POST /payments/mock/:id/fail; POST /payments/mock/:id/refund;
GET/POST/PATCH /me/children; GET /me/entitlements; GET /me/hour-ledger; GET /me/appointments;
POST /service-requests; POST /appointments; PATCH /appointments/:id/reschedule; POST /appointments/:id/cancel;
GET /tutor/assignments; POST /tutor/availability; POST /appointments/:id/plan; POST /appointments/:id/report;
POST /appointments/:id/plan/approve; POST /appointments/:id/report/confirm;
GET/POST/PATCH /admin/products, /admin/tutors, /admin/assignments, /admin/content;
POST /admin/refunds/:id/approve; GET /admin/orders; GET /admin/audit.
Exact request and response schemas live in /src/lib/contracts with Zod, generated API documentation optional. Critical mutation APIs check both session and ownership/role.

## 6. Example payment & service acceptance journey
Demo parent chooses zh-HK, signs in by email, buys trial learning-companion package (fixture HK$299), mock FPS payment success. Order PAID, grant entitlement once. Create child and service request, pick proposed slot. Admin assigns synthetic tutor; tutor posts plan; parent approves; tutor marks session and submits report; parent confirms; hour ledger consumes session time once; parent dashboard shows updated available/held/used hours. Switch to English: same order data with localized labels. Re-run payment success: no extra entitlement. Tutor B is denied access to tutor A appointment; parent B denied access to parent's order.

## 7. Notifications
Abstract notifier with email and in-app adapters. Templates in zh-HK / zh-CN / en. Events order-paid, appointment-created/confirmed, tutor-assigned, plan-ready, report-ready, refund-status. Local development can log email to Mailpit/test inbox; secrets stay server-side. Never claim message sent unless adapter accepted it.

## 8. Security and privacy
CSRF/cookie protections per Better Auth; credential validation and rate limiting on auth; social OAuth state verification; email verification; secure password hashing; PII encryption as needed; RBAC and row ownership; no exposing tutoring child notes publicly. Explicit guardian consent for minor's data and consent log; allow admin assisted data export/deletion with retention policy. Audit admin actions and financial/hour ledger transitions. Avoid unrestricted image uploads; enforce MIME and size; object storage signed upload paths if enabled.

## 9. Testing
Unit: price totals, locale fallback, Zod, state transitions, hour balance, auth guards.
Integration: DB migrations, seed idempotency, mock payment idempotency, refund checks, tutor ownership, appointment collisions under concurrent requests.
E2E Playwright: three-language shopping; email signup; social buttons/redirect configuration; checkout mock successful/failed; parent booking; tutor plan; admin assignment; feedback; cancel/rebook; authorization denial; English SEO metadata.
CI: lint, tsc --noEmit, tests, migration health, Playwright smoke. Document failures.

## 10. Milestones
T01 scaffold; T02 schema+seed; T03 trilingual desktop components; T04 auth/RBAC; T05 products+tutors; T06 cart/orders; T07 mock payments; T08 hour cards; T09 parent child/service request; T10 booking availability; T11 allocation + plans + feedback; T12 dashboards; T13 admin CRUD; T14 notifications; T15 tests/security; T16 reproducible demo deployment.
No live payment checkout, wallet cash top-up or bank account secrets in demo. No false professional certifications, fabricated reviews or real child personal data.

## 11. Environment variables (names only)
DATABASE_URL, BETTER_AUTH_SECRET, BETTER_AUTH_URL, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, FACEBOOK_CLIENT_ID, FACEBOOK_CLIENT_SECRET, PAYMENT_PROVIDER=mock, DEMO_MODE=true, EMAIL_PROVIDER=dev, EMAIL_FROM, R2_ACCOUNT_ID, R2_BUCKET, R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY, NEXT_PUBLIC_SITE_URL. Never commit real values.

## 12. Codex invocation
Read AGENTS.md, docs/PROJECT_CONTEXT.md and this specification. Inspect repo before touching anything. Start from T01–T04 and open a draft PR with working, tested code. Implement every actual business feature later using the milestones above. Ask questions only when an uncertain decision would make irreversible real-world financial/legal commitments; otherwise choose safe documented defaults.

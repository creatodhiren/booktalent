# BookTalent — Product Requirements Document

## Original Problem Statement
Build a production-ready BookTalent talent marketplace (Indian artist booking platform) from scratch with the premium dark luxury theme from the supplied HTML references. Must include real backend workflows — auth (with email verification), artist profiles + media, packages, full booking workflow, Razorpay payments, wallet & withdrawals, reviews, PDF contracts/invoices, KYC, admin panel.

## Architecture
- **Backend**: FastAPI + MongoDB + JWT (`bcrypt`, `pyjwt`), Razorpay SDK (`razorpay==1.4.2`), Resend email (`resend==2.32.2`), PDF generation (`reportlab==4.2.0`)
- **Frontend**: React 19 + react-router-dom v7
- **Styling**: Custom dark-luxury CSS (gold + purple, Cormorant Garamond + Inter)
- **Media storage**: base64 in MongoDB (local, per user preference — no Cloudinary)
- **Payments**: Razorpay live integration with safe mock fallback (when keys empty)
- **Email verification**: Resend with safe mock fallback (test OTP `123456` when keys empty)

## User Personas
1. Customer / Event Planner — books artists, manages bookings, downloads PDFs, leaves reviews
2. Artist — manages profile/media/packages/availability, accepts bookings, withdraws earnings
3. Agency / Corporate — bulk-booking workflows
4. Admin — KYC moderation, payout release, coupons, disputes, all contracts visibility

## Implementation log

### Iteration 1 (2026-06-24) — MVP
- All MVP modules: auth, artist discovery + filters, profile, 5-step booking, customer/artist/admin dashboards
- Wallet, reviews, KYC, coupons, boost, disputes, notifications, messaging
- 6 seeded demo artists + 1 customer + 2 coupons. 28/28 backend tests passing.

### Iteration 2 — Razorpay + PDFs
- Full Razorpay integration (`/api/payments/init`, `/verify` with signature, `/webhook` HMAC, `/refund`)
- PDF generation (contracts + GST invoices) via ReportLab — `application/pdf` streams
- Frontend opens real Razorpay modal when keys present; mock OTP fallback otherwise
- Admin can list all contracts. 35/35 backend tests passing.

### Iteration 3 — Email Verification (current)
- New **Resend-based email verification** replacing SMS for signup
  - `POST /api/auth/email/send` (60-sec cooldown, returns `test_otp` in mock mode)
  - `POST /api/auth/email/verify` (10-min expiry)
  - `POST /api/auth/register` now requires a verified email_otp record (consumed on register)
  - `GET /api/auth/config` exposes provider status to the frontend
- Premium dark-luxury HTML email template (inline CSS + table layout)
- 4-step signup UI: Role → Details → Verify Email → Finish
- Media storage **kept local** (base64 in MongoDB) per user preference
- 43/43 backend tests passing, frontend signup E2E verified

## Switching to live integrations
Add the relevant keys to `/app/backend/.env`, then `sudo supervisorctl restart backend`:
```env
# Email
RESEND_API_KEY=re_xxxxxxxxxxxx
SENDER_EMAIL="BookTalent <noreply@yourdomain.com>"

# Payments
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=xxxxxxxxxxxxxx
RAZORPAY_WEBHOOK_SECRET=xxxxxxxxxxxxxx
```
Both integrations automatically activate when keys are filled — no code change.

## Currently MOCKED (will go live the moment keys are added)
- **Email** — `RESEND_API_KEY` empty → OTP `123456` returned in API + shown in UI hint
- **Razorpay** — `RAZORPAY_KEY_ID` empty → mock OTP `123456` accepted to confirm bookings
- **Phone SMS OTP** endpoints still exist (`/api/auth/otp/*`) but are no longer used in the signup flow — can be deprecated

## Decisions per user
- ✅ Email verification (not SMS) for signup
- ✅ Local storage (base64 in MongoDB) — no Cloudinary/S3 migration
- ✅ Razorpay keys dummy/empty — integration wired and ready

## Backlog (P1)
- WebSocket real-time chat (currently REST polling)
- Digital signature pad on contracts
- AI semantic search over artist bios
- Agency dashboard
- Hash OTPs in DB (security hardening)
- Throttle wrong-OTP attempts (currently unlimited within 10-min window)

## P2
- Charts on analytics (recharts)
- Booking confirmation email on accept (function exists, hook into accept handler)
- Referral program
- 2FA
- i18n (Hindi + English)

## Test Credentials — see /app/memory/test_credentials.md
- Admin: `admin@booktalent.com / Admin@123`
- Customer: `customer@booktalent.com / Customer@123`
- Artist: `priya@booktalent.com / Artist@123`
- Mock email OTP: `123456`
- Mock payment OTP: `123456`

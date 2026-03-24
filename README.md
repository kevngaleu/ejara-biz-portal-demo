# Ejara FX — Business Web Portal

**Product:** Ejara FX Business
**Version:** V1 (MVP)
**Status:** Draft
**Date:** March 24, 2026
**Positioning:** Self-service cross-border payment portal for formal businesses — faster than banks, safer than dealers, with the compliance controls your finance team requires. No hidden fees — one transparent exchange rate.

**👉 [Live prototype →](https://kevngaleu.github.io/ejara-biz-portal-demo/)**

---

## 1. Problem Statement

The current Ejara FX product is agent-assisted: businesses send a representative to a physical kiosk where a trained agent executes the transfer. This works for cash-first market traders, but **formal businesses** (established companies, 5–10M XAF/week) have different needs:

- They cannot interrupt operations to visit a kiosk for every supplier payment
- They need a digital audit trail and compliance receipts for their accountants
- They need internal controls — more than one person should be involved in authorizing a payment
- They have multiple employees who need access at different permission levels

**Competitors already serving this segment:** Luo Pay, Capi Money, Afriex Business. Ejara has no digital self-service offering for businesses today.

**The opportunity:** Ejara's existing KYB infrastructure (Dotfile), CPN integration, and Jara Cash brand trust give us a path to launch a competitive product quickly — without rebuilding from scratch.

---

## 2. Target Users

**Primary persona — Finance Manager / CFO (Formal Business)**
- Runs a company with regular Nigeria supplier payments (5–10M XAF/week)
- Currently uses UBA or informal dealers. Frustrated by bank delays (2+ days, high failure rate) and dealer risk (no recourse)
- Needs: same-day to 48hr delivery, digital receipts, internal payment approval controls
- Values reliability and documentation over cost savings

**Secondary persona — Finance Team Member (Initiator)**
- Prepares and submits payment requests on behalf of their company
- Does not have authority to approve; needs a workflow that separates creation from authorization
- Values: saved beneficiaries (no re-entering supplier details), clear status tracking

**Business account Admin**
- IT or senior finance person who sets up the company account, manages team access
- Needs: simple role assignment, ability to disable a departed employee instantly

---

## 3. The One Metric

V1 succeeds if **50% of onboarded businesses complete a second transfer within 30 days.**

This measures trust and workflow fit. If they return, the product has replaced their existing solution. If not, we have a friction problem in the flow.

---

## 4. Key Scenarios

**Scenario 1 — Regular supplier payment**
> Aminata is the finance manager at an electronics import company in Douala. Every Friday she pays three Nigerian suppliers. She logs into Ejara FX Business, selects "Chukwudi Electronics — GTBank" from saved beneficiaries, types "5,000,000 NGN" in the receive field — the XAF amount (2,016,129 XAF) fills in automatically. She gets a live quote (rate locked for 60s), accepts it. Her colleague Théodore, the CFO, gets a notification on his phone. He reviews, approves. Payment submitted. Monday morning, Chukwudi confirms receipt. Aminata downloads the PDF receipt for accounting.

**Scenario 2 — New company onboarding (existing Ejara customer)**
> Martin runs a textile import business. His company already completed KYB with Ejara's Dotfile integration for another product. He registers on the web portal, enters his company email. The system recognizes the existing Dotfile case — his account is activated immediately. He invites his assistant as Initiator, sets himself as Approver. First payment is sent the same day.

**Scenario 3 — New company onboarding (first time)**
> Alice's trading company is new to Ejara. She registers, gets redirected to the Dotfile onboarding portal, uploads company registration documents and her ID. The Ejara compliance team reviews the case in Dotfile's dashboard and approves within 24 hours. Alice receives an email, logs in, and sends her first payment.

**Scenario 4 — Approver rejects a suspicious payment**
> An initiator submits a payment to an unfamiliar account. The CFO approver doesn't recognize the beneficiary, adds a comment "Wrong account number — check with supplier," and rejects. The initiator is notified, corrects the account, resubmits. The CFO approves. No money moved until both parties confirmed.

---

## 5. Core User Flow

### Onboarding

**Path A — Existing Dotfile-verified customer:**
```
Register (company name + admin email)
  → Backend looks up Dotfile case by email/company ID
  → Case found + APPROVED
  → Account activated immediately
  → USDC wallet created (backend only)
  → Admin invited to set password and invite team
```

**Path B — New business:**
```
Register (company name + admin email)
  → No Dotfile case found
  → Redirected to Dotfile embedded KYB portal
      (company registration doc, admin ID + selfie, proof of address, sector)
  → Dotfile runs AML/sanctions checks
  → Ejara compliance team reviews in Dotfile dashboard
  → Dotfile webhook fires: APPROVED or REJECTED
  → APPROVED: account activated, USDC wallet created, email sent
  → REJECTED: email with reason, business can resubmit
```

### Payment Initiation (Maker-Checker)

```
[Initiator]
1. Dashboard → "New Transfer"
2. Enter amount (XAF) + select destination country (all CPN-supported corridors)
3. "Get Quote" → backend checks USDC reserves:
     Sufficient → proceed with 60-second quote
     Insufficient → extend estimated delivery to 20min (Ejara funds in background)
4. Amount entry — two modes:
     Enter XAF to send → destination currency amount calculated in real time
     Enter destination currency amount to receive → XAF to send calculated in real time
5. "Get Quote" → quote displayed: amount to send (XAF), exchange rate, amount to receive (destination currency), 60-second timer
6. Accept quote → beneficiary selection:
     Choose saved beneficiary (dropdown) OR enter new details (name, bank, account)
     Option to save new beneficiary for future use (optional — not required to proceed)
     Optional: upload invoice
6. Submit for approval → status: PENDING_APPROVAL

[Approver — notified by email + SMS]
7. Review: amount, rate, beneficiary, total cost, initiator name, invoice (if uploaded)
8. Approve:
     Approver sees the rate locked at initiation — this rate is honored regardless of when the Approver acts
     Ejara absorbs any FX movement between initiation and approval
     Backend funds USDC wallet with exact amount at locked rate
     USDC wallet debited → payment submitted to CPN
     Status: SUBMITTED
     Email to Initiator (confirmation + expected delivery date)
     Email + SMS to Approver (confirmation of their action)
   Reject → add optional comment → Initiator notified by email:
     Initiator can edit and resubmit (new quote required — new rate locked at that point)
     Nothing was funded, no cost incurred
```

**Critical design principle:** Nothing is funded until the Approver explicitly confirms. Rejections are free and instant.

---

## 6. Feature Scope

### Must Have

**Onboarding & KYB**
- Company registration form (name, country, admin email)
- Dotfile API lookup on registration: detect existing approved case → auto-activate
- Dotfile embedded KYB portal redirect for new businesses
- Dotfile webhook handler: activate/reject account on compliance decision
- Email notifications: account approved, account rejected (with reason)

**User & Team Management**
- Admin invites team members by email with assigned role (Initiator / Approver)
- Email invitation with set-password link
- Admin can change roles and disable/remove users
- One user may hold multiple roles (e.g., Admin + Approver)
- Separation: Initiator cannot approve their own payment

**Payment Initiation**
- Amount entry + destination country selection (all CPN-supported corridors: Nigeria, China, Dubai, and others)
- Live quote with 60-second countdown timer
- Quote displays: amount to send (XAF), exchange rate, amount to receive (destination currency)
- **Bidirectional amount entry:** user can enter either the XAF amount to send OR the destination currency amount the beneficiary should receive — the other field updates in real time
- Accept or reject quote before proceeding
- Beneficiary selection: saved beneficiaries list or manual entry
- Option to save new beneficiary at payment time (optional — not required to proceed)
- Invoice file upload (optional)
- Submit for approval → enters PENDING_APPROVAL queue

**Approval Flow (Maker-Checker)**
- Approver notification: email + SMS with payment summary
- Approvals queue on dashboard
- Full payment detail view before approving: amount, rate, beneficiary, initiator, invoice
- Approve action → fund + debit USDC → submit to CPN
- Rate shown to Approver is the rate locked by the Initiator at submission — no re-confirmation needed
- Reject with optional comment → Initiator notified by email, can edit and resubmit

**Settings — Approval Configuration (Admin only)**
- Toggle to enable/disable the Maker-Checker requirement entirely
- Approvers list: Admin selects which team members act as Approvers
- Approval threshold: payments below this XAF amount are sent automatically without approval; set to 0 to require approval on all payments

**Beneficiary Management**
- Saved beneficiaries per business account (name, bank, account number, country, nickname)
- Create, edit, delete beneficiaries
- Select saved beneficiary during payment initiation
- Beneficiary visible in transaction history and PDF receipt

**Transaction History & Receipts**
- Filterable transaction list: date range, status, beneficiary, initiator
- Status badges: PENDING_APPROVAL, SUBMITTED, COMPLETED, FAILED, REJECTED
- Transaction detail page: full audit trail including who initiated, who approved, timestamps
- Downloadable PDF receipt on completion (exchange rate, fees, beneficiary, reference)

**Notifications (non-configurable in V1, always on)**
- Payment submitted for approval → email + SMS to Approvers
- Payment approved or completed → email to Initiator
- Rate alert (when admin improves rate on a corridor) → email to all active businesses

**Backend — USDC Wallet**
- One USDC wallet per business, created at KYB approval (address stored, no balance)
- Wallet funded at Approver confirmation with exact payment amount
- Wallet immediately debited to submit payment to CPN
- Atomic operation: fund + debit must both succeed or fail (ACID)
- If USDC reserves insufficient at quote time: auto-extend delivery time, Ejara funds in background

**Admin Panel (Ejara Internal — extends existing)**
- Business account list with KYB status
- Manual KYB override (approve/reject) for edge cases not handled by Dotfile webhook
- Transaction monitoring across all business accounts
- **Exchange rate management:** set and update retail rate per corridor (XAF → NGN); changes apply to all new quotes immediately; in-flight 60-second quotes are honored at their locked rate
- **Rate change audit log:** who changed, old rate, new rate, timestamp
- Alert: USDC reserves < $500

### Out of Scope V1
- Visible wallet balance or top-up UI for businesses
- Bulk payment CSV upload
- API access for business ERP integrations
- Scheduled/recurring payments
- Instant delivery (<1hr)
- Mobile app for business users

---

## 7. User Roles & Permissions

| Action | Admin | Initiator | Approver |
|---|:---:|:---:|:---:|
| Invite / remove users | ✓ | | |
| Assign roles | ✓ | | |
| View all transactions | ✓ | | ✓ |
| View own transactions | ✓ | ✓ | ✓ |
| Initiate payment | | ✓ | |
| Approve / reject payment | | | ✓ |
| Manage saved beneficiaries | ✓ | ✓ | |
| Download receipts | ✓ | ✓ | ✓ |
| Update company settings | ✓ | | |

> One user may hold multiple roles. Initiator cannot approve their own submission.

---

## 8. KYB & Compliance

**KYB provider:** Dotfile (dotfile.com)
- Ejara does not handle documents in-house
- Dotfile's embeddable portal collects documents, runs AML/sanctions checks
- Ejara compliance team reviews and approves/rejects cases inside Dotfile's dashboard

**Business tier:**
- Documents required: official ID, proof of residence, company registration, sector declaration, AML checks
- Monthly transfer limit: up to 555,000,000 XAF (~$1M USD)

**Existing customer fast path:**
- On registration, backend queries Dotfile API by business email / company identifier
- If an approved case exists: account activated without re-KYB
- `dotfile_case_id` stored on Business record for audit trail

**Regulatory note:** Ejara operates as OFI with CPN. CPN's local BFI partners handle in-country licensing for each destination corridor (Nigeria, China, Dubai, etc.). Ejara does not need separate licenses per destination — CPN's network covers it.

---

## 9. Monetization

Ejara monetizes through the **exchange rate spread** — not through explicit transaction fees. Ejara buys liquidity from providers at a wholesale rate and sells to businesses at a retail rate. The spread is Ejara's margin.

**What the business sees:**
- One exchange rate (e.g., 1 XAF = 0.00248 NGN)
- Amount sent (XAF) and amount received (NGN) derived from that rate
- No separate fee line items — the rate is the all-in price

**How it works internally:**

| | Example (500K XAF) |
|---|---|
| Provider/wholesale rate | 1 XAF = 0.00250 NGN → 1,250 NGN received |
| Ejara retail rate offered | 1 XAF = 0.00248 NGN → 1,240 NGN received |
| Spread captured by Ejara | Difference between wholesale and retail rates |
| Business receives | Exactly the NGN shown at quote — no surprise deductions |

**Rate management — Ejara Admin:**
- The Ejara admin sets and updates the retail exchange rate via the admin panel
- Rate changes take effect on all new quotes immediately (in-flight quotes at 60-second lock are honored)
- Admin can set rates per corridor (one rate entry per CPN-supported destination: Nigeria, China, Dubai, etc.)
- Audit log of all rate changes (who changed, old rate, new rate, timestamp)

**V2 opportunity:** Preferential rates for businesses committing to monthly volume minimums (e.g., >10M XAF/month gets a tighter spread). Retention and upsell lever.

---

## 10. Growth Mechanics (Product-Led Growth)

Every core action in the product should pull someone new in or bring an existing user back. The product grows because using it inherently involves other people.

---

### Loop 1 — The Beneficiary Receipt (Nigeria-side acquisition) `V1`

**Mechanic:** Every completed payment sends a receipt notification to the Nigerian beneficiary (supplier). That supplier receives money and sees Ejara FX's name for the first time — with zero marketing spend.

**Trigger:** Payment status → COMPLETED
**Channel:** SMS + email to beneficiary
**Message:** *"You received 5,000,000 NGN from [Company Name] via Ejara FX. Does your business also send XAF payments? Sign up at ejara.io/business."*

**Why it works:** The supplier already trusts Ejara — money arrived. This is the highest-trust moment for acquisition. Nigerian suppliers paying Cameroonian buyers are a natural expansion audience.

---

### Loop 2 — The Savings Card (shareability after payment) `V1`

**Mechanic:** Immediately after a payment completes, show the Initiator a shareable summary card: how much they sent, the rate locked, and the equivalent bank cost comparison.

**Trigger:** Payment status → COMPLETED
**Shown in:** Web portal (dismissable overlay) + included in the completion email
**Message example:**
> *"Payment delivered. You sent 2,016,129 XAF → 5,000,000 NGN at 2.48. A typical bank transfer would have taken 2+ days and cost you more. Share this with your network."*

**Share targets:** WhatsApp (primary in Cameroon), LinkedIn
**Why it works:** Finance managers in the same industry network talk to each other. The card makes the ROI visible and gives them something concrete to share. No new infrastructure needed — rides on the completion event.

---

### Loop 3 — The Approver as Champion `V1`

**Mechanic:** The Approver's first experience is a carefully crafted approval request notification. They didn't sign up — the product found them. If the email is excellent (clear, fast to act on, trustworthy), they become internal champions and push adoption at other companies they advise or work with.

**Trigger:** Payment enters PENDING_APPROVAL
**Channel:** Email + SMS
**Email design principles:**
- Full payment detail above the fold — no login required to see what they're approving
- Single CTA: Approve or Reject (deep link directly to the action)
- Ejara branding is prominent but not loud — trust, not marketing

**Drop-off recovery:** If Approver hasn't acted in 1hr → reminder email. In 24hrs → escalation SMS.

---

### Loop 4 — Invite Your Colleague (team viral loop) `V1`

**Mechanic:** Right after account activation (not during onboarding — timing matters), prompt the Admin: *"Who on your team handles supplier payments? Invite them now."* Strike when motivation is highest.

**Trigger:** Business account status → APPROVED (first login after activation)
**Shown in:** Onboarding checklist on the dashboard
**Nudge sequence:**
1. Account activated → prompt to invite team
2. 48hrs later, if no team members invited → email: *"You're the only one on your account. Add a colleague to split payment duties."*

---

### Loop 5 — Rate Alert (re-engagement) `V1`

**Mechanic:** When the XAF/NGN rate moves favorably (admin updates rate to a better spread), send a re-engagement email to all active business accounts that haven't sent a payment in the last 7 days.

**Trigger:** Admin updates exchange rate to a more competitive level for a given corridor
**Channel:** Email (targeted to businesses that have previously paid to that corridor)
**Message example:** *"The XAF/NGN rate just improved — lock it now for your next payment."*

**Why it works:** Creates urgency without being spammy. Directly tied to a real product event, not a generic marketing push.

---

### Loop 6 — Monthly Statement (passive word-of-mouth) `V2`

**Mechanic:** A beautiful monthly summary email sent to Admin + Approver on the 1st of each month: total sent, number of payments, average rate, time saved vs. bank estimate.

**Why it works:** CFOs and finance managers forward these internally and to their accountants. The product proves its own ROI every month. Makes churning psychologically harder — they can see the history.

---

### Loop 7 — Invite Your Supplier `V2`

**Mechanic:** After a payment completes, prompt the Initiator: *"Does [Chukwudi Electronics] also send payments to Cameroon? Invite them to Ejara FX Business."*

**Why it works:** B2B trade networks are dense — the same supplier often has multiple buyers paying them across corridors. One successful payment creates a warm referral opportunity into the supplier's own business.

---

### Drop-off Recovery Emails (automated) `V1`

| Drop-off point | Delay | Message |
|---|---|---|
| Registered, KYB not started | 24hrs | "Complete your verification — here's what you'll need (2 documents)" |
| KYB started, not submitted | 48hrs | "Pick up where you left off — your application is saved" |
| KYB pending, no user action | 3 days | "Your account is under review — typically approved within 24hrs" |
| Quote requested, payment not submitted | 2hrs | "You checked the rate — ready to send? Rates update in real time" |
| Payment submitted, Approver hasn't acted | 1hr | Reminder to Approver: "A payment is waiting for your approval" |
| Payment submitted, Approver hasn't acted | 24hrs | SMS escalation to Approver |
| First payment done, no second in 14 days | 14 days | "Your suppliers are ready when you are — send your next payment" |

---

## 11. Competitive Positioning

| Feature | Luo Pay | Capi Money | Afriex Biz | **Ejara FX Web** |
|---|---|---|---|---|
| Quote-first flow | 15-min | Instant | Unknown | **60-sec live rate** |
| Fee transparency | 0 (hidden in FX spread) | Not disclosed | Not disclosed | **Spread-based — one clean rate, no line-item fees** |
| Settlement | 24hrs | 24–48hrs | Fast | **1–3 days V1** |
| Maker-Checker | Unknown | Unknown | Unknown | **Yes — default in V1** |
| Corridors | Multi | 30+ | 50+ | **XAF → all CPN corridors (Nigeria, China, Dubai, others)** |
| KYB speed | Fast | Same-day | Minutes | **Instant (existing) / 24hr (new)** |
| Local presence | No | Yes (Douala) | No | **Yes — Jara agents as backup** |
| Saved beneficiaries | Unknown | Yes | Unknown | **Yes** |

**Differentiation story:** "Ejara FX Business is the only platform built specifically for XAF importers — with a live rate you lock before committing, a built-in approval workflow for your finance team, and a local team in Douala if anything goes wrong."

---

## 12. Technical Architecture

### Data Model

```
Business
  id, name, country, registration_number, sector
  kyb_status: PENDING | APPROVED | REJECTED
  dotfile_case_id
  monthly_limit_xaf
  usdc_wallet_address  (backend only, no UI)
  created_at

BusinessUser
  id, business_id, email, name, phone
  roles: [ADMIN, INITIATOR, APPROVER]  (array — user can hold multiple)
  status: INVITED | ACTIVE | DISABLED
  created_at

BusinessBeneficiary
  id, business_id
  nickname  (e.g. "Lagos Supplier – GTBank")
  name, bank, account_number, country
  created_by (user_id), created_at

BusinessTransaction
  id  (BTXR-XXXXXX)
  business_id
  initiated_by (user_id), approved_by ([user_id])  (array — supports dual-approver in V2)
  required_approvals  (default: 1 in V1; extendable per business or amount tier)
  beneficiary_id (nullable — if saved), beneficiary_name, beneficiary_bank, beneficiary_account
  destination_country, destination_currency  (e.g. NGN, CNY, AED)
  amount_xaf, amount_destination, exchange_rate
  wholesale_rate  (internal only — for margin tracking)
  spread_captured  (internal only — Ejara's margin on this transaction, in destination currency)
  invoice_url
  quote_id, quote_expires_at
  status: DRAFT | PENDING_APPROVAL | SUBMITTED | COMPLETED | FAILED | REJECTED
  rejection_comment
  created_at, approved_at, completed_at
```

### API Endpoints

```
# Onboarding & KYB
POST   /api/business/register
POST   /api/business/kyb/webhook          (Dotfile webhook receiver)
GET    /api/business/:id/status

# Auth & Users
POST   /api/business/auth/login
POST   /api/business/auth/accept-invite
POST   /api/business/users/invite
PATCH  /api/business/users/:id
DELETE /api/business/users/:id

# Payments
POST   /api/business/quotes
POST   /api/business/transactions
GET    /api/business/transactions
GET    /api/business/transactions/:id
POST   /api/business/transactions/:id/approve
POST   /api/business/transactions/:id/reject
GET    /api/business/transactions/:id/receipt

# Beneficiaries
GET    /api/business/beneficiaries
POST   /api/business/beneficiaries
PATCH  /api/business/beneficiaries/:id
DELETE /api/business/beneficiaries/:id
```

### Tech Stack
- **Frontend:** React web app (separate from the agent kiosk app)
- **Auth:** Email + password with JWT (no PIN — web, not kiosk)
- **Backend:** Extends existing Node.js/Express service
- **KYB:** Dotfile REST API + webhooks
- **Payments:** Circle Payment Network (CPN) — same as agent product
- **Notifications:** Email (primary) + Africa's Talking SMS (secondary)

### Key Pages
1. **Onboarding:** Register → KYB status → Activated
2. **Dashboard:** Pending approvals badge, recent transactions, quick stats
3. **New Transfer:** Amount → Quote (60s timer) → Beneficiary → Submit
4. **Approvals Queue:** List of PENDING_APPROVAL payments with approve/reject
5. **Transaction History:** Filterable list, status badges, PDF download
6. **Beneficiaries:** Saved suppliers list with add/edit/delete
7. **Team Management:** User list with roles, invite, disable
8. **Settings:** Company profile, Maker-Checker configuration (toggle, approvers, threshold), notification preferences

---

## 13. Regulatory & Compliance

- **Jurisdictions:** Cameroon (XAF) as source currency; destination countries determined by CPN-supported corridors (Nigeria, China, Dubai, and others)
- **Ejara operates as:** OFI with CPN. CPN's local BFI partners handle in-country licensing per destination corridor
- **KYB:** Dotfile holds and processes all business documents (Ejara does not store raw KYB docs)
- **AML monitoring:** Configured in Dotfile; ongoing screening via Dotfile's monitoring module
- **Transaction monitoring alerts (admin panel):**
  - USDC reserves < $500 → urgent notification
  - Business approaches monthly limit (>80% used) → compliance alert
  - Success rate < 95% → alert

---

## 14. Key Risks

**Risk 1: Dotfile existing-customer lookup fails**
Not all existing customers may be indexed the same way in Dotfile (email vs. company ID mismatch).
Mitigation: Fallback to manual verification by Ejara admin; allow compliance team to manually link a Dotfile case to a portal account.

**Risk 2: FX movement between initiation and approval**
The rate is locked at Initiator submission and honored at Approver confirmation. Ejara absorbs any FX movement in between.
Mitigation: Monitor spread exposure on pending approvals. If a payment sits in PENDING_APPROVAL for an unusually long time (e.g., >24hrs), the drop-off recovery email to the Approver reduces this window. Ejara's spread should be sized to absorb normal intraday FX movement.

**Risk 3: Businesses find Maker-Checker too slow for urgent payments**
A two-person approval adds latency.
Mitigation: Approver gets push notification immediately. If adoption data shows this is a blocker, add an "urgent" flag that sends an SMS directly to the Approver's phone. Consider single-user approval for businesses below a monthly volume threshold in V2.

**Risk 4: Low KYB completion rate (drop-off at Dotfile)**
Businesses may abandon the KYB portal if document requirements feel heavy.
Mitigation: Show upfront what's needed before starting. Dotfile's resume functionality lets them complete in multiple sessions.

---

## 15. Open Questions

**Q1: Dotfile lookup identifier**
What identifier will be used to find existing Dotfile cases on registration — business email, tax ID, or company registration number? Needs alignment with the compliance team and Dotfile API capabilities.

**Q2: Quote risk policy — RESOLVED**
The rate locked at Initiator submission is honored at Approver confirmation. Ejara absorbs FX movement between the two steps. No action needed from the Approver on rate changes.

**Q3: Dual-approver threshold — CONFIRMED FOR V2**
For large payments above a threshold (TBD with compliance), two Approvers will be required. Not built in V1 but the data model must accommodate it: `approved_by` should be an array of user IDs, and `BusinessTransaction` should include a `required_approvals` count field set per business or per amount tier.

**Q4: USDC reserve minimum**
Current agent product alert threshold is $500. For business payments that can go up to 555M XAF (~$1M), this threshold may need to be higher. Align with Finance/Operations.

**Q5: Failed payment refund handling**
If CPN fails to deliver and funds are returned, how does the business get refunded? Credit to a future payment or bank transfer back? Align with Finance.

---

## 16. Definition of Done — V1

V1 complete when:
- 5+ businesses onboarded (at least 2 via existing Dotfile fast path, 3 via new KYB)
- 50+ successful transactions across business accounts
- Maker-Checker flow tested end-to-end (submit → approve → CPN submit)
- Dotfile webhook integration live and tested (approve + reject paths)
- Saved beneficiaries working across multiple payments
- PDF receipt downloadable with correct data
- Admin panel shows business transactions and KYB status
- Notifications firing correctly: email + SMS to Approver on submission; email to Initiator on approval/completion

---

## 17. Success Criteria

| Metric | Target |
|---|---|
| Repeat rate | 50% (2nd transfer within 30 days) |
| KYB completion rate | >70% of started applications approved |
| Approval turnaround | Approver acts within 30 min on >80% of submissions |
| Payment success rate | >95% |
| Time to first payment (new biz) | <48hrs from registration |
| Time to first payment (existing biz) | <1hr from registration |

---

## Next Steps

1. Design — wireframes for the 7 key pages (Figma)
2. Dotfile API review — confirm lookup endpoint and webhook events with Dotfile team
3. Backend — extend existing schema with Business, BusinessUser, BusinessBeneficiary, BusinessTransaction entities
4. Frontend — new React web app (separate repo or monorepo module)
5. Compliance — align on Q1–Q5 open questions before build starts
6. Pilot launch — 5 businesses from existing Ejara merchant network

---

## Prototype

The `index.html` file in this repository is a self-contained clickable prototype covering all 7 key pages. No build step required — open it directly in any browser or visit the [live GitHub Pages link](https://kevngaleu.github.io/ejara-biz-portal-demo/).

**How to use the prototype:**
- Log in as **Aminata Diallo (Initiator)** or **Théodore Ngassa (Approver)**
- Use the role switcher bar at the bottom to switch between roles mid-session
- The full Maker-Checker flow: log in as Aminata → New Transfer → complete the 4 steps → switch to Théodore → Approvals → approve or reject
- Language toggle (EN/FR) in the top-right corner

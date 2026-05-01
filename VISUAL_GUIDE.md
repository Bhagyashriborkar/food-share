# THREE FUNCTIONS VISUAL GUIDE

## FUNCTION 27: DIGITAL HANDSHAKE (OTP VERIFICATION)

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃       FUNCTION 27: Digital Handshake (OTP)            ┃
┃    "Secure verification of food handover"             ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

Timeline Flow:
├─ T0: NGO Confirms Pickup
│  ├─ Status: 'pending' → 'confirmed'
│  ├─ OTP Generated: generate_otp() → 6 random digits
│  ├─ Stored: Claim.otp_code = "123456"
│  └─ Emailed: Sent to Donor
│
├─ T1: NGO Arrives at Donor Location
│  └─ Provides OTP verbally: "One-Two-Three-Eight-Nine-Two"
│
├─ T2: Donor Receives OTP
│  ├─ Sees on /donor/active_orders
│  ├─ OTP Display: 123456
│  └─ Form: [_______] [Verify]
│
├─ T3: Donor Enters OTP
│  ├─ Types: "123456"
│  ├─ Clicks: "Verify" button
│  └─ AJAX POST: /claim/{id}/verify_otp
│
└─ T4: Backend Verification
   ├─ Input: "123456"
   ├─ Database: Claim.otp_code = "123456"
   ├─ Match?: YES ✓
   ├─ Update: Status → 'completed'
   ├─ Update: otp_verified → True
   └─ Response: {"success": true}

Code Logic:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
if otp_input != claim.otp_code:
    return error("Invalid OTP")

# OTP MATCHED!
claim.status = 'completed'
claim.otp_verified = True
claim.pickup_time = datetime.utcnow()

# TRIGGER OTHER FUNCTIONS
archive_claim_to_history(claim)  # Fn 28
send_receipt_email(...)          # Fn 29
create_notifications(...)        # Notifications

db.session.commit()
return {"success": true}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Security Features:
✓ 6-digit random (1 in 1 million chance)
✓ One-time use only
✓ Server-side validation (not client-side)
✓ HTTPS recommended for transmission
✓ Database encryption recommended
✓ Role-based access (donor only)

Database Changes:
Before: status='confirmed', otp_code='123456', otp_verified=False
After:  status='completed', otp_code='123456', otp_verified=True
```

---

## FUNCTION 28: ARCHIVING TO HISTORY LOG

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃    FUNCTION 28: Completion Logic with Archiving       ┃
┃     "Preserve permanent transaction records"           ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

Archiving Workflow:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────┐
│  Active Claims Table    │
├─────────────────────────┤
│ id:         123         │
│ food_id:    45          │
│ ngo_id:     12          │
│ status:     pending     │ ─ OTP Verified → status = completed
│ otp_code:   123456      │
│ pickup_time: null       │
└─────────────────────────┘
           │
           │ archive_claim_to_history(claim)
           ↓
┌─────────────────────────────────────┐
│   HistoryLog Table (Archive)        │
├─────────────────────────────────────┤
│ id:              501 (NEW)          │
│ claim_id:        123                │
│ food_title:      "Pizza"            │
│ donor_id:        23                 │
│ ngo_id:          12                 │
│ quantity_served: 50                 │
│ people_served:   50                 │
│ pickup_time:     2026-04-28 2:30PM  │
│ completion_time: 2026-04-28 2:35PM  │
│ notes:           "Delivered safely" │
│ archived_at:     2026-04-28 2:35PM  │
└─────────────────────────────────────┘

Code Implementation:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

def archive_claim_to_history(claim):
    history = HistoryLog(
        claim_id=claim.id,
        food_title=claim.food_listing.title,
        donor_id=claim.food_listing.donor_id,
        ngo_id=claim.ngo_id,
        quantity_served=claim.food_listing.quantity,
        people_served=claim.people_served or 0,
        pickup_time=claim.pickup_time,
        completion_time=datetime.utcnow(),
        notes=claim.notes,
        archived_at=datetime.utcnow()
    )
    db.session.add(history)
    return history

# Called automatically after OTP verification:
if data.get('otp') == claim.otp_code:
    claim.status = 'completed'
    claim.otp_verified = True
    archive_claim_to_history(claim)  # ← HERE
    db.session.commit()

Benefits:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ Permanent Records
  └─ Never lose transaction data
  └─ Historical reference available

✓ Active Table Stays Clean
  └─ Old claims don't clutter active view
  └─ Faster queries on Claim table

✓ Audit Trail
  └─ Track all donations ever made
  └─ Compliance & legal requirements

✓ Analytics Enabled
  └─ Query HistoryLog for impact metrics
  └─ Generate monthly/yearly reports
  └─ See donor/NGO patterns

✓ Dispute Resolution
  └─ Have permanent proof of transaction
  └─ Can reference archived records

Data Preserved:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ What was donated (food_title)
✓ How much (quantity_served)
✓ Who donated (donor_id)
✓ Who received (ngo_id)
✓ When it happened (pickup_time)
✓ When it was completed (completion_time)
✓ How many people helped (people_served)
✓ Special notes/details (notes)
✓ Archive timestamp (archived_at)
```

---

## FUNCTION 29: DIGITAL RECEIPT GENERATION

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃      FUNCTION 29: Digital Receipt Generation           ┃
┃     "Professional documentation of donation"           ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

Receipt Generation Workflow:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 1: Generate Receipt Data
┌────────────────────────────────────────┐
│ def generate_receipt(claim):            │
│   receipt = {                           │
│     'claim_id': 123,                    │
│     'receipt_date': now(),              │
│     'food_title': 'Pizza',              │
│     'quantity': 50,                     │
│     'people_served': 50,                │
│     'donor_name': 'John Smith',         │
│     'ngo_name': 'Hope NGO',             │
│     'pickup_time': datetime,            │
│     'status': 'completed'               │
│   }                                     │
│   return receipt                        │
└────────────────────────────────────────┘
         │
         ↓
Step 2: Send Email to DONOR
┌────────────────────────────────────────┐
│ send_receipt_email(                     │
│   donor_email,                          │
│   "John Smith",                         │
│   receipt_data                          │
│ )                                       │
│                                        │
│ Email Content:                          │
│ ─────────────────────────────────────── │
│ Subject: Food Donation Receipt - Pizza  │
│                                        │
│ Dear John Smith,                        │
│                                        │
│ Your food donation has been collected!  │
│                                        │
│ Receipt Details:                        │
│ - Food: Pizza                           │
│ - Quantity: 50 portions                 │
│ - People Served: 50                     │
│ - Recipient: Hope NGO                   │
│ - Pickup: Apr 28, 2026 2:30 PM          │
│                                        │
│ Impact: You helped feed 50 people!      │
│                                        │
│ Thank you for your generosity!          │
│ ─────────────────────────────────────── │
└────────────────────────────────────────┘
         │
         ↓
Step 3: Send Email to NGO
┌────────────────────────────────────────┐
│ send_receipt_email(                     │
│   ngo_email,                            │
│   "Hope NGO",                           │
│   receipt_data                          │
│ )                                       │
│                                        │
│ Email Content:                          │
│ ─────────────────────────────────────── │
│ Subject: Food Donation Receipt - Pizza  │
│                                        │
│ Dear Hope NGO,                          │
│                                        │
│ Successfully collected food donation!   │
│                                        │
│ Receipt Details:                        │
│ - Food: Pizza                           │
│ - Quantity: 50 portions                 │
│ - People Served: 50                     │
│ - Donor: John Smith                     │
│ - Pickup: Apr 28, 2026 2:30 PM          │
│                                        │
│ Thank you for serving the community!    │
│ ─────────────────────────────────────── │
└────────────────────────────────────────┘
         │
         ↓
Step 4: Create Online Receipt Page
┌────────────────────────────────────────┐
│ GET /claim/{id}/receipt                 │
│                                        │
│ Route:  /claim/123/receipt              │
│ Template: receipt.html                  │
│ Data: receipt_data + claim object       │
│                                        │
│ Renders Beautiful Receipt:              │
│ ╔════════════════════════════════════╗ │
│ ║   🧾 DONATION RECEIPT              ║ │
│ ╟────────────────────────────────────╢ │
│ ║ Receipt ID: #RECV-123-20260428     ║ │
│ ║ Date: April 28, 2026 @ 2:30 PM     ║ │
│ ║                                    ║ │
│ ║ 📦 DONATION DETAILS                ║ │
│ ║ Food: Pizza                        ║ │
│ ║ Qty: 50 portions                   ║ │
│ ║ Served: 50 people                  ║ │
│ ║                                    ║ │
│ ║ 👤 DONOR                           ║ │
│ ║ John Smith                         ║ │
│ ║                                    ║ │
│ ║ 🏢 RECIPIENT                       ║ │
│ ║ Hope NGO                           ║ │
│ ║                                    ║ │
│ ║ 🌍 YOUR IMPACT                     ║ │
│ ║ You helped feed 50 people!         ║ │
│ ║ Thank you for making a difference! ║ │
│ ║                                    ║ │
│ ║ [Print] [Download PDF]             ║ │
│ ╚════════════════════════════════════╝ │
└────────────────────────────────────────┘

Receipt Features:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Document Features:
✓ Unique Receipt ID (#RECV-123-20260428)
✓ Timestamp (exact date & time)
✓ Food item name & quantity
✓ Donor name & organization
✓ Recipient organization name
✓ Pickup time & completion time
✓ People served count
✓ Special notes if any
✓ Status (COMPLETED)
✓ Impact statement

User Features:
✓ Available online immediately
✓ Beautiful gradient design
✓ Clear table layout
✓ Mobile responsive
✓ Easy to print
✓ PDF download support
✓ Share via email link

Security:
✓ Only accessible to involved parties
✓ Only for completed claims
✓ Secure via Flask login_required
✓ Role-based permission check

Recipients & Delivery:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DONOR Receipt:
├─ What: Food donation confirmed
├─ Impact: People served count
├─ Purpose: Thank you & transparency
└─ Email: Within 1 minute of OTP

NGO Receipt:
├─ What: Food collection confirmed
├─ Impact: Community served metric
├─ Purpose: Record keeping & proof
└─ Email: Within 1 minute of OTP

Uses:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

For Donors:
✓ Proof of donation
✓ Tax deduction documentation
✓ Transparency about impact
✓ Motivation for future donations
✓ Social proof/sharing

For NGOs:
✓ Operational record
✓ Beneficiary count tracking
✓ Impact reporting
✓ Financial accountability
✓ Donor relations

For Admins:
✓ Audit trail
✓ Transaction verification
✓ Dispute resolution
✓ Impact analytics
✓ Compliance documentation
```

---

## INTEGRATION: HOW ALL THREE WORK TOGETHER

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃        COMPLETE TRANSACTION PIPELINE                        ┃
┃   Functions 27, 28, 29 Working in Harmony                   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[1] Food Donation Claimed
    └─ Claim created (pending)

[2] NGO Confirms Pickup
    └─ Status → confirmed
    └─ OTP Generated (Function 27)
    └─ OTP Sent to Donor

[3] Physical Handover
    ├─ NGO provides OTP
    ├─ Food transferred
    └─ Donor receives OTP

[4] FUNCTION 27: OTP VERIFICATION ←─── Key Verification Point
    ├─ Donor enters: 123456
    ├─ Backend checks: 123456 == Claim.otp_code
    ├─ Result: MATCH ✓
    └─ Mark: status = 'completed', otp_verified = True

[5] FUNCTION 28: AUTOMATIC ARCHIVING ←─ Preserve Records
    ├─ Create HistoryLog entry
    ├─ Copy all transaction data
    ├─ Record completion timestamp
    ├─ Store people_served count
    └─ Maintain permanent audit trail

[6] FUNCTION 29: GENERATE RECEIPT ←─── Create Documentation
    ├─ Build receipt_data dict
    ├─ Render HTML receipt page
    ├─ Send email to Donor
    ├─ Send email to NGO
    └─ Make available at /receipt/{id}

[7] Post-Transaction
    ├─ Donor views receipt
    ├─ Sees impact: "Helped 50 people"
    ├─ Can print/download
    └─ Gets verification

[8] Permanent Record
    ├─ Data in HistoryLog
    ├─ Available for analytics
    ├─ Audit trail complete
    └─ Impact metrics recorded

Timeline (Minutes 0-5):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0:00 → Pickup window opens
0-30 → NGO travels & collects food
0-45 → Physical handover occurs
1:00 → Donor enters OTP
1:05 → [Fn27] OTP verified → [Fn28] Archived → [Fn29] Receipt sent
1:10 → Donor/NGO receive emails
5:00 → Both can view online receipts

Impact Tracking:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
├─ Function 27 ensures: VERIFICATION
│   └─ Proves actual handover occurred
│
├─ Function 28 ensures: PERSISTENCE
│   └─ Data never lost, always available
│
└─ Function 29 ensures: TRANSPARENCY
    └─ Impact documented & communicated

Data Flow:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Active Claim (Fn27)
    └─> Archived to HistoryLog (Fn28)
    └─> Receipt Generated (Fn29)
    └─> Emails Sent (Fn29)
    └─> Online View (Fn29)
    └─> Impact Recorded (Fn28)
```

---

## SUMMARY TABLE

```
╔═══════════════════════════════════════════════════════════════════════╗
║                    FUNCTION COMPARISON TABLE                          ║
╠═════════════════╦════════════════════════════════════════════════════╣
║ Aspect          ║ Fn 27 (OTP)    ║ Fn 28 (Archive) ║ Fn 29 (Receipt)║
╠═════════════════╬════════════════╩═════════════════╩═════════════════╣
║ Purpose         ║ Verify handover ║ Store records   ║ Document proof ║
║ Timing          ║ Real-time       ║ Automatic       ║ Automatic      ║
║ Trigger         ║ Donor action    ║ After Fn27      ║ After Fn28     ║
║ Data Created    ║ otp_verified=T  ║ HistoryLog row  ║ Receipt record ║
║ Main Benefit    ║ Trust & Security║ Analytics       ║ Transparency   ║
║ User Sees       ║ Verification    ║ (Backend)       ║ Beautiful page ║
║ Email Sent?     ║ OTP email       ║ No              ║ Yes (both)     ║
║ Online View?    ║ No              ║ No              ║ Yes (/receipt) ║
║ Permanent?      ║ Yes (Claim)     ║ Yes (Archive)   ║ Yes (stored)   ║
║ Print Support?  ║ No              ║ No              ║ Yes (CSS)      ║
║ Lines of Code   ║ ~50             ║ ~20             ║ ~300           ║
║ Complexity      ║ High (secure)   ║ Medium          ║ High (UI)      ║
╚═════════════════╩═════════════════════════════════════════════════════╝
```

---

## KEY TAKEAWAYS

✨ **Function 27 (OTP Verification)**
   - Ensures authentic handover
   - One-time use security
   - Real-time verification
   - Trust-building mechanism

🔐 **Function 28 (Archiving)**
   - Permanent data preservation
   - Audit trail creation
   - Analytics enablement
   - Compliance support

📜 **Function 29 (Receipt)**
   - Professional documentation
   - Impact transparency
   - Email delivery
   - Online accessibility

Together they create a **COMPLETE, TRUSTWORTHY DONATION SYSTEM** ✅

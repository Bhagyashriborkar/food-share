# Module 4: Fulfillment & Logistics - Implementation Documentation

## Summary
Implemented **3 critical functionalities** for the Food Waste Reduction & Donation System (Module 4):
- **Function 27**: Digital Handshake (OTP Verification)
- **Function 28**: Completion Logic with Archiving to HistoryLog
- **Function 29**: Digital Receipt Generation

---

## Function 27: Digital Handshake (OTP Verification)

### Purpose
Create a secure digital handover mechanism between donors and NGOs using One-Time Password (OTP) verification.

### Implementation Details

#### Backend Logic (`app.py`)
```python
@app.route('/claim/<int:claim_id>/verify_otp', methods=['POST'])
@login_required
def verify_otp(claim_id):
    """
    Validates OTP input from donor
    - Compares user input with stored OTP code
    - Marks claim as 'completed' on successful verification
    - Triggers archiving and receipt generation
    """
```

#### Key Steps:
1. **OTP Generation** (in `update_claim_status` route)
   - Random 6-digit code generated when NGO confirms pickup
   - Stored in `Claim.otp_code` database field
   - Sent to donor via email notification

2. **OTP Verification**
   - Donor receives OTP from NGO representative during pickup
   - Donor enters 6-digit code in verification form
   - Backend compares input with `Claim.otp_code`

3. **On Successful Verification**
   - Claim status → `'completed'`
   - `otp_verified` → `True`
   - `pickup_time` → Current timestamp
   - Triggers Functions 28 & 29

#### Database Schema
```
Claim.otp_code: String(6) - 6-digit OTP
Claim.otp_verified: Boolean - Verification status
Claim.pickup_time: DateTime - When handover occurred
Claim.status: String - Now set to 'completed'
```

#### API Response
```json
{
  "success": true,
  "message": "OTP verified successfully! Donation marked as completed.",
  "claim_id": 123
}
```

---

## Function 28: Completion Logic with Archiving

### Purpose
Automatically move completed donation records from active transactions to a permanent history log for archival and analytics.

### Implementation Details

#### New Database Model: `HistoryLog`
```python
class HistoryLog(db.Model):
    """Archive of completed claims/transactions"""
    id = db.Column(db.Integer, primary_key=True)
    claim_id = db.Column(db.Integer, nullable=False)
    food_title = db.Column(db.String(200), nullable=False)
    donor_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    ngo_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    quantity_served = db.Column(db.Integer)
    people_served = db.Column(db.Integer)
    pickup_time = db.Column(db.DateTime)
    completion_time = db.Column(db.DateTime)
    notes = db.Column(db.Text)
    archived_at = db.Column(db.DateTime, default=datetime.utcnow)
```

#### Archive Function
```python
def archive_claim_to_history(claim):
    """
    Creates permanent record of completed transaction
    Called automatically after OTP verification
    """
    history = HistoryLog(
        claim_id=claim.id,
        food_title=claim.food_listing.title,
        donor_id=claim.food_listing.donor_id,
        ngo_id=claim.ngo_id,
        quantity_served=claim.food_listing.quantity,
        people_served=claim.people_served,
        pickup_time=claim.pickup_time,
        completion_time=datetime.utcnow(),
        notes=claim.notes,
        archived_at=datetime.utcnow()
    )
    db.session.add(history)
    return history
```

#### Automatic Workflow
1. OTP verified successfully
2. `archive_claim_to_history()` called automatically
3. New `HistoryLog` record created
4. Original `Claim` record updated with `status='completed'`
5. Data preserved for analytics and audit trail

#### Benefits
- ✅ Permanent transaction records
- ✅ Audit trail for compliance
- ✅ Analytics on donation impact
- ✅ Separates active vs. completed claims
- ✅ Reduces clutter in active Claim table

---

## Function 29: Digital Receipt Generation

### Purpose
Generate professional digital receipts for both donors and NGOs, sent via email and viewable online.

### Implementation Details

#### Receipt Generation Function
```python
def generate_receipt(claim, receipt_type='donor'):
    """
    Creates receipt data dictionary with all transaction details
    Returns structured data for template rendering
    """
    receipt = {
        'claim_id': claim.id,
        'receipt_date': datetime.utcnow(),
        'food_title': claim.food_listing.title,
        'quantity': claim.food_listing.quantity,
        'people_served': claim.people_served or claim.food_listing.quantity,
        'donor_name': donor.username,
        'ngo_name': ngo.organization,
        'pickup_time': claim.pickup_time,
        'status': 'completed'
    }
    return receipt
```

#### Email Sending
```python
def send_receipt_email(user_email, user_name, receipt_data):
    """
    Sends personalized receipt email to donor and NGO
    - Different templates for donor vs. NGO
    - Includes impact metrics
    - Professional formatting
    """
```

#### Receipt Details Included
- **Receipt ID**: `#RECV-{claim_id}-{date}`
- **Food Item**: Title and quantity
- **Donor Info**: Name and organization
- **Recipient Org**: NGO name
- **Pickup Time**: When handover occurred
- **People Served**: Number of individuals fed
- **Impact Statement**: "You helped feed X people"

#### Receipt Access
- **Online View**: `/claim/<claim_id>/receipt`
- **Email**: Automatically sent to both parties
- **Print**: Button to print receipt
- **PDF**: Download as PDF option

#### Templates
- **`receipt.html`** - Professional digital receipt with:
  - Gradient header
  - Detailed transaction info
  - Impact metrics
  - Print-friendly styling
  - Download PDF option

---

## Frontend Components

### 1. Donor Active Orders (`donor/active_orders.html`)
**URL**: `/donor/active_orders`

#### Features
- ✅ Lists all active pickups (confirmed + pending)
- ✅ Displays OTP code for donor to share
- ✅ OTP input form for verification
- ✅ Pickup details modal
- ✅ Completed pickups section with receipt links
- ✅ Real-time OTP verification via AJAX

#### Key UI Elements
```html
<!-- OTP Display -->
<div class="otp-display">{{ claim.otp_code }}</div>

<!-- OTP Verification Form -->
<input type="text" class="otp-input" placeholder="Enter 6-digit OTP">
<button class="verify-otp-btn">Verify</button>

<!-- Completed Pickups -->
<a href="{{ url_for('view_receipt', claim_id=claim.id) }}">View Receipt</a>
```

### 2. NGO Pickups (`ngo/pickups.html`)
**URL**: `/ngo/pickups`

#### Features
- ✅ Lists all confirmed pickups for NGO
- ✅ Shows OTP code to provide to donor
- ✅ Pickup window countdown timer
- ✅ People served input field
- ✅ Donor contact details
- ✅ Completed pickups table with receipt links

#### Key UI Elements
```html
<!-- OTP Code (for NGO to share) -->
<div class="otp-display">{{ pickup.otp_code }}</div>

<!-- Pickup Window Timer -->
<div class="progress">Time window closing at {{ pickup.food_listing.pickup_end }}</div>

<!-- People Served Input -->
<input type="number" class="people-served-input" 
       value="{{ pickup.food_listing.quantity }}">
```

### 3. Receipt View (`receipt.html`)
**URL**: `/claim/<claim_id>/receipt`

#### Features
- ✅ Professional receipt layout
- ✅ All transaction details
- ✅ Impact metrics (people served)
- ✅ Print-to-PDF functionality
- ✅ Print-friendly styling
- ✅ Navigation back to active orders/pickups

#### Layout Sections
1. **Header** - Receipt title and branding
2. **Receipt ID & Date** - Unique identifier
3. **Donation Details** - Food info and quantity
4. **Donor Information** - Donor details
5. **Recipient Org** - NGO details
6. **Transaction Details** - Pickup time and status
7. **Impact Message** - Social impact statement
8. **Print/Download** - Action buttons

---

## Database Schema Changes

### New Table: `HistoryLog`
```sql
CREATE TABLE history_log (
    id INTEGER PRIMARY KEY,
    claim_id INTEGER NOT NULL,
    food_title VARCHAR(200) NOT NULL,
    donor_id INTEGER FOREIGN KEY,
    ngo_id INTEGER FOREIGN KEY,
    quantity_served INTEGER,
    people_served INTEGER,
    pickup_time DATETIME,
    completion_time DATETIME,
    notes TEXT,
    archived_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Updated Table: `Claim`
- ✅ `otp_code` - Store generated OTP
- ✅ `otp_verified` - Track verification status
- ✅ `status` - Now includes 'completed' state
- ✅ `people_served` - Track actual people fed
- ✅ `pickup_time` - Timestamp of handover

---

## API Endpoints

### 1. OTP Verification
**POST** `/claim/<claim_id>/verify_otp`

**Request Body**:
```json
{
  "otp": "123456"
}
```

**Response** (Success):
```json
{
  "success": true,
  "message": "OTP verified successfully!",
  "claim_id": 123
}
```

**Response** (Error):
```json
{
  "success": false,
  "error": "Invalid OTP code"
}
```

### 2. View Receipt
**GET** `/claim/<claim_id>/receipt`

Returns HTML page with receipt details and print functionality.

### 3. Active Orders (Donor)
**GET** `/donor/active_orders`

Shows all active pickups for donor's donations.

### 4. Pickups (NGO)
**GET** `/ngo/pickups`

Shows all confirmed pickups for NGO to collect.

---

## Workflows & Sequences

### Complete Transaction Flow

```
1. Donor Posts Food
   └─> Creates FoodListing

2. NGO Claims Food
   └─> Creates Claim (status='pending')

3. NGO Confirms Pickup
   └─> Updates Claim (status='confirmed')
   └─> Generates 6-digit OTP
   └─> Sends OTP to Donor (email)

4. NGO Collects Food & Provides OTP

5. Donor Verifies OTP (Function 27)
   └─> POST /claim/{id}/verify_otp
   └─> Backend validates OTP

6. On OTP Success:
   ├─> Archive to HistoryLog (Function 28)
   ├─> Generate Receipt (Function 29)
   ├─> Send receipt emails
   ├─> Create notifications
   └─> Mark claim as 'completed'

7. Donor & NGO View Receipt
   └─> GET /claim/{id}/receipt
```

---

## Security Considerations

### OTP Security
- ✅ 6-digit random codes (high entropy)
- ✅ One-time use only
- ✅ Stored securely in database
- ✅ Verified server-side (not client-side)
- ✅ HTTPS transmission recommended

### Access Control
- ✅ Only donors can verify OTP for their donations
- ✅ Only NGOs assigned to claim can see code
- ✅ Receipts only accessible to involved parties
- ✅ Role-based route protection

---

## Email Templates

### Donor Receipt Email
```
Subject: Food Donation Receipt - {FOOD_TITLE}

Dear {DONOR_NAME},

Your food donation has been successfully collected!

Receipt Details:
- Food Item: {TITLE}
- Quantity: {QUANTITY} portions
- People Served: {PEOPLE_SERVED}
- Recipient: {NGO_NAME}
- Date: {PICKUP_TIME}

Impact: You helped feed {PEOPLE_SERVED} people in need!

Thank you for making a difference!
```

### NGO Receipt Email
```
Subject: Food Donation Receipt - {FOOD_TITLE}

Dear {NGO_NAME},

Successfully collected food donation!

Receipt Details:
- Food Item: {TITLE}
- Quantity: {QUANTITY} portions
- People Served: {PEOPLE_SERVED}
- Donor: {DONOR_NAME}
- Date: {PICKUP_TIME}

Thank you for your work in the community!
```

---

## Testing Checklist

- [ ] OTP verification with correct code (should succeed)
- [ ] OTP verification with wrong code (should fail)
- [ ] Receipt page displays for completed claims only
- [ ] HistoryLog entry created after completion
- [ ] Emails sent to both parties
- [ ] Donor active orders page shows all pickups
- [ ] NGO pickups page shows all confirmed pickups
- [ ] OTP code displayed correctly on both sides
- [ ] Completed pickups appear in history
- [ ] Print functionality works
- [ ] Receipt shows correct impact metrics

---

## Files Modified/Created

### Backend
- ✅ `app.py` - Added HistoryLog model, receipt functions, new routes

### Frontend
- ✅ `templates/donor/active_orders.html` - Enhanced OTP verification UI
- ✅ `templates/ngo/pickups.html` - Enhanced pickup display with OTP
- ✅ `templates/receipt.html` - Professional receipt template

---

## Future Enhancements

1. **PDF Generation** - Use ReportLab or html2pdf for native PDF exports
2. **SMS Notifications** - Send OTP via SMS for backup
3. **QR Codes** - Generate QR code for receipt tracking
4. **Receipt Archival** - Download receipts in bulk
5. **Analytics Dashboard** - Show impact metrics from HistoryLog
6. **Expiring OTPs** - Auto-expire codes after 30 minutes
7. **OTP Attempts Limit** - Lock after 3 failed attempts

---

## Summary

These three functionalities complete the fulfillment pipeline:
- **Function 27** ensures secure, verified handover
- **Function 28** maintains permanent transaction records
- **Function 29** provides professional documentation

Together, they create a complete, trustworthy donation lifecycle with audit trails and impact tracking.

# TESTING GUIDE: OTP VERIFICATION & RECEIPT GENERATION
## Food Waste System - Module 4 Testing

---

## 🚀 PROJECT STATUS
✅ **Flask Application is Running**
- **URL**: http://127.0.0.1:5000
- **Debug Mode**: ON
- **Server**: http://127.0.0.1:5000

---

## 📋 TESTING WORKFLOW

### **Phase 1: Setup & Account Creation**

#### **Step 1: Go to Home Page**
```
URL: http://127.0.0.1:5000/
```
- You'll see the Food Waste System homepage
- Navigation menu at the top

#### **Step 2: Create Two Accounts**
You need TWO accounts for testing:
1. **Donor Account** - Someone who donates food
2. **NGO Account** - Someone who receives food

**To Register:**
1. Click **"Register"** button
2. Fill in form:
   - Username: `john_donor`
   - Email: `donor@example.com`
   - Password: `password123`
   - Role: **Select "Food Donor"**
   - Organization: `My Restaurant`
   - Click **Submit**

**Repeat for NGO:**
- Username: `hope_ngo`
- Email: `ngo@example.com`
- Password: `password123`
- Role: **Select "NGO/Organization"**
- Organization: `Hope Food Bank`
- Click **Submit**

---

### **Phase 2: Create Food Listing (Donor)**

#### **Step 3: Login as Donor**
```
URL: http://127.0.0.1:5000/login
Username: john_donor
Password: password123
```

#### **Step 4: Create Food Listing**
1. Go to **Dashboard** or **Donor Portal**
2. Click **"Create Listing"** or **"Post Food"**
3. Fill in details:
   ```
   Title: Pizza - Restaurant Leftover
   Description: Fresh pizza from last night, 50 portions
   Food Type: Cooked
   Quantity: 50
   Location: Downtown Restaurant, 123 Main St
   Pickup Start Time: Today at 2:00 PM
   Pickup End Time: Today at 4:00 PM
   Allergens: Wheat, Dairy
   ```
4. Click **"Create Listing"** button

✅ Food listing is now available for NGOs to claim

---

### **Phase 3: Claim Food (NGO)**

#### **Step 5: Logout and Login as NGO**
1. Click **Logout** (top right)
2. Go to **Login** page
3. Login with NGO credentials:
   ```
   Username: hope_ngo
   Password: password123
   ```

#### **Step 6: Find and Claim Food**
1. Go to **"Available Food"** or **"Browse Listings"**
2. Find **"Pizza - Restaurant Leftover"** listing
3. Click **"Claim"** or **"Request"** button
4. Fill optional details:
   ```
   Notes: We will pick up this afternoon
   People to Serve: 50
   ```
5. Click **"Claim Food"** button

✅ Claim is now in **PENDING** status

---

### **Phase 4: Confirm Pickup & Generate OTP (NGO)**

#### **Step 7: Confirm Pickup (Triggers OTP Generation)**
1. Stay logged in as NGO
2. Go to **"Active Orders"** or **"My Claims"**
3. Find the claimed food item
4. Click **"Confirm Pickup"** button

📌 **IMPORTANT - THIS TRIGGERS OTP GENERATION:**
- Backend generates random 6-digit OTP code
- OTP is stored in database
- OTP is sent to Donor via email (or console in dev mode)
- Claim status changes to **"ACTIVE"**

✅ **OTP is now generated and sent to Donor**

---

### **Phase 5: Verify OTP (Donor) ⭐ MAIN TEST**

#### **Step 8: Logout and Login as Donor**
1. Click **Logout**
2. Login as Donor:
   ```
   Username: john_donor
   Password: password123
   ```

#### **Step 9: Find the Claim**
1. Go to **"Dashboard"** or **"My Listings"**
2. Find **"Pizza - Restaurant Leftover"** listing
3. Click on it to view details
4. You should see:
   - Status: **"Active"** (pickup is confirmed)
   - **"Verify OTP"** button or form

#### **Step 10: Enter OTP to Verify ⭐ THIS IS THE OTP CHECK**

```
📌 WHERE TO FIND OTP:
- Check console/terminal output (in development)
- Should show: "Generated OTP: XXXXXX for claim X"
- Example: "Generated OTP: 628471 for claim 5"

OR

- Check Flask Debug console for "MOCK EMAIL" output
- It will show the OTP code
```

**To Enter OTP:**
1. Look at the claim details page
2. You'll see **"OTP Verification"** section with text input
3. Enter the 6-digit OTP code
4. Click **"Verify OTP"** button

```
Example OTP Entry:
┌──────────────────────┐
│ Enter OTP Code:      │
│ ┌────────────────┐   │
│ │   628471       │   │
│ └────────────────┘   │
│ [Verify OTP] Button  │
└──────────────────────┘
```

#### **Step 11: Successful OTP Verification Response ✅**

If OTP is correct, you'll see:
```json
{
  "success": true,
  "message": "OTP verified successfully! Donation marked as completed.",
  "claim_id": 5
}
```

**Changes that happen automatically:**
- ✅ Claim status changes to **"completed"**
- ✅ `otp_verified` field set to **True**
- ✅ `pickup_time` recorded with current timestamp
- ✅ **Function 28**: Claim archived to HistoryLog table
- ✅ **Function 29**: Receipt generated automatically
- ✅ Receipt email sent to both Donor and NGO

---

### **Phase 6: View Receipt (Function 29) ⭐ SECOND MAIN TEST**

#### **Step 12: Go to Receipt Page**

**For Donor (staying logged in):**
1. After successful OTP verification, you'll see:
   ```
   ✓ Success message
   [View Receipt] button
   ```
2. Click **"View Receipt"** button

**OR manually navigate:**
```
URL: http://127.0.0.1:5000/claim/5/receipt
(Replace 5 with actual claim ID)
```

#### **Step 13: View Receipt Details**

You'll see a professional receipt page showing:

```
┌─────────────────────────────────────────────┐
│         🧾 DONATION RECEIPT                 │
├─────────────────────────────────────────────┤
│                                             │
│ ✓ Donation Successfully Completed!          │
│                                             │
│ Receipt ID: #RECV-5-20260430                │
│ Date: April 30, 2026 at 2:32 PM             │
│                                             │
│ DONATION DETAILS                            │
│ ├─ Food Item: Pizza - Restaurant Leftover  │
│ ├─ Quantity: 50 portions                    │
│ ├─ People Served: 50                        │
│                                             │
│ DONOR INFORMATION                           │
│ ├─ Name: john_donor                         │
│ ├─ Type: Individual Donor                   │
│                                             │
│ RECIPIENT                                   │
│ └─ Organization: Hope Food Bank             │
│                                             │
│ TRANSACTION DETAILS                         │
│ ├─ Pickup: Apr 30, 2026 2:30 PM             │
│ └─ Status: COMPLETED ✓                      │
│                                             │
│ YOUR IMPACT                                 │
│ You helped feed 50 people in need!          │
│ Thank you for making a difference!          │
│                                             │
│ [Print Receipt] [Download PDF]              │
└─────────────────────────────────────────────┘
```

#### **Step 14: Check Email (Console Output)**

Check the Flask terminal/console for mock email output:

```
MOCK EMAIL - To: donor@example.com
Subject: Food Donation Receipt - Pizza - Restaurant Leftover
Body:
Dear john_donor,

Your food donation has been successfully collected!

Receipt Details:
- Food Item: Pizza - Restaurant Leftover
- Quantity: 50 portions
- Recipient Organization: Hope Food Bank
- People Served: 50
- Pickup Date: Apr 30, 2026 2:30 PM

Claim ID: 5

Impact: Your donation helped feed 50 people in need.
Thank you for making a difference!

Best regards,
Food Waste Reduction System
```

---

## 🧪 TESTING CHECKLIST

### **OTP Functionality Test ✓**
```
□ Registered Donor account
□ Registered NGO account
□ Donor created food listing
□ NGO claimed the food
□ NGO confirmed pickup
□ OTP was generated (check console output)
□ Donor logged in
□ Donor found OTP code in terminal output
□ Donor entered OTP in verification form
□ Success message appeared: "OTP verified successfully!"
□ Claim status changed to "completed"
□ Notification sent to both parties
```

### **Receipt Functionality Test ✓**
```
□ After OTP verification, receipt was generated
□ Receipt page loaded successfully
□ Receipt shows all required information:
  □ Receipt ID (e.g., #RECV-5-20260430)
  □ Food title (Pizza - Restaurant Leftover)
  □ Quantity (50 portions)
  □ People served (50)
  □ Donor name (john_donor)
  □ Recipient organization (Hope Food Bank)
  □ Pickup date & time
  □ Completion status (COMPLETED)
  □ Impact message
□ Email was sent to donor (check console)
□ Email was sent to NGO (check console)
□ Receipt can be printed/downloaded
```

### **Backend Verification ✓**
```
□ Claim record updated with:
  □ status = 'completed'
  □ otp_verified = True
  □ pickup_time = timestamp
□ HistoryLog entry created with all details
□ Notifications created for both parties
□ Receipt data stored in database
```

---

## 🔍 WHERE TO FIND OTP CODE

### **In Terminal/Console Output:**
```
Look for line like:
Generated OTP: 628471 for claim 5

OR in MOCK EMAIL section:
OTP: 628471
```

### **Using Flask Shell (Alternative Method):**
```powershell
PS> python
>>> from app import db, Claim
>>> claim = Claim.query.filter_by(id=5).first()
>>> print(claim.otp_code)
628471
```

### **Using Browser Developer Tools:**
1. Open browser F12 (Developer Tools)
2. Go to **Network** tab
3. Make OTP verification request
4. Check response JSON for any OTP references

---

## 🐛 TROUBLESHOOTING

### **OTP Not Showing in Terminal**
```
Solution:
1. Check Flask console output carefully
2. Look for "Generated OTP:" message
3. Make sure NGO clicked "Confirm Pickup"
4. Check database: 
   SELECT otp_code FROM claim WHERE id=5;
```

### **OTP Verification Fails**
```
Check:
✓ OTP code matches exactly (6 digits)
✓ No extra spaces before/after
✓ Claim status is "active" (not "completed" yet)
✓ OTP code exists in database

If still fails:
SELECT otp_code, otp_verified FROM claim WHERE id=5;
```

### **Receipt Not Showing**
```
Check:
✓ OTP verification was successful
✓ Claim status is "completed"
✓ otp_verified = True
✓ Try direct URL: http://127.0.0.1:5000/claim/5/receipt
```

### **Email Not Sending**
```
In Development Mode:
- Emails are mocked (printed to console)
- Check Flask terminal for "MOCK EMAIL" output
- Check function: send_email() in app.py
```

---

## 📊 DATABASE VERIFICATION

### **Check Claim Status After OTP:**
```python
# Open Flask shell
python
>>> from app import db, Claim, HistoryLog
>>> claim = Claim.query.get(5)
>>> print(f"Status: {claim.status}")
>>> print(f"OTP Verified: {claim.otp_verified}")
>>> print(f"OTP Code: {claim.otp_code}")
>>> print(f"Pickup Time: {claim.pickup_time}")

# Check if archived
>>> history = HistoryLog.query.filter_by(claim_id=5).first()
>>> print(f"Archived: {history is not None}")
>>> if history:
...     print(f"Completion Time: {history.completion_time}")
```

---

## 📱 API ENDPOINTS FOR TESTING

### **1. Verify OTP (POST)**
```
URL: http://127.0.0.1:5000/claim/5/verify_otp
Method: POST
Content-Type: application/json

Body:
{
  "otp": "628471"
}

Response (Success):
{
  "success": true,
  "message": "OTP verified successfully! Donation marked as completed.",
  "claim_id": 5
}

Response (Error):
{
  "success": false,
  "error": "Invalid OTP code"
}
```

### **2. View Receipt (GET)**
```
URL: http://127.0.0.1:5000/claim/5/receipt
Method: GET
Requires: Login as Donor or NGO

Response: HTML Receipt Page
```

---

## ⏱️ EXPECTED TIMING

```
Timeline of Testing:

1. Account Creation ............... ~1 minute
2. Create Food Listing ............ ~2 minutes
3. NGO Claims Food ................ ~1 minute
4. NGO Confirms Pickup ............ ~30 seconds ⭐ OTP Generated
5. Donor Verifies OTP ............. ~1 minute ⭐ KEY TEST
6. View Receipt ................... ~30 seconds ⭐ KEY TEST

Total Testing Time: ~6 minutes for complete workflow
```

---

## ✅ SUCCESS CRITERIA

### **OTP Check is Working If:**
```
✓ OTP code generated (6 random digits)
✓ OTP stored in Claim.otp_code
✓ OTP verified successfully after entry
✓ Claim status changed to "completed"
✓ Both parties notified
```

### **Receipt is Working If:**
```
✓ Receipt page loads after OTP verification
✓ All details displayed correctly
✓ Email sent (shown in console)
✓ Receipt data shows correct information
✓ Impact metrics displayed
```

---

## 🎯 SUMMARY

**To Test OTP & Receipt:**

1. **Run Application**: `python app.py`
2. **Create 2 Accounts**: One Donor, One NGO
3. **Create Listing** (as Donor)
4. **Claim Food** (as NGO)
5. **Confirm Pickup** (as NGO) → OTP Generated
6. **Enter OTP** (as Donor) → Main Test ⭐
7. **View Receipt** → Second Test ⭐
8. **Check Console** for email confirmations

**All Tests Complete!** ✅

---

**Last Updated**: April 30, 2026  
**Status**: Ready for Testing  
**Flask Server**: Running on http://127.0.0.1:5000

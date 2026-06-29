# ⏱ PunchTrack — Office Time Management System

A full-featured office punch-time calculator with a Google Sheets + Apps Script backend and a GitHub Pages frontend. Supports multiple users, OTP email verification, admin approval workflow, tap-based time recording, salary deduction calculation, and more.

---

## 📁 File Overview

| File | Purpose |
|------|---------|
| `Code.gs` | Google Apps Script backend — all API logic |
| `index.html` | Frontend single-page app — deploy to GitHub Pages |
| `README.md` | This setup guide |

---

## 🚀 Setup — Step by Step

### STEP 1 — Create the Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a **new blank spreadsheet**.
2. Name it something like **PunchTrack Data**.
3. Copy the **Spreadsheet ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/  ►COPY_THIS_PART◄  /edit
   ```
   *(You'll need this if using a standalone script — see Step 2.)*

---

### STEP 2 — Set Up Google Apps Script

#### Option A — Bound Script (Recommended)

1. In your Google Sheet, click **Extensions → Apps Script**.
2. Delete the placeholder `function myFunction() {}` code.
3. Paste the **entire contents of `Code.gs`** into the editor.
4. Click **💾 Save** (Ctrl+S).

#### Option B — Standalone Script

1. Go to [script.google.com](https://script.google.com) and create a new project.
2. Paste the contents of `Code.gs`.
3. At the top of the file, add your Spreadsheet ID:
   ```javascript
   // Add this near the top, after the constants
   function ss() {
     return SpreadsheetApp.openById('YOUR_SPREADSHEET_ID_HERE');
   }
   ```

---

### STEP 3 — Run Initial Setup

1. In the Apps Script editor, select the function **`setup`** from the dropdown.
2. Click **▶ Run**.
3. Grant the requested permissions when prompted (Gmail for sending OTPs, Sheets access).
4. You should see `PunchTrack setup complete!` in the execution log.

> This creates all required sheets with headers and adds the default admin account.

---

### STEP 4 — Deploy as Web App

1. Click **Deploy → New deployment**.
2. Click the ⚙ gear icon next to "Select type" → choose **Web app**.
3. Configure:
   - **Description:** `PunchTrack v1`
   - **Execute as:** `Me`
   - **Who has access:** `Anyone` *(or "Anyone, even anonymous" if available)*
4. Click **Deploy**.
5. **Copy the Web App URL** — it looks like:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```
   > ⚠️ Keep this URL. You need it for the frontend.

> **Every time you edit `Code.gs`**, you must create a **New Deployment** (not update existing) for changes to take effect, then update `SCRIPT_URL` in `index.html`.

---

### STEP 5 — Deploy Frontend to GitHub Pages

1. Create a new **GitHub repository** (e.g., `punch-tracker`).
2. Upload `index.html` to the repository root.
3. Go to **Settings → Pages**.
4. Set Source: **Deploy from a branch** → Branch: `main` → Folder: `/ (root)`.
5. Click **Save**.
6. Your site will be live at:
   ```
   https://YOUR_USERNAME.github.io/punch-tracker/
   ```

---

### STEP 6 — Connect Frontend to Backend

Open `index.html` in a text editor (or on GitHub). Find this line near the top of the `<script>` section:

```javascript
const SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_DEPLOYMENT_URL';
```

Replace it with your actual deployment URL from Step 4:

```javascript
const SCRIPT_URL = 'https://script.google.com/macros/s/AKfycb.../exec';
```

Commit and push the change to GitHub.

---

### STEP 7 — (Optional) Update Email Verification Link

In `Code.gs`, the admin notification email contains a link back to your app. Update this constant at the top of the file:

```javascript
// This is only used for admin notification emails — not required for OTP to work
const APP_URL = 'https://YOUR_USERNAME.github.io/punch-tracker';
```

---

## 🔑 Default Admin Account

| Field | Value |
|-------|-------|
| Email | `rafi11.work@gmail.com` |
| Password | `666666` |
| Role | Admin |

> The admin account is hardcoded and **always works** — it bypasses email verification. Change the password inside `Code.gs` (`DEFAULT_ADMIN` constant) and redeploy when ready.

---

## 👤 User Registration Flow

```
1. User fills registration form → clicks "Create Account & Send Code"
2. System creates pending account → sends 6-digit OTP to email
3. User enters OTP (auto-submits when all 6 digits entered)
4. System marks email as verified → notifies admin
5. Admin logs in → Admin Panel → Approves the user
6. User receives approval email → can now log in
```

---

## ⏱ How Punch Recording Works

### Tap-Based (Primary Method)
- Tap **📍 RECORD CURRENT TIME** each time you arrive at or leave the office.
- **First tap of the day** = IN time.
- **Last tap of the day** = OUT time.
- Middle taps are logged for reference.
- Salary/hours calculations use the first and last tap automatically.

### Manual Entry (Override/Correction)
- Use the **Manual Entry & Corrections** section on the Punch page.
- Directly sets the IN/OUT times for any date.
- Useful for forgotten entries or corrections.
- Overrides tap-calculated times for that date.

---

## 📊 Dashboard Metrics

| Metric | Description |
|--------|-------------|
| **Progress Ring** | % of weekly target hours completed |
| **Remaining** | Hours left to complete this week |
| **Days Left** | Working days remaining (excluding holidays) |
| **Avg Needed** | Hours/day required to meet target |
| **Avg Done** | Your actual average hours/day so far |
| **Last Month** | Whether salary will be deducted, and by how much |

---

## 💰 Salary Deduction Formula

Deduction is calculated at the end of each month:

```
Required Minutes  = Working Days × Min Daily Hours × 60
Shortfall Minutes = max(0, Required − Actual)
Hour Deduction    = (Shortfall / Required) × Monthly Salary
Leave Deduction   = Deductible Holidays × (Salary / Working Days per Month)
Total Deduction   = Hour Deduction + Leave Deduction
```

> Configure `Salary`, `Min Daily Hours`, and `Working Days / Month` in **Rules & Settings**.

---

## 🗓 Holiday Types

| Type | Effect |
|------|--------|
| **Weekly Holidays** | Configured days (e.g., Fri + Sat) — excluded from working day count |
| **Government Holidays** | Admin-managed one-off dates — excluded from working day count |
| **Personal (Non-deductible)** | Approved leave — excluded from working days, no salary impact |
| **Personal (Deductible)** | Unapproved/unpaid leave — excluded from working days, salary deducted |

---

## ⚙️ Settings Reference

| Setting | Default | Description |
|---------|---------|-------------|
| Target Hours / Week | 48h | Total hours to complete per week |
| Min Hours / Day | 8h | Minimum to count as a full work day |
| Min Hours / Week | 40h | Used for deduction threshold |
| Week Starts On | Sunday | First day of the work week |
| Monthly Salary | 0 | Gross salary in BDT |
| Working Days / Month | 22 | Used for daily-rate leave deduction |
| Weekly Off Days | Fri, Sat | Days automatically excluded |

---

## 🛠 Google Sheets Structure

The system auto-creates these sheets on first run:

| Sheet | Purpose |
|-------|---------|
| `Users` | Registered users, roles, verification status |
| `Punch` | Daily IN/OUT records and total minutes |
| `Taps` | Individual time taps (raw records) |
| `Settings` | Per-user configuration |
| `GovHols` | Government holidays |
| `PersHols` | Personal holiday/leave records |
| `Verify` | OTP codes for email verification |
| `Sessions` | Active login sessions |

---

## 🔄 Redeployment Checklist

After any change to `Code.gs`:

- [ ] Save the file in Apps Script editor
- [ ] **Deploy → New deployment** (not "Manage deployments → Edit")
- [ ] Copy the new deployment URL
- [ ] Update `SCRIPT_URL` in `index.html`
- [ ] Commit and push `index.html` to GitHub

---

## 🐛 Troubleshooting

**"Server returned non-JSON" error**
→ The Apps Script URL is wrong, or the deployment isn't set to "Anyone" access. Redeploy.

**Admin can't log in**
→ Run `setup()` from the Apps Script editor to reset the admin account.

**OTP email not received**
→ Check spam folder. Make sure Gmail sending is authorized (run `setup()` and accept permissions).

**Changes to Code.gs not reflected**
→ You must create a **New Deployment** each time — editing an existing deployment doesn't update the live URL.

**"SCRIPT_URL not configured" toast appears**
→ You haven't replaced the placeholder in `index.html`. See Step 6.

---

## 📱 Browser Support

Works on all modern browsers. Optimized for both desktop and mobile.  
No build step required — pure HTML/CSS/JS frontend.

---

## 🔒 Security Notes

- Passwords are hashed (MD5 + salt) before storage in Google Sheets.
- All API calls use HTTPS.
- Sessions expire after 7 days.
- OTP codes expire after 30 minutes and are single-use.
- The admin fast-path only works with the exact hardcoded credentials.

---

*Built for Bangladesh office context — defaults to Asia/Dhaka timezone, BDT currency, and a Sunday–Thursday work week.*

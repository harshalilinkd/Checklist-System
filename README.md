# Checklist-System
High-performance Google Apps Script checklist system with real-time sync, role-based access, smart task scheduling, and optimized master checklist for scalable task management and automation.
## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Google Apps Script (V8) |
| Database | Google Sheets |
| Frontend | Vanilla JavaScript, Tailwind CSS (CDN) |
| CSV Loader | PapaParse 5.4.1 (Web Worker) |
| Icons/Fonts | Material Symbols, Inter (Google Fonts) |

---

## Project Files

| File | Purpose |
|---|---|
| `Code.gs` | All backend logic — sheet operations, auth, frequency engine |
| `Index.html` | Frontend — HTML, CSS, all JavaScript inlined |

---

## Google Sheets Schema

### Doers
`Name | Department | Email Address`

### Tasks
`Task | Doer Name | Department | Frequency | From Date | To Date | Status | Task ID | Assign By`

### Master Checklist
`Name | Email | Department | Task ID | Freq | Task | Planned | Actual | Status | Occurrence Key`

### Users (Login)
`Emp ID | Name | Department | Access | Email | Password`

> Archive sheet mirrors Master Checklist schema. All sheets are auto-created on first run.

---

## Frequency Codes

| Code | Schedule |
|---|---|
| `D` | Daily |
| `W` | Weekly |
| `F` | Fortnightly |
| `M` | Monthly |
| `Q` | Quarterly |
| `Y` | Yearly |
| `E1ST` – `E4TH` | 1st–4th of each month |
| `ELAST` | Last day of each month |
| `SM` | 15th and last day of each month |

---

## Roles & Access

| Role | Master Checklist | Dashboard / Doers / Tasks |
|---|---|---|
| Admin | All users' tasks | ✅ Full access |
| User | Own tasks only | ❌ Hidden |

Login via email + password stored in the `Users` sheet.

---

## Setup

1. Open your Google Sheet → **Extensions → Apps Script**
2. Paste `Code.gs` and create an HTML file named `Index` with `Index.html` contents
3. Update `CSV_CONFIG.MASTER_URL` in `Index.html` with your sheet's published CSV URL
   - Sheets → **File → Share → Publish to web** → Master Checklist sheet → CSV → Copy URL
4. Add credentials to the `Users` sheet (`Access` = `Admin` or `User`)
5. **Deploy → New deployment → Web app** → Execute as: Me → Access: Anyone
6. Run `repairChecklist()` once from the editor to backfill any missing IDs on existing data
7. Run `setupTriggers()` to enable the daily 9AM email digest and archive job

---

## Admin Utilities
Run these manually from the Apps Script editor when needed.

| Function | What it does |
|---|---|
| `diagnoseChecklist()` | Reports missing IDs and blank rows (read-only) |
| `repairChecklist()` | Backfills missing Task IDs and Occurrence Keys |
| `realignMasterTaskIds()` | Fixes Task ID mismatches between Tasks and Master sheets |
| `compactMaster()` | Removes blank rows from the Master sheet |
| `cleanOrphanedMasterRows()` | Clears Master rows whose parent task was deleted |

---

## Key Notes

- **`Done` is the only persisted status.** All others (Delayed, Today, Upcoming Focus, Scheduled) are computed at read time.
- **Occurrence Key** (`taskId|plannedISO`) is the stable row identifier — more reliable than row numbers.
- **CSV publishing delay:** After a write, the published CSV can take up to 5 minutes to reflect changes. The app handles this with optimistic UI updates.
- **`Session.getActiveUser()` is not used** — the app has its own login system to avoid Workspace restrictions.

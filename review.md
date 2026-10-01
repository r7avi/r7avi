
# Attendance Flow — ERP by Modulery Technology

## 1. Setup (one time)

| What | Where | Notes |
|---|---|---|
| Shifts | Finance & HR → Shifts | Start, end, grace (0–180 min), punch-out window (0–720 min), **Applies to** Office or Site |
| Defaults | auto-created per company | **OFFICE** 10:00 AM–06:00 PM · **SITE** 08:30 AM–06:00 PM · 15 min grace · 120 min window |
| Shift locations | on the shift | No location = default for all offices / all sites. One shift per office/site. |
| Boundaries | Branch / Site master | Geofence circle. No circle = punch accepted **unvalidated** |
| Employee | Employees | Branch + linked login. Project team posting gives sites. No shift field on the employee. |
| Permissions | Roles | `hr.attendance.punch` (own punch) · `hr.attendance.mark` (HR marks others) · `hr.attendance.view` · `hr.attendance.trail.view` · `hr.shift.view` / `hr.shift.manage` |

## 2. Which shift applies

1. **Punched in at one of their sites**: that site's SITE shift → else default SITE shift → else office rules.
2. **No site punch** (punched at office, unvalidated, employee page, HR mark):
   - Has a branch → office shift: branch's OFFICE shift → default OFFICE → first-linked OFFICE.
   - No branch, posted to sites → first site's shift.
3. Nothing fits → no shift. The day is still recorded; only a missing punch-out is judged.

The shift is **stamped on the day at creation** and never rewritten. Editing a shift later does not change past days.

## 3. Punch In (mobile, online only)

Checks run in this order. Any failure means **nothing is saved**.

1. Login linked to an employee, else: "not linked to an employee record".
2. Capture time: a future time is refused, and anything older than the offline limit is refused `ATT_CAPTURE_TOO_OLD`.
3. Location-tracking consent accepted, else the consent notice is shown.
4. Geofence, using boundaries as at the capture time:
   - GPS accuracy too poor → `ATT_LOCATION_UNRELIABLE`
   - Outside every circle → `ATT_OUT_OF_GEOFENCE` (shows the distance)
   - No boundary set, or enforcement off → accepted, flagged **Not validated**
5. **Selfie mandatory** → `ATT_SELFIE_REQUIRED` / `ATT_SELFIE_NOT_FOUND`.
6. Already punched in that day → refused.
7. Saved: day = **PRESENT**, shift stamped (§2), IP, app version, device model and OS recorded.

Accepted **any time**. An early punch (07:00, or 00:01 for an 08:30 shift) is valid.

## 4. Punch Out

1. Same login, capture-time and geofence checks. **Selfie optional.**
2. The day it closes: the capture date's open day; failing that, **yesterday's open day if it has a shift** (night shift). Otherwise `ATT_NOT_PUNCHED_IN`.
3. **Window**: allowed until **shift end + window** (e.g. 06:00 PM + 120 min = 08:00 PM). Later → `ATT_PUNCH_OUT_WINDOW_CLOSED`, nothing saved.
4. No shift → no window; can punch out any time that date.
5. **Offline (Android only)**: punch-out is queued. The **capture time** decides, not the sync time.

## 5. Hours

| Field | Rule |
|---|---|
| **Total Hrs** (`workedMinutes`) | Only the time **inside the shift**. Early in counts from the shift start; out after the end counts to the end; late in counts from the actual in-time. |
| **Extra uncounted Hrs** (`extraMinutes`) | Time before the start + after the end. Shown **only on the web day page**. Kept for OT. |
| No shift | Total Hrs = out − in. No extra. |
| Always | Total + Extra = out − in. No break deducted, no OT computed. Across midnight +24 h. |

Example (shift 08:30 AM–06:00 PM): in 07:00 AM, out 07:00 PM → **Total 9:30**, **Extra 2:30**.

## 6. Normal / Abnormal (Remarks)

Only **PRESENT** days are judged. Computed on every read, never stored.

| Case | Result |
|---|---|
| In within the grace (08:30–08:45) | **Normal** |
| In before the shift start | **Normal** + note "Early in by N min" |
| In after start + grace | **Abnormal** — "Late in by N min" |
| Not punched out by start + length + window | **Abnormal** — "No punch-out" (same evening) |
| No shift, left open past the date | **Abnormal** — "No punch-out" |
| Early or late punch-out | Never abnormal |
| Absent / Leave | Not judged (`-`) |

## 7. HR side (web)

| Action | Screen | Notes |
|---|---|---|
| Mark / regularize a day | Attendance → Mark attendance | Any in/out time, no window, no geofence. Stamps a shift on a new day. |
| Close a missed punch-out | Mark attendance | Set the real out-time |
| Daily roster | Today's attendance | Present · Not punched in · On leave · Absent. "Still in · until 08:00 PM" |
| Month register | Attendance | Shift · In · Out · Total Hrs · Remarks |
| Day detail | Attendance → open day | Shift, Total Hrs, **Extra uncounted Hrs**, selfies, location trail, geofence map, device info |
| Unvalidated review | Attendance → Unvalidated | Days accepted without a geofence check, with the reason |
| Timesheet | Attendance report | 1 / 3 / 6 / 12 months, filters, export (xlsx/csv/pdf) |

## 8. Exports

Report columns: Date · Emp Code · Employee · Department · Branch · Shift · Shift Start · Shift End · In · Out · Total Hours (H:MM) · Worked Hours (decimal) · Status · Abnormal (Yes/No) · Remarks · Location Verified.
Remarks examples: "Early in by 20 min", "Late in by 16 min; No punch-out".

## 9. Mobile (My Attendance)

Shows the shift before punch-in ("No shift set up for your office or site" if none). After punch-in it shows the shift, "Punch-out closes at", the Normal/Abnormal badge with the note, and Total Hrs. The Punch Out button turns into "Punch-out closed" once the window ends. All times are 12-hour (09:26 AM).

## 10. Retention

Selfies are purged after **90 days** (configurable). Attendance records are kept.

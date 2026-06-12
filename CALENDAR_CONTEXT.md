# NTU Business Solutions — Year 1 Training Calendar
## Project Context for Claude

---

## What this is
A single-file HTML/CSS/JS training calendar for NTU Business Solutions (BSCC), AY 2026/27.
Hosted as a static file on GitHub. All session details are stored in a Supabase database so edits are shared across everyone who opens the link.

**File:** `ntu-bizsol-y1-training-calendar.html`
**Repo:** `https://github.com/lohxtoshiki/Bizsol-Stuff`
**Branch:** `main`

---

## Tech stack
- Pure HTML/CSS/JS — no build step, no framework
- **Supabase** (PostgreSQL) for shared persistence via REST API (`fetch`)
- Google Fonts: Playfair Display, Jost, IBM Plex Mono
- Hosted via GitHub (raw or GitHub Pages)

---

## Supabase setup
- **Project ref:** `irwejdihqvukbrthiwol`
- **URL:** `https://irwejdihqvukbrthiwol.supabase.co`
- **Key to use:** publishable/anon key (NOT the secret key)
- **Edit password:** `BSCC`

### Database table: `session_details`
```sql
create table session_details (
  session_no   smallint primary key,
  time_override text,
  am_content    text,
  pm_content    text,
  location      text,
  case_of_week  text,
  directors     text,
  remarks       text,
  is_buffer     boolean,
  updated_at    timestamptz default now()
);
```
RLS policies: public SELECT, public INSERT, public UPDATE (password enforced in JS).

### Config constants (top of `<script>` block, ~line 857)
```js
const SUPABASE_URL  = "https://irwejdihqvukbrthiwol.supabase.co";
const SUPABASE_KEY  = "...anon/publishable key...";
const EDIT_PASSWORD = "BSCC";
```

---

## Session schedule (AY 2026/27)

| No | Date        | Time              | AM                          | PM                          |
|----|-------------|-------------------|-----------------------------|-----------------------------|
| 1  | 29 Aug 2026 | 9:00 AM – 5:00 PM | Appreciating the case       | STI & Sliding               |
| 2  | 5 Sep 2026  | 9:00 AM – 5:00 PM | Present STI                 | Recommendation              |
| 3  | 12 Sep 2026 | 9:00 AM – 5:00 PM | Present STI + Rec.          | Finance                     |
| 4  | 19 Sep 2026 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 5  | 26 Sep 2026 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 6  | 10 Oct 2026 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 7  | 17 Oct 2026 | 9:00 AM – 12:30 PM| Present case                | —                           |
| —  | 24 Oct 2026 | Buffer            | No session                  | —                           |
| 9  | 9 Jan 2027  | 9:00 AM – 5:00 PM | Present case                | Component refresher         |
| 10 | 16 Jan 2027 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 11 | 23 Jan 2027 | 8:00 AM – 7:30 PM | 7-hr case · 0800–1500       | Present · 1530–1930 · Tactics|
| 12 | 30 Jan 2027 | 9:00 AM – 4:00 PM | Present case                | Present                     |
| 13 | 13 Feb 2027 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 14 | 20 Feb 2027 | 9:00 AM – 12:30 PM| Present case                | —                           |
| 15 | 27 Feb 2027 | TBC               | 24hr finale / week-long case| —                           |

Note: S8 (24 Oct) is a buffer — no session number displayed on the calendar.
Note: S9 skips from S7 intentionally (the buffer occupies the S8 slot internally).

---

## Key design decisions

### Colour scheme
```css
--white:  #FFFFFF   /* page background */
--navy:   #041840   /* primary text */
--gold:   #A67D4B   /* accent, session cells */
--royal:  #030A8C   /* buffer cells */
--black:  #0D0D0D
```

### Cell types
- `.cell.session` — gold left border, clickable
- `.cell.buffer` — royal blue tint
- `.cell.holiday` — red tint (public holidays)
- `.cell.recess` — dashed border (recess weeks)
- `.cell.brk` — light paper background (winter/summer break)
- `.cell.event` — gold tint (social events like Welcome Dinner)
- `.cell.off` — plain (no special day)

### In-lieu holidays display the same as the main holiday day
(soft:true was removed from Aug 10, Nov 9, Feb 8 so all in-lieu days are full colour)

### Modal flow
1. Click session/buffer cell → **View mode** (read-only info)
2. Click "Edit Details" → **Auth mode** (password prompt)
3. Enter `BSCC` → **Edit mode** (form with all fields)
4. Save → writes to Supabase → returns to View mode

### Edit form fields
- Session Type toggle: Training Session ↔ Buffer — No Session
- Time, AM Content, PM Content (override base schedule)
- Location, Case for the Week, Training Directors, Remarks

### getEffective(s, stored)
Core helper that merges Supabase-stored overrides onto the base SESSIONS array entry:
- `stored.isBuffer` overrides `s.kind === "buffer"` when explicitly set
- `stored.time` overrides `s.time`
- `stored.amContent` / `stored.pmContent` override `s.morning` / `s.afternoon`

---

## Page load flow
1. Show loading overlay (`#loading`)
2. `init()` calls `loadAllData()` — fetches all rows from Supabase into `storedMap`
3. `renderCalendar()` and `renderList()` run with stored overrides applied
4. Click handlers attached to all `.cell[data-sno]` and `.lrow[data-sno]`
5. Loading overlay hidden

---

## Known gotcha — curly quotes in footer
The footer JS string contains Unicode curly quotes around the word "replacement":
```js
"in-lieu days marked “replacement”"
```
Whenever the file is rewritten with a tool (Write tool, Python, etc.), these can get flattened to plain ASCII `"` which breaks JS syntax. Always run this fix after any file rewrite:
```python
c = c.replace('marked "replacement"', 'marked “replacement”')
```
Or check with: `node --check /tmp/cal_check.js` after extracting the script block.

---

## Break / recess ranges
```js
RECESS_RANGES = [["2026-09-28","2026-10-02"], ["2027-03-01","2027-03-05"]]
BREAK_RANGES  = [["2026-12-07","2027-01-08"]]
```
Oct 25–31 is NOT a break (plain off days). Nov is NOT a break (plain off days).

---

## Public holidays
National Day 9 Aug, in-lieu 10 Aug  
Deepavali 8 Nov, in-lieu 9 Nov  
Christmas 25 Dec  
Chinese New Year 6–7 Feb 2027, in-lieu 8 Feb  
Hari Raya Puasa ~10 Mar 2027 (tentative)  
Good Friday 26 Mar 2027  

---

## Special events
Welcome Dinner: 28 Aug 2026, 6:00 PM – 10:00 PM (gold cell, not clickable)

---

## Git workflow
- Always `git pull` before pushing if edits were made on GitHub.com
- After any full file rewrite, run the curly-quote fix and JS syntax check before committing

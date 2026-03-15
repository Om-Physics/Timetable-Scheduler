# Changelog

All notable changes to the Python Timetable Scheduler are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [2.0.0] — 2026-03-15

### Major Release — Python Notebook → Full Web Application

The project has been completely reimplemented as a standalone interactive web application (`index.html`). The original Python notebook is preserved in `notebooks/`.

---

### Added

#### Web Application (index.html)
- Full 7-step setup wizard with sidebar navigation
- School/college name field — displayed as header on all timetable output
- Institution type selector — School mode vs College mode
- Working day picker — click-to-toggle any combination of Sunday–Saturday
- Class range selector — "From class" / "To class" with live preview of all class-sections
- Dynamic section manager — add/remove sections (A, B, C…)
- School hours configuration — start time, end time, period duration
- Break configuration — start time and duration in minutes
- Live capacity estimator — shows periods per day and per week based on inputs
- Dynamic subject/teacher entry — add unlimited subjects, each with teacher name, max grade, and part-time flag
- Part-time teacher availability system — per-teacher, per-day time windows
- Periods-per-week grid — full matrix of class × subject with totals and over-capacity warnings
- Friday-only subject selector (school mode)
- Color-coded timetable output — each subject has a unique color across all cells
- Teacher name displayed in every timetable cell
- Part-time teacher `PT` badge in timetable cells
- Break row displayed as a distinct styled row
- Free Period displayed for unfilled slots (with dashed border)
- School info header on timetable — name, type, class range, sections, date generated
- Schedule info bar — school hours, period duration, break time, periods/day, working days
- Subject legend below timetable
- Full teacher summary table — subjects, max grade, full-time/part-time status, availability windows
- Class-section tab navigation
- Regenerate button — re-runs scheduler with different random seed
- Print button — landscape print layout that hides UI chrome
- Sidebar navigation — jump between steps; completed steps marked with checkmark
- Sidebar summary — shows institution name, type, day count, class range, subject count after setup

#### Scheduler Engine (JavaScript port of Python algorithm)
- Time slot generation with correct break insertion
- Break skip on Fridays
- Fallback break insertion when break time falls after all class periods
- Teacher conflict detection — no teacher double-booked across classes at the same time
- Teacher-class-day deduplication — same teacher not repeated in same class on same day
- Part-time availability window enforcement
- Grade eligibility check — teacher only assigned up to their `maxGrade`
- School mode: max 1 period per subject per day; no back-to-back same subject
- College mode: max 2 periods per subject per day
- Fallback relaxation: if no subject passes strict filters, relaxes daily-repeat limit to avoid ghost slots
- Friday-only subject enforcement
- Post-generation warning report for unscheduled periods

---

### Fixed (from original Python notebook `Time_Table.ipynb`)

| Bug | Original | Fixed |
|-----|----------|-------|
| Break insertion off-by-one | `insert_index` loop overwrote itself on each iteration, potentially placing break in wrong position | Corrected to `findIndex` — finds first slot at or after `bkStart` |
| Period duration ignored | `subjects[subject].append((teacher, 45))` stored duration per teacher but `generate_routine` never read it — `durations = [45]` hardcoded separately | Period duration is now a single user-configurable field applied everywhere |
| Ghost periods in school mode | `assigned_per_day_count[day][subj] > 0` blocked all subjects when weekly periods > available days, permanently leaving slots empty | Added fallback relaxation pass when strict filtering returns no candidates |
| `wrap_cell` defined twice | Defined globally and again inside `view_routine` | Single definition, used everywhere |
| `is_school` scope inconsistency | Returned from `collect_user_settings` but also passed as parameter to `generate_routine` | Unified as a single state variable |
| `PdfPages` unused import | Imported from `matplotlib.backends.backend_pdf` but never used | Removed |
| No "Free Period" fallback | Unfilled slots silently skipped — no visual indication | Empty slots render as "Free Period" |

---

### Changed

- Project structure reorganised: `notebooks/`, `docs/`, root `index.html`
- README completely rewritten with live demo link, feature table, algorithm explanation, configuration reference

---

## [1.0.0] — 2025 (original)

### Added
- Python Jupyter notebook (`Time_Table.ipynb`)
- Interactive terminal-based input collection
- Constraint-based timetable generation algorithm
- Teacher conflict detection
- Part-time teacher availability
- Break insertion with Friday exception
- Friday-only subjects
- Matplotlib table visualisation
- Support for school and college modes
- Per-class-section period count configuration

# Technical Documentation

**Project:** Python Timetable Scheduler  
**Version:** 2.0.0  
**License:** GPL-3.0  
**Last updated:** March 2026

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Algorithm Deep Dive](#3-algorithm-deep-dive)
4. [Configuration Parameters](#4-configuration-parameters)
5. [Constraint System](#5-constraint-system)
6. [Known Limitations](#6-known-limitations)
7. [Testing Guide](#7-testing-guide)
8. [Contributing](#8-contributing)
9. [Glossary](#9-glossary)

---

## 1. Project Overview

The Python Timetable Scheduler is a tool for automatically generating weekly class schedules for schools and colleges. It takes institutional configuration (classes, sections, subjects, teachers, timings) as input and produces a complete timetable with no teacher conflicts.

**Origin:** The project began as a Python Jupyter notebook with terminal-based input. Version 2.0 is a complete reimplementation as a browser-based single-file web application, preserving and improving the original scheduling algorithm.

**Design philosophy:**
- Zero installation — runs in any browser
- No server or backend required
- All data stays local (nothing is transmitted or stored)
- Deterministic in structure, randomised in ordering (regenerate for variety)

---

## 2. Architecture

### v2.0 — Web Application

```
index.html
├── CSS  (embedded)  — layout, timetable styles, print styles
├── HTML (embedded)  — app shell, sidebar, content area
└── JS   (embedded)  — state management, wizard steps, scheduler engine
```

The app is entirely self-contained. No external libraries, no CDN calls. State is held in a plain JavaScript object. The UI re-renders from state on every interaction.

**State object structure:**

```javascript
{
  step: Number,               // current wizard step (0–6)
  maxUnlocked: Number,        // furthest step reached (enables sidebar nav)
  schoolName: String,
  institutionType: 'school' | 'college',
  weekDays: String[],         // ordered subset of ALL_DAYS
  classFrom: Number,
  classTo: Number,
  sections: String[],
  startTime: String,          // HH:MM
  endTime: String,
  breakStart: String,
  breakDuration: Number,      // minutes
  periodDuration: Number,     // minutes
  subjects: [
    { name, teacher, maxGrade, isPartTime }
  ],
  partTime: [
    { teacher, day, from, to }  // HH:MM strings
  ],
  periodsGrid: {
    'classNum-section': { subjectName: periodsPerWeek }
  },
  fridayOnly: String[],       // subject names
  routine: Object | null,     // generated timetable
  subjectColors: Object,      // subject → hex color
  activeClass: String | null,
  warnings: String[],
  errors: Object
}
```

### v1.0 — Python Notebook

The original notebook uses Python globals for configuration and a series of input() prompts to collect settings. The scheduler logic is structurally identical to v2.0 but written in Python. Output is rendered using Matplotlib tables.

---

## 3. Algorithm Deep Dive

### 3.1 Time Slot Generation

`buildTimeSlots(start, end, dur, bkStart, bkDur, day)`

Produces an ordered list of slot objects `{type, start, end}` for a single school day.

**Steps:**
1. Walk from `start` to `end` in increments of `dur` (period duration)
2. When `current >= bkStart`, insert a Break slot and skip ahead by `bkDur`
3. Break is **skipped entirely on Fridays**
4. If `bkStart` is never reached during the walk (e.g. break falls after all periods), insert the break at the correct sorted position as a fallback

**Edge case handled:** In the original notebook, the fallback break insertion used a loop that kept overwriting `insert_index`, meaning the break could be placed at the wrong position. Fixed in v2.0 using `Array.findIndex`.

### 3.2 Subject Candidate Filtering

For each Class period slot, the algorithm builds a list of candidate subjects:

**Strict filters (school mode):**
- `periodsRemaining[subject] > 0`
- If subject is friday-only → only include on Fridays
- Subject not already used today (`daySubCount[day][sub] >= 1`)
- Subject is not the same as the immediately previous period (no back-to-back)

**Strict filters (college mode):**
- Same as above except the daily cap is 2 instead of 1

**Fallback relaxation:**  
If strict filtering returns zero candidates, the algorithm retries with only the `periodsRemaining > 0` and friday-only filters active. This prevents permanent empty (ghost) slots when the number of required periods exceeds the number of available days.

### 3.3 Teacher Assignment

For each candidate subject (in shuffled order), the algorithm finds a valid teacher:

1. Filter teachers by `maxGrade >= classNumber`
2. Shuffle the filtered list
3. For each teacher candidate:
   - Check no time overlap with their existing schedule that day (`teacherDaySlots`)
   - Check they haven't already been used in this class-section-day (`teacherDayClasses`)
   - If part-time: check the slot falls within one of their availability windows
4. Assign the first teacher who passes all checks

### 3.4 Randomisation

`shuffle()` applies a Fisher-Yates shuffle to both the subject candidates and the teacher candidates before each assignment attempt. This means:

- The same configuration produces different timetables on each run
- The **Regenerate** button re-runs the full algorithm with a new random ordering
- Results are not reproducible between runs (no seed is set)

### 3.5 Warning Collection

After all class-sections are processed, the algorithm checks `periodsRemaining` for each class. Any subject with `remainingCount > 0` is added to `state.warnings` and displayed in the result panel.

Common causes of warnings:
- More periods required than available slots in the week
- All teachers for a subject conflict with other assignments at every available slot
- A part-time teacher's availability window is too narrow

---

## 4. Configuration Parameters

### Timing Parameters

| Parameter | Type | Constraint | Description |
|-----------|------|------------|-------------|
| `startTime` | HH:MM | Must be before `endTime` | When school begins |
| `endTime` | HH:MM | Must be after `startTime` | When school ends |
| `breakStart` | HH:MM | Must be within school hours | When break begins |
| `breakDuration` | Integer (min) | 5–120 | How long the break lasts |
| `periodDuration` | Integer (min) | 20–120 | Duration of each class period |

**Estimated periods per day:**
```
periodsPerDay = floor((endTime - startTime - breakDuration) / periodDuration)
```

### Teacher Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `teacher` | String | Display name — used in timetable cells |
| `maxGrade` | Integer | Highest class number this teacher can teach. Set to 99 for no limit. |
| `isPartTime` | Boolean | If true, must have availability windows in Step 5 |

### Part-time Availability

Each record specifies one window: `{ teacher, day, from, to }`.

A teacher can have multiple windows across different days. The scheduler checks that a slot's `[start, end)` falls entirely within at least one of their windows for that day.

If a part-time teacher has no windows set for a given day, they are effectively unavailable that day.

---

## 5. Constraint System

| Constraint | Mode | Rule |
|------------|------|------|
| No teacher double-booking | Both | A teacher cannot appear in two different classes at the same time |
| No teacher repeat in class-day | Both | A teacher is not scheduled twice in the same class on the same day |
| Daily subject cap — school | School | Each subject appears at most once per day per class |
| Daily subject cap — college | College | Each subject appears at most twice per day per class |
| No back-to-back | School | The same subject cannot occupy two consecutive periods |
| Friday-only | School | Designated subjects appear only in Friday slots |
| Break skip on Friday | Both | No break row is inserted on Fridays |
| Part-time window | Both | Part-time teacher only assigned within their declared windows |
| Grade eligibility | Both | Teacher only assigned to classes ≤ their `maxGrade` |

---

## 6. Known Limitations

**Algorithm type:** The scheduler is a **randomised greedy algorithm**. It assigns periods one at a time without backtracking. This means:
- It cannot guarantee a globally optimal schedule
- A valid complete schedule may exist but the algorithm might fail to find it if it makes a poor early assignment
- Regenerating typically resolves most conflicts for realistic configurations

**No persistence:** All data is held in browser memory. Refreshing the page clears everything. There is no save/load feature in v2.0.

**Single teacher per subject per class:** The current model assigns one teacher per subject. If two teachers share the same subject for different sections, add them as separate subject entries with the same subject name but different teachers.

**Fixed period duration:** All periods in a day have the same duration. Variable-length periods (e.g. double periods, lab sessions) are not supported.

---

## 7. Testing Guide

### Suggested test scenarios

**Scenario 1 — Basic school (5 minutes)**
- School name: Test School
- Type: School, Days: Mon–Fri
- Classes: 11–12, Sections: A, B
- Times: 07:00–13:00, Break: 10:00, 20 min, Period: 45 min
- 3 subjects, 3 teachers, all full-time
- 3 periods/week each
- Expected: Clean timetable, no warnings

**Scenario 2 — Part-time teacher**
- Add one part-time teacher with availability only Mon/Wed 09:00–12:00
- Set their subject to 4 periods/week
- Expected: Teacher appears only in Monday/Wednesday slots within their window

**Scenario 3 — Overload (stress test)**
- Set 6 subjects × 5 periods = 30 periods/week, but only ~25 slots available
- Expected: Timetable generates with warning listing unscheduled subjects

**Scenario 4 — Friday-only subject**
- Mark one subject as Friday-only
- Expected: That subject appears only in Friday column across all classes

**Scenario 5 — College mode**
- Switch to College
- Set one subject to 10 periods/week across 5 days
- Expected: Subject appears up to twice per day; no back-to-back restriction applies

### What to check as a tester

- [ ] Does the school name appear at the top of the timetable?
- [ ] Is the break row in the correct position on all days?
- [ ] Is the break absent on Fridays?
- [ ] Are teacher names shown in every period cell?
- [ ] Do part-time teachers only appear in their declared windows?
- [ ] Does regenerating produce a different timetable?
- [ ] Does the print layout look clean?
- [ ] Are warnings shown when periods can't be scheduled?

---

## 8. Contributing

### Reporting issues

Open a GitHub Issue with:
- Browser and OS
- Step in the wizard where the issue occurred
- What you expected vs what happened
- Screenshot if relevant

### Suggesting features

Open a GitHub Issue with the label `enhancement`. Good candidates for future versions:
- Export to PDF / Excel
- Save and load configurations (localStorage)
- Multiple teachers per subject
- Variable period durations (double periods)
- Drag-and-drop manual adjustment of generated timetable
- Conflict highlighting in the output

### Pull requests

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature-name`
3. Make your changes to `index.html`
4. Test across Chrome, Firefox, and Edge
5. Submit a pull request with a clear description

All contributions must be licensed under GPL-3.0.

---

## 9. Glossary

| Term | Definition |
|------|------------|
| **Class-section** | A specific group, e.g. `11-A` (Class 11, Section A) |
| **Period** | A single timetable slot of fixed duration |
| **Break** | A non-teaching slot inserted at a configured time each day |
| **Free Period** | A teaching slot that the scheduler could not fill |
| **Part-time teacher** | A teacher available only during specified time windows |
| **Max grade** | The highest class number a teacher is qualified to teach |
| **Friday-only subject** | A subject restricted to appearing only in Friday's schedule |
| **Ghost slot** | A period that is silently skipped — prevented in v2.0 by fallback relaxation |
| **Greedy algorithm** | An approach that makes the locally best choice at each step without backtracking |
| **Conflict** | Two assignments requiring the same teacher at the same time |

# 📅 Python Timetable Scheduler

> A smart, interactive web-based timetable generator for schools and colleges — built from a Python notebook into a full no-install browser app.

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen)]()
[![Platform: Web](https://img.shields.io/badge/Platform-Web%20%7C%20Browser-orange)]()

---

## 🌐 Live Demo

**Try it instantly — no install needed:**

👉 **[https://om-physics.github.io/Python_Timetable_Scheduler/](https://om-physics.github.io/Python_Timetable_Scheduler/)**

Open in any browser. Works on desktop and mobile. No sign-up required.

---

## 📌 What Is This?

This project started as a Python Jupyter notebook (`Time_Table.ipynb`) for generating school and college timetables using a constraint-based scheduling algorithm. It has since been fully rebuilt into a **standalone, interactive web application** — a single HTML file that runs entirely in the browser with no server, no Python, and no dependencies.

The scheduling logic is preserved and improved from the original notebook, with a full UI wrapped around it.

---

## ✨ Features

### Scheduling Engine
- Automatic weekly timetable generation for multiple classes and sections
- **Conflict detection** — no teacher is double-booked across classes at the same time
- **Part-time teacher support** — assign specific availability windows per teacher per day
- **Friday-only subjects** — e.g. Islamic Education, Moral Science (school mode)
- **College mode** — allows up to 2 periods of the same subject per day
- **School mode** — enforces no back-to-back same subject; max 1 per day
- Graceful fallback: unfilled slots show as "Free Period" with a warning summary
- Break slot insertion with correct positioning (skipped on Fridays)

### User Interface (7-Step Wizard)
| Step | What You Configure |
|------|--------------------|
| 1 | Institution name, type (School/College), working days |
| 2 | Class range (e.g. Class 9 to 12) and sections (A, B, C…) |
| 3 | School hours, break time, break duration, period duration |
| 4 | Subjects, teachers, max grade per teacher, part-time flag |
| 5 | Part-time teacher availability (day + time window) |
| 6 | Periods per week per class per subject, Friday-only subjects |
| 7 | Generated timetable with full info header |

### Timetable Output
- **School name header** with date, type, class range, sections
- **Schedule info bar** — hours, period duration, break time, periods/day
- Color-coded subjects with teacher name in every cell
- Part-time teacher `PT` badge in cells and teacher summary
- Tabbed navigation across all class-sections
- Subject legend with teacher mapping
- **Full teacher summary table** — subjects, max grade, type, availability
- **Print-ready** — landscape print layout, hides UI chrome

---

## 🚀 Getting Started

### Option A — Use the live web app (recommended)
Visit the live demo link above. No setup needed.

### Option B — Run locally
1. Download or clone this repository
2. Open `index.html` in any modern browser (Chrome, Firefox, Edge, Safari)
3. That's it — fully offline capable

```bash
git clone https://github.com/Om-Physics/Python_Timetable_Scheduler.git
cd Python_Timetable_Scheduler
# Open index.html in your browser
```

### Option C — Original Python notebook
The original Jupyter notebook is preserved at `notebooks/Time_Table.ipynb`.

Requirements:
```bash
pip install pandas matplotlib
```
Run in Jupyter:
```bash
jupyter notebook notebooks/Time_Table.ipynb
```

---

## 📁 Repository Structure

```
Python_Timetable_Scheduler/
│
├── index.html                  # ← Main web app (open this in browser)
│
├── notebooks/
│   └── Time_Table.ipynb        # Original Python notebook
│
├── docs/
│   ├── DOCS.md                 # Full technical documentation
│   └── CHANGELOG.md            # Version history
│
├── README.md                   # This file
└── LICENSE                     # GPL-3.0
```

---

## 🧠 How the Scheduler Works

The algorithm operates in passes over each class-section:

1. **Time slot generation** — splits the school day into fixed-duration periods, inserting a break at the configured time (skipped on Fridays)
2. **Candidate filtering** — for each slot, finds subjects that still have remaining periods, respect daily limits, aren't friday-only (unless it's Friday), and aren't the same as the previous slot (school mode)
3. **Teacher assignment** — for each candidate subject, checks teachers for: grade eligibility, no time conflicts across classes, no repeat in the same class-day, and part-time availability windows
4. **Fallback relaxation** — if strict filtering leaves no options, relaxes the daily-repeat rule to avoid ghost (empty) slots
5. **Warning collection** — after all classes are scheduled, reports any subjects with remaining unscheduled periods

---

## ⚙️ Configuration Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| Institution type | School or College | School |
| Working days | Any subset of Sun–Sat | Sun–Thu |
| Class range | From class / To class | 11–12 |
| Sections | e.g. A, B, C | A, B |
| Start time | School opening time | 07:00 |
| End time | School closing time | 14:00 |
| Break start | When break begins | 10:00 |
| Break duration | Minutes | 20 |
| Period duration | Minutes per class | 45 |
| Max grade (teacher) | Highest class a teacher can teach | 99 |
| Part-time availability | Teacher → Day → Time window | — |
| Periods per week | Per subject per class | 3 |
| Friday-only subjects | Only scheduled on Fridays | — |

---

## 🤝 Contributing

Contributions are welcome! See [CONTRIBUTING](docs/DOCS.md#contributing) for guidelines.

To suggest improvements, open a GitHub Issue with:
- A clear description of the problem or feature
- Steps to reproduce (for bugs)
- A screenshot if relevant

---

## 🔒 Known Limitations

- The scheduler uses a **randomized greedy algorithm** — it is not guaranteed to find a globally optimal schedule. Regenerating may produce better results.
- Very tight schedules (many subjects, few days) may leave some periods unscheduled. The warning panel will report these.
- The app runs fully client-side — no data is saved between sessions.

---

## 📄 License

This project is licensed under the **GNU General Public License v3.0**.

You are free to use, modify, and distribute this software under the terms of the GPL-3.0. Any derivative work must also be released under GPL-3.0.

See the [LICENSE](LICENSE) file for full terms, or visit [gnu.org/licenses/gpl-3.0](https://www.gnu.org/licenses/gpl-3.0).

---

## 👤 Author

**Om-Physics**
GitHub: [@Om-Physics](https://github.com/Om-Physics)

---

*Built with ❤️ for teachers and administrators who deserve better tools.*

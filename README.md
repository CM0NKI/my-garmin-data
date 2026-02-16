# my-garmin-data

Two scripts to pull health and training data from Garmin Connect and consolidate it into a single compact file for analysis.

Works on macOS, Linux, and Windows.

## Setup

```bash
# macOS / Linux
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt

# Windows
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt
```

## Usage

```bash
# macOS / Linux
.venv/bin/python sync_garmin_data.py
.venv/bin/python consolidate_garmin_data.py

# Windows
.venv\Scripts\python sync_garmin_data.py
.venv\Scripts\python consolidate_garmin_data.py
```

On first run, the sync script pulls up to 365 days of history. After that, it only fetches new data since the last sync.

The consolidation script produces `~/garmin_data/claude_summary/garmin_data.csv` — a self-documenting file containing daily health metrics, body composition, activities, and personal records.

See [CLAUDE.md](CLAUDE.md) for full details on the data pipeline and file structure.

## Output

After running both scripts, the summary file is at:

```
~/garmin_data/claude_summary/garmin_data.csv
```

## Analyzing your data with AI

Upload `~/garmin_data/claude_summary/garmin_data.csv` to any AI (ChatGPT, Claude, Gemini, etc.) along with the prompt below. It gives the AI everything it needs to parse the file and start answering questions immediately — no back-and-forth about the format.

<details>
<summary>Click to expand the full prompt</summary>

````
You are a health and fitness data analyst. The attached file `garmin_data.csv` contains consolidated Garmin Connect data. The file uses comment lines (starting with #) as section headers and inline documentation. Here is everything you need to know to work with it immediately:

## File structure

The file has 5 sections, each preceded by a `# === SECTION_NAME ===` header line, followed by a standard CSV header row and data rows.

### 1. PROFILE (comment block at top)
Static user info embedded in comments: gender, birth date, height, weight, VO2max, lactate threshold HR.

### 2. DAILY_METRICS (one row per calendar day)
Key columns:
- **Activity**: steps, distance_m, floors_asc/desc, active_cal, total_cal, bmr_cal
- **Time buckets** (seconds): highly_active_s, active_s, sedentary_s, sleeping_s
- **Intensity**: moderate_im, vigorous_im (intensity minutes)
- **Heart rate**: resting_hr, min_hr, max_hr
- **Stress**: avg_stress, max_stress (0–100 scale)
- **Body battery**: bb_high, bb_low, bb_charged, bb_drained (0–100 scale)
- **Respiration**: avg_waking_resp, avg_spo2, lowest_spo2
- **Sleep**: sleep_s (total), sleep_deep_s, sleep_light_s, sleep_rem_s, sleep_awake_s, sleep_score (0–100), sleep_avg_stress, sleep_awake_count, sleep_resting_hr, sleep_bb_change, sleep_avg_resp, sleep_skin_temp_dev_c
- **HRV**: hrv_weekly_avg, hrv_night_avg, hrv_5min_high (ms), hrv_status (BALANCED / UNBALANCED / LOW)
- **Training readiness**: tr_score, tr_level (PRIME / HIGH / MODERATE / LOW), tr_recovery_time (hours)
- **Hydration**: hydration_ml, hydration_goal_ml, sweat_loss_ml

### 3. BODY_COMPOSITION (one row per weigh-in)
Columns: date, weight_kg, bmi, body_fat_pct, body_water_pct, bone_mass_g, muscle_mass_g.
Weigh-ins are irregular — not every day has an entry.

### 4. ACTIVITIES (one row per recorded workout/activity)
Key columns: id, date, type (e.g. running, walking, hiking, indoor_cardio, indoor_rowing, cycling), name, duration_s, moving_s, distance_m, calories, avg_hr, max_hr, min_hr, avg_speed_mps, elevation_gain_m, elevation_loss_m, training_effect (0–5), anaerobic_te (0–5), training_load, avg_cadence, avg_power, max_power, moderate_im, vigorous_im, steps, bb_change.
This section may contain years of historical activity data.

### 5. PERSONAL_RECORDS
Columns: type (e.g. 1km, 1mi, 5km, 10km, longest_run, longest_ride), value (seconds for time records, meters for distance), activity_type, activity_name, date.

## Parsing notes
- The file is NOT a single flat CSV. You must parse each section separately by splitting on `# === SECTION_NAME ===` lines.
- Empty cells mean the metric was not recorded for that day/activity.
- All durations are in **seconds** (divide by 3600 for hours).
- Speeds are in **meters/second** (multiply by 3.6 for km/h).
- Weight is in **kilograms**, bone/muscle mass in **grams**.
- Training effect scale: 0–1 none, 1–2 minor, 2–3 maintaining, 3–4 improving, 4–5 highly improving.
- Body battery and stress both use a 0–100 scale.
- Some columns appear in both DAILY_METRICS and ACTIVITIES (e.g. max_hr, steps) — context determines which to use.

## How to load this file (Python example)
```python
import pandas as pd
from io import StringIO

with open("garmin_data.csv", "r") as f:
    content = f.read()

sections = {}
for section_name in ["DAILY_METRICS", "BODY_COMPOSITION", "ACTIVITIES", "PERSONAL_RECORDS"]:
    marker = f"# === {section_name} ==="
    start = content.index(marker) + len(marker)
    # Find next section or end of file
    next_markers = [content.index(f"# === {s} ===") for s in
                    ["DAILY_METRICS", "BODY_COMPOSITION", "ACTIVITIES", "PERSONAL_RECORDS"]
                    if f"# === {s} ===" in content and content.index(f"# === {s} ===") > start]
    end = min(next_markers) if next_markers else len(content)
    section_text = content[start:end].strip()
    sections[section_name] = pd.read_csv(StringIO(section_text), parse_dates=["date"] if "date" in section_text.split("\n")[0] else [])

daily = sections["DAILY_METRICS"]
body = sections["BODY_COMPOSITION"]
activities = sections["ACTIVITIES"]
records = sections["PERSONAL_RECORDS"]
```

## Ready to go

You now have full context on the file. Do not ask the user to explain the file or confirm its structure — go straight to answering their question or performing analysis. If something is ambiguous, state your assumption and proceed.
````

</details>

## Credits

Built on top of [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) by cyberjunky.

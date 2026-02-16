# Garmin Connect Python Project

## What This Is

A Python library (`garminconnect`) for the Garmin Connect API, plus two personal scripts for syncing and consolidating health data.

## How to get fresh data and update the summary

Every time you want up-to-date Garmin data (e.g. after a new workout, a new day, or a weigh-in), run these two commands in order:

```bash
cd ~/dev/python-garminconnect

# 1. Pull new data from Garmin Connect (needs internet + auth)
.venv/bin/python sync_garmin_data.py

# 2. Rebuild the consolidated summary file
.venv/bin/python consolidate_garmin_data.py
```

Or as a one-liner:

```bash
cd ~/dev/python-garminconnect && .venv/bin/python sync_garmin_data.py && .venv/bin/python consolidate_garmin_data.py
```

**What happens:**

1. `sync_garmin_data.py` downloads new daily health data and activities from Garmin Connect into `~/garmin_data/`. It only fetches days/activities that don't already exist locally. If your stored auth tokens (`~/.garminconnect`) are expired, it will prompt for email, password, and MFA code.
2. `consolidate_garmin_data.py` reads all the raw JSON files and produces one compact summary file at `~/garmin_data/claude_summary/garmin_data.csv`. It only processes new days/activities (incremental). Original data is never modified.

After running both, the summary file is ready. That's it.

## How to ask Claude about the data

Point Claude to this single file:

```
~/garmin_data/claude_summary/garmin_data.csv
```

The file is self-documenting — it has a comment header at the top explaining every column, every scale, and every section. It contains:

1. **PROFILE** — gender, birth date, height, weight, VO2max, lactate threshold HR
2. **DAILY_METRICS** — one row per day (~50 columns): steps, sleep breakdown, HRV, stress, body battery, training readiness, hydration, respiration, SpO2
3. **BODY_COMPOSITION** — one row per weigh-in: weight (kg), BMI, body fat %, body water %, bone mass, muscle mass
4. **ACTIVITIES** — one row per activity (~24 columns): type, duration, HR, training effect, training load, power, cadence, elevation, body battery impact
5. **PERSONAL_RECORDS** — best performances (fastest 1K/5K, longest run/ride, etc.)

Only go to the raw JSON files in `~/garmin_data/daily/` or `~/garmin_data/activities/` if granular intra-day or per-activity detail is needed beyond what the summary provides.

## Data locations

| Path | Contents |
| --- | --- |
| `~/garmin_data/daily/{YYYY-MM-DD}/` | 13 JSON files per day (summary, sleep, hrv, stress, etc.) |
| `~/garmin_data/activities/` | One JSON file per activity (by activity ID) |
| `~/garmin_data/body_composition/` | Body comp and weigh-in data |
| `~/garmin_data/weekly/` | Weekly step/stress trends (not in summary — derivable from daily data) |
| `~/garmin_data/profile/` | User profile and devices |
| `~/garmin_data/personal_records.json` | Personal bests |
| `~/garmin_data/claude_summary/garmin_data.csv` | **The single file to read for analysis** |
| `~/garmin_data/claude_summary/processing_state.json` | Tracks which dates and activities have been consolidated |
| `~/garmin_data/sync_state.json` | Tracks last sync date for incremental pulls |

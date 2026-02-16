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

## Credits

Built on top of [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) by cyberjunky.

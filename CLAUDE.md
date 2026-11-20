# CLAUDE.md

Guidance for Claude Code in this repository.

## What it is

A server service that turns the daily Solinst LevelSender report emails
(water level, temperature, conductivity per site) into:

- a SQLite archive of every sample (`water_monitor/store.py`);
- ThingsBoard telemetry, one device per site, over the HTTP device API
  (`water_monitor/tb.py`), for the dashboard;
- `water-report.xlsx` and `water-report.ods`: a Summary sheet and one tab
  per site with daily lows, highs and averages and four trend charts
  (`water_monitor/workbook.py`), which the `email` command mails.

`perl/` is a Perl port for Perl readers that writes only the `.xlsx`; it
reads the same `water.toml` and the same archive. Keep the two in step:
a change to the parsing, the daily statistics or the spreadsheet layout is
made in both, and both test suites check the same numbers.

## Rules

- Times: stored as UTC, shown as US Eastern (America/New_York); a day is an
  Eastern calendar day. Sample times in the emails carry no zone; each
  site's `utc_offset` in `water.toml` says what its logger clock is on.
- Pure cores (parser, daily statistics, sheet rows, message bodies) take no
  I/O, so tests drive them directly; IMAP, SMTP, HTTP and files stay thin.
- The repository is public: no real site data, addresses or secrets.
  Fixtures in `tests/fixtures/` are anonymised copies of real reports.
  Secrets come from the environment variables `water.toml` names.
- Every change has a test, and the tests pass before a push.

## Commands

```bash
pip install -r requirements.txt pytest ruff
ruff check .
pytest -q
(cd perl && prove -l t/)
python -m water_monitor -c water.toml --no-send ingest report.eml
python -m water_monitor -c water.toml export
```

CI (`.github/workflows/ci.yml`) runs ruff, pytest, the Perl tests and a
Docker build.

# Canada SWE Internship Finder Version 1

A command-line tool that finds **software engineering internship / co-op
positions in Canada**, filtered by how recently they were posted. By default it
shows roles posted in the **last 24 hours**, but every filter is adjustable.

It works **year-round** (Fall, Winter, Spring, Summer) and **auto-updates across
cycles**, so you can keep using the same script next semester and next year
without editing anything.

---

## What it does

- Pulls live job data from a public, structured internship feed (refreshed hourly).
- Keeps only roles that are **active**, **located in Canada**, **software-related**,
  and **posted within your chosen time window**.
- Prints clean results to the terminal and can export to **CSV**, **Excel (.xlsx)**,
  or **JSON**.

### Where the data comes from

The tool reads the public `listings.json` published by the **SimplifyJobs × Pitt
CSC** internship project — a community + automated feed that scrapes the career
pages of hundreds of top tech companies and normalizes them into one file.

**Why this instead of scraping LinkedIn / Indeed?** The major job boards block
automated scraping and forbid it in their terms of service. This feed is public,
structured, refreshed hourly, and legal to consume — which makes it both more
reliable and lower-risk than scraping.

Although the source repository is *named* after a summer cycle
(e.g. `Summer2026-Internships`), its feed actually carries **every term** — Fall,
Winter, Spring, and future summers all live in the same file. That's why the tool
works in any season.

### How it stays current across years

SimplifyJobs launches a fresh `Summer{YEAR}-Internships` repository for each new
cycle. On every run, the tool **auto-detects the newest existing repo** (it checks
next year, this year, then last year) and uses it. When the 2027 cycle launches,
the script switches over automatically — no code changes needed. If GitHub can't
be reached, it falls back to a known-good repo.

---

## Requirements

- **Python 3.8+** — no third-party packages required for normal use.
- **`openpyxl`** — only needed if you use `--xlsx` (Excel export):
  ```bash
  pip install openpyxl
  ```

No API key, login, or account is required.

---

## Quick start

```bash
# Software internships in Canada posted in the last 24 hours
python3 find_canada_swe_interns.py

# Widen the window to the last 3 days
python3 find_canada_swe_interns.py --hours 72

# Only Fall roles, and also export to Excel
python3 find_canada_swe_interns.py --term fall --xlsx fall_jobs.xlsx
```

---

## Parameters

| Flag | Argument | Default | Description |
|------|----------|---------|-------------|
| `--hours` | number | `24` | Only show roles posted within this many hours. |
| `--term` | text | all terms | Season filter: `fall`, `winter`, `spring`, `summer`, or a specific cycle like `"winter 2027"`. |
| `--source` | `internships` \| `newgrad` | `internships` | Which feed to use. `newgrad` switches to full-time new-grad roles. |
| `--year` | number | auto-detect | Force a specific internship cycle year (e.g. `2027`). Normally left off so the active repo is detected automatically. |
| `--source-url` | URL | — | Override completely with a custom `listings.json` URL. |
| `--province` | code | all | Restrict to one province/territory, e.g. `ON`, `QC`, `BC`, `AB`. |
| `--include-ai` | flag | off | Also include `AI/ML/Data` category roles. |
| `--all-tech` | flag | off | Include every tech category (hardware, quant, product, etc.). |
| `--rescue-adjacent` | flag | off | Also catch frontend/backend/devops/cloud/SRE roles even when the feed mis-tags them under another category. Hardware/firmware titles stay excluded. |
| `--roles` | keywords | — | Comma-separated title keywords to narrow results, e.g. `--roles devops,backend,frontend`. |
| `--csv` | path | — | Also write results to a CSV file. |
| `--xlsx` | path | — | Also write results to a formatted Excel file (requires `openpyxl`). |
| `--json` | path | — | Also write the raw matching entries to a JSON file. |

You can combine any of these freely.

---

## How the filtering works

**Canada detection.** A role counts as Canadian if any of its locations contains
`Canada`/`CAN` or ends in a Canadian province code (`ON`, `QC`, `BC`, `AB`, `MB`,
`SK`, `NS`, `NB`, `NL`, `PE`, `NT`, `YT`, `NU`). Note that some roles list multiple
countries (e.g. "Remote in USA, Remote in Canada") — these are kept because they
are open to Canada.

**Internship vs. new-grad.** For the default internship source, the title must look
like an internship or co-op (`intern`, `co-op`, `coop`). This title check is
**skipped** when you pass `--source newgrad`, since full-time roles don't say
"intern."

**What counts as "software."** Roles are matched by the feed's **category** field,
not by requiring the literal words "software engineer" in the title. By default the
tool includes the `Software` and `Software Engineering` categories — which already
covers frontend, backend, full-stack, mobile, web, and most DevOps roles. Use
`--include-ai` to add `AI/ML/Data`, or `--all-tech` for everything.

**Catching mis-tagged roles.** The upstream feed occasionally files a software-
adjacent role (a frontend or cloud role, say) under `AI/ML/Data` or another
category. `--rescue-adjacent` recovers those by matching the job title against
software role keywords (frontend, backend, devops, SRE, full-stack, mobile, etc.)
while deliberately excluding hardware/firmware titles (embedded, FPGA, PCB...) so
you don't get electrical-engineering roles.

---

## Output

### Terminal

Each match prints the title, company, category, location, posting time (with how
many hours ago), sponsorship note, and the application link. If nothing matches
your window, the tool says so and suggests widening the search.

### CSV (`--csv`)

Columns: `title`, `company`, `category`, `locations`, `posted_utc`, `hours_ago`,
`sponsorship`, `url`.

### Excel (`--xlsx`)

A formatted worksheet with a frozen header row and filters enabled. Columns:

`Job Title` · `Company` · `Location` · `Category` · `Date Posted` ·
`Deadline to Apply` · `Description & Application Link` (a clickable hyperlink).

> **About the deadline column:** the data feed does **not** publish application
> deadlines, so this column is flagged "Not listed — check posting" rather than
> left misleadingly blank. Many internships are rolling ("until filled") and have
> no fixed deadline; for the rest, open the link to confirm. A cell note explains
> this in the file itself.

### JSON (`--json`)

The raw matching entries from the feed, useful if you want to post-process the data
yourself.

---

## Example recipes

```bash
# Last 48 hours, Ontario only
python3 find_canada_swe_interns.py --hours 48 --province ON

# Fall internships across Canada, broad net, exported to Excel
python3 find_canada_swe_interns.py --term fall --rescue-adjacent --xlsx fall.xlsx

# Only DevOps / backend internships in the last week
python3 find_canada_swe_interns.py --hours 168 --roles devops,backend --rescue-adjacent

# Full-time new-grad SWE roles in Canada, last 30 days, to CSV
python3 find_canada_swe_interns.py --source newgrad --hours 720 --csv newgrad.csv

# Include AI/ML internships too
python3 find_canada_swe_interns.py --include-ai --hours 72

# Target a future cycle explicitly (once it exists)
python3 find_canada_swe_interns.py --year 2027 --term "summer 2027"
```

---

## Notes and expectations

- **A strict 24-hour window is often sparse in off-peak months.** Summer-internship
  posting peaks roughly August–January, so in late spring/summer a 24h search may
  return few or zero results. Widen with `--hours 72` / `--hours 168`, or filter by
  the season you actually want with `--term`.
- **Future cycles fill in gradually.** Postings for an upcoming term don't appear
  until companies open them (usually a few months ahead), so `--term "summer 2027"`
  will stay thin until late in the prior year.
- **Category accuracy depends on the upstream feed.** Most roles are tagged well,
  but `--rescue-adjacent` is a title-based heuristic, not a guarantee.
- **Results reflect what the community/automated feed has logged.** It's broad and
  updated hourly, but no single source captures every posting everywhere.

---

## Suggested next steps (optional)

- **Schedule it.** Run it on a cron job or GitHub Action (e.g. daily) and have it
  append new postings to a running spreadsheet.
- **Add more sources.** Company ATS boards (Greenhouse, Lever, Ashby) expose public
  JSON endpoints and could be added to widen coverage beyond this feed.

---

## License / terms

This tool consumes a public, openly published data feed and does not scrape any
site that prohibits it. Always review individual companies' application pages and
terms before applying.

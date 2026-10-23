<p align="center"><a href="https://apify.com/DevilScrapes/bamboohr-jobs-scraper"><img src="assets/banner.svg" alt="BambooHR Jobs Scraper — Python example" width="100%"></a></p>

# BambooHR Jobs Scraper in Python: small-employer job data for analysis

<p align="center">
<a href="https://apify.com/DevilScrapes/bamboohr-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/bamboohr-jobs-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/bamboohr-jobs-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

BambooHR is an HR system used mostly by small and mid-sized organizations, from software firms to non-profits and arts institutions, and many of them publish vacancies on a careers page at `<company>.bamboohr.com/careers`. This repository is a Python client for the **[BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper)** on Apify: give it those subdomains and it returns one row per open job with department, employment type, city, state, remote flag and, if you ask for it, the posting date and the complete description text. The Actor does its work on Apify; your script saves `results.json` and `results.csv`, which load straight into pandas.

> **Actor:** [BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [bamboohr-jobs-scraper-javascript](https://github.com/devilscrapes/bamboohr-jobs-scraper-javascript)

## 🔥 What you get

- **Two depths of data**: the careers list supplies title, department, employment status, city, state and the remote flag; switch on `fetchFullDescription` and each posting also gains its status, country, posting date and full body.
- **Descriptions in two shapes**: `description_html` keeps the employer's markup for display, while `description_text` carries the same body with tags removed and whitespace collapsed, ready for search or keyword counts.
- **The cap also limits detail requests**: `maxJobsPerCompany` trims each company's list before descriptions are fetched, so a small cap keeps both the row count and the run time down.
- **Remote is three-valued**: `is_remote` is `true`, `false` or `null`, and `null` means the employer never said, which is not the same as on-site.
- **Input checked before any request**: bare subdomains and `*.bamboohr.com` URLs are accepted; a custom-domain URL or a malformed entry stops the run with an error that names the entry.
- **Unclaimed subdomains are not errors**: a subdomain that BambooHR redirects away from, or a company with no openings, simply yields zero rows.
- **Partial failures degrade, they do not drop rows**: temporary errors are retried with backoff, and if one posting's detail request still fails, the row is kept with its list fields and the detail columns left `null`.

## 🎯 Use cases

### Study hiring demand among smaller employers

Most labour-market samples lean towards large companies with big career portals. A list of BambooHR subdomains gives you the other side: department mix, employment types and locations for organizations a big-employer sample would miss.

### Mine descriptions for skills and benefits

With `fetchFullDescription` on, `description_text` is plain prose. Tokenise it in a notebook, count mentions of tools, certifications or perks, and compare the counts by department or by company.

### Track how long roles stay open

`date_posted` comes from each posting's detail record. Run the script weekly from cron and you can measure days open per role and see which departments keep seats empty the longest.

### Keep a sector watchlist

Analysts following, say, regional healthcare providers or cultural institutions can keep their subdomain list in a text file, rerun it on a schedule and append every CSV to a history table.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/bamboohr-jobs-scraper-python.git
cd bamboohr-jobs-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

The script adds flags, files and error handling, but the integration underneath fits in a handful of lines: start the Actor with your input, wait for it, and walk the run's dataset.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/bamboohr-jobs-scraper").call(run_input={'companies': ['adobe', 'nycballet'], 'maxJobsPerCompany': 5, 'fetchFullDescription': True, 'proxyConfiguration': {'useApifyProxy': True}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`input.json` uses the two subdomains from the Actor's own prefill, `adobe` and `nycballet`, with five jobs each and full descriptions switched on, so a test returns at most ten rows. Add subdomains to `companies`, or set `fetchFullDescription` to `false` when titles and locations are all you need and you would rather the run finished sooner.

```json
{
  "companies": [
    "adobe",
    "nycballet"
  ],
  "maxJobsPerCompany": 5,
  "fetchFullDescription": true,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `companies` | array | yes | `["adobe", "nycballet"]` | — |
| `maxJobsPerCompany` | integer | no | `100` | — |
| `fetchFullDescription` | boolean | no | `true` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

One row per posting. `job_id` is BambooHR's numeric id and is only unique inside one company, so key on `company` plus `job_id` when you combine employers. Five columns (`job_status`, `location_country`, `date_posted`, `description_html`, `description_text`) are filled only when descriptions are fetched.

| Field | Type | Always present |
|---|---|---|
| `company` | string | yes |
| `job_id` | integer | yes |
| `job_title` | string | yes |
| `job_url` | string | yes |
| `department` | string or null | no |
| `employment_type` | string or null | no |
| `job_status` | string or null | no |
| `location_city` | string or null | no |
| `location_state` | string or null | no |
| `location_country` | string or null | no |
| `is_remote` | boolean or null | no |
| `date_posted` | string or null | no |
| `description_html` | string or null | no |
| `description_text` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.0015 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

With pandas (`pip install pandas`), the snippet below converts `date_posted` into days open, labels the three remote states, prints a company-by-department table and saves roles open longer than 60 days to a CSV.

```python
import json

import pandas as pd

rows = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
rows["date_posted"] = pd.to_datetime(rows["date_posted"], errors="coerce")
rows["days_open"] = (pd.Timestamp.today().normalize() - rows["date_posted"]).dt.days
rows["remote"] = rows["is_remote"].map({True: "remote", False: "on-site"}).fillna("not stated")
rows["department"] = rows["department"].fillna("(none)")

table = rows.pivot_table(
    index=["company", "department"], columns="remote", values="job_id", aggfunc="count", fill_value=0
)
table["median_days_open"] = rows.groupby(["company", "department"])["days_open"].median()
print(table.to_string())

stale = rows[rows["days_open"] > 60].sort_values("days_open", ascending=False)
stale[["company", "job_title", "department", "date_posted", "days_open", "job_url"]].to_csv(
    "open-over-60-days.csv", index=False
)
```

## ❓ FAQ

### How do I find a company's BambooHR subdomain?

Look at the address of its job listings. If they open at `something.bamboohr.com/careers`, the subdomain is `something`. When the company shows jobs on its own domain, look for a BambooHR link or iframe on that page to find the underlying subdomain; the custom domain itself is not accepted.

### Why are `date_posted` and the descriptions `null`?

Either `fetchFullDescription` was off, or the detail request for that posting failed after its retries. In the second case the row is still written with its list fields intact, and the run log names the posting.

### Is there salary information?

Not as a column. The public careers data has no structured pay field, so none is returned. Some employers mention pay inside the description, which you can search for in `description_text`.

### Will one bad subdomain stop the batch?

A well-formed subdomain that does not exist gives zero rows and the batch carries on. A malformed entry, such as a custom-domain URL or text with spaces, is rejected during input validation before any request is sent, and the run fails with that entry named.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [BambooHR Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/bamboohr-jobs-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/bamboohr-jobs-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Workable Jobs Scraper](https://apify.com/DevilScrapes/workable-jobs-scraper) — the equivalent board export for employers that recruit through Workable.
- [Personio Jobs Scraper](https://apify.com/DevilScrapes/personio-jobs-scraper) — Personio career pages, read from Personio's public XML feed.
- [Teamtailor Jobs Scraper](https://apify.com/DevilScrapes/teamtailor-jobs-scraper) — career sites built on Teamtailor, with publish dates and descriptions.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with BambooHR.

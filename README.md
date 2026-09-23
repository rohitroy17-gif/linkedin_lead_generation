# LinkedIn Lead Generation Automation

An n8n-style workflow that scrapes LinkedIn job postings via Apify actors, enriches each listing with company and contact details, and appends the results to a Google Sheet as a running leads list.

## Overview

The workflow is triggered by a form submission (e.g. a search query or keyword), scrapes LinkedIn job listings through Apify, loops through the results in batches, enriches each item with additional data (via an HTTP request and a second Apify actor run), transforms the fields, merges the enriched branches back together, and writes everything to Google Sheets.

## Workflow Steps

1. **On form submission** — Trigger. Starts the workflow with 1 input item (likely a search term, job title, or location submitted via a form).

2. **Run an Actor and get dataset** — Calls an Apify Actor to scrape an initial set of LinkedIn job listings, returning 10 items.

3. **Loop Over Items** — Batches the 10 items into groups of 5 and iterates over them so downstream steps process manageable chunks.

4. **Run an Actor and get dataset1** — For each batch, runs a second Apify Actor (likely to pull deeper details per job/company, such as company profile data).

5. **HTTP Request** — Makes an external API call per item, probably to fetch additional enrichment data (e.g. company website, contact email lookup, or a similar service).

6. The enriched data then splits into two parallel branches:
   - **Code in JavaScript** — Custom JS logic to parse, clean, or reshape the HTTP response (5 items).
   - **Edit Fields** — Manually maps/renames fields such as company name, website, sector, experience level, and job apply URL (5 → 10 items).

7. **Merge** — Combines the two branches back into a single dataset by matching inputs.

8. **Append or update row in sheet** — Writes the final 10 rows to a Google Sheet, appending new rows or updating existing ones based on a key (likely email or company name).

## Output

The destination Google Sheet contains one row per lead, with these columns:

| Column | Description |
|---|---|
| Email address | Contact email for the company/lead |
| Company name | Name of the hiring company |
| Website | Company website URL |
| Sector | Industry/sector (e.g. Software Development, IT Services) |
| Experience | Seniority level of the job posting (e.g. Entry level, Associate, Mid-Senior, Not Applicable) |
| Job apply URL | Direct LinkedIn link to the job posting |

## Requirements

- **Apify** account with actor(s) configured for LinkedIn job scraping and company data extraction
- **Google Sheets** connection with edit access to the target spreadsheet
- An HTTP-accessible enrichment API (used in the HTTP Request step)
- A form trigger (e.g. n8n form, Typeform, or similar) to kick off each run

## Notes / Assumptions

- Exact configuration of the two Apify actor nodes, the HTTP Request endpoint, and the JavaScript code node isn't visible from the workflow diagram — descriptions above are inferred from node names, item counts, and the final sheet output. Update this section with the actual actor names/endpoints for full accuracy.
- The "Not Applicable" experience values suggest some listings don't specify a seniority level, which the pipeline passes through as-is.
- The Merge node's match/combine logic (e.g. by index or by a shared key) should be confirmed and documented once known.

## Customization

- Adjust the **Loop Over Items** batch size if Apify or the HTTP enrichment API has rate limits.
- Add a **deduplication step** before the sheet append if the same company/job may appear across multiple runs.
- Consider adding error handling (e.g. an IF/Error Trigger node) around the HTTP Request step in case the enrichment API fails for a given item.

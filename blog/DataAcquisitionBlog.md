---
title: "Data Acquisition Blog"
author: "Jillian Baker"
date: "2026-03-11"
format: html
---

# What Does It Take to Win Survivor? A Data-Driven Look at 20 Seasons of Contestants

Last semester, every Wednesday night my apartment turned into a little sanctuary. Friends from all corners of my life would pile in — a smorgasbord of different people — and we'd spend the night watching Survivor together, snacking on whatever treat we'd picked for the week. It was one of those rare, low-pressure rituals that helped everyone decompress mid-week. No agenda, just good company and great TV.

Watching such different people bond over the show made me wonder if different types of people actually play the game differently.

This got me thinking: *who actually does well on this show, and why?* Are there patterns in age, profession, or background that predict how far someone goes? As someone learning data science, I figured there was no better way to find out than to go get the data myself.

---

## The Motivating Question

**Does a contestant's age, profession, or hometown predict how far they go in Survivor?**

Survivor is equal parts social game, physical endurance, and strategic maneuvering. I wanted to know whether any measurable background characteristic correlates with success — or whether the game is truly anyone's to win.

---

## Is It Ethical to Scrape This Data?

Before writing a single line of code, I checked Wikipedia's [`robots.txt`](https://en.wikipedia.org/robots.txt) file to confirm that scraping is permitted. Wikipedia explicitly allows bots to access most of its content, and all contestant data on the site is publicly available and crowd-maintained. No login, no paywall, no personal or private information involved.

I also followed responsible scraping practices:
- Used a descriptive `User-Agent` header to identify my requests
- Scraped only what I needed (two specific tables)
- Made no repeated or rapid-fire requests

---

## Getting the Data

Wikipedia hosts a detailed page listing every [Survivor (US) contestant](https://en.wikipedia.org/wiki/List_of_Survivor_(American_TV_series)_contestants) by season, including their age, hometown, profession, and finish placement. I targeted seasons 31–50 (2016–2026), which gave a large enough sample while keeping the data relatively modern.

The core scraping logic is straightforward with `pandas` and `requests`:

```python
import requests
import pandas as pd
from io import StringIO

url = 'https://en.wikipedia.org/wiki/List_of_Survivor_(American_TV_series)_contestants'
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0'}

response = requests.get(url, headers=headers)
all_tables = pd.read_html(StringIO(response.text))

recent_seasons = all_tables[4]  # seasons 41–50
other_seasons  = all_tables[3]  # seasons 31–40
df = pd.concat([recent_seasons, other_seasons], ignore_index=True)
```

`pd.read_html()` does the heavy lifting — it detects and parses all HTML tables on the page into a list of DataFrames. From there it's just a matter of identifying which table indices correspond to the seasons you want and merging them.

---

## Cleaning & Transforming the Data

Raw Wikipedia data is messy. Here's what needed fixing:

- **Footnote symbols** (`◊`, `^`, `‡`) were embedded in contestant names and stripped with regex
- **Finish placements** were stored as strings like `"1st"`, `"Winner"`, `"Runner-up"` — standardized and converted to integers
- **Season names** (e.g., *"Survivor: David vs. Goliath"*) were mapped to their season numbers
- **Hometowns** were split into separate city and state columns, with Canadian entries flagged separately
- **Profession text** was freeform, so I wrote a keyword-based categorizer to group ~300 unique job titles into 11 broad categories (Business, Medical, Student, Legal, etc.)

One important derived column: **Finish Percentile**, which normalizes placement across seasons of different cast sizes:

```
FinishPct = (1 - (Finish - 1) / (SeasonSize - 1)) × 100
```

This means 100 = winner and 0 = first boot, regardless of how many people were cast — making seasons directly comparable.

---

## Dataset Overview

| Feature | Description |
|---|---|
| `Name` | Contestant name |
| `Season` | Season number (31–50) |
| `Age` | Age at time of filming |
| `Hometown` | Contestant's listed hometown |
| `State` | Extracted U.S. state |
| `Profession` | Raw profession text from Wikipedia |
| `ProfCategory` | Grouped profession category |
| `Finish` | Final placement (1 = winner) |
| `SeasonSize` | Number of contestants that season |
| `FinishPct` | Normalized finish (0–100, higher = better) |
| `AgeGroup` | Binned age category (e.g., 25–29, 30–34) |

**Final dataset: ~380 rows × 11 columns** after cleaning and deduplication.

A few quality notes worth knowing:
- Returning players appear as separate rows since each appearance is independent
- Some hometown entries lacked a U.S. state and were excluded from state-level maps
- Profession text is self-reported and varies in specificity — the category groupings are a useful approximation, not a perfect taxonomy

---

## What I Found

A few highlights from the analysis:

- **Age doesn't predict success.** The relationship between age and finish percentile is essentially flat. Survivor really can be won at any age.
- **Business/consulting professionals are the most represented group** — dealmaking and relationship management are core Survivor skills, so this tracks.
- **Average contestant age has crept upward** in recent seasons, likely reflecting a shift toward casting returning players.
- **Los Angeles, New York, and Brooklyn** dominate the hometown map — partly population size, partly proximity to entertainment casting pipelines.

Here's a look at two of the standout visuals. Note that both charts are filtered to groups with sufficient sample sizes to keep the results meaningful:

**Average Performance by State** *(U.S. states with 5+ contestants only)*
<iframe 
  src="/images/state_performance_map.html" 
  width="100%" 
  height="500px" 
  frameborder="0">
</iframe>

**Average Performance by Profession** *(profession categories with 5+ contestants only)*
<iframe 
  src="/images/profession_performance.html" 
  width="100%" 
  height="500px" 
  frameborder="0">
</iframe>

---

## Resources & Further Reading

- 📄 [Wikipedia: List of Survivor Contestants](https://en.wikipedia.org/wiki/List_of_Survivor_(American_TV_series)_contestants) — the data source
- 🤖 [Wikipedia robots.txt](https://en.wikipedia.org/robots.txt) — confirms scraping is permitted
- 📦 [`pandas.read_html()` docs](https://pandas.pydata.org/docs/reference/api/pandas.read_html.html) — the workhorse of this project
- 📊 [Plotly Express docs](https://plotly.com/python/plotly-express/) — used for all visualizations
- 💻 [Project GitHub Repo](https://github.com/jilliangbaker97/Data_Acquisition_Blog/tree/main) — full code and dataset

---

*Whether or not you have a Wednesday night Survivor crew of your own, I hope this gives you a useful starting point for scraping and analyzing structured data from Wikipedia. The same pattern — `requests` → `read_html` → clean → analyze — transfers easily to almost any Wikipedia list page.*

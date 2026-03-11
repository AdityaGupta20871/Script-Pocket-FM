# Pocket FM — Amazon Kindle Scraper Pipeline

AI-powered scraper for Amazon Kindle Paranormal Romance bestsellers.
Extracts Top 100 books across 2 paginated pages into a structured Excel dataset.

---

## Architecture

```
Amazon URL
  → Playwright (headless Chromium, US-spoofed)
    → BeautifulSoup (list parsing — rank, title, author, rating, price, URL)
      → Groq LLM / Llama 3.1 (detail extraction — description, publisher, pub date)
        → Excel (.xlsx)
```

**Why LLM-as-parser?**  
No brittle CSS selectors. LLM extracts fields semantically from raw text.
When Amazon changes their UI, the pipeline self-heals.

---

## Setup

### 1. Create virtual environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Mac / Linux
python3 -m venv venv
source venv/bin/activate
```

### 2. Install dependencies
```bash
pip install playwright beautifulsoup4 groq openpyxl
python -m playwright install chromium
```

### 3. Add your Groq API key
Open `scraper_v6.py` and set your key on line 16:
```python
GROQ_API_KEY = "YOUR_GROQ_API_KEY_HERE"
```
Get a free key at: https://console.groq.com

### 4. Run
```bash
python scraper_v6.py
```

---

## Output

| File | Description |
|------|-------------|
| `output/kindle_paranormal_romance_YYYYMMDD_HHMM.xlsx` | Main dataset — 100 books, 10 columns |
| `output/checkpoint_part1.json` | Intermediate save after list parsing |
| `output/checkpoint_final.json` | Final data before Excel export |
| `output/debug_page_1.html` | Saved HTML of page 1 (for debugging) |
| `output/debug_page_2.html` | Saved HTML of page 2 |

---

## Dataset Columns

| Column | Description |
|--------|-------------|
| Rank | 1–100 (across both pages) |
| Title | Full book title |
| Author | Author name |
| Rating | Star rating (e.g. 4.5) |
| Num Reviews | Total review count |
| Price | Listed price |
| URL | Clean Amazon product URL (no tracking params) |
| Description | Book summary (LLM-extracted, max 400 chars) |
| Publisher | Publisher name (LLM-extracted) |
| Publication Date | YYYY-MM-DD (LLM-extracted) |

---

## Known Issues & Fixes Applied

| Issue | Fix |
|-------|-----|
| Amazon serving Indian content on Indian IPs | Spoof US geolocation + USD cookie in Playwright |
| Wrong category being scraped | Corrected URL: node ID 6190484011 (Paranormal) not 6361470011 (Dystopian) |
| Duplicate ranks across pages | Page 2 items get rank_offset=+50 applied |
| LLM hallucinating book titles | LLM only sees extracted text sections, never full page HTML |
| Review count picking wrong numbers | 3-tier extraction: aria-label > customerReviews href > fallback span |
| Groq token limit exceeded | Input trimmed to description + details sections, max 3000 chars |

---

## Scaling

To run at scale across multiple categories:

- Parameterise `BASE_URL` into a config list
- Use Celery + Redis task queue for distributed processing
- Add rotating residential proxies (ScraperAPI / Bright Data)
- Cache HTML by URL + date hash to reduce redundant fetches
- Store results in PostgreSQL with `scraped_at` timestamp for trend tracking

---

## Files

```
pipeline/
├── scraper_v6.py          ← Main script (run this)
├── README.md              ← This file
└── output/
    ├── kindle_paranormal_romance_*.xlsx
    ├── checkpoint_part1.json
    ├── checkpoint_final.json
    ├── debug_page_1.html
    └── debug_page_2.html
```

---

**Author:** Aditya Gupta  
**Assignment:** Pocket FM — US Romantasy Intern  
**Date:** March 2026

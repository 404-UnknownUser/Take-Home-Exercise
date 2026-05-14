# Take-Home Exercise — Ana Pereira
**AI Engineering Internship — Strypes Portugal, Summer 2026**

---

## Part 1 — Build a Small Data Fetcher (Python)

### How to run

1. Install the only dependency:
   ```bash
   pip install requests
   ```

2. Run the script:
   ```bash
   python fetch_hn_jobs.py
   ```

3. Results are printed to the terminal and saved to `hn_jobs.json`.

### Script (`fetch_hn_jobs.py`)

```python
import requests
import json
import html
import re
from datetime import datetime
from typing import Union

# Constants
ALGOLIA_URL = "https://hn.algolia.com/api/v1/search"
FIREBASE_URL = "https://hacker-news.firebaseio.com/v0/item/{}.json"
HN_ITEM_URL = "https://news.ycombinator.com/item?id={}"
OUTPUT_FILE = "hn_jobs.json"
NUM_JOBS = 10


def get_latest_hiring_thread_id():
    """Find the most recent 'Ask HN: Who is Hiring?' thread via Algolia."""
    params = {"query": "Ask HN: Who is Hiring?", "tags": "ask_hn", "hitsPerPage": 1}
    response = requests.get(ALGOLIA_URL, params=params, timeout=10)
    response.raise_for_status()
    hits = response.json().get("hits", [])
    if not hits:
        raise ValueError("No 'Who is Hiring?' thread found.")
    return hits[0]["objectID"]


def get_item(item_id):
    """Fetch a single HN item from the Firebase API."""
    response = requests.get(FIREBASE_URL.format(item_id), timeout=5)
    response.raise_for_status()
    return response.json() or {}


def parse_job(item):
    """
    Extract title, date, and URL from a raw job comment.
    Job posts are HTML comments — we strip tags and take the first line
    as the title, since comments have no 'title' field unlike top-level posts.
    """
    plain_text = re.sub(r"<[^>]+>", " ", html.unescape(item.get("text", "")))
    first_line = next((l.strip() for l in plain_text.splitlines() if l.strip()), "N/A")

    return {
        "title": first_line[:120],
        "posting_date": datetime.fromtimestamp(item["time"]).strftime("%Y-%m-%d") if item.get("time") else "N/A",
        "url": HN_ITEM_URL.format(item["id"]),
    }


def main():
    # Step 1 — find the latest monthly hiring thread
    thread_id = get_latest_hiring_thread_id()
    thread = get_item(thread_id)
    child_ids = thread.get("kids", [])[:NUM_JOBS]  # 'kids' are the individual job posts

    # Step 2 — fetch and parse each job posting
    jobs = []
    for item_id in child_ids:
        try:
            job = parse_job(get_item(item_id))
            jobs.append(job)
            print("[{}] {}\n    {} — {}\n".format(len(jobs), job["title"], job["posting_date"], job["url"]))
        except Exception as e:
            print("Skipping item {}: {}".format(item_id, e))

    # Step 3 — save to JSON
    with open(OUTPUT_FILE, "w", encoding="utf-8") as f:
        json.dump(jobs, f, indent=4, ensure_ascii=False)
    print("Saved {} jobs to {}".format(len(jobs), OUTPUT_FILE))


if __name__ == "__main__":
    main()
```

---

## Part 2 — Design an AI Agent

*(Written answers in the submitted .docx)*

---

## Part 3 — Challenge the Brief

*(Written answers in the submitted .docx)*

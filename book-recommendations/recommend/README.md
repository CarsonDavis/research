# Book Recommendation Pipeline

A tool for generating personalized book recommendations. The LLM orchestrates the pipeline by making decisions and calling CLI commands; Python handles mechanical tasks like searching and scraping.

## Pipeline Overview

```
┌─────────────────────────────────────────────────────────────┐
│  1. CANDIDATE GENERATION                                     │
│     LLM generates search queries → Python executes searches │
│     LLM reviews results → adds promising candidates          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  2. DATA COLLECTION                                          │
│     Python scrapes Goodreads for metadata and reviews        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  3. ANALYSIS                                                 │
│     LLM reads reviews, matches against user profile          │
│     LLM sets recommendation tier and reasoning               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  4. OUTPUT                                                   │
│     Python generates recommendations.md report               │
└─────────────────────────────────────────────────────────────┘
```

---

## CLI Commands

### Check Status

```bash
uv run python -m recommend status
```

Shows pipeline state: how many candidates exist, which need scraping, which need analysis.

### List Candidates

```bash
uv run python -m recommend list
```

Shows all candidates with their current status (pending, has metadata, has reviews).

### Add Candidate

```bash
uv run python -m recommend add <goodreads_id> "<title>" "<author>" --note "<source>"
```

Manually add a book to the candidate pool.

**Examples:**
```bash
# Add a book similar to a favorite
uv run python -m recommend add 17934530 "Annihilation" "Jeff VanderMeer" --note "Similar to Piranesi"

# Add unread book by favorite author
uv run python -m recommend add 18007564 "The Just City" "Jo Walton" --note "By favorite author"

# Add from style-based search
uv run python -m recommend add 34051011 "The Poppy War" "R.F. Kuang" --note "Fantasy with hard magic"
```

### Scrape Candidates

```bash
# Scrape metadata and reviews for top N candidates
uv run python -m recommend scrape-candidates --top 20

# Scrape only metadata (faster)
uv run python -m recommend scrape-candidates --metadata-only --top 50
```

### List Ready for Analysis

```bash
# Show books that have reviews but need analysis
uv run python -m recommend list-ready

# Output just IDs (for scripting)
uv run python -m recommend list-ready --ids-only
```

### Check If Already Read

```bash
uv run python -m recommend check-read "<title>" "<author>"
```

Returns "YES - already read" (exit code 1) or "NO - not in read list" (exit code 0). Uses fuzzy matching for title/author variations.

### Check Series / Find Book 1

```bash
# Check if a book is part of a series and find Book 1
uv run python -m recommend check-series <goodreads_id>

# Automatically add Book 1 to library if found
uv run python -m recommend check-series <goodreads_id> --add
```

Returns Book 1's ID, title, and author if the given book is Book 2+. Returns "Not part of a series, or already Book 1" otherwise.

### Generate Report

```bash
uv run python -m recommend generate-report
```

Creates `output/recommendations.md` from all analyzed candidates.

---

## Important Notes

### Series Resolution

The `add` command does **not** automatically resolve to Book 1. Use `check-series` to handle this:

```bash
# Check if book is Book 2+ and get Book 1 info
uv run python -m recommend check-series <id>

# Or check and add Book 1 in one step
uv run python -m recommend check-series <id> --add
```

### Already-Read Filtering

The `add` command does **not** check against already-read books. Use `check-read` first:

```bash
uv run python -m recommend check-read "Book Title" "Author Name"
```

### Duplicate Handling

Adding the same Goodreads ID twice will merge the sources (useful for tracking that a book appeared in multiple searches).

---

## Orchestration Workflow

### Phase 1: Generate Candidates

Read `user_profile.md` to understand preferences, then generate search queries.

**For S-tier books**, search for similar reads:
```
"books similar to Piranesi"
"books like Children of Time"
"if you liked Roadside Picnic"
```

**For favorite authors**, find their other works:
```
"Ted Chiang books"
"Ursula K. Le Guin bibliography"
```

**For style preferences**, search by attributes:
```
"fantasy immersive worldbuilding"
"science fiction unreliable narrator"
"books with creeping revelation"
```

For each search, review results and add promising candidates:
```bash
uv run python -m recommend add <id> "<title>" "<author>" --note "<source>"
```

### Phase 2: Collect Data

Once candidates are added, scrape their data:
```bash
uv run python -m recommend scrape-candidates --top 30
```

Check progress:
```bash
uv run python -m recommend status
```

### Phase 3: Analyze Candidates

List candidates ready for analysis:
```bash
uv run python -m recommend list-ready
```

For each candidate, read the book data:

```bash
uv run python -c "
from recommend.library import BookLibrary
import json
library = BookLibrary()
book = library.get_book('<goodreads_id>')
print(json.dumps(book, indent=2))
"
```

Compare reviews against user profile:
- Do praised elements match what user loves?
- Do criticized elements match user's hate patterns?
- Any dealbreakers present?

Set the recommendation:

```bash
uv run python -c "
from recommend.library import BookLibrary
library = BookLibrary()
library.set_recommendation(
    '<goodreads_id>',
    tier='high',  # high, medium, try, or skip
    reasoning='Matches user preference for immersive worldbuilding and creeping revelation',
    dealbreakers=[]
)
"
```

### Phase 4: Generate Output

```bash
uv run python -m recommend generate-report
```

---

## Key Files

| File | Purpose |
|------|---------|
| `user_profile.md` | Reader preferences (loves, hates, genre affinities) |
| `data/read_books_with_genres.csv` | Already-read books (for filtering) |
| `library/` | Cached data (candidates, metadata, reviews) |
| `output/recommendations.md` | Final recommendations |

---

## Recommendation Tiers

| Tier | Criteria |
|------|----------|
| **high** | Multiple love-list matches, no hate-list matches, high genre affinity |
| **medium** | Some love-list matches, minor concerns, good genre fit |
| **try** | Few matches but worth considering, or mixed signals |
| **skip** | Dealbreaker present, multiple hate-list matches, low genre affinity |

---

## Example Session

```bash
# 1. Check current state
uv run python -m recommend status

# 2. Add candidates from searches (verify not already read first)
uv run python -m recommend add 17934530 "Annihilation" "Jeff VanderMeer" --note "Similar to Piranesi"
uv run python -m recommend add 13642710 "A Fire Upon the Deep" "Vernor Vinge" --note "Epic SF worldbuilding"
uv run python -m recommend add 77197 "The Dispossessed" "Ursula K. Le Guin" --note "Favorite author"

# 3. Scrape data
uv run python -m recommend scrape-candidates --top 10

# 4. List books ready for analysis
uv run python -m recommend list-ready

# 5. [LLM reads each book's data and sets recommendations]

# 6. Generate report
uv run python -m recommend generate-report
```

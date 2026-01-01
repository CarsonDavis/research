# Goodreads Tools

Utilities for processing Goodreads library exports.

## Installation

```bash
cd book-recommendations
uv sync
```

## Utilities

### 1. Clean Export

Converts a raw Goodreads CSV export into a clean format with only essential columns.

```bash
uv run python -m goodreads clean <input.csv> <output.csv>
```

**Input**: Raw Goodreads export (all columns)
**Output**: Clean CSV with columns:
- `goodreads_id` - Book ID for URL construction
- `title` - Book title
- `author` - Primary author
- `my_rating` - Your rating (0-5)
- `my_review` - Your review text

Only books marked as "read" are included.

### 2. Add Genres

Scrapes genre information from Goodreads and adds it to a clean export.

```bash
uv run python -m goodreads genres <input.csv> <output.csv>
```

**Features**:
- Async HTTP requests with configurable concurrency
- Automatic retries with exponential backoff
- Progress bar with ETA
- Graceful failure handling (saves progress on interrupt)

**Options**:
- `--concurrency N` - Number of parallel requests (default: 5)
- `--retry N` - Max retry attempts (default: 3)

**Output**: Same CSV with added `genres` column (pipe-separated list).

## URL Format

Goodreads book pages: `https://www.goodreads.com/book/show/{goodreads_id}`

## Project Structure

```
book-recommendations/
├── data/
│   ├── goodreads_export_raw.csv   # Original Goodreads export
│   ├── read_books.csv             # Cleaned (read books only)
│   └── read_books_with_genres.csv # With scraped genres
├── goodreads/
│   ├── __init__.py
│   ├── clean.py      # Export conversion
│   ├── scrape.py     # Genre scraping (async)
│   └── cli.py        # Command-line interface
└── README.md
```

## Example Workflow

```bash
# Clean raw export
uv run python -m goodreads clean data/goodreads_export_raw.csv data/read_books.csv

# Add genres
uv run python -m goodreads genres data/read_books.csv data/read_books_with_genres.csv
```

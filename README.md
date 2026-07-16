# Newsly

An automated pipeline that scrapes AI news from YouTube channels, OpenAI, and Anthropic, processes the content into readable digests, and emails a daily summary — all in one scheduled run.

## What It Does

Every run of the pipeline performs five steps:

1. **Scrape** — Pulls recent content from configured YouTube channels (with transcripts), the OpenAI blog, and the Anthropic blog, filtered to a configurable time window (default: last 24 hours).
2. **Process Anthropic articles** — Converts scraped Anthropic markdown into clean, structured content.
3. **Process YouTube transcripts** — Extracts and prepares video transcripts for summarization.
4. **Generate digests** — Creates concise digests/summaries for each article and video using an LLM.
5. **Send email** — Compiles the top digests into a single email and sends it to the configured recipient.

Results from every stage (counts scraped, processed, failed, digested, email status) are logged and returned as a structured summary.

## Tech Stack

- **Language:** Python 3.12+
- **Database:** PostgreSQL (via SQLAlchemy + psycopg2)
- **LLM:** OpenAI API (for digest generation)
- **Scraping/Parsing:** `requests`, `beautifulsoup4`, `feedparser`, `docling`, `markdownify`, `youtube-transcript-api`
- **Config/Validation:** `pydantic`, `python-dotenv`
- **Package management:** [`uv`](https://github.com/astral-sh/uv)

## Project Structure

```
ai-news-aggregator/
├── app/
│   ├── config.py              # Source configuration (e.g. YouTube channel IDs)
│   ├── daily_runner.py        # Orchestrates the full daily pipeline
│   ├── runner.py               # Runs all scrapers and persists raw results
│   ├── scrapers/               # YouTube / OpenAI / Anthropic scrapers
│   ├── services/               # Processing: Anthropic markdown, YouTube transcripts, digests, email
│   ├── database/               # Repository layer for persistence
│   └── agent/                  # LLM-driven digest/summary generation
├── main.py                     # Entry point for running the pipeline
├── pyproject.toml
├── uv.lock
└── example.env                 # Template for required environment variables
```

## Setup

1. **Clone the repository and install dependencies:**
   ```bash
   uv sync
   ```

2. **Configure environment variables:**
   Copy `example.env` to `.env` and fill in your credentials:
   ```bash
   cp example.env .env
   ```
   ```
   OPENAI_API_KEY=

   MY_EMAIL=
   APP_PASSWORD=

   POSTGRES_USER=postgres
   POSTGRES_PASSWORD=postgres
   POSTGRES_DB=ai_news_aggregator
   POSTGRES_HOST=localhost
   POSTGRES_PORT=5432
   ```

3. **Set up PostgreSQL** (locally or via Docker) and ensure the database defined above exists.

4. **Configure sources:**
   Edit `app/config.py` to add or remove YouTube channel IDs to track:
   ```python
   YOUTUBE_CHANNELS = [
       "UCawZsQWqfGSbCI5yjkdVkTA",  # Matthew Berman
   ]
   ```

## Usage

Run the full daily pipeline:

```bash
python main.py [hours] [top_n]
```

- `hours` (optional, default `24`) — how far back to look for new content.
- `top_n` (optional, default `10`) — number of articles/videos to include in the email digest.

Example:
```bash
python main.py 48 15
```

You can also run the pipeline programmatically:

```python
from app.daily_runner import run_daily_pipeline

result = run_daily_pipeline(hours=24, top_n=10)
print(result["success"])
```

## Scheduling

For automated daily runs, schedule `main.py` (or `daily_runner.py`) via cron, a task scheduler, or a Docker-based scheduler.

## License

Add your license here.

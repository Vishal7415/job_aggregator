# Job Aggregator (Django)

Aggregate job listings from LinkedIn, Indeed, and Naukri into one place, with optional resume-based matching. Built with Django 5, Bootstrap 5, Crispy Forms, and Django Allauth.

## Features

- **Aggregated Search**: Queries multiple sources via `scrapers/` and combines results.
- **Resume Matching (optional)**: Upload PDF/TXT; jobs are scored by token overlap.
- **Per-source Stats**: Summary counts by provider.
- **User Accounts**: Registration, login, profile via `users/` and Allauth.
- **History**: Saves your search queries.
- **Responsive UI**: Bootstrap 5, icons, and clean templates.

## Project structure

- **`core/`**: Django settings, URLs, WSGI/ASGI
- **`jobs/`**: Models, forms, views, URLs, and job templates
- **`scrapers/`**: `linkedin_scraper.py`, `indeed_scraper.py`, `naukri_scraper.py`, and `job_aggregator.py`
- **`users/`**: Auth-related views, forms, URLs (profile, login, register)
- **`templates/`**: Page templates, including `base/` and `jobs/`
- **`static/`**: CSS and assets
- **`manage.py`**: Django management entrypoint

## Requirements

- Python 3.12+
- pip / venv

## Quick start

```bash
git clone https://github.com/Vishal7415/job_aggregator.git
cd job_aggregator
python3 -m venv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
# optionally edit .env (SECRET_KEY, GOOGLE keys, etc.)
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

Open http://127.0.0.1:8000 and register/login to start searching.

## Environment variables

See `.env.example` for a full list. Key values:

- **`SECRET_KEY`**: Django secret key (change in production)
- **`DEBUG`**: `True/False`
- **`ALLOWED_HOSTS`**: Comma-separated (e.g., `localhost,127.0.0.1`)
- **`CSRF_TRUSTED_ORIGINS`**: Comma-separated origins for POSTs (add preview proxy if needed)
- **`GOOGLE_CLIENT_ID`**, **`GOOGLE_CLIENT_SECRET`**: For Google login via Allauth (optional)
- **`TIME_ZONE`**: Defaults to `Asia/Kolkata`

## How it works

Request flow:

```text
Home (/)
  -> jobs/views.py: home() renders templates/base/home.html (search form)
  -> POST /jobs/search/ -> jobs/views.py: search_jobs()
     - Validates form, saves SearchHistory
     - Calls scrapers/job_aggregator.py: JobAggregator.search_jobs()
       - Invokes LinkedIn/Indeed/Naukri scrapers
       - Normalizes job dicts
     - Saves jobs to jobs/models.py: Job
     - Renders templates/jobs/search_results.html
```

Resume matching (optional): If you upload a PDF/TXT, the view extracts tokens and computes overlap with each job’s text to form a "Resume Matching Jobs" section.

## Scrapers

- **LinkedIn**: `scrapers/linkedin_scraper.py`
- **Indeed**: `scrapers/indeed_scraper.py`
- **Naukri**: `scrapers/naukri_scraper.py`
- **Aggregator**: `scrapers/job_aggregator.py` (mixes, shuffles, and limits results; provides stats)

Notes:

- Sites may block scraping (403). Each scraper provides deterministic fallback sample data so the UI remains functional.
- Rate limiting via `time.sleep(1)` per request.
- For production, prefer official APIs where available.

## Running tips

- If you use a preview proxy (non-8000 port), add it to `CSRF_TRUSTED_ORIGINS` in `core/settings.py` or `.env`.
- Default login/logout/register/profile routes are under `/users/`.
- Static files are served in development via Django.

## Troubleshooting

- **403 CSRF origin check**: Ensure your preview origin is in `CSRF_TRUSTED_ORIGINS`.
- **Port already in use**: Use another port: `python manage.py runserver 0.0.0.0:8001`.
- **Django not installed**: Create venv and `pip install -r requirements.txt`.
- **No jobs shown**: If scrapers are blocked, fallback jobs should appear; check console logs.

## Deployment

- Use a production WSGI/ASGI server (gunicorn/uvicorn) and a reverse proxy (nginx) with static files collected via `collectstatic`.
- Set `DEBUG=False`, configure `ALLOWED_HOSTS` and `CSRF_TRUSTED_ORIGINS` properly.
- Provision environment variables for social auth if needed.

## Security

- `.env` and `db.sqlite3` are included for local dev only. Do not commit sensitive keys in production.
- Review robots/ToS of target sites. Avoid violating anti-scraping rules.

## License

MIT (or update as desired).
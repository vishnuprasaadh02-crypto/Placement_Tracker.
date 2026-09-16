# Placement Tracker

A full-stack CRUD web application for tracking internship and placement
applications — company, role, status, deadlines, and notes — built to the
CRUD Web Application SOP (React + Django REST Framework + SQLite).

Live use case: keep every internship/placement application (Wishlist →
Applied → Online Assessment → Interview → Offer/Rejected) in one dashboard
instead of scattered notes.

## 1. Problem Statement

Students applying to multiple internships/placements lose track of which
companies they've applied to, current round, and upcoming deadlines. This
app centralizes that into a single searchable, filterable dashboard.

## 2. Objectives

- Track applications through their full lifecycle with clear status stages
- Full CRUD via a REST API, consumed by a React frontend
- Server-side + client-side validation
- Search and filter by company, role, notes, and status

## 3. Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 (Vite), plain CSS |
| Backend | Django 5 + Django REST Framework |
| Database | SQLite (dev) — swappable to PostgreSQL/MySQL |
| API Testing | Postman collection (`docs/Placement_Tracker.postman_collection.json`) |
| Version Control | Git / GitHub |

## 4. System Architecture

```
User → React (Vite) frontend  →  REST API (JSON)  →  Django REST Framework
                                                       ↓ ORM
                                                    SQLite database
```

- Frontend runs on `http://localhost:5173`
- Backend runs on `http://localhost:8000`
- CORS is enabled on the backend for the Vite dev origin

## 5. Data Model (ER Summary)

Single entity: **Application**

| Field | Type | Notes |
|---|---|---|
| id | AutoField (PK) | |
| company | CharField(120) | required |
| role | CharField(120) | required |
| application_type | Choice: Internship / Full-Time / Hackathon | |
| status | Choice: Wishlist / Applied / Online Assessment / Interview / Offer / Rejected / Withdrawn | |
| applied_date | Date | nullable |
| deadline | Date | nullable, must be ≥ applied_date |
| job_link | URL | optional |
| notes | Text | optional |
| created_at / updated_at | DateTime | auto |

Constraint: unique `(company, role)` pair — prevents duplicate entries for
the same role at the same company.

## 6. REST API Reference

Base URL: `/api/`

| Operation | Method | Endpoint | Notes |
|---|---|---|---|
| Create | POST | `/applications/` | Validates required fields |
| List | GET | `/applications/` | Supports `?search=`, `?status=`, `?application_type=`, `?ordering=` |
| Retrieve | GET | `/applications/{id}/` | |
| Update | PUT/PATCH | `/applications/{id}/` | |
| Delete | DELETE | `/applications/{id}/` | |
| Dashboard stats | GET | `/applications/stats/` | Counts per status |

Example response (`GET /api/applications/1/`):
```json
{
  "id": 1,
  "company": "Microsoft",
  "role": "SWE Intern",
  "application_type": "INTERNSHIP",
  "application_type_display": "Internship",
  "status": "APPLIED",
  "status_display": "Applied",
  "applied_date": "2026-09-01",
  "deadline": null,
  "job_link": "",
  "notes": "",
  "created_at": "2026-09-17T00:23:58.207191+05:30",
  "updated_at": "2026-09-17T00:23:58.207230+05:30"
}
```

## 7. Validation

- **Client-side**: required fields, URL format for job link, deadline ≥ applied date
- **Server-side** (still enforced even if the frontend is bypassed, e.g. via Postman):
  - `company` / `role` cannot be blank
  - `deadline` cannot precede `applied_date`
  - duplicate `(company, role)` pairs are rejected
  - model-level `clean()` requires an applied date once status reaches "Applied"

## 8. Setup & Execution

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser   # optional, for /admin/
python manage.py runserver
```
Backend now serves the API at `http://127.0.0.1:8000/api/` and the Django
admin at `http://127.0.0.1:8000/admin/`.

### Frontend
```bash
cd frontend
npm install
cp .env.example .env    # adjust VITE_API_BASE_URL if needed
npm run dev
```
Frontend runs at `http://localhost:5173`.

Run both servers at the same time (two terminals) to use the full app.

## 9. Testing

A ready-made Postman collection is at
`docs/Placement_Tracker.postman_collection.json` — import it into Postman
and run against the local backend. It covers:

- Create with valid data
- Create with missing required field (expects `400`)
- Retrieve, update (PATCH), delete
- Delete on a non-existent id (expects `404`)
- Search and filter
- Dashboard stats endpoint

Manual verification already performed during development:
- `POST /api/applications/` → `201`, record persisted
- `GET /api/applications/` → returns the created record
- `PATCH /api/applications/{id}/` → status updated correctly
- `DELETE /api/applications/{id}/` → record removed, confirmed via subsequent `GET`
- `python manage.py check` → 0 issues
- `npm run build` → frontend builds with no errors

## 10. Security Notes

- No secrets are committed; `SECRET_KEY` and `DEBUG` are read from environment
  variables with safe dev-only fallbacks (see `backend/placement_tracker/settings.py`)
- `.gitignore` excludes `db.sqlite3`, `node_modules/`, `dist/`, and `.env`
- All queries go through the Django ORM (parameterized — no raw SQL)
- CORS is restricted to the local dev origin, not wide open in production

## 11. Challenges & Solutions

- **Deadline-before-applied-date bugs**: solved with validation at both the
  serializer and model level, so it's enforced no matter which client calls the API.
- **Stale UI after CRUD actions**: the frontend refetches the list and stats
  together after every create/update/delete instead of guessing new state.

## 12. Future Enhancements

- User accounts/authentication (per-student login instead of a single shared list)
- Reminders/notifications for upcoming deadlines
- CSV export of applications
- Kanban-style drag-and-drop board by status

## 13. Publishing to GitHub

From the project root (this folder):
```bash
git init
git add .
git commit -m "Initial commit: Placement Tracker CRUD app"
git branch -M main
git remote add origin https://github.com/<your-username>/placement-tracker.git
git push -u origin main
```
Create the empty repository first at github.com/new (no README/gitignore
selected there, since this project already has both), then run the commands
above.

## 14. Project Structure

```
placement-tracker/
├── backend/
│   ├── manage.py
│   ├── requirements.txt
│   ├── placement_tracker/      # settings, urls, wsgi/asgi
│   └── tracker/                # models, serializers, views, urls, admin
├── frontend/
│   ├── src/
│   │   ├── components/         # StatusBadge, StatCards, ApplicationForm, ApplicationTable
│   │   ├── App.jsx
│   │   ├── api.js
│   │   └── index.css
│   └── package.json
├── docs/
│   └── Placement_Tracker.postman_collection.json
└── README.md
```

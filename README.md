# Job Portal

A server-rendered job portal built with Django. Users can register as job seekers
or employers, browse job listings, post and manage jobs, and submit applications.
The project uses Django templates and session authentication; it is not currently
a JSON/REST API.

## Features

- Browse job listings and view job details without signing in.
- Register an account with a username, email address, password, and role.
- Sign in and sign out using Django's session-based authentication.
- Create job listings as a signed-in user.
- Edit or delete only job listings owned by the signed-in user.
- Apply to listings and see whether an application has already been submitted.
- View your own job listings and job applications.
- Manage users, jobs, and applications in Django admin.

## Technology

- Python
- Django
- SQLite for local development
- Django templates for the web interface

## Requirements

- Python compatible with the Django version installed for the project.
- Django. The development environment used for this project has Django 6.1.3.

No dependency manifest is currently included. Install Django in a virtual
environment if it is not already available:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install Django
```

On Windows, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

## Setup

Run these commands from the directory containing `manage.py`:

```bash
python manage.py migrate
python manage.py check
python manage.py runserver
```

Open [http://127.0.0.1:8000/](http://127.0.0.1:8000/) in a browser.

To create an administrator account for `/admin/`:

```bash
python manage.py createsuperuser
```

The local SQLite database is stored in `db.sqlite3`. It is ignored by Git; run
migrations to initialize it in a new environment.

## Routes

The application returns HTML pages and redirects, rather than JSON responses.
Forms use standard browser submissions and Django CSRF protection.

| Method | Path | Description | Access |
|---|---|---|---|
| GET | `/` | Browse all jobs | Public |
| GET | `/jobs/<id>/` | View a job and application status | Public |
| GET, POST | `/jobs/create/` | Display and submit the new-job form | Sign-in required |
| GET, POST | `/jobs/<id>/update/` | Display and submit the edit form | Owner only |
| GET, POST | `/jobs/<id>/delete/` | Confirm and delete a job | Owner only |
| GET, POST | `/jobs/<id>/apply/` | Apply to a job; the page submits using POST | Sign-in required |
| GET | `/my-jobs/` | View jobs posted by the signed-in user | Sign-in required |
| GET | `/my-applications/` | View applications submitted by the signed-in user | Sign-in required |
| GET, POST | `/accounts/signup/` | Display and submit the signup form | Public |
| GET, POST | `/accounts/login/` | Display and submit the login form | Public |
| GET, POST | `/accounts/logout/` | Sign out and redirect to the job list | Sign-in required |
| GET | `/admin/` | Django administration | Staff account required |

`<id>` is the integer primary key of a job. Unauthenticated users accessing
protected pages are redirected to the login page. Attempts to edit or delete
another user's job return a not-found response.

## Accounts and roles

The custom account model extends Django's `AbstractUser` and adds a `role` field:

- `seeker` — displayed as **Job Seeker**; the default role.
- `employeer` — displayed as **Employeer** in the current model.

The role is collected during registration. Posting a job currently requires a
signed-in account but is not restricted by role; job updates and deletion are
restricted to the job owner.

## Data models

### CustomUser

Uses Django's standard user fields, including username, email, password, and
authentication flags, with the additional `role` field described above.

### Job

| Field | Description |
|---|---|
| `title` | Job title, up to 100 characters |
| `company_name` | Company name, up to 200 characters |
| `description` | Full job description |
| `location` | Job location, up to 100 characters |
| `salary` | Decimal salary value |
| `posted_by` | User who created the job |
| `date_posted` | Date the job record was created |

Jobs are ordered newest first by posting date.

### Application

| Field | Description |
|---|---|
| `job` | Job the user applied to |
| `applicant` | User who applied |
| `date_applied` | Date the application record was created |

Deleting a job or user also deletes related records according to the models'
`CASCADE` relationships.

## Project layout

```text
job_portal/
├── accounts/                 # Custom user, signup, login/logout routes, admin
├── jobs/                     # Jobs, applications, views, routes, templates
├── job_portal/               # Django project settings and root URL config
├── templates/                # Shared base template
├── db.sqlite3                # Local development database (ignored by Git)
└── manage.py                 # Django management commands
```

## Tests and checks

Run the Django system checks:

```bash
python manage.py check
```

Run the test suite:

```bash
python manage.py test
```

## Development and deployment notes

- `DEBUG` is enabled in the current development settings. Do not use these
  settings as-is for a public production deployment.
- Configure a private `SECRET_KEY`, production `ALLOWED_HOSTS`, and an
  appropriate database for deployment.
- Use a production-grade WSGI or ASGI server instead of `runserver` in
  production.
- The project currently has no dedicated REST API, pagination, or file-upload
  workflow.

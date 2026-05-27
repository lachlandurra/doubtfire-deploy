# OnTrack (Doubtfire) — Development Runbook

This document covers how to start the full application stack, seed test data, and run the
automated test suite. It is written for the `feature/keycloak-prototype-b` branch, which
adds Google account linking and Google sign-in via Keycloak.

---

## Prerequisites

| Tool | Min version | Notes |
|---|---|---|
| Docker Desktop | 4.x | Engine + Compose v2 bundled |
| Docker Compose | v2 (`docker compose`) | v1 (`docker-compose`) also works |
| Free ports | 3000, 4200, 8080, 3306 | Must not be in use before starting |

Clone the repo with submodules:

```bash
git clone --recurse-submodules https://github.com/doubtfire-lms/doubtfire-deploy
cd doubtfire-deploy
git checkout feature/keycloak-prototype-b
git submodule update --init --recursive
```

---

## 1. Starting the Application

All commands run from the `development/` directory:

```bash
cd development
docker compose up --build
```

On first run this builds images for `doubtfire-api` and `doubtfire-web` (~10 minutes).
Subsequent starts reuse the layer cache and are much faster.

### What starts up

| Service | Container name | Port | Description |
|---|---|---|---|
| MariaDB | `doubtfire-dev-db` | — | Database (internal only) |
| Redis | `doubtfire-redis-sidekiq` | — | Background job queue (internal only) |
| API (Rails) | `doubtfire-api` | **3000** | REST API, auto-runs migrations + seed on boot |
| Web (Angular) | `doubtfire-web` | **4200** | Frontend dev server with hot reload |
| Keycloak | `doubtfire-keycloak` | **8080** | Google OAuth broker (admin: `admin`/`admin`) |

The API startup command waits for MariaDB to be ready, then runs:
```
bundle exec rake db:migrate db:seed
bundle exec rails s -b 0.0.0.0
```

`db:seed` creates the five system roles (`Student`, `Tutor`, `Convenor`, `Admin`, `Auditor`)
using `find_or_create_by`, so it is safe on every restart.

The application is ready when the web container prints:
```
** Angular Live Development Server is listening on 0.0.0.0:4200
```

Open **http://localhost:4200** in a browser.

---

## 2. Seeding Test Users

`db:migrate db:seed` only creates roles, not users. To populate a full set of test
accounts and sample units, run the populate task inside the API container:

```bash
docker compose exec doubtfire-api bundle exec rake db:populate
```

> **Warning:** `db:populate` calls `db:drop` and `db:setup` first — it wipes and
> rebuilds the database. Only run it on a fresh or throwaway dev database.

### Default accounts (all passwords: `password`)

| Username | Role | Notes |
|---|---|---|
| `aadmin` / `acain` | Admin | Primary admin accounts |
| `aconvenor` | Convenor | Unit convenor |
| `atutor` | Tutor | |
| `astudent` / `123456X` | Student | Basic student accounts |
| `ajones` | Admin | Secondary admin |
| `aauditor` | Auditor | Read-only access |

All accounts use `password` as their password (set by the `DatabasePopulator`). These are
development-only accounts and must never appear in production.

### Minimal seed (admin only, no wipe)

If you want just the admin account without dropping the database:

```bash
docker compose exec doubtfire-api bundle exec rake db:init
```

This creates `aadmin` (role: Admin, password: `password`) if no users exist yet, and is
safe to run on a live database.

---

## 3. Keycloak Setup (Google Sign-In)

The `doubtfire` realm and the `doubtfire-link` OIDC client are **created automatically**
when Keycloak first starts — no manual realm or client configuration is needed.

`development/keycloak/doubtfire-realm.json` is bind-mounted into the Keycloak container's
import directory. On boot, Keycloak's `--import-realm` flag reads that file and creates
the realm and client with the shared secret already matching `DF_KEYCLOAK_CLIENT_SECRET`
in `docker-compose.yml`. If the realm already exists in the `keycloak_data` volume (e.g.
from a previous run) the import is silently skipped — it is safe to restart the stack.

> **Tests only?** Skip this entire section. The automated test suite stubs all Keycloak
> HTTP calls with WebMock — no running Keycloak instance or real Google credentials are
> needed to run `bundle exec rails test`.

### 3.1 Add the Google identity provider (one-time, manual)

The only remaining step requires real Google OAuth credentials. These come from
[Google Cloud Console](https://console.cloud.google.com/) → **APIs & Services** →
**Credentials** → **Create credentials** → **OAuth 2.0 Client ID** (Application type:
**Web application**).

Register this **Authorised redirect URI** in Google Cloud Console:
```
http://localhost:8080/realms/doubtfire/broker/google/endpoint
```

Then configure the identity provider in Keycloak:

1. Open **http://localhost:8080** → log in as `admin` / `admin`
2. In the realm selector (top-left) choose **doubtfire**
3. **Identity Providers** → **Add provider** → **Google**
4. Paste your **Client ID** and **Client Secret** from Google Cloud Console
5. **Save**

The **Sign in with Google** button on the login page will now redirect through Keycloak
to Google and back.

> **Resetting Keycloak:** If the `keycloak_data` volume already contains a manually
> configured realm from before this automation was added, run `docker compose down -v`
> (wipes all volumes) followed by `docker compose up --build` to let the import run fresh.

---

## 4. Running the Test Suite

Tests run inside the `doubtfire-api` container against an in-memory test database
(configured in `test/test_helper.rb`). Keycloak environment variables are set
automatically by the test helper — no real Keycloak instance is needed.

### Run all tests

```bash
docker compose exec doubtfire-api bundle exec rails test
```

### Run a specific test file

```bash
docker compose exec doubtfire-api bundle exec rails test test/api/auth_test.rb
```

### Run a single test method

```bash
docker compose exec doubtfire-api bundle exec rails test test/api/auth_test.rb \
  -n test_keycloak_google_signin_start_returns_google_broker_url
```

### Keycloak-specific tests (Prototype B)

The auth tests for the Google sign-in / account linking features live in
`test/api/auth_test.rb`. The relevant test methods are:

| Test | What it covers |
|---|---|
| `test_auth_method_exposes_keycloak_google_signin_when_configured` | `GET /auth/method` returns `google_signin_enabled: true` |
| `test_keycloak_google_signin_start_returns_google_broker_url` | `GET /auth/google` returns a correctly structured Keycloak authorization URL |
| `test_keycloak_google_link_start_requires_authenticated_user_and_sets_link_state` | `GET /auth/link/google` requires auth and embeds `user_id` in the state JWT |
| `test_keycloak_google_signin_callback_uses_linked_login_and_issues_one_time_token` | Full sign-in callback flow: code exchange → ID token verify → one-time token |
| `test_keycloak_google_signin_callback_rejects_unlinked_google_account` | Unlinked email redirects to `?error=no_linked_account` |
| `test_keycloak_google_link_callback_records_linked_login` | Link flow creates `UserLinkedLogin` record |
| `test_keycloak_google_link_callback_rejects_provider_identifier_already_linked_elsewhere` | Prevents one Google email from being linked to two users |

All HTTP calls to Keycloak are stubbed with WebMock — the tests pass without a running
Keycloak instance.

---

## 5. Stopping the Stack

```bash
# Stop containers (preserves volumes / data)
docker compose down

# Stop and delete all data (volumes wiped — full clean state)
docker compose down -v
```

Use `down -v` before re-running `db:populate` if you want a guaranteed clean slate.

---

## 6. Useful One-Liners

```bash
# Tail API logs
docker compose logs -f doubtfire-api

# Open a Rails console
docker compose exec doubtfire-api bundle exec rails console

# Re-run only DB migrations (safe on a live DB)
docker compose exec doubtfire-api bundle exec rake db:migrate

# Check migration status
docker compose exec doubtfire-api bundle exec rake db:migrate:status
```

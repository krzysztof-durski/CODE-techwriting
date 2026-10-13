<!-- CODE CLASS README EXAMPLE -->

<a id="readme-top"></a>

> [!NOTE]
> This README is a version for the technical documentation module. It belongs to the original watchpapa.tv repository, which is not part of this repository. Links to files such as `CONTEXT.md`, `Documentation/` and the migrations point to that original repository. Some of these links may not work.

# watchpapa.tv

watchpapa.tv helps you follow movies and TV shows. It shows every upcoming episode and premiere on one calendar and keeps a rating diary of everything you watch.

![React 19](https://img.shields.io/badge/React-19-61dafb) ![Vite 8](https://img.shields.io/badge/Vite-8-646cff) ![Tailwind CSS 4](https://img.shields.io/badge/Tailwind-4-06b6d4) ![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers_%26_Pages-f38020) ![Hono 4](https://img.shields.io/badge/Hono-4-e36002) ![Supabase](https://img.shields.io/badge/Supabase-Postgres_%2B_Auth-3fcf8e)

Visit the live site at [watchpapa.tv](https://watchpapa.tv). 

![The watchpapa.tv home page with a trending hero banner above rows of popular movies and shows](images/watchpapa-home.webp)

<!-- TABLE OF CONTENTS -->
## Table of contents

- [About the project](#about-the-project)
- [Getting started](#getting-started)
- [Usage](#usage)
- [Configuration](#configuration)
- [Development](#development)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)
- [Documentation](#documentation)
- [Change log](#change-log)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Licence](#licence)
- [Acknowledgements](#acknowledgements)
- [AI usage](#ai-usage)

<!-- ABOUT THE PROJECT -->
## About the project

Keeping up with shows is hard. You finish a series and then must remember when the next episode arrives. A second show releases weekly on a different day. A film you want to see opens in autumn.

With watchpapa.tv you can follow all of these titles and see their release dates on one personal calendar. You can also rate, log and organise what you watch.

The site reads its movie and TV information live from The Movie Database (TMDB). Titles, dates, posters and cast stay current. watchpapa.tv stores only your own data, which is your ratings, follows and watchlists.

The project is live in production and under active development.

### Features

- **Releases calendar.** Shows a month grid on desktop and an agenda list or week view on a phone. You cannot newly follow a finished show or an already released film, so the calendar shows only titles worth waiting for.
- **Detail pages.** Every movie, show, season, episode and person has a page. You can rate a title, log a watch, add it to a watchlist and see where to stream it in your regions. Logging several watches creates a rewatch diary.
- **Discovery.** Search movies, shows and people in one list. Browse daily trending titles, personal recommendations, collections, filmographies and age certifications.
- **My Services.** Filters popular, top-rated and suggested titles to the streaming services you pay for. This page needs a Pro plan or higher.
- **Library and profile.** Keep several named watchlists and choose five favourites. A public profile shows your favourites and stats. You can also observe other people to see their activity, and you can share image cards.
- **Import and export.** Bring your history from a Letterboxd or watchpapa CSV file. The import runs in the background, so you can close the tab. CSV means comma-separated values.
- **Language and region.** Choose a content language, translated or original titles, and the regions for release dates and streaming availability.
- **Opt-in adult content.** Adult content is off by default. It needs your date of birth and uses filtering beyond TMDB's own flag.
- **Plans and rewards.** Free, Premium, Pro and Pro+ plans exist, with referrals, reward codes and upgrades granted by administrators. Payments are not available yet.

![The releases calendar for October 2026 with followed shows and movies on their air dates](images/watchpapa-calendar.webp)

### How it works

The React frontend and the application programming interface (API) both run on Cloudflare. The API is a Cloudflare Worker. It reads movie and TV data from TMDB on request and caches the data at the edge. Supabase stores user data in a PostgreSQL database. Row-level security (RLS) lets each user read and write only their own rows.

```mermaid
flowchart LR
    B([Browser]) -->|loads app| P["Frontend<br/>React on Cloudflare Pages"]
    X([Social and chat crawlers]) -->|link previews| F["Pages Function<br/>real 404s and OG / JSON-LD tags"]
    F -->|GET /api/meta| W
    P -->|own user data, RLS-gated| S[("Supabase<br/>Postgres and Auth")]
    P -->|content and multi-table reads<br/>JWT bearer| W["Worker API<br/>Hono on Cloudflare Workers"]
    W -->|live content, edge-cached| T[TMDB API v3]
    W -->|SQL via Hyperdrive| S
    C[Cron trigger<br/>every minute] -->|advances import jobs| W
    G["GitHub Actions<br/>nightly"] -->|pg_dump, age-encrypted| R[("Cloudflare R2<br/>30-day retention")]
    S -.-> G
```

Four design decisions shape the project.

- **No content database.** Language and region travel as query parameters, so the edge cache keeps one copy per locale.
- **Built for the Workers Free plan.** Every endpoint that fans out to many requests has a cap. The caps keep each request within the plan's limits.
- **Stateless sign-in checks.** The Worker verifies Supabase tokens locally. It reads a user's role from the database on every request.
- **Rules live in the database.** RLS protects every table. Database triggers enforce plan limits, so the interface is not the only guard.

### Built with

* [React 19](https://react.dev/) and [React Router 7](https://reactrouter.com/)
* [Vite 8](https://vite.dev/)
* [Tailwind CSS 4](https://tailwindcss.com/)
* [TipTap](https://tiptap.dev/) for the announcements editor
* [react-easy-crop](https://github.com/ValentinH/react-easy-crop) for avatars
* [Hono 4](https://hono.dev/) on [Cloudflare Workers](https://developers.cloudflare.com/workers/)
* [Supabase](https://supabase.com/) for the database, sign-in and file storage
* [postgres.js](https://github.com/porsager/postgres) over Cloudflare Hyperdrive
* [jose](https://github.com/panva/jose) for token verification
* [TMDB API v3](https://developer.themoviedb.org/) for movie, show and people data
* [Resend](https://resend.com/) for sign-in emails
* [Cloudflare Pages](https://pages.cloudflare.com/) for hosting
* [Cloudflare R2](https://developers.cloudflare.com/r2/) for encrypted backups
* [Vitest](https://vitest.dev/), [Playwright](https://playwright.dev/) and Node's built-in test runner

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Getting started

Follow these steps to run a local copy.

### Prerequisites

* Node.js 22 and npm
  ```sh
  node --version   # should print v22.x
  ```
* A [TMDB API key](https://www.themoviedb.org/settings/api)
* A Supabase project that has the watchpapa schema. You need its URL, publishable key and session-pooler connection string. The numbered files in [Backend/src/db/migrations/](https://github.com/watchpapa/watchpapa/tree/production/Backend/src/db/migrations) create the schema.

### Installation

1. Clone the repository.
   ```sh
   git clone https://github.com/watchpapa/watchpapa.git
   cd watchpapa
   ```
2. Install the packages for the Worker and the frontend.
   ```sh
   (cd worker && npm install) && (cd Frontend && npm install)
   ```
   Both installs finish without errors. Only the root test suites need the root `package.json`.
3. Create the local configuration files from the examples. Then add your TMDB key and your Supabase URL and key.
   ```sh
   cp worker/.dev.vars.example worker/.dev.vars   # TMDB_API_KEY_SECRET
   cp Frontend/.env.example Frontend/.env         # VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY
   ```
4. Start the Worker API. Put the database connection string in a shell variable, not in `.dev.vars`.
   ```sh
   export CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE="postgres://USER:PASS@HOST:5432/postgres?sslmode=require"
   cd worker && npm run dev
   ```
   Wrangler starts the API on `http://localhost:8787`.
5. Open a second terminal and check that the API runs.
   ```sh
   curl http://localhost:8787/health
   ```
   You should see this output.
   ```json
   {"status":"ok"}
   ```
6. Start the frontend.
   ```sh
   cd Frontend && npm run dev
   ```
7. Open `http://localhost:5173`. The home page shows live posters from TMDB. Vite forwards `/api` requests to the Worker, so you do not need to configure cross-origin access locally.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->
## Usage

### Follow titles and use the calendar

1. Create an account and choose a username.
2. Search for a show or movie and open its page.
3. Select **Follow**.
4. Open **Calendar** to see the upcoming episodes and premieres of your followed titles.

> [!NOTE]
> Each plan has a follow limit. If you exceed the limit, the calendar stays blocked until you unfollow some titles or upgrade.

![Search results for "blade runner" with Movies, Shows and People tabs, sorting and follow buttons](images/watchpapa-search.webp)

### Rate and log what you watch

1. Open a title.
2. Rate it, log a watch or add it to a watchlist.

Rating a title or logging a watch removes it from your watchlists. The site keeps your rewatches in a diary and keeps a history of your own rating. The share button creates image cards.

Community ratings appear after 10 people rate a title. Until then, the page shows the TMDB average.

![The Matrix detail page with a Where to watch panel and a rating and watchlist sidebar](images/watchpapa-movie.webp)

### Import your history

1. Open **Import**.
2. Upload a Letterboxd export or a watchpapa CSV file. Your browser reads the file.
3. Choose how to handle conflicts and which watchlist to use.
4. Select the button that starts the import.

The import runs as a background job, so you can close the tab. A status pill shows the progress. You can export all your data as a CSV file from the same page.

### Choose your language, region and services

1. Open **Settings**.
2. Choose a content language and select translated or original titles.
3. Pick the regions for release dates and **Where to watch**.
4. Select your streaming services.

You can also reorder or hide any row on the home page.

> [!WARNING]
> Adult content is off by default. You must enter a date of birth before you can turn it on.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONFIGURATION -->
## Configuration

| What | Where | Notes |
|---|---|---|
| TMDB API key | `worker/.dev.vars` locally. `wrangler secret put TMDB_API_KEY_SECRET` in production. | A secret. Never commit it. |
| Database connection | Shell variable `CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE` locally. `HYPERDRIVE` binding in production. | Use the Supabase session pooler URL. |
| Worker settings | `vars` in [worker/wrangler.jsonc](https://github.com/watchpapa/watchpapa/blob/production/worker/wrangler.jsonc) | `ALLOWED_ORIGINS`, `SUPABASE_URL`, `BATCH_MAX`, `BATCH_SUBREQ_BUDGET`, `IMPORT_CHUNK_MAX`, `RELEASES_MAX_SHOWS`, `SITEMAP_PAGES`, `RECOMMENDATIONS_MAX_SEEDS`, `RECOMMENDATIONS_PROVIDER_CHECK_MAX`, `SHOW_STATUS_BACKFILL_MAX` |
| Rate limits and cron | `worker/wrangler.jsonc` | The `RL_GLOBAL` and `RL_MUTATION` bindings, and a cron trigger that runs every minute |
| Frontend settings | `Frontend/.env` locally. Pages environment variables in production. | `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_API_BASE_URL` |
| Test settings | Root `.env` | `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `TMDB_API_KEY_SECRET` |
| Backup and deploy secrets | GitHub Actions secrets | `SUPABASE_DB_POOLER_URL`, `AGE_PUBLIC_KEY`, `R2_*`, `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` |
| Database schema | `Backend/src/db/migrations/` | Numbered SQL migrations that you only append to |

> [!IMPORTANT]
> In production, set `VITE_API_BASE_URL` to `https://api.watchpapa.tv`. The frontend calls the Worker from another origin. Without this value, the app cannot reach the API.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- DEVELOPMENT -->
## Development

Run `npm install` once in the repository root before you run the root test suites.

| Command | Where | What it does |
|---|---|---|
| `npm run dev` | `Frontend/` | Starts the Vite development server |
| `npm run build` | `Frontend/` | Creates a production build in `dist/` |
| `npm run lint` | `Frontend/` | Runs ESLint |
| `npm run dev` | `worker/` | Starts the Worker locally with Wrangler |
| `npm test` | `worker/` | Runs the Worker unit tests |
| `npm run test:rls` | root | Tests the RLS policies and audit triggers against a live database inside rolled-back transactions |
| `npm run test:unit` | root | Tests the breached-password check and the password-write guard |
| `npm run test:e2e` | root | Runs Playwright against the real interface with all network calls mocked. Run `cd Frontend && npx playwright install chromium` first. |
| `npm run test:backup` | root | Tests the backup pipeline and runs a restore drill. Needs Docker and [`age`](https://github.com/FiloSottile/age). |
| `npm run test:smoke` | root | Checks the live Supabase project and Worker. Creates and deletes one temporary user. |

The project has this layout.

```text
watchpapa/
├── Frontend/                 # React app (Cloudflare Pages)
│   ├── functions/            #   Pages Function: real 404s and link-preview tags
│   ├── src/pages/            #   one component per route: app/, auth/, admin/
│   ├── src/features/         #   data hooks grouped by domain (content, rating, follows)
│   ├── src/components/       #   shared interface components
│   └── src/lib/              #   API client, Supabase client, TMDB image URLs
├── worker/                   # Cloudflare Worker API (Hono)
│   ├── src/routes/           #   one file per route group, admin routes in routes/admin/
│   ├── src/tmdb/             #   TMDB client, normalisation, recommendations, adult-content filter
│   ├── src/import/           #   CSV import: resolve titles, commit rows
│   ├── test/                 #   Vitest unit tests
│   └── wrangler.jsonc        #   bindings, variables, cron trigger, routes
├── Backend/src/db/migrations/  # numbered SQL migrations
├── scripts/db-backup.sh      # nightly encrypted database backup
├── tests/                    # rls_tests, unit, e2e, backup, smoke, plus pytest checks
├── Documentation/            # release notes, deployment, database, security, compliance
├── .github/workflows/        # nightly database backup, manual Worker deploy
└── CONTEXT.md                # codebase reference: routes, schema, conventions
```

Keep these rules in mind when you change code.

- Never store content such as movies, shows or people. Read it live through the Worker. Use the hooks in `Frontend/src/features/content/` to call those endpoints.
- Key user tables by `tmdb_id`. The frontend reads them with supabase-js under RLS. Route anything that needs several tables at once through the Worker.
- Do not pass arrays as bind parameters in Worker queries. The database connection sets `fetch_types: false`, so `= ANY(${arr})` fails. Pass a comma-separated string with `string_to_array(...)` or use `IN ${sql(arr)}`.
- Add a new numbered migration file. Do not edit an existing one.
- Update [CONTEXT.md](https://github.com/watchpapa/watchpapa/blob/production/CONTEXT.md) after every meaningful change.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- DEPLOYMENT -->
## Deployment

A push to the `production` branch deploys both parts automatically.

| Part | How it deploys |
|---|---|
| Frontend | Cloudflare Pages git build. The project is `watchpapa` and the root directory is `Frontend/`. |
| Worker API | Cloudflare Workers Builds. The project is `watchpapa-api` and the root directory is `worker/`. The API runs at `api.watchpapa.tv`. |
| Database | You apply new files from `Backend/src/db/migrations/` to Supabase. |

If Workers Builds is unavailable, run the **Deploy Worker (manual backup)** workflow from the GitHub Actions tab.

A nightly GitHub Actions job backs up the database. It runs `pg_dump`, encrypts the dump with `age` and uploads it to a private Cloudflare R2 bucket. The bucket keeps backups for 30 days. The Supabase Free plan has no built-in backups. [Documentation/deployment.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/deployment.md) describes the secrets and the restore steps.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- TROUBLESHOOTING -->
## Troubleshooting

| Problem | What to check |
|---|---|
| The site loads but every API call fails | Set `VITE_API_BASE_URL` to `https://api.watchpapa.tv` in the Pages environment. Add your origin to `ALLOWED_ORIGINS`. |
| Sign-in or database calls fail in the browser | Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` to the Pages production environment. Make sure no `Frontend/wrangler.toml` file exists. |
| The Worker returns 500 errors on user-data routes | Run `npx wrangler tail` and check the Hyperdrive binding ID in `wrangler.jsonc`. Also check that no query passes an array as a bind parameter. |
| A background import stops | Run `npx wrangler tail` and look for `[IMPORT]` lines every minute. The next run retries a step that fails. |
| Unknown URLs return 200 instead of 404 | Deploy `Frontend/functions/_middleware.js`. The Pages root directory must be `Frontend/`. |
| The Supabase project is paused | The Free plan pauses inactive projects. The Worker's cron query normally prevents this. Restore the project in the Supabase dashboard. |

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- DOCUMENTATION -->
## Documentation

| I want to | Read |
|---|---|
| Look up a route, table or convention | [CONTEXT.md](https://github.com/watchpapa/watchpapa/blob/production/CONTEXT.md) |
| Deploy, set secrets or restore a backup | [Documentation/deployment.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/deployment.md) |
| See the database tables, policies and functions | [Documentation/database/schema.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/database/schema.md) |
| Understand plans, referrals and reward codes | [Documentation/subscription_system.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/subscription_system.md) |
| Review the threat model and security controls | [Documentation/security/security.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/security/security.md) |
| Browse all documents | [Documentation/README.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/README.md) |

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CHANGE LOG -->
## Change log

[Documentation/releases.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/releases.md) lists what each release added.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->
## Roadmap

- [x] Live TMDB content through a Cloudflare Worker
- [x] Releases calendar, ratings, watchlists and rewatch diary
- [x] Letterboxd and CSV import as background jobs
- [x] Language, region, streaming-service and adult-content preferences
- [x] My Services page, collections and certifications
- [x] Plans, reward codes, referrals and bulk tier rewards
- [x] Link previews, structured data and real 404 pages
- [x] Encrypted nightly database backups with a tested restore
- [ ] Payments for paid plans

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->
## Contributing

watchpapa.tv is a personal project under an all-rights-reserved licence. The project does not accept outside contributions without written permission. To offer help, contact the owner through the [contact page](https://watchpapa.tv/contact) first.

To report a bug or ask a question, [open an issue](https://github.com/watchpapa/watchpapa/issues) or use the contact page.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENCE -->
## Licence

Copyright © 2026 Krzysztof Durski. All rights reserved. See [`LICENCE`](https://github.com/watchpapa/watchpapa/blob/production/LICENCE) for details.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

- TMDB provides the movie, TV and people data and images. _This product uses the TMDB API but is not endorsed or certified by TMDB._
- JustWatch provides the streaming availability data through TMDB.
- The [Have I Been Pwned](https://haveibeenpwned.com/Passwords) range API powers the breached-password check.
- The project builds on [Cloudflare Workers and Pages](https://developers.cloudflare.com/), [Supabase](https://supabase.com/), [Hono](https://hono.dev/) and [React](https://react.dev/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- AI USAGE -->
## AI usage

AI tools (Cursor and Claude Code) helped with brainstorming, implementation, debugging and documentation. The author made the architecture, planning and technical decisions. The author reviewed and adapted all AI output before committing it. Commits that AI assisted carry a `Co-authored-by` line that names the tool.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

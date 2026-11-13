版本：2026-11-11

# TrailTales

> Turn travel photos into AI-written travelogues.

TrailTales is a photo travelogue web app. Upload your travel photos and it groups them into trips, recognizes where each trip went, and writes a travelogue for every trip — ready to read, edit, and share.

## What it does

- **Bulk upload** — drag-and-drop or pick photos; upload progress and per-file retry, with a summary of completed / in-progress / failed files.
- **Trips, auto-grouped** — photos are grouped into trips by time (a new trip starts after 72 hours without a photo). Trips can be split or merged before confirming.
- **Location recognition** — GPS coordinates embedded in the photo are used first when available; otherwise AI (Gemini) guesses the location from the photos; the author confirms, edits, or skips per trip.
- **AI travelogues** — one travelogue per trip, generated from the trip's photos and locations, with per-place sections and photos. Regenerating creates a new draft; the published version stays live until the new one is published.
- **Reading & editing** — rich-text editing (TipTap) with paragraph structure you control; font size and night mode saved per reader.
- **Portal** — public list of published travelogues with search, five sort orders, and rule-based recommendations ("you may like").
- **Accounts & roles** — Super Admin, Author, User, Guest. Authors manage drafts and published posts; Super Admins can pin / unpin and take down posts (taken-down posts go to the archive, restorable for 60 days).
- **Messages** — private messages and system notifications (likes, comments, archive reminders).
- **Responsive** — one codebase for desktop and mobile, with skeleton screens on every page.

## Tech stack

- **Next.js 13** (App Router) · **React 18** · **TypeScript**
- **MongoDB Atlas M0** (free tier, Mongoose) — text and metadata only; photo files live on disk
- **Google Gemini AI** — location recognition and travelogue generation
- **TipTap v2** — rich-text editing (headless core, custom toolbar)
- **Sharp** — image processing (HEIC → JPEG, thumbnails, blur placeholders)
- **bcryptjs** — password hashing (server-side, cost factor 10)
- **Playwright** — end-to-end tests, plus API tests and CI

## Getting started

Prerequisites: **Node.js 16**.

```bash
npm install
```

Create a `.env.local` file in the project root:

```env
MONGODB_URI=mongodb+srv://<user>:<password>@<cluster>/<database>
GEMINI_API_KEY=<your-gemini-api-key>
```

Then run the dev server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

> To run it locally, paste your own Gemini API key into `.env.local` (`GEMINI_API_KEY`). The key is only used server-side (Route Handlers call Gemini); it never goes into front-end code or the repository.

## Project structure

One repository, front end and back end together (Next.js 13 App Router):

```
app/                  # pages + API routes
  layout.tsx          # shared layout: top nav + footer
  page.tsx            # portal page (home)
  my-travelogues/     # my travelogues
  upload/             # upload page
  trips/              # trip list + trip detail / photo wall
  trips/locations/[id]/  # location confirm page
  generate/[id]/      # travelogue generation status page
  travelogue/[id]/    # travelogue reading page
  author/             # author management (drafts / published)
  profile/            # profile page
  messages/           # message center
  archive/            # archive / recycle bin
  api/                # backend Route Handlers, grouped by feature
    auth/ upload/ trips/ locations/ generate/ travelogues/
    photos/ comments/ likes/ search/ portal/ profile/
    messages/ notifications/ archive/
components/           # custom components, grouped by feature
  layout/ photo/ trip/ travelogue/ upload/ auth/ message/ common/
lib/                  # server logic: db, auth, gemini, sharp,
                      # cluster (72-hour rule), sort, upload
models/               # Mongoose models: User, Session, Travelogue,
                      # Trip, Photo, Comment, Message, Notification,
                      # Archive, GenerateTask
public/               # static files: logo, favicon
uploads/              # photo files (originals, 400px thumbnails,
                      # 20px blur placeholders) — project root, NOT
                      # under public/; served through authenticated
                      # API routes, not direct URLs
styles/               # global styles (responsive breakpoints)
tests/                # Playwright E2E + backend API tests
.env.local            # secrets (Gemini key, database connection
                      # string) — never committed
```

## API overview

12 groups of endpoints. Shared conventions: every response is a
success/failure + data/error structure; unauthenticated requests get
"not logged in", insufficient permission gets "no permission";
paginated endpoints always return the total count (the front end
computes total pages from total ÷ page size). IDs are passed as
request parameters, not built into the URL path — the one exception
is the reading page route `/travelogue/[id]`.

| Group | Endpoints include |
|---|---|
| Auth | `POST /api/auth/register`, `POST /api/auth/login` (returns user id, username, role, avatar URL), `POST /api/auth/logout`, `GET /api/auth/me`, `POST /api/auth/admin-reset-password` (Super Admin only; checks username + email, then generates a one-time temporary password) |
| Upload | `POST /api/upload` (multiple files; returns a batch id, per-file id, filename, status, thumbnail URL), `DELETE /api/upload`, `GET /api/upload/status` |
| Trips | `POST /api/trips/cluster`, `GET /api/trips`, `POST /api/trips/confirm`, `POST /api/trips/confirm-all`, `POST /api/trips/split`, `POST /api/trips/merge` |
| Locations | `POST /api/locations/recognize` (place name + confidence, an integer 0–100), `POST /api/locations/confirm`, `POST /api/locations/edit`, `POST /api/locations/skip` |
| Generate | `POST /api/generate`, `GET /api/generate/status`, `POST /api/generate/retry` |
| Travelogues | `GET /api/travelogues`, `PUT /api/travelogues`, `POST /api/travelogues/publish`, `POST /api/travelogues/unpublish`, `DELETE /api/travelogues`, `GET /api/travelogues/mine` (paginated, with total count) |
| Photos | `GET /api/trips/photos`, `GET /api/photos/file` (serves the image itself; published photos are public, unpublished ones only for their author), `POST /api/photos/move`, `DELETE /api/photos` |
| Comments / likes / favorites | `POST /api/travelogues/comments`, `GET /api/travelogues/comments`, `DELETE /api/comments`, `POST /api/travelogues/like`, `POST /api/travelogues/favorite` |
| Search / portal / recommendations | `GET /api/search` (title, content, author), `GET /api/portal`, `GET /api/portal/recommend` |
| Profile | `GET /api/profile`, `PUT /api/profile/password`, `PUT /api/profile/username`, `POST /api/profile/avatar` |
| Messages / notifications | `GET /api/messages`, `POST /api/messages`, `GET /api/messages/detail`, `GET /api/notifications`, `POST /api/notifications/read` |
| Archive | `GET /api/archive`, `POST /api/archive/restore`, `POST /api/archive/purge` |

## Photos

For testing and development, about **100 photos** are enough. Anna
will trim the current set of 533 photos down to around 100 later.
The goal is verifying the features work — this is not a load test,
so there is no need to stress-test with the full set.

## Deployment

Local-first: the app runs locally and is **not** deployed to any
overseas cloud server (including Vercel). The main way to show it to
an interviewer is a **demo video** — the interviewer watches the
video and does not need to visit or run the app. The GitHub
repository link is shared as usual, and the repository includes test
images.

## Project status

🚧 In active development. Requirements, UI design, and technical design are done (2026-11-11); implementation starts next.

## License

No license — all rights reserved.

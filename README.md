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
- **MongoDB Atlas** (Mongoose) — text and metadata only; photo files live on disk
- **Google Gemini AI** — location recognition and travelogue generation
- **TipTap** — rich-text editing (headless core, custom toolbar)
- **Sharp** — image processing (HEIC → JPEG, thumbnails, blur placeholders)
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

## Project status

🚧 In active development. Requirements and UI design are done; technical design is in progress.

## License

No license — all rights reserved.

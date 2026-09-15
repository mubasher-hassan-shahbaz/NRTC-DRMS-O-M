# NRTC Safe City — Daily Operations & SLA Reporting Portal

Internal reporting portal for NRTC field engineers (37 Punjab districts) and
the NRTC PMU, built for the Punjab Safe Cities Authority (PSCA) O&M contract.

## Stack
- Next.js 15 (App Router, Server Actions, TypeScript)
- Tailwind CSS v4
- Supabase (Postgres + Auth + Row Level Security)
- Recharts, ExcelJS, @react-pdf/renderer, lucide-react

## 1. Set up Supabase
1. Create a project at supabase.com (or use the one whose keys are already in `.env.local`).
2. Open the SQL editor and run `supabase/schema.sql` — it creates all tables
   (`districts`, `profiles`, `daily_reports`, `district_employees`,
   `attendance_logs`, `district_expenses`), seeds the 37 districts, and
   enables RLS policies.
3. Create your first users under **Authentication → Users**, then insert a
   matching row in `profiles` for each (role `DISTRICT_USER` with a
   `district_id`, or `ADMIN` for PMU staff). `profiles.id` must equal the
   `auth.users.id` of that user.

### Note on the UPDATE policies
`schema.sql` defines two separate `UPDATE` policies on `daily_reports`
("District users can update unapproved reports" and "Admins have full
update access"). Postgres RLS policies of the same command type are
combined with `OR`, so this is intentional and correct — a district user is
covered by the first policy, an admin by the second (and technically also
qualifies under the first's `ADMIN`/`AUDITOR` clause). No conflict, nothing
to change.

## 2. Environment variables — nothing required for now
Your Supabase URL and anon key are baked in as defaults in
`lib/supabase/config.ts`. This is safe: the anon key is meant to be public
(access control comes from the RLS policies in `schema.sql`, not from
hiding this key), so the app deploys and runs correctly with **zero**
environment variables configured — no Vercel dashboard step needed.

If you ever rotate keys or point this at a different Supabase project,
either edit the fallback values in `lib/supabase/config.ts` directly, or
set these on Vercel/locally (they override the fallback — regular,
non-shared project environment variables are free on Vercel's Hobby plan,
no upgrade needed):

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

## 3. Run locally
```
npm install
npm run dev
```
Visit http://localhost:3000 — you'll be redirected to `/login`.

## 4. Deploy on Vercel (free plan)
1. Push this folder to a GitHub repo (or use Vercel's "Upload" import if you
   don't want to use git).
2. Import the repo in Vercel.
3. In **Project Settings → Environment Variables**, add the same three
   `NEXT_PUBLIC_SUPABASE_*` variables from `.env.local`.
4. Deploy. No other build configuration is needed — it's a standard Next.js
   app.

## Project structure
```
app/
  login/                  Login page
  dashboard/              District user: report history + 6-step wizard
  admin/                  PMU: Punjab overview, all-reports, review workflow
actions/                  Server Actions (auth, reports CRUD, review)
components/
  wizard/                 The 6-step daily report form
  admin/                  Overview cards, chart, districts table, inspector modal
  ui/                     Small shared UI primitives (Button, Card, Input, ...)
lib/
  supabase/               Browser / server / middleware Supabase clients
  calculations.ts         Availability % and offline-count formulas
  export/                 Excel (ExcelJS) and PDF (@react-pdf/renderer) export
supabase/schema.sql       Full schema + RLS (as provided)
```

## What's implemented vs. left as a next step
Implemented: auth + RBAC (district-locked vs Punjab-wide), the full 6-step
wizard with live availability calculations, draft/submit, admin overview
dashboard with a 7-day trend chart, filterable all-reports table, the
approve / needs-revision / reject workflow with PMU remarks, and Excel/PDF
export.

Not yet built (schema already supports these, so they're a natural next
phase): the `district_employees` / `attendance_logs` geofenced attendance
screens, and the `district_expenses` claims workflow. Say the word and I'll
build either of those next in the same style.

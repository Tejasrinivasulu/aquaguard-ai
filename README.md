# AquaGuard AI — Clean Oceans Platform

AquaGuard AI is a **full-stack-style web application** (React + Vite) that models an end-to-end **plastic pollution monitoring and response ecosystem** for coastal and inland waters. It connects **citizens**, **government / authority**, **NGO partners**, and **field workers** around AI-assisted reporting, UAV dispatch, mission management, and rewards.

> **Demo mode:** By default, business data lives in the **browser (`localStorage`)** — no backend required. Optionally run the **`server/`** app to persist **citizen bundles** in **SQLite** (see [Database (optional)](#database-optional)). Demo accounts and synthetic rows seed on first load.

---

## Table of contents

- [Tech stack](#tech-stack)
- [Quick start](#quick-start)
- [Database (optional)](#database-optional)
- [Who uses the system](#who-uses-the-system)
- [Feature map by role](#feature-map-by-role)
- [End-to-end workflows](#end-to-end-workflows)
- [UAV, plastic AI & satellite scheduler](#uav-plastic-ai--satellite-scheduler)
- [Maps & location](#maps--location)
- [Demo data & test logins](#demo-data--test-logins)
- [Project structure](#project-structure)
- [Scripts](#scripts)

---

## Tech stack

| Layer | Choice |
|--------|--------|
| UI | React 18, TypeScript, Vite |
| Styling | Tailwind CSS, shadcn/ui (Radix), `next-themes` (light/dark) |
| Routing | React Router v6 |
| Maps | Leaflet, React-Leaflet |
| Charts | Recharts |
| Forms | React Hook Form + Zod |
| State (demo) | `localStorage` + small in-memory version counters + React Query (provider) |
| API + DB (optional) | Node + [Hono](https://hono.dev/) + [Prisma](https://www.prisma.io/) + **SQLite** (`server/`) |

---

## Quick start

```bash
npm install
npm run dev
```

Open the URL Vite prints (usually `http://localhost:5173`).

```bash
npm run build    # production build
npm run preview  # serve dist locally
npm run lint
npm run test     # Vitest
```

---

## Database (optional)

Citizen reports, profile, settings, notifications, etc. can be mirrored to a **SQLite** file via a small API in `server/`.

1. **Copy env** — `cp server/.env.example server/.env` (or create `server/.env` with `DATABASE_URL="file:./dev.db"` and `PORT=3001`).
2. **Install & schema** — from `server/`:

   ```bash
   cd server
   npm install
   npx prisma generate
   npx prisma db push
   npm run dev
   ```

   The API listens on `http://localhost:3001` (see `/health`).

3. **Point the SPA at the API** — in the **project root**, create `.env.local`:

   ```bash
   VITE_CITIZEN_API_URL=http://localhost:3001
   ```

   Restart `npm run dev`. On citizen dashboard load, the app **GET**s `/citizen/bundle` (header `X-Aquaguard-User`: session identifier); each **write** **PUT**s the full bundle. **localStorage is still updated** so admin / authority views on the same browser keep working.

**Production:** swap `DATABASE_URL` for PostgreSQL (or host SQLite on disk), set `CORS_ORIGIN` to your site origin(s), and deploy the `server` process next to your static Vite build.

---

## Who uses the system

| Role | Purpose | Typical login |
|------|---------|----------------|
| **Citizen** | Report pollution, view maps, rewards, profile | [`/login`](/login) — role *Citizen* |
| **NGO partner** | Triage citizen requests, run missions, analytics | [`/login/ngo`](/login/ngo) or `/login` with role *NGO* |
| **Field worker** | See regional missions, track assignments | `/login` — role *Field worker* (match registry email/name) |
| **Admin / Authority** | Verify reports before NGOs, verify cleanup proofs, UAV console, analytics | [`/login/admin`](/login/admin) or `/login` — role *Admin / Authority* |

Sessions are stored in `localStorage` under a small **session** object (`aquaguard_session`): identifier + role.

---

## Feature map by role

### Public marketing site (`/`)

- Landing hero, product sections (About, Benefits, Features), footer.
- **Session-aware navbar:** if signed in, shows **Dashboard** + **Sign out**; otherwise **Login** / **Sign up**.
- **Public site** links from admin/NGO/worker sidebars open in a **new tab** so dashboard sessions stay open.

---

### Citizen dashboard (`/dashboard/citizen`)

| Feature | Route | What it does |
|---------|--------|----------------|
| **Overview** | `/dashboard/citizen` | KPIs, severity breakdown, map markers, recent activity, system status (drone / alerts). |
| **Report pollution** | `/dashboard/citizen/report` | Upload image/video (or camera), pin location, notes; **simulated AI** (severity %, bounding boxes, plastic flag); routing hint **drone vs NGO** by severity. |
| **Nearby pollution map** | `/dashboard/citizen/map` | Leaflet map: seeded hotspots + your reports; satellite-style basemap option. |
| **My reports** | `/dashboard/citizen/my-reports` | List/filter reports; **authority** and **NGO partner** status badges; detail view. |
| **Notifications** | `/dashboard/citizen/notifications` | In-app alerts (pollution nearby, drone, cleanup, NGO). |
| **Rewards & achievements** | `/dashboard/citizen/rewards` | Points, badges, leaderboard (demo), points history. |
| **Profile** | `/dashboard/citizen/profile` | Display name, email, photo, stats. |
| **Settings** | `/dashboard/citizen/settings` | Push toggles, alert types, digest, location sharing, live GPS while using dashboard. |
| **Help / support** | `/dashboard/citizen/help` | FAQ and support ticket stub. |

**Citizen logic highlights**

- Points: upload bonuses, plastic detection, high-severity / drone routing, authority verification, cleanup milestones (see in-app copy and `citizen-store`).
- New citizen bundles can include **seed reports** (Andhra Pradesh / Telangana–style demo locations).

---

### NGO partner dashboard (`/dashboard/ngo`)

| Feature | Route | What it does |
|---------|--------|----------------|
| **Overview** | `/dashboard/ngo` | Mission KPIs, alerts, charts (trends, severity, status). |
| **Citizen requests** | `/dashboard/ngo/requests` | Queue of **authority-approved** citizen cases; accept → creates **field mission**; decline; shows **assigned worker groups** when authority set them. |
| **Missions** | `/dashboard/ngo/missions` | Full mission lifecycle: assigned → in progress → **submit after-cleanup photo** → *pending authority verification* → completed; NGO partner points on verified completion. |
| **Analytics** | `/dashboard/ngo/analytics` | Deeper charts for partner operations. |
| **Resources** | `/dashboard/ngo/resources` | Reference / partner resources (demo content). |

---

### Admin / authority dashboard (`/dashboard/admin`)

| Feature | Route | What it does |
|---------|--------|----------------|
| **Command center** | `/dashboard/admin` | Cross-cutting analytics: citizen bundles, NGO KPIs, worker status charts. |
| **Verify reports** | `/dashboard/admin/verify` | **First gate:** approve/reject citizen uploads before they appear in the **NGO incoming** queue; optional **worker group** assignment (crews, not only individuals); citizen notification + points on approve. |
| **Verify cleanups** | `/dashboard/admin/verify-cleanup` | **Second gate:** after NGO submits **after photo**, authority approves/rejects; unlocks **NGO partner points** and **worker reward points** tied to mission notes. |
| **Citizens** | `/dashboard/admin/citizens` | Per–storage-key breakdown: reports, severity, NGO flags, points. |
| **NGO & missions** | `/dashboard/admin/ngo` | NGO-side mission / queue insights for oversight. |
| **Workers** | `/dashboard/admin/workers` | Field registry: roles (cleanup, drone, inspector), status, regions, hours, missions, **reward points**. |
| **UAV & plastic AI** | `/dashboard/admin/uav-plastic` | Fleet status, plastic threshold policy, dispatch log (citizen sync vs satellite interval), map of activity. |
| **Public site** | (sidebar) | Opens marketing home in a **new tab**. |

**Worker groups**

- Defined in the worker registry layer; authority selects **groups**; members are expanded to worker IDs for NGO notes and **reward distribution**.

---

### Field worker dashboard (`/dashboard/worker`)

| Feature | Route | What it does |
|---------|--------|----------------|
| **Overview** | `/dashboard/worker` | Personal stats, regional mission hints. |
| **Missions** | `/dashboard/worker/missions` | Missions matched to home **region** (heuristic) + open missions. |
| **Profile** | `/dashboard/worker/profile` | Ties to **worker registry** when email/name matches; otherwise a demo shell. |

Workers earn **reward points** when authority verifies NGO cleanup proofs for missions that list **Authority-assigned workers** in notes.

---

## End-to-end workflows

### A. Citizen report → authority → NGO

1. Citizen submits a report (**Report pollution**).
2. Report is held for **authority** (`pending_review`) until approved or rejected.
3. On **approve**, citizen gets verification points; report can enter **NGO citizen requests**.
4. NGO **accepts** → **field mission** is created; **decline** stops that path for the demo queue.

### B. NGO mission → proof → authority → rewards

1. NGO advances mission and uploads **after-cleanup image** → status **pending verification**.
2. **Admin → Verify cleanups** approves or rejects proof.
3. On approve: mission **completed**; **NGO partner points**; **worker rewards** parsed from mission notes (assigned worker IDs).

### C. High severity & UAV

- Policy in **plastic / UAV** logic can auto-suggest or dispatch **UAV** for severe or water-linked cases; admin **UAV & plastic AI** shows fleet and **dispatch history** (e.g. `citizen_sync`, `satellite_interval`).

---

## UAV, plastic AI & satellite scheduler

- **UAV fleet** and **dispatch records** live in `localStorage` (see `uav-plastic-automation.ts`).
- A background **satellite-linked scheduler** component (`UavSatelliteScheduler`) can trigger periodic scans consistent with the same policy (demo timers).
- **Admin UAV** page documents thresholds and lets you reset / inspect demo UAV state.

---

## Maps & location

- Default map center and seeded hotspots target **Andhra Pradesh & Telangana** demo geography (e.g. Visakhapatnam area as default center).
- Citizens can use **geolocation** or manual pin where the UI allows.
- Map tiles use configurable providers (see `map-tile-providers.ts`).

---

## Demo data & test logins

On first load, `ensurePlatformDemoData()` (see `src/lib/demo-platform-seed.ts`) may create:

- **Synthetic citizen bundles** (e.g. `demo.raghav@vizag.ap`, `demo.sita@hyd.tg`, …) so **Admin → Citizens** and charts are populated.
- **Synthetic NGO incoming** rows (pending) when the queue is empty.
- **Authority queue** entry from a seeded pending report (idempotent).

**Field workers** in the registry use Indian names and **AP/TG** regions; use emails like `ananya.sharma@aquaguard.in` from `worker-store` to align the worker app with a registry row.

To see a **fresh** seed for NGO missions that include every status (e.g. `pending_verification`), clear `localStorage` key `aquaguard_ngo_workspace_v1` (or use a private window).

---

## Project structure (high level)

```text
server/                   # Optional Hono + Prisma + SQLite API (`/citizen/bundle`)
src/
  App.tsx                 # Routes + global providers + UAV scheduler
  main.tsx                # Boot + demo seed
  components/             # UI, dashboards, maps, system
  layouts/                # Role-specific shells
  pages/                  # Marketing, auth, role dashboards
  lib/                    # Domain stores (citizen, ngo, worker, authority, UAV, …)
  hooks/                  # Version subscriptions for reactive dashboards
ml/                       # Optional / separate ML experiments (see ml/*/README.md)
```

Core domain modules (all under `src/lib/`):

- `citizen-store.ts` — reports, hotspots, rewards, notifications, bundles  
- `authority-review-queue.ts` — verify-before-NGO queue  
- `ngo-store.ts` — missions, alerts, partner points, proof flow  
- `ngo-citizen-requests.ts` — NGO incoming queue  
- `worker-store.ts` — workers, groups, rewards parsing  
- `uav-plastic-automation.ts` — fleet + dispatch log  
- `demo-platform-seed.ts` — optional synthetic platform data  
- `admin-analytics.ts` — aggregates across bundles for admin overview  

---

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Preview production build |
| `npm run lint` | ESLint |
| `npm run test` | Vitest (once) |
| `npm run test:watch` | Vitest watch mode |
| `npm run api:dev` | SQLite API (`server/`) — run from repo root after `cd server && npm install` |

---

## License / attribution

This repository is configured as a **private** npm package in `package.json`. Add your team’s license and attribution here when you publish or distribute the project.

---

**AquaGuard AI** — *AI-powered detection, coordinated response, and measurable impact for cleaner waterways.*

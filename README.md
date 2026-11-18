# DisasterAI — Intelligent Emergency Response & Rescue Coordination Platform

DisasterAI is a full-stack, real-time emergency reporting, AI-assisted incident triage, GIS mapping, and rescue dispatch platform built for Pakistan (Karachi, Lahore, Islamabad, and coastal Sindh). It connects citizens, emergency command center operators, field rescue squads, and nearby hospitals on a unified operational network.

---

## Key Capabilities

### 1. Direct Pakistan Emergency Calling (`tel:`)
- **One-Tap Dispatch Confirmation Modals**: Visible across the public safety pages, portal headers, Citizen Dashboard, and emergency reporting views:
  - **Edhi Ambulance (`115`)** — Direct `tel:115` medical & ambulance dispatch confirmation.
  - **Chhipa Ambulance (`1020`)** — Direct `tel:1020` rapid ambulance & emergency transport confirmation.
  - **Fire Brigade (`101`)** — Direct `tel:101` fire suppression & trapped-occupant rescue confirmation.
  - **Rescue (`1122`)** — Direct `tel:1122` unified provincial emergency service confirmation.
- **Safety & Fallback**: Always prompts the user to confirm before initiating the device dialer, never initiates silent or repeated calls, and displays copyable dial numbers if telephony is unsupported on desktop browsers.

### 2. Nearby Hospital First-Aid & Trauma Standby Notifications
- Automatically identifies the closest hospitals using Haversine geospatial lookup across seeded Pakistani medical centers (e.g., JPMC, Civil Hospital Karachi, Aga Khan University Hospital, Liaquat National, Indus Hospital, PIMS, Mayo Hospital).
- **Citizen & Operator Triggers**: Automatically alerts the nearest hospitals on new emergencies and provides one-click **Notify Nearby Hospitals** actions on citizen tracking and command center incident detail pages.
- **Hospital Readiness Tracking**: Tracks each hospital alert across `Notified` → `First Aid Dispatched` → `ER Trauma Standby` in real time, alongside `H+` markers on the live map.

### 3. Explainable AI Severity Triage & Multimodal Hazard Analysis
- **Multimodal Gemini Engine**: Uses server-side `@google/genai` (`gemini-4-hydrogen`) to analyze incident descriptions, casualty/trapped flags, secondary hazards, and uploaded emergency photos.
- **Explainable `0–100` Scoring**: Maps incidents to `LOW` (`0–24`), `MEDIUM` (`25–49`), `HIGH` (`50–74`), and `CRITICAL` (`75–100`) with itemized risk factor impacts, confidence scores, and recommended response actions.
- **Deterministic Fallback**: Automatically runs a transparent rule-based NLP/heuristic triage scorer whenever `GEMINI_API_KEY` is unconfigured or unavailable.

### 4. Geospatial Duplicate Detection & Intelligent Team Dispatch
- **Duplicate Clustering & Safe Merge**: Evaluates spatial proximity and semantic similarity against open incidents; operators can merge duplicate reports into a primary case while preserving original witness records.
- **Rescue Squad Recommendation**: Ranks active teams (`Medical Response Team`, `Fire & Rescue Team`, `Flood Rescue Team`, `Disaster Recovery Team`, `General Emergency Team`) by category match, Haversine distance, ETA, and availability.
- **OSRM Road Routing & Live Leaflet Maps**: Plots emergency pins, live rescue vehicles, hospital markers, and road polylines fetched via OSRM with an arterial fallback.

---

## Three Role-Based Portals

Visit `/login` or `/register` to sign in with Google (Firebase Auth) or launch any portal immediately in **Demo Mode**:

| Portal | Key Routes | Capabilities |
| :--- | :--- | :--- |
| **Public & Auth** | `/`, `/login`, `/register`, `/about`, `/safety` | Platform overview, public safety guides, role login, direct `115` / `1020` / `101` / `1122` emergency dialers |
| **Citizen Portal** | `/citizen/dashboard`, `/citizen/report-emergency`, `/citizen/my-reports`, `/citizen/track/:incidentId`, `/citizen/safety-center` | GPS & map-pin reporting, multi-image upload, live AI severity preview, 8-stage timeline tracking, nearby hospital alerts |
| **Emergency Command Center** | `/admin/dashboard`, `/admin/live-map`, `/admin/incidents`, `/admin/incidents/:incidentId`, `/admin/rescue-teams`, `/admin/assignments`, `/admin/resources`, `/admin/analytics`, `/admin/notifications`, `/admin/settings` | Real-time KPIs, interactive GIS command map, AI triage override & duplicate merge, team dispatch, hospital standby management, fleet/resource inventory, Recharts analytics, audit logs, demo simulator |
| **Rescue Team Portal** | `/team/dashboard`, `/team/assignments`, `/team/assignments/:assignmentId`, `/team/resources`, `/team/history` | Accept/reject dispatches, stream `En Route` / `On Site` / `In Progress` status & GPS updates, request backup resources, submit completion reports |

---

## Tech Stack

- **Frontend**: React 19, TypeScript, React Router, Tailwind CSS v4 (`@tailwindcss/vite`), Lucide Icons, Motion (`motion`), Leaflet (`leaflet` + OpenStreetMap tiles), Recharts (`recharts`).
- **Backend**: Node.js, Express (`express`), WebSockets (`ws` at `/ws`), rate limiting (`240 req/min` per IP on `/api/*`), Vite dev middleware / static SPA serving on port `3000`.
- **AI Engine**: `@google/genai` (`gemini-4-hydrogen`) + deterministic rule-based triage fallback.
- **Persistence & Auth**: Firebase Authentication (Google Sign-In) & Cloud Firestore (`firebase-blueprint.json`, `firestore.rules`) paired with a real-time server store seeded for Karachi and Pakistani cities.

---

## Project Structure

```text
├── server.ts                  # Express REST API, WebSocket (/ws) server & Vite/dist host
├── server/
│   ├── aiEngine.ts            # Gemini multimodal triage & rule-based fallback
│   ├── geoEngine.ts           # Haversine distance, duplicate detection, OSRM routing, hospital & team ranking
│   ├── seed.ts                # Seeded Karachi/Pakistan incidents, teams, hospitals, alerts, resources
│   └── store.ts               # Authoritative real-time state & WebSocket event broadcaster
├── src/
│   ├── App.tsx                # Role routes & ErrorBoundary wrapper
│   ├── context/
│   │   └── DisasterContext.tsx# Global state, WebSocket sync, Auth, and API actions
│   ├── lib/
│   │   └── firebase.ts        # Firebase SDK initialization & Firestore persistence helpers
│   ├── types/
│   │   └── disaster.ts        # Shared TypeScript interfaces & enums
│   ├── components/
│   │   ├── layout/AppShells.tsx
│   │   ├── map/EmergencyMap.tsx
│   │   └── ui/DisasterPrimitives.tsx  # Direct emergency call modal (115/1020/101/1122), KPI cards, badges
│   └── pages/
│       ├── PublicPages.tsx
│       ├── CitizenPages.tsx
│       ├── AdminPages.tsx
│       ├── AdminOpsPages.tsx
│       └── TeamPages.tsx
├── firebase-applet-config.json
├── firebase-blueprint.json    # Firestore schema & entity definitions
├── firestore.rules            # Role-based Firestore security rules
├── firestore.rules.test.ts
├── .env.example
└── package.json
```

---

## Getting Started

### Prerequisites
- **Node.js** 20+ and **npm**

### 1. Environment Variables
Copy `.env.example` to `.env` (or configure secrets in your runtime environment):

```env
GEMINI_API_KEY="MY_GEMINI_API_KEY"
APP_URL="MY_APP_URL"

# Optional: enables SMS alerting when configured
# TWILIO_ACCOUNT_SID=""
# TWILIO_AUTH_TOKEN=""
# TWILIO_FROM_NUMBER=""
```

> If `GEMINI_API_KEY` is omitted or left at its placeholder value, DisasterAI automatically falls back to its built-in deterministic rule-based severity and hazard classifier.

### 2. Install & Run Development Server
```bash
npm install
npm run dev
```
The Express + Vite + WebSocket server runs on `http://localhost:3000`.

### 3. Typecheck, Build & Production Start
```bash
npm run lint    # Runs tsc --noEmit
npm run build   # Builds frontend bundle into dist/
npm start       # Serves production API, WebSockets, and dist/ on port 3000
```

---

## Firebase & Firestore Security Setup

- **`firebase-applet-config.json`**: Holds the provisioned Firebase client configuration and Firestore database ID.
- **`firebase-blueprint.json`**: Defines normalized collections:
  - `/users/{userId}` and `/users/{userId}/private/{docId}` (PII isolation)
  - `/admins/{userId}`
  - `/incidents/{incidentId}` and `/incidents/{incidentId}/statusHistory/{historyId}`
  - `/rescue_teams/{teamId}`, `/team_assignments/{assignmentId}`, `/resources/{resourceId}`
  - `/hospital_alerts/{alertId}`, `/notifications/{notificationId}`
  - `/rescue_operation_reports/{reportId}`, `/audit_logs/{logId}`
- **`firestore.rules`**: Enforces authenticated role-based access control, immutable audit/timeline writes, and owner/admin boundaries.

---

## REST API & WebSocket Reference

### System & Real-Time Stream
- `GET /api/health` — Returns service health plus `geminiConfigured` and `smsConfigured` flags.
- `GET /api/state` — Returns full synchronized platform state (incidents, teams, assignments, hospitals, hospital alerts, resources, notifications, reports, audit logs, and `demoMode`).
- `WS /ws` — Real-time WebSocket stream emitting `state:sync`, `incident:created`, `incident:updated`, `assignment:created`, `assignment:updated`, `team:updated`, `resource:updated`, `hospital:alert`, `hospital:updated`, `report:created`, `notification:created`, and `notification:updated`.

### AI Triage, Routing & Hospitals
- `POST /api/ai/analyze` — Runs multimodal AI classification and explainable `0–100` severity scoring.
- `POST /api/ai/check-duplicates` — Checks geospatial + semantic overlap against active incidents.
- `GET /api/incidents/:id/recommendations` — Returns ranked rescue squad recommendations with distance and ETA.
- `GET /api/hospitals/nearby?lat=&lng=&limit=` — Finds nearest hospitals ordered by Haversine distance.
- `POST /api/incidents/:id/notify-hospitals` — Dispatches first-aid notifications to nearby hospitals.
- `PATCH /api/hospital-alerts/:id` — Updates hospital alert status (`Notified`, `First Aid Dispatched`, `ER Trauma Standby`).
- `GET /api/route?fromLat=&fromLng=&toLat=&toLng=` — Proxies OSRM driving polyline routing with arterial fallback.

### Incidents, Teams, Resources & Notifications
- `POST /api/incidents` — Submits a new emergency report (`INC-2026-XXXX`), runs AI triage & duplicate check, alerts nearby hospitals when appropriate, and broadcasts via `/ws`.
- `PATCH /api/incidents/:id` — Updates incident status, severity override, public alert flag, or operator notes.
- `POST /api/incidents/:id/merge` — Merges a duplicate report into a primary incident.
- `POST /api/incidents/:id/assign` — Dispatches a rescue team to an incident.
- `PATCH /api/assignments/:id` — Updates squad assignment status (`Accepted`, `Rejected`, `En Route`, `On Site`, `In Progress`, `Completed` with completion metrics/report).
- `POST /api/teams/:id/location` — Streams live team GPS coordinates and availability status.
- `POST /api/resources` / `PATCH /api/resources/:id` — Adds or allocates emergency equipment and fleet inventory.
- `PATCH /api/notifications/:id/read` / `POST /api/notifications/read-all` — Marks alerts as read.

### Demo Simulator
- `POST /api/demo/mode` — Toggles `demoMode`.
- `POST /api/demo/simulate` — Executes live demonstration actions: `reset`, `new_flood`, `critical_fire`, `nearby_duplicate`, `team_accept`, `team_enroute`, `team_arrive`, `resolve_incident`.

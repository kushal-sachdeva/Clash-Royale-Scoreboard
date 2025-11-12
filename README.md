# Clash Royale Scoreboard

A fun little side project I hacked together in just a couple of hours for me and my friends’ Clash group.  
It’s basically a **live scoreboard** where we can add matchups, track wins/losses, and keep things a bit competitive.
A lightweight single-page app that lets Clash Royale groups log head-to-head matchups and track scores in real time. The UI, styling, and client logic all live in `index.html`, while Firebase Firestore provides shared persistence so every viewer sees the same scoreboard instantly.

### Features
- Add new matchups (Player A vs Player B, with optional mode/title).
- Update scores with a simple `+1` button.
- Cooldowns built in to prevent spam/tampering (2 min for +1, 10 min for reset/delete).
- Reset or delete matchups (two-tap confirmation with cooldown).
- Real-time sync using **Firebase Firestore** so everyone sees the same scores instantly.
- Hosted on GitHub Pages — no setup required.
## Project Structure

### Why?
Mostly for fun — we wanted a quick and easy way to keep score in our Clash Royale group without overthinking it.  
And honestly, it was cool to see how fast something like this can come together in just a short build sprint.
```
.
├── index.html   # static HTML + embedded CSS/JS powering the entire experience
└── README.md    # project overview and code walkthrough (this file)
```

### Tech Stack
- **HTML** (vanilla, no frameworks).
- **Firebase Firestore** for real-time database.
- **GitHub Pages** for hosting.
Because everything is bundled into a single HTML file, deploying the project is as simple as serving `index.html` from any static host (the original project uses GitHub Pages).

## Runtime Flow

1. **Bootstrap:** The page loads custom styles and then initializes Firebase using the compat CDN builds. A Firestore instance (`db`) is created and reused for all operations.
2. **Event wiring:** Button handlers are registered for creating matchups, incrementing scores, arming resets/deletes, and filtering. Global delegated click listeners keep the logic resilient to DOM re-renders.
3. **Realtime sync:** `listen()` attaches an `onSnapshot` listener on the `matchups` collection ordered by creation time. Whenever data changes, the UI is re-rendered from scratch and cached in `__docs` for countdown updates.
4. **Cooldown bookkeeping:** A `setInterval` loop runs every second to refresh badge states, button enablement, and cooldown overlays without re-rendering the entire list.

## Data Model (Firestore `matchups` documents)

| Field         | Type                                | Purpose |
|---------------|-------------------------------------|---------|
| `a`, `b`      | `string`                            | Player names shown on the card header (inline editable).
| `title`       | `string` (optional)                 | Extra context such as mode or tournament round.
| `sa`, `sb`    | `number`                            | Current score for player A and player B.
| `created`     | `firebase.firestore.Timestamp`      | Server timestamp used for ordering newest matchups first.
| `lastUpdate`  | `firebase.firestore.Timestamp/null` | Tracks the last `+1` action to enforce cooldowns.
| `resetArmAt`  | `firebase.firestore.Timestamp/null` | Start of the 10-minute reset confirmation window.
| `deleteArmAt` | `firebase.firestore.Timestamp/null` | Start of the 10-minute delete confirmation window.

Firestore server timestamps are set inside transactions to guarantee accurate cooldown calculations across clients.

## Key UI Behaviors

### Adding and Editing Matchups
- The "Add Matchup" button (or pressing <kbd>Enter</kbd> in any input) validates that both player names are provided before writing a new document with zeroed scores.
- Clicking a player name prompts the user to rename that side. The `rename` helper issues a targeted update for either `a` or `b`.

### Incrementing Scores with Cooldown Protection
- `bump(id, side)` wraps score increments in a Firestore transaction so concurrent clients cannot overwrite each other.
- A 2-minute (`SCORE_COOLDOWN_MS`) timer is enforced per matchup. Once any score is recorded, a transparent overlay disables the buttons until the cooldown expires.
- Toast notifications use a centrally positioned popup (`#toast`) with a backdrop to communicate success and cooldown messages.

### Reset/Delete Arm Windows
- Both reset and delete actions are two-step flows: the first click "arms" the action by storing a timestamp; the second click is only accepted after a 10-minute (`ARM_WINDOW_MS`) wait.
- Badges at the top of each card communicate whether the action is ready, armed, or still waiting, ensuring tamper resistance.

### Filtering and Live Updates
- The search box filters matchups client-side across player names and titles while the Firestore listener keeps `__docs` in sync with the backend.
- `updateBadges()` reuses the cached document state to update badge text, button states, and overlays every second without re-rendering cards, keeping the UI responsive even with frequent updates.

## Local Development

No build tooling is required. To experiment locally:

```bash
# from the repo root
python3 -m http.server 8000
# visit http://localhost:8000/index.html
```

You will need to ensure the configured Firebase project (`clashroyalescore-362e0`) is accessible from your environment, or swap in your own Firebase credentials.

## Opportunities for Enhancement

- **Authentication:** Gate score updates behind Firebase Authentication to prevent anonymous tampering.
- **Responsive layout tweaks:** Break long player names gracefully on smaller screens and offer a compact mobile layout.
- **Analytics/History:** Store per-matchup history to review past games, streaks, or leaderboards.
- **Offline safeguards:** Queue updates locally when the network drops and replay them once Firestore reconnects.

---

Just a small project, nothing serious — but it’s been fun to build and use.  
This document should provide enough context for new contributors to understand how the scoreboard works and where to extend it next.

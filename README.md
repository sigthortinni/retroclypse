# RETROCLYPSE

An 80s synthwave zombie shooter built in the browser with Three.js and plain HTML/CSS.
Single-player or peer-to-peer co-op multiplayer over WebRTC (PeerJS).

Play: **[retroclypse.web.app](https://retroclypse.web.app)**

## Features

- **7 maps**: DAY, NIGHT, BEACH, FOREST, DESERT, CITY, SNOW — each a fully different scene
- **3 game modes**: NORMAL (2-minute survival), WAVES (5 escalating waves), FREE PLAY (no zombies)
- **8 guns** in the in-game shop: Pistol, Paintball, AK-47, Machine Gun, Shotgun, Sniper, Rocket, Laser Cannon
- **Super-powers** via mystery balls: Ghost Mode, Super Speed, Mega Jump, Ammo, Flashlight, Shield, Grenade, Extra Life
- **Procedural zombies** with HP tiers, chase AI around obstacles, some carry pistols
- **Two-gun loadout** — press 1/2 to switch
- **Stim healing**, perks, coin economy, shop, loadout
- **Multiplayer co-op** via PeerJS (WebRTC P2P)
- **Friends-only leaderboard** via Firebase Firestore (optional, no personal info, group-scoped)
- **Kid-friendly**: no Google login required, local-first identity, self-chosen names

## Tech

- Three.js (via unpkg CDN + import map)
- PeerJS for WebRTC signaling
- Firebase Hosting (static site)
- Firebase Firestore (optional online leaderboard, configured via `FIREBASE_CONFIG` in `index.html`)
- Pure HTML/CSS/JS — no build step, no bundler

## Running locally

```bash
# Option A: Firebase's built-in serve
firebase serve --only hosting

# Option B: Python
python3 -m http.server 8000
```

Then open `http://localhost:5000` (Firebase) or `http://localhost:8000` (Python).

> Opening `index.html` directly via `file://` may work but some browsers block ES
> module imports from local files — use a local HTTP server for reliable testing.

## Deploying

Install the Firebase CLI, then:

```bash
firebase login
firebase use --add   # pick/create a Firebase project
firebase deploy --only hosting
```

`firebase.json` is already configured to deploy to the `retroclypse` Firebase Hosting site.

## Enabling the online leaderboard

1. In your Firebase project console, go to **Firestore Database → Create database → Test mode**.
2. In **Project settings → Your apps → Web app → SDK setup**, copy your `firebaseConfig` object.
3. Open `index.html` and find the `FIREBASE_CONFIG` block near the bottom of the `<script type="module">`. Paste your config, set `FIREBASE_ENABLED = true`.
4. Redeploy.

Recommended Firestore security rules (no-auth-required, anti-spam, group-scoped):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /leaderboard/{doc} {
      allow read: if true;
      allow create: if
        request.resource.data.score is number &&
        request.resource.data.score >= 0 &&
        request.resource.data.score < 10000 &&
        request.resource.data.name is string &&
        request.resource.data.name.size() > 0 &&
        request.resource.data.name.size() <= 16 &&
        request.resource.data.groupCode is string &&
        request.resource.data.groupCode.size() > 0 &&
        request.resource.data.groupCode.size() <= 16;
      allow update, delete: if false;
    }
  }
}
```

## Multiplayer

Everything real-time (positions, zombies, shots) flows **peer-to-peer over WebRTC**.
PeerJS provides the initial signaling handshake for free, after which data goes directly
between browsers.

- Click **HOST ROOM** on the menu → get a `FPS-xxxx` code.
- Share the code. Others click **JOIN ROOM** and paste the code.
- Host is authoritative for zombies. Guest shots are routed to host for damage resolution.

## Controls

- **WASD** — move
- **Mouse** — aim
- **Click** — shoot (hold for Machine Gun / Laser Cannon)
- **R** — reload
- **Space** — jump
- **1 / 2** — switch active weapon
- **E** — use stim (heal to full)
- **F** — throw grenade
- **M** — toggle music
- **ESC** — pause

## Files

- `index.html` — the entire game (HTML + CSS + all JS in one file)
- `firebase.json` — Firebase Hosting config
- `.firebaseignore` — files to exclude from deploy uploads
- `.gitignore` — files to exclude from git
- This `README.md`

## License

Personal / hobby project. No explicit license.

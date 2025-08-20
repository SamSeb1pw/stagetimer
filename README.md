# StageTimer — Remote Presentation Timer for Live Stage Sync

[![Releases](https://img.shields.io/badge/Releases-latest-blue)](https://github.com/SamSeb1pw/stagetimer/releases)

![StageTimer Hero](https://images.unsplash.com/photo-1517694712202-14dd9538aa97?auto=format&fit=crop&w=1600&q=60)

Live sync presentation timer for stage shows, conference rooms, classrooms, and rehearsals. Control the timer from a central controller. Display the timer on multiple screens. Keep all clients in sync with WebSocket updates.

Topics: express · nextjs · presentation-time · presentation-timer · remote-control · remotecontroltimer · stage-timer · stagetimer · timer · websocket

Table of Contents
- Features
- How it works
- Demo and screenshots
- Quick start
- Local development
- Production build
- WebSocket protocol
- API endpoints
- Configuration
- Deployment tips
- Security and best practices
- Contributing
- Releases
- License
- Contact

Features
- Central controller UI to start, pause, reset, and set timer.
- Live updates to all connected displays using WebSocket.
- Express backend for REST endpoints and WebSocket upgrade handling.
- Next.js frontend for the controller and display pages.
- Role separation: controller vs display client.
- Persistent state option (in-memory default, optional Redis).
- Adjustable tick rate and offset for stage drift handling.
- Client reconnection and delta sync for seamless recovery.
- Simple REST API for remote automation and integration.

How it works
- The server runs Express for REST APIs and a WebSocket server for real-time sync.
- A controller client sends commands (start, pause, reset, adjust) to the server.
- The server broadcasts state changes to all display clients over WebSocket.
- Each display applies server time or server offsets and shows the synced timer.
- On reconnect, a client requests current state and the server replies with authoritative time and mode.
- Optionally, store current state in Redis to survive server restarts.

Demo and screenshots
![Controller UI](https://images.unsplash.com/photo-1508385082359-f92c7d0f6c52?auto=format&fit=crop&w=1200&q=60)
Controller UI shows remaining time, big start/pause/reset buttons, and per-segment markers.

![Audience Display](https://images.unsplash.com/photo-1519337265831-281ec6cc8514?auto=format&fit=crop&w=1200&q=60)
Display UI shows large digits, optional color cues, and stage offset.

Quick start (local)
1. Clone the repo
   git clone https://github.com/SamSeb1pw/stagetimer.git
2. Install dependencies
   cd stagetimer
   npm install
3. Copy example env
   cp .env.example .env
4. Run dev servers
   npm run dev

Local development runs two parts:
- Next.js frontend on port 3000
- Express + WebSocket backend on port 4000 (proxied by Next in some setups)

Local environment variables
- PORT=4000           # backend port
- NEXT_PUBLIC_WS_URL  # ws://localhost:4000 for dev
- REDIS_URL           # optional, for persistent state
- SESSION_SECRET      # optional, for auth if enabled

npm scripts
- npm run dev — start both frontend and backend in dev mode
- npm run build — build Next.js for production
- npm run start — start the production server (Express + built Next app)
- npm run lint — run linters
- npm run test — run tests

Architecture
- Next.js serves the controller UI and display pages.
- Express hosts REST endpoints and upgrades connections to WebSocket.
- ws (or native WebSocket server) handles low-latency messages.
- Optional Redis stores the last known state and session info.
- Static assets served by Next.js for display reliability.

WebSocket protocol
Messages use JSON. Keep messages small and predictable.

Basic message types:
- controller -> server
  - { type: "start", at: 1234567890 }         # start at epoch ms (server will accept server-side time)
  - { type: "pause" }
  - { type: "reset", duration: 600000 }       # set timer duration in ms
  - { type: "adjust", offset: -500 }          # adjust server time by offset ms
  - { type: "heartbeat" }                     # keepalive from clients

- server -> clients
  - { type: "state", mode: "running", timeLeft: 450000, tick: 1620000000000 }
  - { type: "paused", timeLeft: 300000 }
  - { type: "reset", duration: 600000 }
  - { type: "error", message: "reason" }

Client reconnection flow
- Client opens WS, sends { type: "hello", role: "display" }
- Server replies with current state: { type: "state", ... }
- Client syncs UI to server time and begins local tick based on server tick rate.

API endpoints
- GET /api/state — returns current state { mode, timeLeft, duration, tick }
- POST /api/control — accept JSON { action: "start" | "pause" | "reset" | "adjust", payload }
- GET /api/version — returns current build and commit SHA

Use the REST API for automated control or fallback when WebSocket is unavailable.

Configuration keys (example)
- TICK_RATE=1000         # ms between server ticks
- MAX_CLIENT_DRIFT=2000  # ms allowed before force resync
- REDIS_URL              # redis://host:port
- ALLOW_ANONYMOUS        # true/false to allow controller without auth

Running production
1. Build Next app
   npm run build
2. Start server
   npm run start

The start script runs Express and serves the built Next app. Use a process manager like PM2 or systemd in production.

Docker
- Use the included Dockerfile to containerize the app.
- Build:
  docker build -t stagetimer:latest .
- Run:
  docker run -e PORT=4000 -p 4000:4000 stagetimer:latest

Deploy tips
- Terminate TLS at a reverse proxy (NGINX, Traefik) and forward WS upgrades.
- Use sticky sessions or separate WebSocket routing for scale.
- Use Redis for shared state across instances.
- Set proper CORS and origin checks for safety.
- Monitor latency and packet loss; WS will reconnect but drift may appear.

Security and best practices
- Use HTTPS and WSS in production.
- Restrict controller access via auth and roles.
- Validate messages server-side and sanitize payloads.
- Rate-limit control APIs to prevent abuse.
- Keep dependencies up to date and run security scans.

Contributing
- Fork the repo.
- Create a feature branch.
- Run tests and linting.
- Submit a pull request with a clear description and test coverage.

Release downloads
Download the release asset from the Releases page and execute the file as needed. Visit the releases page and pick the asset for your platform:
https://github.com/SamSeb1pw/stagetimer/releases

You can also click the badge at the top to open the Releases page:
[![Download Release](https://img.shields.io/badge/Download%20Release-Get%20Binary-green)](https://github.com/SamSeb1pw/stagetimer/releases)

If you use a packaged release, the asset name will list the target OS. Download and run the asset, for example on Linux:
- chmod +x stagetimer-linux
- ./stagetimer-linux --env .env

If a release asset does not run or you prefer source, build from source:
- npm install
- npm run build
- npm run start

Testing and diagnostics
- Use headless browser tests for display rendering.
- Simulate network drop and reconnection to verify sync.
- Use log output for server tick and client heartbeat events.
- Use simple scripts to send POST /api/control for automated checks.

Common troubleshooting
- If displays fall out of sync, check server tick rate and client clock skew.
- If WS fails to connect behind proxy, ensure Upgrade and Connection headers pass through.
- If state resets on restart, enable REDIS to persist state.

Examples
- Start a 10 minute timer from controller:
  { type: "reset", duration: 600000 } then { type: "start" }
- Pause at any time:
  { type: "pause" }
- Adjust for stage drift:
  { type: "adjust", offset: -2000 }  # subtract 2 seconds

Integrations
- Use REST API with automation platforms to trigger timers.
- Use lightweight hooks to integrate with AV switchers and show control software.
- Use WebSocket client to embed a remote display in existing pages.

Changelog and releases
- Check releases for binaries, release notes, and migration instructions.
- Download and execute appropriate asset for your platform from:
  https://github.com/SamSeb1pw/stagetimer/releases

License
- MIT License. See LICENSE file for full text.

Contact
- Report issues via GitHub Issues.
- Open pull requests for fixes and features.
- For urgent production problems, open an issue with logs and reproduction steps.
An arcade-style 3D racing game that runs in the browser
Built with React, React Three Fiber (Three.js), Rapier physics, Zustand, and Firebase.
Disclaimer: This is an independent fan-inspired project and is not affiliated with Gameloft or the Asphalt franchise. Use only original or properly licensed assets (see Asset Sources & Licensing).
Table of Contents
Features
Tech Stack
Getting Started
Project Structure
Game Architecture
Firebase Integration
Asset Sources & Licensing
Performance Tips
Roadmap
Features
Arcade vehicle physics — raycast-vehicle model with tunable top speed, acceleration, handling, and grip.
Nitro system — 3-segment nitro bar (Asphalt-style), flame effects, FOV kick, speed lines.
Drifting — handbrake/drift mechanic with tire trails, smoke, and nitro charge on drift.
Ramps & airborne sections — jump ramps with air control and landing impact.
Takedowns — ram opponents into walls/obstacles to earn bonus nitro and a takedown banner.
AI opponents — waypoint-following rivals with difficulty tiers and rubber-banding.
Garage — pick, rotate/preview, and upgrade cars; progression with credits.
Online features (Firebase) — authentication, cloud saves, global leaderboards, ghost replays.
Data-driven design — cars, tracks, and controls are configured through plain config files.
Tech Stack
Area	Technology
UI framework	React 18 + Vite
3D rendering	Three.js via @react-three/fiber
3D helpers	@react-three/drei (loaders, camera helpers, Environment, etc.)
Physics	@react-three/rapier (Rapier) — Cannon is a valid alternative
Post-processing	@react-three/postprocessing (bloom, motion blur, vignette)
Global state	Zustand
Backend	Firebase (Auth, Firestore, Storage, Hosting)
Audio	Howler.js or the Web Audio API
Getting Started
Prerequisites
Node.js 18+
A Firebase project (free Spark plan is enough for development)
Installation
bash
# 1. Clone the repository
git clone <your-repo-url> racing-game
cd racing-game

# 2. Install dependencies
npm install

# 3. Create your environment file
cp .env.example .env
# Fill in your Firebase credentials (see below)

# 4. Start the dev server
npm run dev
Environment Variables (.env)
bash
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
Scripts
Command	Description
npm run dev	Start the Vite dev server
npm run build	Create a production build
npm run preview	Preview the production build locally
firebase deploy	Deploy to Firebase Hosting (after npm run build)
Project Structure
racing-game/
├── public/
├── src/
├── .env.example
├── firestore.rules
├── firebase.json
├── package.json
└── vite.config.js
public/ — Static assets served as-is
Files here are loaded at runtime by URL (e.g. /models/cars/car_chiron.glb). Large binary assets live here so Vite does not bundle them.
public/
├── audio/
│   ├── sfx/        # Sound effects
│   └── music/      # Background music
└── models/         # All 3D assets (.glb / .gltf)
    ├── cars/
    ├── tracks/
    └── props/
Path	Purpose
audio/sfx/	Short sound effects: nitro.mp3, drift.mp3, crash.mp3, engine.mp3 (looped engine sample whose pitch is driven by RPM).
audio/music/	Background music: one track for menus and one or more for in-race.
models/cars/	Vehicle models, e.g. car_chiron.glb, car_vulcan.glb. Each should have separate meshes for body and four wheels so wheels can spin/steer independently.
models/tracks/	Track/environment models, e.g. track_tokyo.glb, track_nevada.glb. Ideally split into visual mesh and a simplified collision mesh.
models/props/	Interactive/decorative objects: ramp.glb (jump ramp), nitro_bottle.glb (pickup), barriers.glb.
Tip: Compress models with gltf-transform (Draco/Meshopt + WebP textures) before committing them — it often cuts file size by 70–90%.
src/ — Application source code
src/assets/ — Bundled 2D assets
Assets imported directly in code (processed and hashed by Vite).
Path	Purpose
icons/	UI icons (nitro, trophy, credits, settings, etc.).
textures/	Effect textures: skid_mark.png (tire trail decal), lens_flare.png (sun/nitro flare), smoke.png (drift/exhaust particle sprite).
Fonts	Arcade/racing fonts, referenced from index.css.
src/config/ — Data-driven game balance
All tunable values live here. Balancing the game should never require touching gameplay code.
File	Purpose
cars.config.js	Per-car stats and metadata: id, display name, model path, topSpeed, acceleration, handling, nitroEfficiency, mass, price, upgrade tiers, wheel offsets.
tracks.config.js	Per-track data: model path, spawn/grid positions, ordered checkpoint list, AI waypoints, lap count, pickup locations, ramp positions.
controls.config.js	Keybindings (WASD / Arrow keys, Space for nitro/drift, etc.). Used by useKeyboardControls; makes remapping easy.
src/hooks/ — Reusable React hooks
File	Purpose
useKeyboardControls.js	Listens for key events and maps them to input flags (accelerate, brake, left, right, nitro, drift) using controls.config.js.
useGameAudio.js	Manages SFX and music playback; adjusts engine pitch/volume from RPM and applies volume settings from useSettingsStore.
useWindowSize.js	Tracks viewport size for responsive HUD scaling and canvas resizing.
src/stores/ — Global state (Zustand)
File	Purpose
useGameStore.js	The game state machine: 'MENU' | 'COUNTDOWN' | 'RACING' | 'PAUSED' | 'GAMEOVER'. Also holds the selected car/track, current lap, race timer, and nitro/speed values read by the HUD.
usePlayerStore.js	Persistent player profile: race position/results, best lap times, credits, owned cars, car upgrades. Synced with local storage and Firebase.
useSettingsStore.js	User settings: graphics quality (low / medium / high), SFX/music volume, control preferences.
src/services/ — Firebase & backend layer (add this folder)
File	Purpose
firebase.js	Initializes the Firebase app and exports auth, db, storage.
authService.js	Sign-in/sign-up (Google, email, anonymous), sign-out, auth state listener.
saveService.js	Cloud save/load of the player profile (credits, garage, upgrades) in Firestore.
leaderboardService.js	Submit lap/race times and fetch top-N rankings per track.
ghostService.js (optional)	Upload/download compressed ghost replays for time-trial mode.
src/game/ — 3D engine and gameplay logic (core of the project)
game/core/ — Canvas and scene setup
File	Purpose
GameCanvas.jsx	Creates the R3F <Canvas>: renderer settings, DPR cap, shadow map, tone mapping, post-processing, and resize handling.
GameScene.jsx	Composes everything inside the canvas: <Physics>, lighting, environment/skybox, track, player car, AI cars, pickups, camera, and effects.
game/camera/ — Third-person camera system
File	Purpose
FollowCamera.jsx	Chase camera behind the car with smoothed (lerp) position and look-at; adapts distance/height with speed.
CameraEffects.jsx	Dynamic effects: screen shake on collisions, FOV kick and slight pull-back when nitro is active.
game/physics/ — Physics layer (Rapier or Cannon)
File	Purpose
VehicleRaycaster.jsx	Raycast vehicle model: suspension springs, engine/traction force, tire friction, lateral grip, drift behavior, anti-roll, and air control.
Colliders.jsx	Static and dynamic colliders: track boundaries, barriers, ramps (jump launchers), and obstacles. Uses collision groups from utils/collisionFilters.js.
game/entities/ — Objects that exist in the world
entities/car/
File	Purpose
Car.jsx	The player car: connects the 3D model, the physics body, and input. Applies engine force, steering, nitro, and drift; reports speed/nitro to stores.
CarWheel.jsx	Wheel visuals: rolling rotation based on speed and steering angle on front wheels.
CarAudio.jsx	Positional 3D audio attached to the car (engine, tire squeal, nitro whoosh).
entities/ai/
File	Purpose
AICar.jsx	An opponent car using the same vehicle model as the player but driven by AI inputs.
WaypointFollower.js	AI steering/throttle logic: follows track waypoints, looks ahead for corners, adds speed variation per difficulty, supports rubber-banding.
entities/track/
File	Purpose
Track.jsx	Loads the track .glb, builds trimesh/convex colliders, and places props from the track config.
Checkpoints.jsx	Invisible trigger volumes that validate lap progress (prevents cutting corners) and compute live race ranking.
Pickups.jsx	Collectible nitro bottles and coins: spawn, rotate/animate, collect, respawn timers.
entities/props/
File	Purpose
(components)	Breakable or interactive scenery: wooden crates, signs, cones. Triggers sparks/debris on impact.
game/effects/ — Arcade visual effects
File	Purpose
NitroFlames.jsx	Flames from the exhaust while boosting (shader mesh or particle system).
TireTrails.jsx	Skid marks drawn on the road while drifting/braking.
SparkParticles.jsx	Sparks when scraping walls or other cars.
SpeedLines.jsx	Wind/speed streaks at screen edges during nitro.
src/ui/ — 2D interface (HTML/CSS over the 3D canvas)
ui/hud/ — In-race heads-up display
File	Purpose
Speedometer.jsx	Speed readout (km/h) and current gear.
NitroBar.jsx	Nitro gauge divided into 3 segments, with active/ready states.
MiniMap.jsx	Small track map showing the positions of all cars (rendered with canvas/SVG from waypoints).
RacePosition.jsx	Current rank (1st, 2nd…), lap counter, current/best lap time.
ui/menus/ — Navigation screens
File	Purpose
MainMenu.jsx	Title screen: Play, Garage, Leaderboard, Settings, Login.
GarageMenu.jsx	Car selection with rotating 3D preview, stats comparison, upgrades, and purchases.
TrackSelectMenu.jsx	Track selection with previews, best times, and lap/difficulty options.
PauseMenu.jsx	Pause overlay: Resume, Restart, Settings, Quit to menu.
ui/overlays/ — Transient overlays
File	Purpose
Countdown.jsx	"3… 2… 1… GO!" countdown at race start (switches state COUNTDOWN → RACING).
TakedownBanner.jsx	Animated banner when you take out an opponent, with the bonus awarded.
src/utils/ — Helpers
File	Purpose
math.js	lerp, clamp, damping, vector projection, angle wrapping, and other math helpers.
collisionFilters.js	Collision-group bitmasks (car, wall, pickup, trigger, AI) so only relevant objects collide.
storage.js	Local persistence (LocalStorage/IndexedDB) for offline progress; used as a cache before syncing to Firebase.
Root files in src/
File	Purpose
App.jsx	Top-level orchestrator: renders UI screens or the 3D canvas based on useGameStore state.
index.css	Global styles, CSS variables, arcade fonts, HUD layout.
main.jsx	React entry point; mounts <App />.
Game Architecture
State flow
MENU ──► COUNTDOWN ──► RACING ──► GAMEOVER
  ▲                     │  ▲          │
  │                     ▼  │          │
  │                   PAUSED          │
  └───────────────────────────────────┘
Per-frame loop (inside useFrame)
Read input flags from useKeyboardControls.
Apply forces/steering to the vehicle physics body.
Update nitro, drift, and air state.
Update camera, effects, and audio from the car state.
Write only the values the HUD needs to the store at a throttled rate (e.g. 10–15 Hz) to avoid React re-render overhead.
Design principles
Config over code: balance in src/config/.
Physics in the loop, UI in React: do not store per-frame values (position, velocity) in React state; use refs or transient Zustand updates.
Separation of concerns: gameplay in game/, interface in ui/, persistence in services/.
Firebase Integration
Firebase product	Usage
Authentication	Google / email login, with anonymous login to play instantly.
Firestore	Player profiles, owned cars & upgrades, leaderboards, race history.
Storage	Optional: ghost replay files, user avatars.
Hosting	Deploy the production build.
Suggested Firestore collections:
users/{uid}                  → displayName, credits, ownedCars[], upgrades{}, settings
leaderboards/{trackId}/times → { uid, displayName, carId, lapTime, createdAt }
ghosts/{trackId}/{uid}       → replay metadata + Storage path
Important: Never trust the client for leaderboards. Write strict firestore.rules and, for competitive integrity, validate submitted times with Cloud Functions (e.g. reject impossible lap times).
Asset Sources & Licensing
Always check the license of every asset (CC0, CC-BY, royalty-free, or paid) and keep credits in a CREDITS.md file. Do not use ripped models from commercial games.
3D models (cars, tracks, props)
Source	Notes
Sketchfab	Huge catalog; filter by Downloadable and license; exports .glb. Best for cars.
Kenney	CC0 low-poly packs, including a racing kit. Great for prototyping.
Quaternius	CC0 low-poly models (vehicles, props, environment).
Poly Pizza	Free low-poly models, many CC0/CC-BY.
CGTrader	Free and paid models; good for higher-quality cars.
TurboSquid	Paid, professional models; check the royalty-free license terms.
Free3D	Mixed quality; verify licenses.
itch.io game assets	Indie asset packs, many for racing games.
Blender	Create or modify models yourself; export as .glb.
Audio
Source	Notes
Freesound	Engine, crash, nitro SFX (check each license).
Pixabay	Free music and SFX.
OpenGameArt	Free game audio and art.
Zapsplat	Free SFX with attribution.
Textures / environment maps
Source	Notes
Poly Haven	CC0 HDRIs, textures, and models.
ambientCG	CC0 PBR textures (asphalt, concrete, etc.).
Model preparation checklist
Export as .glb (glTF binary).
Keep car triangle count reasonable (~30k–80k for the player car; fewer for AI).
Name meshes clearly: Body, Wheel_FL, Wheel_FR, Wheel_RL, Wheel_RR.
Set the origin at the car's center, with the forward axis consistent across all cars.
Compress with gltf-transform (Draco/Meshopt, resize/convert textures).
Create a simple collision mesh for tracks instead of using the visual mesh.
Performance Tips
Cap pixel ratio: dpr={[1, 1.5]} on the Canvas.
Use instancing for repeated props (barriers, trees, lights).
Use LOD for distant objects; bake lighting into track textures when possible.
Offer low / medium / high presets (shadows, post-processing, draw distance).
Avoid creating objects in useFrame; reuse vectors/quaternions.
Preload models with useGLTF.preload() and show a loading screen.
Roadmap
 Core driving: acceleration, steering, camera
 Nitro, drift, and effects
 Track with checkpoints and lap logic
 AI opponents
 Ramps, pickups, and takedowns
 HUD and menus
 Garage and progression
 Firebase auth, cloud saves, and leaderboards
 Mobile/touch and gamepad support
 Multiplayer (WebSocket/WebRTC) and ghost replays

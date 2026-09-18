your-username# NMDE 404 — Multiplayer Sketch Specification (v2)
### Reference document for LLM-assisted development

---

## Overview

This document defines the technical contract for multiplayer sketches in NMDE 404. Your sketch runs as a single HTML file in a shared WebSocket session hosted by the instructor. All students connect to the same server via a public ngrok URL. The instructor controls session state (reset, kick, and a live preview switcher) from an admin panel.

Your job: build a Three.js sketch inside this system. The server, login screen, and multiplayer plumbing are already defined. Follow this spec exactly and your LLM will generate code that works without modification.

Submissions are reviewed and, if needed, patched before the live session — but a file that follows this spec exactly needs no changes.

---

## System Architecture

```
Your HTML file (local or on RIT server)
    ↕ WebSocket (wss://)
ngrok tunnel
    ↕
Node.js server (Docker, port 3000, instructor's Mac)
    ↕ broadcasts to all connected clients
Everyone else's HTML file
```

- **Server:** Node.js + WebSocket. Instructor runs it. You never touch it.
- **Transport:** WebSocket over ngrok. The instructor posts the `wss://` URL at the start of class.
- **Session:** All players share one server instance. No rooms, no lobbies.
- **Your file:** One self-contained HTML file. No build step, no bundler, no framework.

---

## File Rules

**One file.** All HTML, CSS, and JavaScript in a single `.html` file. No external JS files, no imports from your own server.

**Name it:** `firstname.html` — exactly matching the name you will log in with. This name must be unique among your classmates — if two students would collide (two "Alex"s, etc.), pick a variant (`AlexP.html`) and confirm it with the instructor before submitting.

**Three.js version:** r128 only. Load via CDN script tags:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
```

Add loaders as needed from the same CDN:
```html
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>
```

**No other multiplayer libraries.** No PeerJS, no Socket.io, no WebRTC. WebSocket only.

**Required description comment.** Place this block at the very top of your `<script>` tag, filled in. This is what gets reviewed before your code is read line by line — it's how the instructor knows what your sketch is *supposed* to do, so it can be checked against what it actually does:

```javascript
// SKETCH: one sentence — what is this?
// MOVE: what the arrow keys do (skip only if it's literally "moves my character")
// SHIFT: what your special effect does
```

---

## Asset Hosting

All assets (3D models, textures, audio, images) must be hosted on the RIT server and referenced with **absolute URLs**. Never use relative paths.

**Upload via FTP** to your RIT web space:
```
Host:     your-username.cad.rit.edu
Protocol: FTP
Path:     /your-username/sketch/your-project-name/
```

**Reference in your HTML:**
```javascript
// Correct — absolute URL
gltfLoader.load('https://your-username.cad.rit.edu/yourusername/sketch/project/model.gltf', ...);
const audio = new Audio('https://your-username.cad.rit.edu/yourusername/sketch/project/sound.mp3');

// Wrong — relative path (breaks when HTML is opened from a different location)
gltfLoader.load('model.gltf', ...);
```

**Supported formats:**
- 3D models: `.gltf` (preferred — self-contained, no external `.bin` files)
- Audio: `.mp3`
- Images/textures: `.png`, `.jpg`
- Video: `.mp4`

**GLTF export note:** When exporting from Blender, choose "glTF Embedded (.gltf)" to produce a single file with no external dependencies.

---

## Login Overlay

Copy this HTML block verbatim before your first `<script>` tag. Change only the title text ("Project Name" and "0X").

```html
<div id="notif"></div>

<div id="login-overlay">
  <div class="ll">
    <div class="li">
      <div>
        <p class="ls">Community Dimensional Draw</p>
        <p class="lh">Project <b>0X</b></p>
      </div>
      <div class="lg">
        <p class="ll-label">Name</p>
        <input class="ll-input" id="l-name" type="text" placeholder="First Name"
          maxlength="20" onkeydown="if(event.key==='Enter')doJoin()">
      </div>
      <div class="lg">
        <p class="ll-label">Color</p>
        <div class="swatches" id="l-swatches"></div>
      </div>
      <div class="lg">
        <p class="ll-label">Server</p>
        <input class="ll-input" id="l-server" type="text"
          value="wss://your-ngrok-url-here"
          placeholder="wss://your-ngrok-url"
          onkeydown="if(event.key==='Enter')doJoin()">
      </div>
      <div class="l-err" id="l-err"></div>
      <button class="join-btn" onclick="doJoin()">Join</button>
    </div>
  </div>
  <div class="lr">
    <span class="lr-text"><b>Mike Minerva</b> &nbsp; Professor of Practice, New Media Design</span>
    <span class="lr-tag">NMDE 404 Interactive 4</span>
  </div>
</div>
```

Copy this CSS block verbatim into your `<style>` tag:

```css
#login-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.85);
  display: flex; align-items: stretch; z-index: 1000;
  opacity: 1; transition: opacity 0.5s ease;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
#login-overlay.dissolve { opacity: 0; pointer-events: none; }
.ll { width: 400px; flex-shrink: 0; padding: 70px 8px 50px 70px; display: flex; align-items: flex-start; }
.li { display: flex; flex-direction: column; gap: 28px; width: 100%; }
.ls { font-size: 18px; color: #fff; font-weight: 400; white-space: nowrap; }
.lh { font-size: 56px; line-height: 1; color: #ececec; font-weight: 400; white-space: nowrap; }
.lh b { font-weight: 700; }
.lg { display: flex; flex-direction: column; gap: 10px; width: 100%; }
.ll-label { font-size: 18px; color: #7f7c76; font-weight: 400; }
.ll-input {
  background: transparent; border: none;
  border-bottom: 1px solid rgba(255,255,255,0.3);
  color: #fff; font-size: 18px; font-family: inherit;
  padding: 0 0 8px 0; width: 100%; outline: none; transition: border-color 0.2s;
}
.ll-input::placeholder { color: rgba(255,255,255,0.2); }
.ll-input:focus { border-bottom-color: rgba(255,255,255,0.7); }
.swatches { display: grid; grid-template-columns: repeat(4, 36px); gap: 10px; }
.swatch {
  width: 36px; height: 36px; border-radius: 50%;
  cursor: pointer; border: none; padding: 0;
  transition: transform 0.15s, box-shadow 0.15s;
}
.swatch:hover { transform: scale(1.12); }
.swatch.sel { box-shadow: 0 0 0 2px rgba(0,0,0,0.85), 0 0 0 4px var(--c); }
.join-btn {
  background: #fb5579; border: none; border-radius: 3px;
  color: #fff; font-size: 28px; font-family: inherit;
  font-weight: 700; padding: 12px 20px;
  cursor: pointer; transition: background 0.15s; line-height: 1;
}
.join-btn:hover { background: #e63e60; }
.l-err { font-size: 11px; color: #fb5579; min-height: 14px; }
.lr {
  position: fixed; right: 0; top: 0; bottom: 0; width: 54px;
  display: flex; flex-direction: column; align-items: center;
  justify-content: flex-start; gap: 60px; padding-top: 70px;
}
.lr-text { writing-mode: vertical-lr; font-size: 13px; color: rgba(255,255,255,0.4); white-space: nowrap; }
.lr-text b { font-weight: 700; color: rgba(255,255,255,0.55); }
.lr-tag { writing-mode: vertical-lr; background: #fb5579; color: #fff; font-size: 13px; font-weight: 700; white-space: nowrap; padding: 10px 16px; }
.name-flag { position: fixed; pointer-events: none; white-space: nowrap; z-index: 500; transform: translate(-50%, -100%); }
.name-flag .label { background: #fb5579; color: #fff; font-size: 11px; font-weight: 600; letter-spacing: 0.5px; padding: 3px 8px; border-radius: 2px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; }
#notif { position: fixed; top: 40px; left: 50%; transform: translateX(-50%); font-size: 11px; letter-spacing: 2px; text-transform: uppercase; color: rgba(255,255,255,0.85); background: rgba(0,0,0,0.55); padding: 5px 14px; border-radius: 3px; opacity: 0; transition: opacity 0.3s; pointer-events: none; white-space: nowrap; z-index: 600; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; }
#notif.show { opacity: 1; }
```

---

## WebSocket Protocol

### Naming your message type

Every message you send that carries your sketch's own state (position, effect state, whatever) uses one custom `type` string. **Prefix it with your first name** so it can't collide with anyone else's — e.g. `'ada-move'`, `'sam-draw'`. With 30+ students on one server, two people independently picking `type:'move'` will break both of their sketches silently (each will think the other's updates are its own).

### Message types your client sends

```javascript
// On join (sent automatically by the login system)
{ type: 'join', name: 'Ada', color: '#4A90D9' }

// Your position/state update — send at ~20fps from your render loop
// Use your prefixed type name, e.g. 'ada-move'
{ type: 'ada-move', /* your state data here */ }
```

Nothing else. In particular: **never send `type: 'reset'`** — reset is instructor-only (see below). If you want a "clear my own trail" action, do it entirely locally; don't tell the server about it.

### Message types your client receives

```javascript
// Existing players when you first join
{ type: 'players', players: [ { name, color }, ... ] }

// New player joined
{ type: 'join', name, color }

// Player left
{ type: 'leave', name }

// Instructor reset — clear your scene state
{ type: 'reset' }

// Instructor broadcast mode — optional, reserved for class-wide moments.
// Most sketches can ignore this entirely. It is NOT related to your Shift
// effect (see Key Bindings Contract below) — that is always on and yours alone.
{ type: 'mode', bonus: true/false }

// You were removed by instructor
{ type: 'kicked', message }

// Your name is rejected — already taken, or banned after a kick
{ type: 'join-rejected', reason }

// Another player's state update, in their own message type
{ type: 'their-prefixed-type', name, /* their state data */ }
```

---

## Multiplayer Boilerplate

Paste this entire block inside your `<script>` tag. Fill in the sections marked `FILL IN` — there are five: `MY_MSG_TYPE`, `addRemote`, `removeRemote`, `connectToServer`, the `reset` handler, and the custom-message handler in `onmessage`.

```javascript
// ── FILL IN: your message type, prefixed with your name ───────────────
const MY_MSG_TYPE = 'firstname-move'; // e.g. 'ada-move'

// ── Color swatches ──────────────────────────────────────────────────
const MP_COLORS = ['#4A90D9','#fb5579','#2E8B57','#9B59B6','#E07840','#8B7EB8','#3CB371','#C8B820'];
let myColor = MP_COLORS[0];
(function() {
  const el = document.getElementById('l-swatches');
  MP_COLORS.forEach(col => {
    const btn = document.createElement('button');
    btn.className = 'swatch' + (col === myColor ? ' sel' : '');
    btn.style.cssText = 'background:' + col + ';--c:' + col;
    btn.onclick = () => {
      myColor = col;
      document.querySelectorAll('.swatch').forEach(s => s.classList.remove('sel'));
      btn.classList.add('sel');
    };
    el.appendChild(btn);
  });
})();

// ── Notification toast ───────────────────────────────────────────────
let notifTimer;
function notify(msg) {
  const el = document.getElementById('notif');
  el.textContent = msg; el.className = 'show';
  clearTimeout(notifTimer);
  notifTimer = setTimeout(() => { el.className = ''; }, 2500);
}

// ── Name flags ───────────────────────────────────────────────────────
// Builds the flag as DOM nodes (not innerHTML) so a player's name can
// never be interpreted as markup. Do not change this to string-concat HTML.
function makeFlag(name, colorHex) {
  const d = document.createElement('div');
  d.className = 'name-flag';
  const label = document.createElement('div');
  label.className = 'label';
  label.style.background = colorHex;
  label.textContent = name;
  d.appendChild(label);
  document.body.appendChild(d);
  return d;
}
function positionFlag(flag, worldPos) {
  // worldPos is a THREE.Vector3 in world space.
  // Requires a `camera` variable to exist in scope — make sure your
  // THREE.PerspectiveCamera is assigned to a variable literally named `camera`.
  const v = worldPos.clone().project(camera);
  if (v.z > 1) { flag.style.display = 'none'; return; }
  flag.style.display = '';
  flag.style.left = ((v.x * 0.5 + 0.5) * window.innerWidth) + 'px';
  flag.style.top  = ((-v.y * 0.5 + 0.5) * window.innerHeight - 20) + 'px';
}

// ── Remote player registry ───────────────────────────────────────────
const remotes = {};  // name → { flag, ...your per-player state }

function addRemote(name, colorHex) {
  if (remotes[name]) return;
  const flag = makeFlag(name, colorHex);

  // ── FILL IN: create visual representation for this remote player ──
  // Example: const mesh = new THREE.Mesh(geometry, material); scene.add(mesh);
  // Store anything you need to update this player later.

  remotes[name] = {
    flag,
    color: new THREE.Color(colorHex),
    // ...your per-player state
  };
}

function removeRemote(name) {
  const r = remotes[name];
  if (!r) return;
  r.flag.remove();

  // ── FILL IN: remove this player's visuals from the scene ──
  // Example: scene.remove(r.mesh); r.mesh.geometry.dispose();

  delete remotes[name];
}

// ── WebSocket ────────────────────────────────────────────────────────
let ws = null;
let myName = '';
let myFlag = null;
let bonusMode = false; // set from the instructor's optional broadcast — see note above
let lastSend = 0;

function connectToServer(url, name, colorHex) {
  myName = name;
  myFlag = makeFlag(name, colorHex);

  // ── FILL IN: set your experience's color/state from login ──
  // Example: myCharacter.color = new THREE.Color(colorHex);

  ws = new WebSocket(url);

  ws.onopen = () => {
    ws._ok = true;
    ws.send(JSON.stringify({ type: 'join', name, color: colorHex }));
    document.getElementById('login-overlay').style.display = 'none';
    notify(name + ' joined');
  };

  ws.onmessage = (e) => {
    try {
      const d = JSON.parse(e.data);

      if (d.type === 'players') {
        d.players.forEach(p => { if (p.name !== myName) addRemote(p.name, p.color); });
      }
      if (d.type === 'join' && d.name !== myName) {
        addRemote(d.name, d.color);
        notify(d.name + ' joined');
      }
      if (d.type === 'leave') {
        removeRemote(d.name);
        notify(d.name + ' left');
      }
      if (d.type === 'mode') {
        bonusMode = d.bonus; // optional — most sketches never read this
      }
      if (d.type === 'reset') {
        // ── FILL IN: reset your own scene state ──
        // Example: clearTrail(); myCharacter.position.set(0,0,0);
        Object.values(remotes).forEach(r => {
          // ── FILL IN: reset each remote player's state ──
        });
      }
      if (d.type === 'kicked') {
        ws._ok = false; ws.close();
        notify('Removed from session');
        setTimeout(() => {
          const ov = document.getElementById('login-overlay');
          ov.classList.remove('dissolve'); ov.style.display = 'flex';
          document.getElementById('l-err').textContent = 'Removed by instructor.';
        }, 600);
      }
      if (d.type === 'join-rejected') {
        const ov = document.getElementById('login-overlay');
        ov.classList.remove('dissolve'); ov.style.display = 'flex';
        document.getElementById('l-err').textContent = d.reason || 'Cannot rejoin.';
      }

      // ── FILL IN: handle your experience-specific message ──
      // Fires when another player sends a state update in their own message
      // type. Use d.name to look up remotes[d.name] and update their visual.
      //
      // if (d.type === 'their-prefixed-type' && d.name !== myName) {
      //   const r = remotes[d.name];
      //   if (r) {
      //     // update r's position, trail, etc. from d's data
      //     positionFlag(r.flag, r.mesh.position);
      //   }
      // }

    } catch(err) { console.error(err); }
  };

  ws.onclose = () => {
    if (!ws._ok) return;
    const ov = document.getElementById('login-overlay');
    ov.classList.remove('dissolve'); ov.style.display = 'flex';
    document.getElementById('l-err').textContent = 'Disconnected.';
  };
}

// ── Send state update (call this from your render loop) ──────────────
// Rate-limited to ~20fps internally. Call every frame.
function sendUpdate(stateData) {
  if (!ws || ws.readyState !== WebSocket.OPEN) return;
  const now = performance.now();
  if (now - lastSend < 50) return;
  lastSend = now;
  ws.send(JSON.stringify({ type: MY_MSG_TYPE, ...stateData }));
  // Update own flag — pass the world position of your character
  // if (myFlag) positionFlag(myFlag, myCharacter.position);
}

// ── Login handler ────────────────────────────────────────────────────
function doJoin() {
  const name   = document.getElementById('l-name').value.trim();
  const server = document.getElementById('l-server').value.trim();
  const err    = document.getElementById('l-err');
  if (!name)   { err.textContent = 'Enter a name'; return; }
  if (!server) { err.textContent = 'Enter the server URL'; return; }
  err.textContent = '';
  const ov = document.getElementById('login-overlay');
  ov.classList.add('dissolve');
  setTimeout(() => {
    ov.style.display = 'none';
    connectToServer(server, name, myColor);
  }, 500);
}
window.doJoin = doJoin;
```

---

## Key Bindings Contract

All sketches must follow this mapping. These are reserved:

| Key | Action |
|-----|--------|
| **Space** (held) + **Arrow keys** | Orbit camera |
| **Arrow keys** (alone) | Move your character |
| **Shift** | Your special effect. Always on — this is yours alone, not gated by anything instructor-side. |
| **C** | Clear / reset your own trail, locally only. Never send this to the server. |

Do not use Space alone as a toggle. Do not assign color changes to Space. The LLM should implement orbit like this:

```javascript
// In your render loop — check if space is held
const spaceHeld = pressedKeys[' '] !== undefined;

if (spaceHeld) {
  // Rotate camera with arrow keys
  if (pressedKeys['ArrowLeft'])  camTheta -= orbitSpeed * dt;
  if (pressedKeys['ArrowRight']) camTheta += orbitSpeed * dt;
  if (pressedKeys['ArrowUp'])    camPhi = Math.max(0.15, camPhi - orbitSpeed * dt);
  if (pressedKeys['ArrowDown'])  camPhi = Math.min(Math.PI - 0.15, camPhi + orbitSpeed * dt);
  updateCamera();
} else {
  // Move your character
  if (pressedKeys['ArrowLeft'])  { /* move left */ }
  if (pressedKeys['ArrowRight']) { /* move right */ }
  // etc.
}
```

Track key state with `pressedKeys`:
```javascript
const pressedKeys = {};
window.addEventListener('keydown', e => { pressedKeys[e.key] = true; });
window.addEventListener('keyup',   e => { delete pressedKeys[e.key]; });
```

**What "your special effect" means:** it's entirely your call — a burst, a size change, a color shift, a sound, whatever fits your sketch. It fires immediately on keydown, every time, for you alone. It's not synchronized with other players and it's not gated by the instructor's optional `mode` broadcast. Describe it in your `SHIFT:` comment (see File Rules) so it can be checked against what the code actually does.

---

## Render Loop Pattern

Your render loop must call `sendUpdate()` every frame. `sendUpdate` handles its own rate-limiting.

```javascript
function renderLoop(timestamp) {
  const dt = Math.min((timestamp - (lastTime || timestamp)) / 1000, 0.05);
  lastTime = timestamp;

  // 1. Read input
  // 2. Update your scene
  // 3. Send your state
  sendUpdate({ /* your state */ });
  // 4. Update flag positions
  if (myFlag) positionFlag(myFlag, myCharacterPosition);
  Object.values(remotes).forEach(r => positionFlag(r.flag, r.somePosition));
  // 5. Render
  renderer.render(scene, camera);
  requestAnimationFrame(renderLoop);
}
requestAnimationFrame(renderLoop);
```

---

## Customizing Name Flags

The name flag is a positioned `div` that follows each player in screen space. The positioning logic is fixed — do not touch `makeFlag()`'s structure or `positionFlag()`. The visual styling is entirely yours.

The flag structure (built as DOM nodes, not raw HTML — see the boilerplate):
```html
<div class="name-flag">
  <div class="label" style="background: [player color]">Ada</div>
</div>
```

Override `.name-flag` and `.name-flag .label` in your CSS to restyle. Examples:

**Pill with outline:**
```css
.name-flag .label {
  background: transparent;
  border: 1.5px solid currentColor;
  color: #fff;
  border-radius: 20px;
  font-size: 10px;
  padding: 3px 10px;
}
```

**Large bold tag:**
```css
.name-flag .label {
  font-size: 16px;
  font-weight: 900;
  letter-spacing: 2px;
  text-transform: uppercase;
  padding: 6px 14px;
  border-radius: 0;
}
```

**Transparent ghost:**
```css
.name-flag .label {
  background: rgba(255,255,255,0.08);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(255,255,255,0.2);
  color: #fff;
  border-radius: 4px;
}
```

You can also add a small arrow, icon, or animated element by appending extra elements to the flag's outer `div` inside `makeFlag()` — just keep the name itself set via `textContent`, never concatenated into an HTML string. The only other constraint: keep `transform: translate(-50%, -100%)` on `.name-flag` so the flag anchors above the player position.

---

Students give this to their LLM along with their sketch idea:

> I am building a multiplayer Three.js sketch for an NMDE 404 class session. The instructor runs a shared WebSocket server. I need to follow a specific technical spec exactly.
>
> Read the attached `multiplayer-spec.md` before writing any code.
>
> My sketch idea: [describe your sketch here — what it looks like, how it moves, what Shift does]
>
> Constraints:
> - Single HTML file, Three.js r128 via CDN, no bundler
> - Use the login overlay HTML/CSS exactly as written in the spec
> - Use the multiplayer boilerplate exactly as written, filling in only the marked sections (`MY_MSG_TYPE`, `addRemote`, `removeRemote`, `connectToServer`, the `reset` handler, and the custom-message handler)
> - All assets hosted at absolute RIT URLs (I will fill these in after uploading)
> - Key bindings must follow the spec (Space+arrows = orbit, arrows alone = move, Shift = my always-on effect, C = clear my trail locally)
> - `MY_MSG_TYPE` must be my first name prefixed, e.g. `'ada-move'`
> - The `camera` variable must be accessible to `positionFlag()`
> - Include the required `SKETCH:` / `MOVE:` / `SHIFT:` comment block at the top of my script
>
> Start by showing me the filled-in boilerplate sections before writing the full file.

---

## Checklist Before Submitting

- [ ] File named `firstname.html` matching your login name exactly, unique among classmates
- [ ] `SKETCH:` / `MOVE:` / `SHIFT:` comment block present at the top of the script
- [ ] Login overlay HTML copied verbatim
- [ ] Login overlay CSS copied verbatim
- [ ] Multiplayer boilerplate included with `MY_MSG_TYPE` and all `FILL IN` sections completed
- [ ] `MY_MSG_TYPE` is prefixed with your first name and doesn't collide with a classmate's
- [ ] `sendUpdate()` called every frame in render loop
- [ ] `camera` variable exists and is what `positionFlag()` uses
- [ ] All assets at absolute RIT URLs
- [ ] No relative asset paths
- [ ] No PeerJS or other P2P libraries
- [ ] Space + arrows = orbit, arrows alone = move
- [ ] Shift always triggers your effect, unconditionally
- [ ] `C` clears your own trail locally — never sent to the server
- [ ] Your client never sends `type: 'reset'`
- [ ] Tested locally — scene renders, login overlay appears

---

*NMDE 404 — RIT New Media Design*
*Mike Minerva — Professor of Practice*

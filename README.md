# Finding Myself - A Creatively Coded Self Portrait 

## 1. 

I set up the project (Vite + React)
I scaffolded a minimal, hot-reloadable environment, by entering the following into the bash terminal:

```
npm create vite@latest finding-myself -- --template react
cd finding-myself
npm install
npm run dev
```
Then I mounted React and kept the entry simple

Here is index.html :

```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Kim Piffy's 'Finding Herself'</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>

```


src/main.jsx :

```
import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'
import './styles.css'

const root = createRoot(document.getElementById('root'))
root.render(<App />)

```

src/App.jsx :

```
import React from 'react'
import AstroSonar from './AstroSonar.jsx'

export default function App() {
  return (
    <div>
      <AstroSonar />
    </div>
  )
}

```

## 2. 

Next, I needed a stable angular timebase to drive both visuals and (later)audio triggers so I built the sonar sweep (state + animation loop) I created a component that holds the sweep angle and advances it with requestAnimationFrame. This is the skeleton that everything else hangs off.

Here is an excerpt from src/AstroSonar.jsx 

```
import React, { useEffect, useRef, useState } from "react";

// helpers
const degToRad = d => (d * Math.PI) / 180;
const wrapDeg = d => ((d % 360) + 360) % 360;

// detect if the sweep crossed a target degree (handles wrap-around)
const crossed = (prev, curr, target) => {
  prev = wrapDeg(prev); curr = wrapDeg(curr); target = wrapDeg(target);
  if (prev <= curr) return prev < target && target <= curr;
  return target > prev || target <= curr;
};

export default function AstroSonar() {
  const [running, setRunning] = useState(true);
  const [tempo, setTempo]   = useState(30);   // deg/sec
  const [angle, setAngle]   = useState(0);

  const rafRef = useRef(null);
  const baseTimeRef = useRef(null);
  const baseAngleRef = useRef(0);
  const prevAngleRef = useRef(0);

  useEffect(() => {
    if (!running) { if (rafRef.current) cancelAnimationFrame(rafRef.current); return; }

    const loop = (ts) => {
      const secs = ts / 1000;
      if (baseTimeRef.current == null) baseTimeRef.current = secs;
      const curr = wrapDeg(baseAngleRef.current + (secs - baseTimeRef.current) * tempo);
      const prev = prevAngleRef.current;

      // ...event triggers will go here...

      prevAngleRef.current = curr;
      setAngle(curr);
      rafRef.current = requestAnimationFrame(loop);
    };

    rafRef.current = requestAnimationFrame(loop);
    return () => rafRef.current && cancelAnimationFrame(rafRef.current);
  }, [running, tempo]);

  // ...render SVG dial here (omitted for brevity)...
  return <div className="app">{/* sonar dial */}</div>;
}
```

## 3. 

To transform my birth chart into something I could animate, I needed to translate astrological positions into a circular system. In astrology, the zodiac is divided into 12 signs of 30° each, starting with Aries at 0°.

So:
Aries = 0°–29°
Taurus = 30°–59°
Gemini = 60°–89°
…and so on, wrapping around the full 360°.

I then took my own birth chart and calculated each planet’s longitudinal placement within this system. For example, if Mercury was at 17° Aquarius, that translates to 300° (start of Aquarius) + 17° = 317° around the circle.
This gave me self-specific data points to place on the sonar dial. Each planet has a glyph, a degree, and (later) a sound.

src/AstroSonar.jsx (excerpt) :

```
// Signs (static glyph ring)
const SIGNS = [
  { key:"Aries", glyph:"♈", deg:0 }, { key:"Taurus", glyph:"♉", deg:30 },
  { key:"Gemini", glyph:"♊", deg:60 }, { key:"Cancer", glyph:"♋", deg:90 },
  { key:"Leo", glyph:"♌", deg:120 },   { key:"Virgo", glyph:"♍", deg:150 },
  { key:"Libra", glyph:"♎", deg:180 }, { key:"Scorpio", glyph:"♏", deg:210 },
  { key:"Sagittarius", glyph:"♐", deg:240 }, { key:"Capricorn", glyph:"♑", deg:270 },
  { key:"Aquarius", glyph:"♒", deg:300 },    { key:"Pisces", glyph:"♓", deg:330 },
];

// My planets — calculated longitudinal placements from my chart
const PLANETS = [
  { key:"Sun", glyph:"☉", deg:311.59, instrument:"lead" },
  { key:"Moon", glyph:"☽", deg:52.1, instrument:"pad" },
  { key:"Mercury", glyph:"☿", deg:317.52, instrument:"arp" },
  { key:"Venus", glyph:"♀", deg:358.29, instrument:"bell" },
  { key:"Mars", glyph:"♂", deg:100.01, instrument:"snare" },
  { key:"Jupiter", glyph:"♃", deg:194.41, instrument:"bass" },
  { key:"Saturn", glyph:"♄", deg:319.52, instrument:"pad" },
  { key:"Uranus", glyph:"♅", deg:289.29, instrument:"hat" },
  { key:"Neptune", glyph:"♆", deg:289.31, instrument:"hat" },
  { key:"Pluto", glyph:"♇", deg:235.21, instrument:"tom" },
  { key:"Lilith", glyph:"⚸", deg:351.59, instrument:"clap" },
  { key:"NNode", glyph:"☊", deg:260.15, instrument:"bell" },
  { key:"Asc", glyph:"ASC", deg:171.45, instrument:"kick" },
  { key:"MC", glyph:"MC", deg:78.8, instrument:"lead" },
];
```

I placed zodiac signs as a static glyph ring and defined planets at the fixed degrees in the method mentioned above.
The sweep calls crossed(prev, curr, targetDeg) to fire events as it passes each planet. This is the part where I “quantified the qualitative” so the ring encodes identity data and then the sweep makes it kinetic. Use of sound was inspired by Sarah Langford's note on considering sound, and by guest lecturers Isabel & Helen's use of movement and especially taking part in their creative type test. 

## 4. 

I was able to wire in sound using Tone.js (and trialed a remix board) To do this I imported Tone.js, drafted a small sound library, and mapped planets to instruments.

At this stage I experimented with a UI to remix myself (swapping instrument assignments live). It worked, but I scaled it back as it was often times sounding chaotic and I came to think the idea of a sonar with set sounds was enough. I wanted to keep the piece focused and stable. The sounded I ended up using create an effect which simultaenously references a sonar 'beep' but also a 'heart beat' bringing it back to the idea of this being a self portrait after all. 

src/AstroSonar.jsx (excerpt)

```
import * as Tone from "tone";

// bright, safe scale
const SCALE = ["G3","A3","B3","D4","E4","G4","A4","B4","D5","E5","G5","A5","B5","D6","E6"];
const pick = i => SCALE[(i % SCALE.length + SCALE.length) % SCALE.length];

// tiny sound library (labels omitted for brevity)
const SOUND_LIBRARY = [
  { key:"kick" }, { key:"snare" }, { key:"clap" }, { key:"hat" }, { key:"tom" },
  { key:"bass" }, { key:"lead" }, { key:"arp" }, { key:"pad" },  { key:"bell" },
  { key:"pluck" }, { key:"pulse" }, { key:"blip" },
];

// preserve the original mapping so I could “reset” after remixing
const ORIGINAL_INSTRUMENTS =
  Object.fromEntries(PLANETS.map(p => [p.key, p.instrument]));

// (In my trials I exposed a mini “board” to choose instruments per planet,
// then I decided to keep the final mapping fixed for clarity.)

function triggerPlanet(p, idx) {
  // pick note from SCALE deterministically
  const note = pick(idx);
  // route to the planet’s instrument (built elsewhere in the graph)
  // e.g., synths[p.instrument].triggerAttackRelease(note, "8n");
}
```

## 5

I added my self portrait mandala as a background image. I dropped in the radial drawing I had made in Procreate. At the time I created it, I had no idea it would end up being the background for this project, it was just a starting point coming from week one and a total experiment. However, once I placed it behind the dial, it clicked so well. 

This reference back to my earlier self-portrait work gave the whole piece a continuity I didn’t expect. The fact that it worked so perfectly felt like fate. Visually it meant the sonar concept was now really personal and started to reference 60’s psychedelia and the popular interest in astrology at that time, which excited me and gave more reference to my interests and background. It also meant my own artwork was a part of the outcome so it wasnt purely code based. It really gave the work a lineage. This was the first moment where the sonar felt less like an abstract experiment and more like a self-portrait.

```
import bg from "./assets/bg.png";

useEffect(() => {
  const el = document.documentElement;
  // Vite resolves the import to the built asset URL
  el.style.setProperty('--bg-url', `url(${bg})`);
}, []);
```

```
.scene-wrapper {
  position: relative;
  display: grid;
  place-items: center;
  overflow: hidden;
}
.scene-wrapper::before {
  content: "";
  position: absolute;
  inset: 0;
  background-image: var(--bg-url);
  background-position: center;
  background-size: cover;
  z-index: 0;
}
```
## 6
Once the background was in place, I started thinking about time and pace. I set the initial BPM to 60, to my resting heart rate. As the dial accelerates, the BPM increases stepwise. I added a little reference to represent my states of being as the BPM increases. This became a metaphor for my own tendency to overwork myself — drifting from rest into intensity, then crashing. The sonar sweep embodied those cycles. The intensite of the sounds sweeping as the bpm increases really instills this concept. 

60 bpm, meditate. 
90–120 bpm, flow. 
150 bpm, work bitch!
180 bpm, burnout

```
const [bpm, setBpm] = useState(60); // start at resting heart rate
const appRef = useRef(null);

useEffect(() => {
  const spb = 60 / bpm;
  const rotDur = `${4 * spb}s`;
  const hueDur = `${8 * spb}s`;
  const el = appRef.current || document.documentElement;
  el.style.setProperty('--bg-rot-dur', rotDur);
  el.style.setProperty('--bg-hue-dur', hueDur);
}, [bpm]);

```

## 7

I enhanced the background with rotation & hue shifts.

After I had the static background image working, I wanted to push it further. I experimented with keyframe animations to slowly rotate the image and cycle its hue. The results were psychoactive, almost hallucinatory, and brought the sonar alive in a way that surprised me.

src/styles.css (excerpt)

```
.scene-wrapper::before {
  /* ... */
  filter: hue-rotate(0deg);
  animation:
    bg-rotate var(--bg-rot-dur, 16s) linear infinite,
    bg-hue var(--bg-hue-dur, 32s) linear infinite;
  animation-play-state: var(--bg-anim-state, running);
}

@keyframes bg-rotate { to { transform: rotate(360deg); } }
@keyframes bg-hue    { to { filter: hue-rotate(360deg); } }
```

This stage was thrilling. The combination of my hand-drawn radial pattern with tempo-driven movement and hue cycling transformed the whole piece. It started to feel like a psychedelic astrological instrument, which was both personal and culturally resonant.

## 8

Once the sonar, background, tempo logic, and animations were in place, I needed to make the project live so I could share it in the Online Studio and with my tutors/peers. I wanted a process where I could just push code and have the site update automatically.  Deployment was the moment my self-portrait became public. It shifted from a private experiment in abstraction to something viewable, remixable, and alive on the web.

To do that, I set up GitHub Pages with Actions:
Every push to the main branch triggers a workflow.

The workflow installs dependencies, builds the Vite project, and publishes the output (dist/) to GitHub Pages.
Pages then serves the site from the gh-pages branch.

This meant I didn’t have to manually rebuild or drag files around. Deployment is the finishing point to the creative flow.

.github/workflows/deploy.yml

```
name: Deploy Vite site to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

And because GitHub Pages serves from a subpath, I added a base to my Vite config:

vite.config.js :

```
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: '/grimiore/',  // matches repo name
})
```







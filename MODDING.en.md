# Mods for Wreckbox

A mod is a small text file with the `.wmod` extension. It adds your own things to the game: tools, weapons, items, materials,
buildings, maps, sounds, rules and mini games. A mod is installed from the game menu (“Mods” → “Add file”), turned on and off with
one switch, and doesn’t get in the way of game updates: when a new version comes out, mods keep working.

This document is written for three kinds of readers at once:

- **no programming** — section 1: a mod with a structure or a map is made in the editor in a minute, without a single line of code;
- **asking a neural network** — section 8: attach this file to your request; it has rules for the AI and a ready-made request text;
- **programmers** — sections 2–7: basics, the full command reference, limits, complete mods and debugging.

Every code example in this document is real: the game runs each of them in an automated test before every release. If an example
doesn’t work in your version of the game, update the game.

There is also a [Russian version](MODDING.md) of this document. Ready-made example mods are in the [examples](examples) folder.

## Contents

1. [A mod without code: a structure or a map from the editor](#1-a-mod-without-code-a-structure-or-a-map-from-the-editor)
2. [Your first mod in five minutes](#2-your-first-mod-in-five-minutes)
3. [Basics](#3-basics)
4. [Reference: every command of the control panel](#4-reference-every-command-of-the-control-panel)
5. [Limits and safety](#5-limits-and-safety)
6. [Recipes: complete mods](#6-recipes-complete-mods)
7. [Debugging and common errors](#7-debugging-and-common-errors)
8. [For a neural network](#8-for-a-neural-network)
9. [Compatibility and rules](#9-compatibility-and-rules)
10. [Appendix: materials, items, sounds](#10-appendix-materials-items-sounds)

---

## 1. A mod without code: a structure or a map from the editor

The game builds the mod for you from what you have built.

**A building.** Build a house — in “Build” mode or in the map editor (“Maps” → “Custom map”).

1. Pick the “Copy” tool and draw a frame around the house.
2. Press “To a mod” → “Framed structure”.
3. Type a name and an author, press “Save file”. You get a file `name.wmod`.
4. Install the mod: menu → “Mods” → “Add file”.

The house appears in the “Building” tool (“Build” → “Building”) — for you and for everyone who installs the mod. It is placed on a lot
like the game’s own houses: the lot is cleared and levelled, and citizens move in if the house has rooms, doors and stairs.

**A map.** Make a map in the editor (“Maps” → “Custom map” → “Create map”), press “⋯” → “Make a mod” → “This map”.
After the mod is installed, the map appears in “My maps” (“Maps” → “Custom map” → “Open list”) marked “mod”: you can open it
to play or in the editor.

**What’s inside such a file.** An ordinary mod: a few lines and one long data string at the end. You don’t need to read or edit it.
You can open the file in Notepad and add your own things to it — for example, a tool next to the house (section 6.9).

<!-- test: skip — what the file looks like, the data is not real -->
```js
// Wreckbox mod: the building “House with a turret”. Made in the map editor: “⋯” → “Make a mod”.
Wreck.mod({
  id: 'house-with-turret-k3f9a',
  name: "House with a turret",
  author: "Your name",
  version: '1.0',
  api: 1,
  description: "A new building in the “Building” tool — in “Build” mode and in the map editor.",
  setup(w) {
    w.addBuilding({ id: 'b1', name: "House with a turret", data: DATA });
  },
});
const DATA = 'V0JMRAEAAAA…';   // structure data — a long string
```

---

## 2. Your first mod in five minutes

### 2.1. Template

Copy all of this into Notepad:

```js
// Popper — my first Wreckbox mod.
Wreck.mod({
  id: 'popper',                          // mod name in Latin letters: the game tells mods apart by it
  name: { ru: 'Хлопушка', en: 'Popper' },
  author: 'Your name',
  version: '1.0',
  api: 1,                                // mod format version — always 1 for now
  description: 'The “Popper” tool: a small blast where you tap.',

  setup(w) {                             // w — the control panel: the mod does everything through it
    w.addTool({
      id: 'popper',
      name: { ru: 'Хлопушка', en: 'Popper' },
      hint: { ru: 'Коснитесь стены или земли.', en: 'Tap a wall or the ground.' },
      slider: { label: { ru: 'Сила', en: 'Power' }, min: 1, max: 10, def: 3 },

      down(p) {                          // tap: p.x, p.y — point in the world, p.value — the slider
        w.explode(p.x, p.y, 10 + p.value * 4, 60 + p.value * 40);
      },
    });
  },
});
```

### 2.2. How to install

1. Save the file as `popper.wmod`. In Notepad: “File” → “Save as”, type “All files”, encoding UTF-8.
2. In the game: menu → “Mods” → “Add file” and pick the file. On a phone, “Paste text” is easier: copy the whole mod text
   and paste it into the box.
3. The game shows a warning (“a mod is a program”). Press “Add mod”.

### 2.3. How to check

Open “Demolish”: the “Popper” has appeared in the tool row, before the eraser. Pick it, move the slider, tap a wall.
In the “Mods” menu you can see what the mod added (“Tools: Popper”), and the bottom of the menu says “1 mod”.

### 2.4. How to fix

Change the text and add the mod again. **A mod with the same `id` replaces the old one** — that’s how new versions are released:
keep the `id`, raise the `version`. The mod’s saved data (section 4.10) stays when it is replaced.

If the mod wasn’t added, the game tells you why: “Couldn’t add the mod: …”. Common causes and what to do — section 7.

---

## 3. Basics

### 3.1. File structure

The whole mod is one call, `Wreck.mod({ … })`. Inside the braces is the description of the mod:

| Field | What it is | Required |
|---|---|---|
| `id` | Mod name in Latin letters: letters, digits, `-` and `_`, up to 24 characters | Recommended (without it the game makes an `id` from the name and author) |
| `name` | Name — a string or in two languages: `{ ru: '…', en: '…' }`, up to 40 characters | Yes |
| `author` | Author | No |
| `version` | Mod version: `'1.0'`, `'1.2.3'` | No |
| `api` | Mod format version. Always `1` for now | Yes |
| `description` | Short description in the mod’s row in the “Mods” menu, up to 220 characters (longer — cut with “…”, the full text opens with the “Learn more” button) | No |
| `details` | Detailed description up to 4000 characters: subtleties, rules, commands, tips. Line breaks are kept. A string or in two languages. If present, the mod gets a “Learn more” button | No |
| `setup(w)` | The function the game calls when the mod is turned on. `w` is the control panel | Yes |

Outside `setup` you can declare your own constants and functions (like `const DATA = '…'` in a mod from the editor). Everything
else happens inside `setup`: that’s where the mod adds tools and items and subscribes to events.

`setup` runs **once** when the mod is turned on: on install, on game start and when the player flips the mod’s switch on.
The functions the mod hands to the game (a tool tap, an event handler) are called later, when the time comes.

### 3.2. `id` and names

- **The mod’s `id`** tells it apart from other mods. Two mods with the same `id` can’t be installed side by side — the second
  replaces the first. Make up your own: `ufo-invasion`, `petya-tools`.
- **An `id` inside the mod** (of a tool, item, material, sound, building, map) is short and in Latin letters: `'cluster'`, `'titan'`.
  They must differ within one kind: two tools `'boom'` in one mod is an error. In different mods they may coincide:
  the game adds the mod’s `id` to them.
- **Names** — everything the player sees (`name`, `hint`, the slider label) — write in two languages: `{ ru: 'Титан', en: 'Titanium' }`.
  The game shows the right one. A plain string works too — then it is shown in both languages. The game language is `w.lang`
  (`'ru'` or `'en'`).

### 3.3. Coordinates

- The world is a cross-section of a city. **The `x` axis points right, the `y` axis points down**: the larger `y`, the lower.
  The unit is a **world pixel**.
- The world is made of **6 × 6 pixel cells** (`w.cell` = 6). A cell is a `col` (column) and a `row`:
  `col = Math.floor(x / w.cell)`, `row = Math.floor(y / w.cell)`. Cell centre: `x = (col + 0.5) * w.cell`.
- Scale: 1 metre is about 15 pixels (2.5 cells). One storey of a house is about 8 cells.
- Each map has its own world width: `w.world.width`. The ground is at a different height in different places: `w.groundY(x)` is
  the top of the ground at point `x`, `w.topY(x)` is the top of anything solid (the roof, if there’s a house above that point).
- What’s on screen now: `w.camera` — `{ x, y, zoom, left, top, right, bottom }` (centre, zoom and edges of the view in world pixels).

In the reference examples (section 4), `x`, `y` are a point in the world and `col`, `row` a cell. In your own mod take them from
a tap (`p.x`, `p.y`), from an event (`e.x`, `e.y`) or from the camera.

### 3.4. Time, speed, gravity

- Time is **in game seconds**. The game runs in steps of 1/60 second. On pause, time stands still; if the player speeds the game up,
  there are more steps per second. `w.world.time` — how many seconds this world has been running.
- Speed is **pixels per second** (`vx` — to the right, `vy` — downwards; to throw something up, use a negative `vy`).
- Gravity “1g” is 450 pixels per second per second. `w.world.gravity` is the current one, in g (1 — Earth, 0 — weightlessness).

### 3.5. Power of blasts and disasters

A blast is defined by two numbers: **radius** (pixels) and **power** (how much strength it takes from blocks). For comparison:

| What | Radius | Power |
|---|---|---|
| Firecracker | 10–20 | 50–120 |
| Gas cylinder | 46 | 340 |
| The game’s bomb (slider from 1 to 10) | 36–108 | 410–880 |
| Limit for a mod | 200 | 2000 |

Big disasters (meteorite, tornado, tsunami, nuclear strike…) are set by a **level from 1 to 10** — like the slider of that tool in the game.

### 3.6. What commands return

- An action (`explode`, `setBlock`, `spawnItem`…) returns `true` — it worked, `false` — it didn’t: the place is taken, the point is
  outside the world, the action budget for this step is used up (section 5). The game doesn’t complain about `false` — simply
  nothing happened.
- If the mod gives a command **wrong data** (no such material, sound or item, a required field is missing), the command throws
  an error. The player sees the message: “Mod “…”: error — …”. An error in `setup` stops the mod from turning on.
- Information (`w.world`, `w.camera`, `findPeople`…) is **a copy**. You can’t change anything in it: people and items are changed
  only with commands.

### 3.7. What a mod can and can’t do

A mod works only through the control panel `w`. It doesn’t see the game’s internals: in mod code, `window`, `document`, `fetch`,
`localStorage` and other page tools are replaced with empty names. Plain JavaScript is all there: `Math`, `JSON`, `Date`, arrays,
functions, classes.

For delayed actions use `w.after(seconds, function)`, not `setTimeout`: `w.after` runs on game time, stands still on pause and
won’t fire if the mod was turned off.

A mod **can**: add its own tools, projectiles, blasts and disasters; blocks, people, items; its own items, materials, buildings,
maps, sounds; game rules (gravity, damage, blast power…); a score and text on screen; its own flying and crawling objects (the mod
moves and draws them itself); console commands; data in the save.

A mod **can’t**: change how citizens behave, drive a vehicle, add new liquids or new interface windows, replace the physics
as a whole, go online.

The control panel has no game particles — debris, sparks, dust, smoke: a mod moves its own fragments and sparks in `update` and
draws them in `draw` (how to keep them from hanging in the air — section 7.4, “Your own objects hang in the air”).

---

## 4. Reference: every command of the control panel

The control panel `w` is the object the mod receives in `setup(w)`. Below is everything in it. For each command: what it does,
what it takes, what it returns and an example. The game clips numbers outside the allowed range to the nearest edge.

### 4.1. Registration: add to the game

#### `w.addTool(description)` — a tool

Adds a button to the tool row (“Demolish”, “Build” or both), before the eraser. Mod tools are not available in the map editor.

| Field | What it is |
|---|---|
| `id` | Short name in Latin letters (required) |
| `name` | Label under the button, up to 28 characters (required) |
| `hint` | Hint in the tool panel, up to 180 characters |
| `mode` | `'destroy'` — in “Demolish” (the default), `'build'` — in “Build”, `'both'` — in both |
| `icon` | Icon: an SVG outline (what goes into `d="…"`) in a 24 × 24 box, drawn as a line like the game’s icons. Without it — the common mod icon |
| `slider` | Slider: `{ label, min, max, def, unit, format }`. `min`, `max`, `def` — integers from −999 to 999; `unit` — text after the number (`'kg'`); `format(v)` — your own label: return a string |
| `down(p)` | Finger or mouse button pressed |
| `move(p)` | Dragging without releasing |
| `up(p)` | Released |
| `update(dt, p)` | Every game step while the tool is selected (not called on pause). `dt` — 1/60 second |
| `draw(ctx, p)` | Every frame while the tool is selected: draw a crosshair or a hint. `ctx` — a canvas in world coordinates |
| `select()`, `deselect()` | The tool was selected, switched to another |

The pointer `p`: `x`, `y` — point in the world; `down` — whether it is pressed; `vx`, `vy` — movement speed; `startX`, `startY` —
where it was pressed; `value` — the slider value.

Inside the tool’s functions, `this` is the description itself: it is a handy place to keep your state (`this.count`). Arrow
functions (`() => …`) have no `this` of their own — for state, declare a variable in `setup` instead.

```js
w.addTool({
  id: 'line',
  name: { ru: 'Линия взрывов', en: 'Blast line' },
  hint: { ru: 'Тяните — взрывы встанут вдоль линии.', en: 'Drag — blasts follow the line.' },
  icon: 'M4 20L20 4M6 14l4 4M14 6l4 4',
  slider: { label: { ru: 'Шаг', en: 'Step' }, min: 10, max: 60, def: 24, unit: 'px' },
  down(p) { this.lastX = p.x; this.lastY = p.y; },
  move(p) {
    if (Math.hypot(p.x - this.lastX, p.y - this.lastY) < p.value) return;
    w.explode(p.x, p.y, 14, 120, { fire: 0 });
    this.lastX = p.x; this.lastY = p.y;
  },
  draw(ctx, p) {
    ctx.strokeStyle = 'rgba(255, 200, 60, 0.9)';
    ctx.lineWidth = 2 / w.camera.zoom;            // width in screen pixels at any zoom
    ctx.beginPath(); ctx.arc(p.x, p.y, 8, 0, Math.PI * 2); ctx.stroke();
  },
});
```

Up to 24 tools in one mod.

#### `w.addCommand(name, function)` — a console command

The console opens with the `~` key (on a phone — menu → “Console”). The name is 2 to 16 letters (Russian or Latin), digits, `-` and `_`.
The function receives everything the player typed after the name (in lower case). If it returns a string, the game shows it as a message.
A name already taken by the game or another mod is an error.

```js
w.addCommand('shower', (arg) => {
  const n = Math.min(40, Number(arg) || 10);
  for (let i = 0; i < n; i++) {
    w.after(i * 0.1, () => {
      const c = w.camera;
      w.shoot({ x: c.left + Math.random() * (c.right - c.left), y: c.top - 40, vy: 300, look: 'ball', radius: 16, power: 150 });
    });
  }
  return 'A shower of ' + n + ' projectiles';
});
```

#### `w.addSound(id, recording)` — your own sound

The recording is embedded in the mod file as a string `'data:audio/mp3;base64,…'` (mp3, ogg, wav — any format the browser can read),
up to 2 MB. It is played with `w.sound(id, …)`. The recording is prepared in the background in a fraction of a second: played in the
very first moment after the mod is turned on, it stays silent. Up to 32 sounds in a mod. How to get such a string from your file —
section 6.10.

```js
w.addSound('beep', 'data:audio/mp3;base64,…');
```

#### `w.addItem(description)` — an item

A new item in the “Mods” tab of the “Items” tool (“Build”). It is placed, moved, falls, burns and breaks like the game’s items.

| Field | What it is | Default |
|---|---|---|
| `id`, `name` | Name in Latin letters and the label (required) | |
| `width`, `height` | Size in world pixels (required): from 3 × 3 to 320 × 220. A cell is 6 | |
| `mass` | Mass, from 0.2 to 300 (for comparison: TV — 2, sofa — 4, car — 15) | 2 |
| `hp` | Strength (from 1 to 3000) | 40 |
| `fragile` | Fragility against hits (from 0.02 to 2) | 0.6 |
| `flammable` | Flammability (from 0 to 1) | 0.3 |
| `value` | Price: added to the damage counter when the item is broken | 100 |
| `sound` | Hit sound: `'wood'`, `'metal'`, `'soft'`, `'glass'`, `'ceramic'`, `'stone'` | `'wood'` |
| `debris` | Debris: `'wood'` (splinters), `'cloth'`, `'metal'`, `'glass'` (shards), `'white'` (white crumbs), `'leaf'`, `'water'` (splashes) | by sound: wood → `'wood'`, metal → `'metal'`, soft → `'cloth'`, glass → `'glass'`, ceramic and stone → `'white'` |
| `variants` | How many looks the item has (up to 8): the player cycles through them in “Items” | 1 |
| `hang` | `true` — hangs from the ceiling above the cursor (chandelier, sign) | |
| `light` | Glows at night: `{ x, y, radius }` — a point from the item’s centre and a radius | |
| `explosive` | Explodes when broken or set on fire: `{ radius, power, fire, fuse }` (`fuse` — seconds of burning before the blast) | |
| `draw(ctx, w, h, variant)` | Drawing in code: the canvas goes from 0 to `w` in width and from 0 to `h` in height | |
| `image` | Drawing as a picture: `'data:image/png;base64,…'` (png, jpeg, webp, gif; up to 1 MB) | |
| `images` | Several pictures — one per look (up to 8) | |
| `onBreak(e)` | Your own “item broken” function (data — as in the `itemBreak` event) | |

The drawing is made once and remembered — don’t make it depend on time. A small picture (one dot per world pixel) stays sharp,
a large one is scaled down smoothly. A broken item falls apart into debris of the `debris` kind. Up to 60 items in a mod.

```js
w.addItem({
  id: 'barrel',
  name: { ru: 'Бочка с краской', en: 'Paint barrel' },
  width: 12, height: 16, mass: 4, hp: 40, sound: 'metal', flammable: 0.6, variants: 3,
  draw(ctx, pw, ph, v) {
    ctx.fillStyle = ['#d94f4f', '#4f8fd9', '#e0c040'][v];
    ctx.fillRect(0, 1, pw, ph - 2);
    ctx.fillStyle = '#2b2d31';
    ctx.fillRect(0, 0, pw, 1.5); ctx.fillRect(0, ph - 1.5, pw, 1.5);
  },
});
```

#### `w.addMaterial(description)` — a material

A new material in the “Brush” (“Build”). A mod material is **a copy of an existing sample material** (`like`) with its own colour and
numbers: it has the sample’s texture and sounds. Whatever the mod doesn’t set is taken from the sample.

| Field | What it is | Limits |
|---|---|---|
| `id`, `name` | Name in Latin letters and the label (required) | |
| `like` | Sample: any brush material (list — section 10.1) | `'brick'` by default |
| `color` | Colour: `'#rrggbb'` | |
| `density` | Density: weight of a block (wood 0.7, brick 1.9, steel 1.4) | 0.1–8 |
| `hp` | Strength against hits and blasts (wood 60, concrete 150, steel 320) | 2–2000 |
| `compress` | How much weight it holds from above (brick 900, concrete 2000) | 10–20000 |
| `shear` | Whether it holds overhangs and spans to the side (brick 28, steel 400) | 1–5000 |
| `tension` | Whether it holds things hanging below (brick 8, steel 1500) | 1–20000 |
| `flammable` | Flammability | 0–1 |
| `burnTime` | How many seconds it burns | 1–60 |
| `brittle` | Brittleness: how easily it crumbles | 0.05–5 |
| `shatter` | How many pieces a fallen chunk splits into | 0.1–4 |
| `bearing` | `false` — not load-bearing: you can’t build on it (like glass) | |
| `value` | Price of a block for the damage counter | |
| `onBreak(e)` | Your own “block destroyed” function (data — as in the `break` event) | |

Special behaviour tied to a game material itself is not copied: glass bursts from a shock wave, steel bends, rebar sticks out of
reinforced concrete — a mod material doesn’t do that, even if it is made on their sample.

The game has 42 numbers for mod materials across all enabled mods. If they run out, the mod works without its material, and the
player sees the message “turn off another mod with materials”. When a mod is turned off, its blocks in the world turn into the sample
material.

```js
w.addMaterial({
  id: 'titan',
  name: { ru: 'Титан', en: 'Titanium' },
  like: 'steel', color: '#9fb0c4',
  density: 0.8, hp: 600, shear: 900, tension: 3000,
});
```

#### `w.addBuilding({ id, name, data })` — a building

A new type in the “Building” tool. `data` is a structure blueprint that the game gives you: “Copy” → frame → “To a mod” → “Framed
structure” (section 1). You don’t write the blueprint yourself. Up to 40 buildings in a mod.

```js
w.addBuilding({ id: 'tower', name: { ru: 'Башня', en: 'Tower' }, data: TOWER });
```

(`TOWER` is the data string from the file the game made; the full mod is recipe 6.9.)

#### `w.addMap({ id, name, data })` — a map

A map in “My maps” marked “mod”. `data` is a map from the editor: “⋯” → “Make a mod” → “This map”. Up to 20 maps in a mod.

```js
w.addMap({ id: 'island', name: { ru: 'Остров', en: 'Island' }, data: ISLAND });
```

#### `w.on(event, function)` — subscribe to an event

The list of events and their data — section 4.9. Up to 16 functions per event in a mod.

```js
w.on('death', (e) => { w.toast('A citizen died'); });
```

### 4.2. Information about the world

All of these are copies at the moment of the call.

| What | What it gives |
|---|---|
| `w.api` | The mod format version the game understands (1 for now) |
| `w.game` | Game version as a string: `'2.0'` |
| `w.lang` | Game language: `'ru'` or `'en'` |
| `w.cell` | Cell size in pixels: 6 |
| `w.world` | `{ width, height, cols, rows, cell, groundY, map, seed, time, night, gravity, paused }`: world size in pixels and in cells, the usual street level (one number for the whole world; on hills and in hollows the ground is higher or lower — the top of the ground there is `w.groundY(x)`), which map and its number, world seconds, whether it’s night, gravity in g, whether the game is paused |
| `w.camera` | `{ x, y, zoom, left, top, right, bottom }`: centre of the view, zoom, edges of the view in world pixels |
| `w.pointer` | The pointer, like a tool’s `p`: `{ x, y, down, vx, vy, startX, startY, value }` |
| `w.stats` | Score: `{ destroyed, damage, dead, people, hurt, debris, zombies }` — share destroyed (%), damage, dead, alive, wounded, debris pieces, zombies |
| `w.weather` | `{ kind, level, season }`: weather (`'clear'`, `'rain'`, `'storm'`, `'fog'`, `'snow'`), its strength 0–10, the season (`'summer'`, `'autumn'`, `'winter'`, `'spring'`) |
| `w.settings` | Game rules (“knobs”, section 4.8) — the current multipliers |
| `w.materials` | Materials accepted by `setBlock` — a list of names |
| `w.items` | Items accepted by `spawnItem` — a list of names |
| `w.sounds` | Game sounds that `w.sound` plays — a list of names |
| `w.groundY(x)` | Top of the ground at point `x` (without buildings), pixels |
| `w.topY(x)` | Top of anything solid at point `x` (the roof if there’s a house), pixels |
| `w.getBlock(col, row)` | The block in a cell: `null` (empty) or `{ material, hp, burning, reinforced, rubble }` — material, share of strength 0–1, whether it burns, whether it has rebar, whether it is rubble |
| `w.blockAt(x, y)` | The same by a point in pixels |

```js
const b = w.blockAt(x, y);
if (b && b.material === 'glass') w.toast('Glass here');
```

#### `w.findPeople({ x, y, radius, alive })` and `w.findItems({ x, y, radius, kind })`

People and items in a circle (without `radius` — in the whole world; no more than 400). `alive: true` — only the living; `kind` — only
items of this kind (your short `id` or a game item name).

A person: `{ id, x, y, vx, vy, hp, dead, down, burning, zombie, armed, panic, kind }` — id, point (the waist), speed, health 0–100,
whether dead, whether lying (fell, thrown), burning, a zombie, armed, panicking; `kind` — `'citizen'`, `'service'` (rescuers,
police, guards, SWAT), `'bandit'`, `'zombie'`.

An item: `{ id, kind, x, y, angle, vx, vy, width, height, mass, hp, burning, hanging }` — id, kind, centre, rotation, speed,
size, mass, share of strength 0–1, whether it burns, whether it hangs.

The `id` works for the “by id” commands (section 4.5). A search costs time: no more than 24 searches per mod per game step.

```js
for (const h of w.findPeople({ x, y, radius: 120, alive: true })) {
  if (h.burning) w.water(h.x, h.y - 10, 2);       // put out burning people
}
```

### 4.3. Blocks

`col`, `row` — a cell. Material — a name from the brush (section 10.1) or the short `id` of a material of your mod.

| Command | What it does | Returns |
|---|---|---|
| `w.setBlock(col, row, material)` | Places a block in an empty cell (not in water, not in an item) | `true` — placed |
| `w.removeBlock(col, row, debris)` | Removes a block. `debris = true` — breaks it with debris, like a hit; otherwise the block just disappears. Doesn’t remove ground and bedrock | `true` — removed |
| `w.damageBlock(col, row, damage)` | Takes strength away (50 by default) | `true` — the block is destroyed |

Reinforced concrete is placed as `'rconcrete'`, and `getBlock` returns it as `'concrete'` with `reinforced: true` — it is the same block.

```js
// a brick wall 5 cells high
for (let k = 0; k < 5; k++) w.setBlock(col, row - k, 'brick');
```

### 4.4. Blasts, projectiles, fire, water

`x`, `y` — world pixels.

| Command | What it does |
|---|---|
| `w.explode(x, y, radius, power, { fire, napalm, silent })` | A blast. Radius 4–200 (40 by default), power 1–2000 (400). `fire` — how much fire, 0–1 (0.4); `napalm: true` — a burning mixture; `silent: true` — no sound |
| `w.shoot(projectile)` | Your own projectile (below) |
| `w.bomb(x, y, radius, power)` | An air bomb falls from above exactly onto the point, punching through the roof and floors above it. 68 and 620 by default |
| `w.rocket(x0, y0, x1, y1, radius, power)` | A rocket flies from point (`x0`, `y0`) towards (`x1`, `y1`) and punches through an outer wall. 37 and 480 by default |
| `w.molotov(x, y, radius)` | A Molotov cocktail thrown from the side. Fire radius 10–90 (42) |
| `w.fire(x, y, strength)` | Set fire to the block at the point (or the wall behind it). Strength 1–255 (140) |
| `w.water(x, y, amount)` | Pour water: 1 — one cell (0.05–4) |
| `w.lava(x, y)` | A drop of lava |

They all return `true` if they worked.

```js
w.explode(x, y, 60, 700, { fire: 0.8 });
w.bomb(x, y);
w.rocket(w.camera.left, y - 100, x, y);
```

#### `w.shoot(projectile)` — your own projectile

Flies along an arc and explodes when it touches a block, the ground or an item, or by the `fuse` timer.

| Field | What it is | Default |
|---|---|---|
| `x`, `y` | Where it flies from (required) | |
| `vx`, `vy` | Speed, pixels per second (up to 2500) | 0 |
| `gravity` | Gravity multiplier (from −3 to 3; 0 — in a straight line) | 1 |
| `fuse` | Explode after this many seconds of flight (up to 14; 0 — only on impact) | 0 |
| `radius`, `power` | Blast on impact | 30 and 300 |
| `fire` | Fire of the blast, 0–1 | 0.4 |
| `look` | Look: `'shell'`, `'bomb'`, `'rocket'`, `'ball'` | `'shell'` |
| `color` | Body colour: `'#rrggbb'` | |
| `size` | Drawing size (0.3–6) | 1 |
| `spin` | Spin in flight (radians per second) | 0 — faces along the flight |
| `trail` | Trail: `'smoke'` or `'fire'` | |
| `onHit(h)` | Your own action instead of the blast: `h` — `{ x, y, vx, vy }`, where and how fast it hit | |
| `draw(ctx)` | Your own drawing: the canvas is already at the projectile’s point and rotated along the flight | |

A projectile lives no longer than 14 seconds and disappears when it flies off the edge of the world.

```js
// a “nesting doll” projectile: on impact it splits into three small ones
w.shoot({
  x, y: y - 300, vy: 200, look: 'bomb', trail: 'smoke',
  onHit(h) {
    for (let i = -1; i <= 1; i++) w.shoot({ x: h.x, y: h.y - 4, vx: i * 160, vy: -220, look: 'ball', radius: 20, power: 200 });
  },
});
```

#### A blast exactly at the point — even in mid-air

A projectile explodes on impact: if the player tapped the sky, the projectile flies past that point and explodes only on the ground
or a wall. To make the blast happen exactly where the player tapped, you need two things:

1. **`gravity: 0`** — the projectile flies in a straight line. With gravity it drops along an arc and misses the target.
2. **`fuse` — the travel time to the point:** the distance divided by the speed. The projectile explodes when it gets there.

If a wall or a roof is in the way, the projectile explodes on it earlier: that’s how it should be. The fuse fires at the end of
a game step (1/60 second), so the miss is no more than one step of travel: at speed 600 — about 10 pixels, less than two cells.

```js
// a rocket comes in from the upper left and explodes exactly at point (x, y)
const sx = x - 400, sy = y - 300;                 // where it starts
const speed = 600;                                // pixels per second
const dist = Math.hypot(x - sx, y - sy);          // how far to fly
w.shoot({
  x: sx, y: sy,
  vx: (x - sx) / dist * speed,                    // speed along the straight line to the target
  vy: (y - sy) / dist * speed,
  gravity: 0,                                     // in a straight line
  fuse: dist / speed,                             // arrived — blast
  look: 'rocket', radius: 40, power: 500,
});
```

The fuse is at most 14 seconds: a far start at low speed won’t make it. The same trick is used in the “Cluster bomb” (section 6.1):
it opens in the air above the target. Your own flying object with a different path (side approach, then a dive) is moved by the mod
itself in `update`, like the UFO in section 6.6, and blown up with `w.explode` wherever needed.

### 4.5. People and items

| Command | What it does | Returns |
|---|---|---|
| `w.spawnPerson(x, y)` | Places a citizen | `true` — placed |
| `w.spawnItem(kind, x, y, { angle, vx, vy })` | Places an item: your own (short `id`) or a game one (section 10.2). Not in a wall and not above the top of the world | `true` — placed |
| `w.hurtPerson(id, damage)` | Wounds a person (health 100; damage 20 by default) | `true` |
| `w.pushPerson(id, vx, vy)` | Pushes a person: they fall and fly at this speed | `true` |
| `w.pushItem(id, vx, vy)` | Adds speed to an item. Service things (elevator car, station vehicles) are not pushed | `true` |
| `w.damageItem(id, damage)` | Takes strength from an item (20 by default) | `true` |
| `w.removeItem(id)` | Removes an item | `true` |

`id` — the number from `findPeople`, `findItems` or from an event.

```js
w.spawnItem('car', x, y - 20);
for (const it of w.findItems({ x, y, radius: 150 })) w.pushItem(it.id, 0, -500);   // everything around — up
```

### 4.6. Big disasters, weather, day and night

The level is from 1 to 10, like the slider of that tool in the game. No more than three disasters per game step (section 5).

| Command | What it does | Default level |
|---|---|---|
| `w.meteor(x, y, size)` | A meteorite falls onto the point | 5 |
| `w.lightning(x, y, level)` | Lightning | 6 |
| `w.quake(level)` | Earthquake | 6 |
| `w.tornado(x, level)` | Tornado | 6 |
| `w.blackHole(x, y, level)` | Black hole | 5 |
| `w.tsunami(side, level)` | Tsunami: `'left'` — wave from the left, `'right'` — from the right, no side — the game chooses (in the port — always from the sea). One tsunami at a time | 6 |
| `w.nuke(x, y, level)` | Nuclear strike: siren, then the blast. One at a time | 4 |
| `w.zombies(count, x)` | A crowd of zombies (1–60) comes in from beyond the edge of the screen towards point `x`. Returns how many came | 12 |
| `w.setWeather(weather, strength)` | Weather: `'clear'`, `'rain'`, `'storm'`, `'fog'`, `'snow'`; strength 0–10 | |
| `w.setNight(true / false)` | Night or day | |

In the map editor, weather and night don’t change.

```js
w.meteor(x, y, 3);
w.setWeather('storm', 8);
w.after(5, () => w.lightning(x, y));
```

### 4.7. Sound, screen, messages

| Command | What it does |
|---|---|
| `w.sound(name, x, y, volume, rate)` | A sound at a point in the world: your own (`addSound`) or a game one (section 10.3). Volume 0–1.5 (0.5), rate 0.25–4 (1; higher — higher pitch and shorter). No such sound — an error |
| `w.shake(strength, x, y)` | Screen shake: strength 0–1 (0.2) |
| `w.toast(text)` | A message at the bottom of the screen (up to 140 characters) |
| `w.log(…)` | A line in the browser’s developer console (section 7) |
| `w.after(seconds, function)` | Do it later: after this many game seconds (up to 600) |

```js
w.sound('glass', x, y, 0.8);
w.shake(0.4, x, y);
w.after(2, () => w.toast('Two seconds have passed'));
```

### 4.8. Game rules (“knobs”)

`w.set(name, multiplier)` changes a game rule. **1 — as in the game**, 2 — twice as much, 0.5 — half as much. If several mods turn
the same knob, their multipliers are multiplied. When the mod is turned off, its rules are reverted. In the “Mods” menu you can see what
the mod changed (“Rules: gravity ×0.5”).

| Name | What it changes | Limits |
|---|---|---|
| `gravity` | Gravity (on top of the player’s “1g / Moon / 0g” choice) | 0–3 |
| `explosionPower` | Power of all blasts | 0.1–5 |
| `explosionRadius` | Radius of all blasts | 0.25–3 |
| `blockDamage` | Damage to blocks from hits and blasts | 0–10 |
| `itemDamage` | Damage to items | 0–10 |
| `peopleDamage` | Damage to people (0 — nobody gets hurt or dies; except those vaporized by a nuclear blast) | 0–10 |
| `fallDamage` | Fall damage | 0–10 |
| `fireSpread` | How fast fire spreads | 0–5 |
| `buildingStrength` | How much weight structures hold | 0.2–10 |
| `timeScale` | Speed of time | 0.1–4 |

`w.get(name)` — the current total for the knob (with all mods). An unknown name or a number outside the limits is an error.

```js
w.set('gravity', 1 / 6);          // like on the Moon
w.set('explosionPower', 2);       // all blasts twice as strong
w.toast('Gravity ×' + w.get('gravity').toFixed(2));
```

### 4.9. Events

`w.on(event, function)`. The event data (`e`) is a copy.

| Event | When | Data |
|---|---|---|
| `update` | Every game step (60 times a second; not on pause) | `dt` — step length, seconds |
| `draw` | Every frame | `ctx` — a canvas in world coordinates (under fire and smoke) |
| `hud` | Every frame, on top of everything | `ctx`, `width`, `height`, `free` — a canvas in screen pixels, its size and the part of the screen free of the game’s panels: `{ top, bottom, left, right }` |
| `explode` | **Before** a blast | `{ x, y, radius, power, fire }`: you may change `radius` and `power`; return `false` — there will be no blast |
| `break` | A block is destroyed | `{ col, row, x, y, material, cause, loose }`: cell, point, material, cause (`'hit'`, `'blast'`, `'fire'`, `'stress'` overload, `'crush'` crushed, `'vapor'` vaporized, `'other'`), `loose` — the block was in a falling piece |
| `itemBreak` | An item is broken | like an item from `findItems` |
| `hurt` | A person is wounded (damage from 2) | like a person from `findPeople` plus `{ cause, damage }` |
| `death` | A person died | like a person from `findPeople` plus `{ cause }` |
| `impact` | A heavy piece of debris or an item hit the ground or a structure | `{ x, y, speed, mass, id, item }`: `item` — the item kind, or `''` for debris |
| `world` | A new world: restart, another map, a save loaded | `{ map, seed, loaded, editor, data }`: `loaded` — from a save, `data` — what the mod put there in `save` |
| `save` | The player saves the world | the mod **returns** its data: a number, a string, a list, a plain object |

Causes in `hurt` and `death` (`cause`): `'blast'`, `'blunt'`, `'fall'`, `'crush'`, `'fire'`, `'thermal'`, `'shock'`, `'electric'`,
`'gas'`, `'suffocation'`, `'hole'` and others.

How events are delivered:

- `update`, `draw`, `hud` and `explode` are called immediately. The others pile up and are delivered **at the end of the game step**.
  So a blast the mod makes in response to an event happens on the next step: a chain reaction runs by itself and doesn’t hang the game.
- Blasts the mod makes right inside an `explode` handler don’t produce `explode` events — otherwise it would be an endless loop.
- No more than 240 events of one kind per game step: when a thousand blocks collapse, the extra ones are dropped.
- A mod turned on in a running game receives `world` only with the next world — set the initial state in `setup`.

```js
// blasts in the sky are cancelled, near the ground they are twice as strong
w.on('explode', (e) => {
  if (e.y < w.groundY(e.x) - 300) return false;
  e.power *= 2;
});
w.on('break', (e) => {
  if (e.material === 'glass' && Math.random() < 0.1) w.sound('glass', e.x, e.y, 0.3);
});
w.on('hud', (ctx, width, height, free) => {
  ctx.fillStyle = '#ffd23f';
  ctx.font = '600 14px sans-serif';
  ctx.fillText('Destroyed: ' + w.stats.destroyed.toFixed(1) + '%', free.left + 12, free.top + 16);
});
// a heavy piece of debris crashed — the screen shakes the harder, the heavier and faster it was
w.on('impact', (e) => {
  if (e.mass > 20 && e.speed > 300) w.shake(Math.min(0.6, e.mass * e.speed / 60000), e.x, e.y);
});
```

### 4.10. Your own storage

`w.store` — the mod’s settings and records on the player’s device. It survives a game restart, turning the mod off and a new version
of the mod; deleting the mod erases the storage too.

| Command | What it does |
|---|---|
| `w.store.get(key, fallback)` | The value by key; if there is none — `fallback` |
| `w.store.set(key, value)` | Remember: a number, a string, a list, a plain object |
| `w.store.remove(key)` | Forget |
| `w.store.keys()` | List of keys |

A key is up to 40 Latin letters, digits, dots and dashes. Up to 64 keys, one value up to 20,000 characters.

```js
let best = w.store.get('best', 0);
w.on('death', () => {
  const n = w.stats.dead;
  if (n > best) { best = n; w.store.set('best', best); }
});
```

### 4.11. Saves

The storage (4.10) is for the mod on this device. **For the world** there is the `save` event: whatever the mod returns goes into
the world save (up to 60,000 characters), and on loading it comes back in the `world` event (`e.loaded === true`, `e.data`).

```js
let score = 0;
w.on('death', () => { score++; });
w.on('save', () => ({ score }));
w.on('world', (e) => { score = e.loaded && e.data ? e.data.score : 0; });
```

A save remembers which mods were enabled and how many of their things are in the world. If the player loads it without the mod, the game
warns them: the mod’s items are removed, its materials become sample materials — the save always opens. A custom map from the editor
remembers the mods whose items and materials are on it.

---

## 5. Limits and safety

### 5.1. Action budget per game step

So that a mod with a bug in a loop can’t hang the game, commands have a budget for one game step (1/60 second). The budget is shared
by a table row: 6000 for all blocks, fire, water and lava together, 80 for all projectiles together, and so on. When it runs out,
the command returns `false` until the next step. On pause the budget is renewed every frame.

| What | Per step |
|---|---|
| Explosions (`explode`) | 30 |
| Projectiles (`shoot`, `bomb`, `rocket`, `molotov`) | 80 |
| People (`spawnPerson`) | 12 |
| Items (`spawnItem`) | 30 |
| Blocks, fire, water, lava (`setBlock`, `removeBlock`, `damageBlock`, `fire`, `water`, `lava`) | 6000 |
| Big disasters and weather | 3 |
| Sounds (`sound`) | 12 |
| Actions on people and items by id | 300 |
| Searches (`findPeople`, `findItems`) | 24 |

Need more — spread it over time: `w.after(i * 0.1, …)` or step by step in `update`.

### 5.2. Sizes

| What | Limit |
|---|---|
| Mod file | 4 MB |
| One sound recording | 2 MB |
| One picture | 1 MB |
| One building blueprint | 1.5 MB |
| Tools / items / sounds / buildings / maps in one mod | 24 / 60 / 32 / 40 / 20 |
| Handlers of one event in a mod | 16 |
| Mod storage | 64 keys of 20,000 characters |
| Mod data in a world save | 60,000 characters |
| Mod materials across the whole game | 42 |

### 5.3. Errors and automatic shutdown

- An error in mod code doesn’t crash the game: the player sees the message “Mod “…”: error — …” (no more than once every 2.5 seconds),
  and the game goes on.
- **5 errors in 20 seconds — the mod turns itself off**: “Mod “…” turned off: too many errors”. Turn it on again in the “Mods” menu.
- An error in `setup` stops the mod from turning on; on game start such a mod is turned off (“Mod “…” failed to start and was turned off”).
- Turning a mod off removes everything it added: tools, commands, items (also from the world), materials (blocks become the sample),
  buildings, maps, sounds, rules. You don’t need to restart the game.

### 5.4. Safety

A mod is a program. It runs inside the game and doesn’t see the page, the internet or the device’s files: all of these are replaced with
empty names in mod code. But this is not complete protection — that’s why the game shows a warning before a mod runs for the first time,
and you should only install mods from places you trust.

---

## 6. Recipes: complete mods

Each recipe is a whole `.wmod` file: copy, save, install.

### 6.1. Cluster bomb

The bomb opens in the air and showers the street with bomblets. A projectile with a time fuse and its own `onHit`.

```js
Wreck.mod({
  id: 'cluster-bomb',
  name: { ru: 'Кассетная бомба', en: 'Cluster bomb' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Бомба раскрывается в воздухе и засыпает улицу мелкими зарядами.', en: 'The bomb opens in the air and showers the street with bomblets.' },

  setup(w) {
    w.addTool({
      id: 'cluster',
      name: { ru: 'Кассетная бомба', en: 'Cluster bomb' },
      hint: { ru: 'Коснитесь цели — бомба раскроется над ней.', en: 'Tap a target — the bomb opens above it.' },
      slider: { label: { ru: 'Зарядов', en: 'Bomblets' }, min: 4, max: 24, def: 12 },

      down(p) {
        const n = p.value;
        const top = Math.min(w.camera.top - 60, p.y - 500);          // the bomb appears above the view
        w.shoot({
          x: p.x, y: top, vx: 0, vy: 420, look: 'bomb', size: 1.4,
          fuse: Math.max(0.3, (p.y - 160 - top) / 520),              // open about ten metres above the target
          onHit(h) {                                                 // our own action instead of the usual blast
            w.explode(h.x, h.y, 16, 80, { fire: 0 });
            for (let i = 0; i < n; i++) {
              w.shoot({
                x: h.x, y: h.y, look: 'ball', spin: 9, trail: 'smoke',
                vx: (i - (n - 1) / 2) * 34 + (Math.random() - 0.5) * 30,
                vy: 120 + Math.random() * 120,
                radius: 26, power: 260,
              });
            }
          },
        });
      },

      draw(ctx, p) {                                                 // crosshair under the cursor
        ctx.strokeStyle = 'rgba(255, 90, 60, 0.9)';
        ctx.lineWidth = 2 / w.camera.zoom;
        ctx.beginPath(); ctx.arc(p.x, p.y, 14, 0, 6.3); ctx.stroke();
      },
    });
  },
});
```

### 6.2. Timer mine

A tap places a charge, a countdown runs above it, at zero — a blast. Your own state, `update` and `draw`.

```js
Wreck.mod({
  id: 'timer-mine',
  name: { ru: 'Мина с таймером', en: 'Timer mine' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Заряды с обратным отсчётом.', en: 'Charges with a countdown.' },

  setup(w) {
    const mines = [];                                    // placed charges: { x, y, t } — t seconds to the blast

    w.addTool({
      id: 'mine',
      name: { ru: 'Мина с таймером', en: 'Timer mine' },
      hint: { ru: 'Коснитесь стены или земли. Ползунок — секунды до взрыва.', en: 'Tap a wall or the ground. Slider — seconds to blast.' },
      slider: { label: { ru: 'Таймер', en: 'Timer' }, min: 1, max: 15, def: 5, unit: { ru: 'с', en: 's' } },
      down(p) {
        if (mines.length >= 20) { w.toast(w.lang === 'ru' ? 'Не больше 20 мин' : '20 mines at most'); return; }
        mines.push({ x: p.x, y: p.y, t: p.value });
        w.sound('click', p.x, p.y, 0.6);
      },
    });

    // every game step: timers run only while the game runs and stand still on pause
    w.on('update', (dt) => {
      for (let i = mines.length - 1; i >= 0; i--) {
        const m = mines[i], sec = Math.ceil(m.t);
        m.t -= dt;
        if (m.t <= 0) { w.explode(m.x, m.y, 45, 500); mines.splice(i, 1); }
        else if (Math.ceil(m.t) < sec) w.sound('tink', m.x, m.y, 0.4);      // a tick once a second
      }
    });

    // every frame: the charge and the digits above it. The canvas is in world coordinates, so the font size is divided by the zoom:
    // the digits are always about 12 points on screen, however you zoom
    w.on('draw', (ctx) => {
      const k = 1 / w.camera.zoom;
      ctx.font = '700 ' + (12 * k).toFixed(2) + 'px sans-serif';
      ctx.textAlign = 'center';
      for (const m of mines) {
        ctx.fillStyle = '#c0392b'; ctx.fillRect(m.x - 4, m.y - 3, 8, 6);
        ctx.fillStyle = '#ffd23f'; ctx.fillText(String(Math.ceil(m.t)), m.x, m.y - 4 - 3 * k);
      }
    });

    w.on('world', () => { mines.length = 0; });          // a new world — no old mines
  },
});
```

### 6.3. Mini game: wreck the block with three bombs

Three bombs, the goal is to destroy a set share of the city. Score on screen, win, loss and a record on the device.

```js
Wreck.mod({
  id: 'three-bombs',
  name: { ru: 'Три бомбы', en: 'Three bombs' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Мини-игра: снесите 3% города тремя бомбами.', en: 'Mini game: wreck 3% of the city with three bombs.' },

  setup(w) {
    const ru = w.lang === 'ru';
    const GOAL = 3;                                      // goal: percent destroyed (on top of what was destroyed before the start)
    let left, start, lastAt, state;                      // bombs left, destroyed before the start, time of the last bomb, 'play' / 'win' / 'lose'
    let best = w.store.get('best', 0);                   // record — the best result in percent

    function reset() { left = 3; start = w.stats.destroyed; lastAt = -1; state = 'play'; }
    reset();
    w.on('world', reset);
    const got = () => Math.max(0, w.stats.destroyed - start);

    w.addTool({
      id: 'bomb3',
      name: { ru: 'Три бомбы', en: 'Three bombs' },
      hint: { ru: 'Сбросьте три бомбы так, чтобы снести как можно больше. Касание после конца — новая попытка.',
              en: 'Drop three bombs to wreck as much as you can. Tap after the end to retry.' },
      down(p) {
        if (state !== 'play') { reset(); w.toast(ru ? 'Новая попытка' : 'New try'); return; }
        if (left <= 0) return;
        left--;
        lastAt = w.world.time;
        w.bomb(p.x, p.y, 90, 1000);
      },
    });

    w.on('update', () => {
      if (state !== 'play') return;
      if (got() >= GOAL) state = 'win';
      else if (left === 0 && w.world.time - lastAt > 8) state = 'lose';    // eight seconds for the collapse after the last bomb
      else return;
      if (got() > best) { best = got(); w.store.set('best', best); }
      w.toast(state === 'win' ? (ru ? 'Победа!' : 'You win!') : (ru ? 'Не хватило. Ещё раз?' : 'Not enough. Again?'));
      w.sound(state === 'win' ? 'horn' : 'thud', w.camera.x, w.camera.y, 0.7);
    });

    w.on('hud', (ctx, width, height, free) => {
      const text = (ru ? 'Бомб: ' : 'Bombs: ') + left + (ru ? ' · снесено ' : ' · wrecked ') + got().toFixed(1) + '% / ' + GOAL + '%'
        + (ru ? ' · рекорд ' : ' · best ') + best.toFixed(1) + '%';
      ctx.font = '600 14px system-ui, sans-serif';
      ctx.textAlign = 'center';
      const tw = ctx.measureText(text).width + 24;
      ctx.fillStyle = state === 'win' ? 'rgba(30, 110, 50, 0.85)' : state === 'lose' ? 'rgba(130, 30, 30, 0.85)' : 'rgba(20, 22, 28, 0.75)';
      ctx.fillRect(width / 2 - tw / 2, free.top, tw, 26);
      ctx.fillStyle = '#fff';
      ctx.fillText(text, width / 2, free.top + 18);
    });
  },
});
```

### 6.4. Moon gravity

A game rule in one line and a command to toggle it.

```js
Wreck.mod({
  id: 'moon',
  name: { ru: 'Луна', en: 'Moon' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Лунная сила тяжести. Команда «moon» включает и выключает её.', en: 'Moon gravity. The console command “moon” toggles it.' },

  setup(w) {
    let on = true;
    w.set('gravity', 1 / 6);                             // 1/6 of Earth’s; turn the mod off and gravity is back
    w.addCommand('moon', () => {
      on = !on;
      w.set('gravity', on ? 1 / 6 : 1);
      return on ? 'Moon gravity' : 'Earth gravity';
    });
  },
});
```

### 6.5. Immortal citizens

```js
Wreck.mod({
  id: 'immortals',
  name: { ru: 'Неуязвимые жители', en: 'Immortal citizens' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Жители не ранятся и не гибнут — летают, падают и бегут, но живы.', en: 'Citizens never get hurt or die.' },

  setup(w) {
    w.set('peopleDamage', 0);                            // 0 — no damage at all (except vaporization by a nuclear blast)
    w.addCommand('immortal', (arg) => {
      const off = arg === 'выкл' || arg === 'off';
      w.set('peopleDamage', off ? 1 : 0);
      return off ? 'Citizens can be hurt again' : 'Citizens are immortal';
    });
  },
});
```

### 6.6. UFO: your own flying object

The game knows nothing about UFOs: the mod moves the saucer in `update`, draws it in `draw` and acts on the world with control panel
commands. The saucer flies to the tapped point, hovers and lifts people with a beam; the “ufo” command chases it away.

```js
Wreck.mod({
  id: 'ufo',
  name: { ru: 'НЛО', en: 'UFO' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Летающая тарелка поднимает людей лучом.', en: 'A flying saucer lifts people with a beam.' },

  setup(w) {
    let ufo = null;                                      // { x, y, tx, beam, t, pull } — pull: seconds to the next pull of the beam

    w.addTool({
      id: 'ufo',
      name: { ru: 'НЛО', en: 'UFO' },
      hint: { ru: 'Коснитесь — тарелка прилетит сюда.', en: 'Tap — the saucer flies here.' },
      icon: 'M3 13c3-2 15-2 18 0c-3 3-15 3-18 0zM8 11c0-4 8-4 8 0M8 17l-2 4M16 17l2 4',
      down(p) {
        if (!ufo) ufo = { x: w.camera.left - 80, y: w.camera.top + 60, t: 0, beam: false, pull: 0 };
        ufo.tx = p.x;
      },
    });

    w.on('update', (dt) => {
      if (!ufo) return;
      const hover = Math.min(w.topY(ufo.tx), w.groundY(ufo.tx)) - 110;     // hover above the roofs
      const dx = ufo.tx - ufo.x;
      ufo.x += Math.max(-260 * dt, Math.min(260 * dt, dx));
      ufo.y += (hover - ufo.y) * Math.min(1, dt * 2);
      ufo.t += dt;
      ufo.beam = Math.abs(dx) < 6;
      // the beam: every 0.2 seconds lift the living under the saucer (searching for people costs time — not every step)
      ufo.pull -= dt;
      if (ufo.beam && ufo.pull <= 0) {
        ufo.pull = 0.2;
        for (const h of w.findPeople({ x: ufo.x, y: ufo.y + 90, radius: 110, alive: true })) {
          if (Math.abs(h.x - ufo.x) < 26 && h.y > ufo.y) w.pushPerson(h.id, (ufo.x - h.x) * 3, -240);
        }
      }
    });

    w.on('draw', (ctx) => {
      if (!ufo) return;
      const { x, y } = ufo;
      if (ufo.beam) {
        const g = ctx.createLinearGradient(0, y, 0, y + 200);
        g.addColorStop(0, 'rgba(140, 255, 180, 0.45)');
        g.addColorStop(1, 'rgba(140, 255, 180, 0)');
        ctx.fillStyle = g;
        ctx.beginPath(); ctx.moveTo(x - 8, y); ctx.lineTo(x + 8, y); ctx.lineTo(x + 30, y + 200); ctx.lineTo(x - 30, y + 200); ctx.fill();
      }
      ctx.fillStyle = '#9aa6b2';
      ctx.beginPath(); ctx.ellipse(x, y, 26, 7, 0, 0, Math.PI * 2); ctx.fill();
      ctx.fillStyle = 'rgba(160, 220, 255, 0.85)';
      ctx.beginPath(); ctx.ellipse(x, y - 5, 10, 8, 0, Math.PI, 0); ctx.fill();
      ctx.fillStyle = Math.floor(ufo.t * 4) % 2 ? '#ffd23f' : '#ff5a3c';     // blinking lights
      for (let i = -2; i <= 2; i++) { ctx.beginPath(); ctx.arc(x + i * 9, y + 2, 1.6, 0, Math.PI * 2); ctx.fill(); }
    });

    w.addCommand('ufo', () => { ufo = null; return 'The UFO flew away'; });
    w.on('world', () => { ufo = null; });
  },
});
```

### 6.7. Dynamite: a block that explodes

A material with its own `onBreak`: a destroyed block explodes, the blast breaks neighbouring dynamite blocks — the wall goes off in a chain.
A blast from `onBreak` happens on the next game step, so the chain runs by itself and doesn’t hang the game. The full version with a
chain record is the file [examples/dynamite.wmod](examples/dynamite.wmod) next to this document.

```js
Wreck.mod({
  id: 'tnt-blocks',
  name: { ru: 'Блоки динамита', en: 'Dynamite blocks' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Материал «Динамит» в «Кисти»: рвётся цепочкой.', en: 'Dynamite material in the Brush: blows up in a chain.' },

  setup(w) {
    let booms = 0;
    w.addMaterial({
      id: 'tnt',
      name: { ru: 'Динамит', en: 'Dynamite' },
      like: 'wood', color: '#c0392b', hp: 30,
      onBreak(e) { booms++; w.explode(e.x, e.y, 34, 380, { fire: 0.2 }); },
    });
    w.addCommand('dynamite', () => 'Blocks blown up: ' + booms);
  },
});
```

### 6.8. Furniture with pictures

An item drawn with a picture. The picture is embedded in the file as a string `data:image/png;base64,…` — below is a real 8 × 8 dot
gift box. A small picture is stretched without blurring, so pixel art stays sharp.

```js
Wreck.mod({
  id: 'gifts',
  name: { ru: 'Подарки', en: 'Gifts' },
  author: 'ItsLamba',
  version: '1.0',
  api: 1,
  description: { ru: 'Подарок в «Предметах» — вкладка «Моды».', en: 'A gift box in Items — the Mods tab.' },

  setup(w) {
    w.addItem({
      id: 'gift',
      name: { ru: 'Подарок', en: 'Gift' },
      width: 12, height: 12, mass: 1, hp: 15, sound: 'soft', debris: 'cloth', value: 50,
      image: GIFT,
    });
  },
});
const GIFT = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAgAAAAICAYAAADED76LAAAALklEQVR42mM4YWT0H4Y/nLABY2Qxhmlycv9hGKYAWYyBoAkwQVyYsAKCVlDsCwDnCJxpdad+NQAAAABJRU5ErkJggg==';
```

How to get such a string from your own picture — the same as for a sound (6.10), only it starts with `data:image/png;base64,`.
Several looks of an item — `images: [PICTURE1, PICTURE2]`.

### 6.9. A building and a map from the editor — in one mod with your own command

Make two mods in the editor (section 1): “Framed structure” and “This map”. Open the files in Notepad, take from each the long string
in quotes after `const DATA =` and paste it instead of `'…'` below. This way one mod has the house, the map and your own command.

```js
Wreck.mod({
  id: 'my-district',
  name: { ru: 'Мой квартал', en: 'My district' },
  author: 'Your name',
  version: '1.0',
  api: 1,
  description: { ru: 'Дом с башенкой, карта квартала и сирена.', en: 'A house, a district map and a siren.' },

  setup(w) {
    w.addBuilding({ id: 'house', name: { ru: 'Дом с башенкой', en: 'House with a turret' }, data: HOUSE });
    w.addMap({ id: 'district', name: { ru: 'Мой квартал', en: 'My district' }, data: MAP });
    w.addCommand('alarm', () => { w.sound('horn', w.camera.x, w.camera.y, 1); return 'Alarm!'; });
  },
});
const HOUSE = '…';   // the string from the “Framed structure” mod
const MAP = '…';     // the string from the “This map” mod
```

### 6.10. Your own sound

The recording is embedded in the file as a base64 string. How to get it:

- **Windows:** open PowerShell in the folder with the file and run (the string ends up on the clipboard):
  `"data:audio/mp3;base64," + [Convert]::ToBase64String([IO.File]::ReadAllBytes("$PWD\honk.mp3")) | Set-Clipboard`
- **macOS:** in Terminal `echo "data:audio/mp3;base64,$(base64 -i honk.mp3)" | pbcopy`
- **Linux:** `echo "data:audio/mp3;base64,$(base64 -w0 honk.mp3)"`

For ogg write `data:audio/ogg;base64,`, for wav — `data:audio/wav;base64,`. Short recordings are better: the mod file grows by a third
more than the recording itself. Use only recordings you are allowed to use.

```js
Wreck.mod({
  id: 'horn-tool',
  name: { ru: 'Гудок', en: 'Horn' },
  author: 'Your name',
  version: '1.0',
  api: 1,
  description: { ru: 'Инструмент «Гудок» со своей записью.', en: 'A horn tool with its own sound.' },

  setup(w) {
    w.addSound('honk', HONK);
    w.addTool({
      id: 'honk',
      name: { ru: 'Гудок', en: 'Horn' },
      hint: { ru: 'Коснитесь — гудок. Ползунок — высота звука.', en: 'Tap to honk. Slider — pitch.' },
      slider: { label: { ru: 'Высота', en: 'Pitch' }, min: 5, max: 20, def: 10 },
      down(p) {
        w.sound('honk', p.x, p.y, 0.9, p.value / 10);
        for (const h of w.findPeople({ x: p.x, y: p.y, radius: 80, alive: true })) w.pushPerson(h.id, h.x < p.x ? -200 : 200, -150);
      },
    });
  },
});
const HONK = 'data:audio/mp3;base64,…';   // your recording goes here
```

---

## 7. Debugging and common errors

### 7.1. How to see what the mod is doing

- **`w.toast(text)`** — the simplest way: a message at the bottom of the screen. Works everywhere, on a phone too.
- **A console command.** Give the mod a command that shows its state and call it with the `~` key (on a phone —
  menu → “Console”): `w.addCommand('mystate', () => 'mines: ' + mines.length)`.
- **`w.log(…)`** writes to the browser’s developer console (F12 → Console), marked `[mod id]`. It works only in the browser version
  of the game; the Windows and phone versions have no developer console.
- **The “Mods” menu**: you can see what the mod added and its last error (“Error: …”).

### 7.2. Errors when adding a mod

The game writes “Couldn’t add the mod: …” and one of the reasons:

| Message | What’s wrong |
|---|---|
| the file is empty | There is no text in the file |
| the file is too large | Larger than 4 MB — make the pictures and sounds smaller |
| error in the code — … | A typo that makes the file unreadable: an extra or missing bracket, comma, quote. After the dash — the browser’s hint |
| this is not a Wreckbox mod: the file has no Wreck.mod({ … }) | There is no `Wreck.mod` call, or it has no `setup` |
| the mod doesn’t state its format version (api: 1) | `api: 1` is missing |
| the mod was made for a newer version of the game — update the game | `api` is higher than this game version understands |
| the mod has no name | There is no `name` |
| error on start — … | `setup` failed: after the dash — what exactly (for example, “addTool: id (Latin letters) and name are required”) |

Example — `api` is missing:

<!-- test: error "the mod doesn’t state its format version (api: 1)" -->
```js
Wreck.mod({
  id: 'no-api',
  name: 'No version',
  setup(w) { w.toast('Hello'); },
});
```

Example — a tool has no `id`, the mod doesn’t start:

<!-- test: error "addTool: id (Latin letters) and name are required" -->
```js
Wreck.mod({
  id: 'no-tool-id',
  name: 'A tool without a name',
  api: 1,
  setup(w) {
    w.addTool({ name: 'Bang', down(p) { w.explode(p.x, p.y, 30, 300); } });
  },
});
```

### 7.3. Errors during the game

The game writes “Mod “…”: error — …”. Common texts (written by the browser):

| Text | Cause | What to do |
|---|---|---|
| `w.explod is not a function` | A typo in a command name | Check against section 4 |
| `x is not defined` | The variable isn’t declared, or is declared in another function | Declare `let x` in `setup` |
| `Cannot read properties of undefined (reading 'x')` | Reading a field of something that isn’t there: an empty result, no such person | Check: `if (b) …` |
| `Cannot read properties of null` | `getBlock` returned `null` — the cell is empty | Check the result |
| `window is not defined`, `document is not defined` | The mod doesn’t see the page | Do everything through the control panel |
| `fetch is not defined` | The mod doesn’t go online | — |
| `sound: no sound “…”` | No such sound | The list — `w.sounds` and section 10.3 |
| `setBlock: no such material “…”` | No such material | The list — `w.materials` and section 10.1 |
| `spawnItem: no item “…”` | No such item | The list — `w.items` and section 10.2 |
| `set: “gravity” needs a number from 0 to 3` | A number outside the knob’s limits | Section 4.8 |

If errors come often (5 in 20 seconds), the mod turns itself off — turn it on again in the “Mods” menu once you’ve fixed it.

### 7.4. If nothing happens

- The command returned `false` — see why (section 3.6): the cell is taken, the point is outside the world, the step budget ran out (section 5.1).
- `update` is not called on pause — unpause.
- The tool isn’t visible — it is in “Demolish” or “Build” depending on `mode`; mod tools are not available in the map editor.
- The item didn’t appear in “Items” — it is in the “Mods” tab.
- Your own recording is silent in the first moment after the mod is turned on — it is still being prepared.
- The `world` event didn’t come — the mod was turned on in a running game; it will come with the next world.
- **Your own objects hang in the air** (fragments, shell casings, marks on walls). The game moves only its own things: a dot the mod
  drew on a wall won’t fall when the wall is destroyed. Remember which block the object lies on or is stuck in, and check it every
  step with `w.blockAt`; if the block is gone, the object falls until it lands on another one. And don’t forget to remove old objects.

```js
// your own fragments: they fly, land on a block, fall on when the block is destroyed, vanish after 8 s
const bits = [];                                    // { x, y, vx, vy, t, lie, sx, sy }
for (let k = 0; k < 20; k++) bits.push({ x, y: y - 60, vx: (Math.random() - 0.5) * 300, vy: -250 * Math.random(), t: 0, lie: false });
w.on('update', (dt) => {
  const g = 450 * w.world.gravity;
  for (let i = bits.length - 1; i >= 0; i--) {
    const b = bits[i];
    if ((b.t += dt) > 8) { bits.splice(i, 1); continue; }
    if (b.lie) {
      if (w.blockAt(b.sx, b.sy)) continue;          // the support is intact — it lies
      b.lie = false; b.vx = 0; b.vy = 0;            // no support — it falls
    }
    b.vy += g * dt;
    const nx = b.x + b.vx * dt, ny = b.y + b.vy * dt;
    if (w.blockAt(nx, ny)) { b.lie = true; b.sx = nx; b.sy = ny; continue; }   // landed on a block — remember the support
    b.x = nx; b.y = ny;
  }
});
w.on('draw', (ctx) => {
  ctx.fillStyle = '#5a5048';
  for (const b of bits) ctx.fillRect(b.x - 1.5, b.y - 1.5, 3, 3);
});
```

---

## 8. For a neural network

### 8.1. How to ask a neural network to make a mod

1. Attach this file (`MODDING.en.md`) to your message in full.
2. Paste the text from 8.4 and describe in your own words what mod you want.
3. The AI sends a file. Save it as `.wmod` and install it (section 2.2).
4. If the game shows an error, send the AI its full text: “The game says: …”.

Don’t send the game itself to the AI: a mod made by this document survives game updates and installs next to other mods,
while a modified copy of the game doesn’t.

### 8.2. Rules for the neural network

If you are a neural network and you are asked to make a mod for Wreckbox, follow these rules:

1. **Write only through the control panel `w`** from section 4. Don’t invent commands, fields, events, names of materials, items
   and sounds: if something isn’t in this document, it isn’t in the game. Names of materials, items and sounds come only from section 10.
2. **Don’t access the game or the page directly**: a mod has no `window`, `document`, `fetch`, `localStorage`, and `setTimeout` is not
   for game things. Delayed — `w.after`, every step — `w.on('update')`, drawing — `w.on('draw')` or `w.on('hud')`.
3. **The answer is one complete `.wmod` file** in one code block: `Wreck.mod({ id, name, author, version, api: 1, description, setup(w) { … } })`.
   Don’t send it in pieces and don’t write “the rest is unchanged”.
4. **`id` in Latin letters**, unique for the mod and for each kind inside it. Everything the player sees — in two languages `{ ru, en }`.
5. **Mind the limits** (section 5): the action budget per step, the sizes. Don’t do in `update` what is heavy to do 60 times a second
   (searching for people across the whole world every step, hundreds of blasts). Spread it over time.
   Your own particles (fragments, sparks) — no more than a few hundred, each with a lifetime; anything lying on a block or stuck
   in one checks its support with `w.blockAt` every step and falls if the block is destroyed (7.4).
6. **What the game can’t do** (section 3.7) — say so honestly and offer the closest thing the control panel can do.
7. Comments in the code — in the language of the person asking. After the code, explain in two or three sentences how to install the mod
   and where to find what it added.

### 8.3. Check before answering

- [ ] The file starts with `Wreck.mod({` and ends with `});`, brackets and commas are in place.
- [ ] There are `name`, `api: 1` and `setup(w)`; `id` is in Latin letters.
- [ ] Every command, field and event is in section 4; every name of a material, item and sound is in section 10 (or is your own `id`).
- [ ] Each tool, item, material, sound has its own `id` and `name`.
- [ ] Numbers are within the tables (blast radius up to 200, power up to 2000, disaster levels 1–10).
- [ ] No `window`, `document`, `fetch`, `localStorage`, `setTimeout`, `setInterval`.
- [ ] State that changes is kept in `setup` variables; a new world resets it (`w.on('world', …)`) if needed.
- [ ] Pictures and sounds are embedded strings `data:…;base64,…` (if the person didn’t send any, draw in code and use the game’s sounds).
- [ ] The blast must be where the player tapped — a projectile with `gravity: 0` and `fuse` (4.4, “A blast exactly at the point”) or your
      own flying object in `update` with `w.explode` at the right point. An ordinary projectile explodes only on impact and flies past a point in the air.

### 8.4. Ready-made request text

Copy it, finish the last paragraph and send it together with the file `MODDING.en.md`:

```text
Make a mod for the game Wreckbox. Attached is the document MODDING.en.md — the complete description of what mods can do.
Follow it strictly: only commands of the control panel w from section 4, names of materials, items and sounds from section 10,
don't invent anything. Follow the rules from section 8.2 and check the mod against the list in 8.3.
Send one complete .wmod file, in one code block, and briefly explain how to install it and where to find in the game what it added.
If the game can't do something from my request, say so and offer the closest thing.

My mod: (describe in your own words what it should do)
```

---

## 9. Compatibility and rules

### 9.1. Versions

- `api: 1` — the first version of the mod format (Wreckbox 2.0). As long as the game understands `api: 1`, the commands in this document
  **don’t disappear and aren’t renamed**: a mod made today works in future versions of the game.
- New commands appear in the control panel as versions are released. If a mod uses a new command and the player hasn’t updated yet,
  check whether it exists: `if (w.newCommand) …`. The game version is `w.game`.
- A mod with an `api` higher than the game understands doesn’t install: “the mod was made for a newer version of the game — update the game”.
- Mods don’t update themselves: the player installs a new version the same way as the first — with the same `id` it replaces the old one.

### 9.2. Rules

- Making and sharing mods is **free**.
- Not allowed: selling mods; passing a mod off as an official version of the game; publishing the game itself, whole or modified.
- Put your name in the mod (`author`) and don’t pass someone else’s mod off as yours. If you took something from someone, credit the author.
- A mod must do what it says it does. Don’t hide in a mod anything the player doesn’t know about.
- Pictures and sounds — only those you are allowed to use.
- Share `.wmod` files where the game’s community gathers (links are in the game menu).

---

## 10. Appendix: materials, items, sounds

### 10.1. Materials

For `setBlock` and for the `like` sample of `addMaterial`:

| Name | Material | Strength | Density |
|---|---|---|---|
| `wood` | Wood | 60 | 0.7 |
| `brick` | Brick | 100 | 1.9 |
| `concrete` | Concrete | 150 | 2.3 |
| `rconcrete` | Reinforced concrete (concrete with rebar) | 150 | 2.3 |
| `steel` | Steel | 320 | 1.4 |
| `glass` | Glass | 10 | 1.0 |
| `drywall` | Drywall | 24 | 0.45 |
| `tile` | Roof tiles | 30 | 1.1 |
| `panel` | Concrete panels | 160 | 2.2 |
| `aerated` | Aerated concrete | 40 | 0.6 |
| `stone` | Rubble stone | 120 | 2.3 |
| `castiron` | Cast iron | 220 | 2.6 |
| `lead` | Lead | 110 | 4.8 |
| `glassblock` | Glass blocks | 45 | 1.5 |
| `slate` | Slate | 22 | 0.8 |

`getBlock` and the `break` event name materials the same way, and they can also return things a mod can’t place: `'earth'` (ground),
`'rock'` (bedrock), `'lava'`, `'basalt'`, `'rebar'`, `'ice'`. Reinforced concrete there is `'concrete'` with `reinforced: true`.
A mod material is called by its full name: `mod_<mod id>_<material id>`.

### 10.2. Game items

For `spawnItem` and `findItems` (the full list — `w.items`):

- **Home:** `sofa`, `armchair`, `ctable`, `dtable`, `chair`, `bed`, `nightstand`, `wardrobe`, `bookshelf`, `sideboard`, `trunk`, `tv`, `crt`,
  `fridge`, `stove`, `counter`, `washer`, `bathtub`, `toilet`, `bsink`, `radiator`, `piano`, `plant`, `flamp`, `chandelier`, `clamp`,
  `flpanel`, `box`.
- **Office & shop:** `desk`, `ochair`, `filing`, `cooler`, `reception`, `rack`, `vending`, `shelf`, `showcase`, `counter2`, `shopsign`,
  `teller`, `safe`, `atm`, `wchairs`, `qline`, `ratesboard`, `banksign`, `hbed`, `medcab`, `optable`, `console`, `mimic`, `swatsign`.
- **Street:** `tree`, `bench`, `slamp`, `hydrant`, `trash`, `gassign`, `hvac`, `beacon`, `buoy`, `lhlamp`.
- **Vehicles:** `car`, `policecar`, `ambulance`, `truck`, `firetruck`, `swatvan`, `forklift`, `mtruck`, `fueltruck`, `lifeboat`, `tug`, `container`.
- **Machinery:** `transformer`, `bigtrafo`, `diesel`, `press`, `lathe`, `conveyor`, `crate`, `turbine`, `condenser`, `deaerator`, `steamgen`, `gcn`, `fvu`.
- **Military:** `tank`, `btr`, `heli`, `sam`, `aagun`, `ammobox`, `shell`, `bunk`, `gunrack`, `mapboard`, `target`, `tanktarget`, `wreck`.
- **Explosive:** `gascyl` (gas cylinder), `barrel`, `tnt` (TNT crate), `boiler`, `fuelpump`, `fueltank`, `ammobox`, `shell`.

Your mod’s own items — by their short `id`.

### 10.3. Game sounds

For `w.sound` (the full list — `w.sounds`):

- **Blasts:** `boom`, `boom_far`, `boom_low`, `grenade`, `flashbang`, `whistle` (falling bomb), `whoosh`.
- **Hits and breaks:** `wood`, `wood_break`, `metal`, `metal_break`, `stone`, `concrete`, `brick`, `earth`, `soft`, `car`, `ceramic`,
  `glass`, `glass_big`, `crunch`, `thud`, `crack`, `creak`, `groan` (a building groaning), `tink`, `chain`, `piano`, `tree`.
- **Fire, water, electricity:** `ignite`, `hiss`, `steam`, `splash`, `splash_big`, `zap`, `arc`, `thunder`.
- **Other:** `shot`, `laser`, `pop`, `bulb`, `click`, `horn`, `ship_horn`, `gull`, `zombie`, `growl`.

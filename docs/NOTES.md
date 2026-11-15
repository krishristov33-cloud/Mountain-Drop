# The Mountain Drop: Prototype Notes (Lumen)
Prototype v0.4, 2026-11-15. One file, `index.html` (three.js 0.164 from unpkg, no build step). Open it in Chrome or Edge with internet access, or serve the folder (`python3 -m http.server`). v0.3.1 is kept as `index_v3_1.html`; the broken v0.3 is kept as `index_v3_broken.html` (for reference only), v0.2 as `index_v2.html`, v0.1 as `index_v1.html`.
Canonical design: Krea's `Apex/Projects/The Mountain Drop.md`. Motion: Aether's `Apex/Projects/The Mountain Drop/Motion Spec - Aether.md`. These notes cover only what the prototype does.

## v0.6 changelog: realistic bike mechanics (Kris: "I want the ride to feel more realistic mechanics-wise")
- **Momentum:** speed comes from the slope.
  - The model is gravity along the slope, minus rolling resistance (Crr), minus v² air drag (ρ·CdA/2m).
  - Rolling resistance is higher off-road, in snow and in rain. Tucking (W at speed) cuts CdA.
  - Pedalling is power-limited (420 W, capped at 330 N). It's strong from a standstill and fades with speed, and it does nothing above 16 m/s.
- **Lean steering:** A/D set a lean target. The bike leans first, with lag, then turns at yawRate = g·tan(lean)/v. That means steering gets slower as speed rises. Walking pace blends to direct steering.
- **Tyre grip limit:** the sideways force in a turn (v·yawRate) is capped by grip (μg × weather grip × tyre gear), and braking uses up part of that grip.
  - Over the limit, the bike understeers and slides out gradually, and the slide scrubs speed.
  - Sustained heavy sliding above 13 m/s ends in "WASHOUT — LOST THE FRONT".
- **Brakes:** S is progressive (the lever ramps in) and deceleration is capped by grip. Past about 92% of grip the wheels lock and skid, with spray, lower effectiveness and dulled steering.
  - Stopping from 15 m/s takes about 24 m in clear weather and about 50 m in snow. 4-Piston Brakes raise the force, not the grip limit.
- **Suspension:** a spring-damper is driven by terrain bumps and landing impacts. The bike bobs, pitches with the compression and dives under braking. DH Fork and Coil Shock soften the spring and the landing hit.
- **Landing quality:** speed kept depends on how well the bike's pitch matches the slope and on the vertical impact (both scaled by suspension gear).
  - "BUTTER LANDING" keeps speed. "FLAT LANDING −x%" loses up to 45%.
  - A hard, nose-heavy landing crashes.
- **Air:**
  - Flip and spin rotation keeps its momentum after you release Y/H/Q/E and damps slowly. Self-levelling only kicks in once rotation is slow.
  - Popping (Space) on a rising kicker lip adds the lip's lift (popLip), so timing matters.
- **Camera:** slightly more chase lag, plus shake scaled to suspension movement, speed and surface (much less on asphalt).
- **Tuning:** all values are in `PHYS` at the top of the script, and live in the console as `MD.PHYS`. Keys: mass, pedalW, pedalFmax, crr, cdA, rho, mu, maxLean, leanRate, lowSpeedTurn, slideGain, slideScrub, brakeRamp, skidAt, suspK, suspC, suspKick, landMatch, spinDamp, spinAcc, popLip, camLag, camShake.
- **Tests:** headless full autopilot runs (with pedalling below 8 m/s), no errors and no crashes. #1 Cedar Pass 2:06, #15 Fox-Fire Switchbacks 3:33, #12 Silver Peak Straight 2:32. Probes: lean lag measured, snow turn slides (slide 0.23) while clear doesn't, Y flip coasts (−1.28 → −2.84 rad after release). Screenshot: `shot_v6.png`.
- **Known issues:**
  - Tuned blind, with no hands-on play.
  - Holding S skids quickly on low-grip surfaces. Feather it or raise `skidAt`.
  - Rotation momentum makes over-rotation easier, so flips need an earlier release.
  - The drift (C) path still uses the v0.5 model.
  - Without pedalling, the autopilot stalled once on a flat section of #1 (fixed by lower Crr plus pedalling).

## v0.5 changelog: Trail Hub, 20 trails, gear unlocks (Kris: "add a hub screen, where you choose from 20 different trails ...")

### Hub screen (山道 TRAIL HUB)
- Flow: title → **TRAIL HUB** → pick a trail → trail intro card → **DROP IN**. "QUICK RIDE" on the title still drops straight into the loaded trail.
- Look: modern-Japan blue hour, with a night-sky gradient, ridge silhouettes, a torii and village lights.
- Layout: a 5×4 grid of trail cards. Each card shows:
  - the Japanese name with an English subtitle;
  - difficulty pips;
  - a mini-map of the actual trail curve;
  - a straight↔curvy bar and label (Straight / Flowing / Twisty / Switchbacks);
  - length and estimated time;
  - weather kanji for the 5 sections and the palette (blue, dusk, dawn or night);
  - the best score and time, or a CLEAR tag;
  - for locked trails, a lock strip listing exactly what's missing, with gear needs in orange.
- Header: bank, trails cleared (x/20) and gear owned (x/11).
- Detail bar: full facts and a context button. It reads **RIDE ▶** if the trail is open, **GET GEAR ▶** if you've finished the trail before it but lack the gear (it opens the shop on that item), and LOCKED otherwise.
- Controls: mouse (hover selects; click a selected card to ride) or keyboard (arrows/WASD move, Enter/Space ride, G opens the gear shop, Esc goes back to the title).
- Return to Hub:
  - The end-of-run summary has **◀ TRAIL HUB**, RIDE AGAIN and GEAR SHOP. It also lists newly unlocked trails (解放 UNLOCKED) and "ridden but needs gear" hints.
  - The pause menu has **◀ RETURN TO HUB**. Bailing mid-run banks 50% of the run points.

### Trails
- All 20 trails are generated from parameters: seed, curviness, length, gradient, jumps/km and kicker height, rails, road crossings, tunnel and cliff, the 5-section weather set, palette, and prop mix (lamp spacing, guardrail density, vending machines, power poles, maples, senbon-torii corridors).
- **Trail #1 is the original v0.4 run, unchanged** (same seed, curve and layout).
- Curvy trails can turn up to ~45°. The terrain now flattens perpendicular to the trail, so tight bends stay rideable.
- Terrain resolution and vegetation counts scale with length.
- Night trails get a darker palette. Helmet Light lights the road.
- Trail switching reloads the page with `?trail=N&ride=1` (the world builds once per load, which is robust and file:// safe). The trail intro card's DROP IN is the click that restarts audio.

| # | Trail | Diff | Curviness | Length | Autopilot time | Weather (5 sections) | Palette | Unlock rule |
|---|---|---|---|---|---|---|---|---|
| 1 | 杉ノ峠 Cedar Pass | ◆ | Flowing (0.45) | 1.25 km | 1:51 | clear → wind → rain → fog → snow | blue | open from the start |
| 2 | 九十九折 Switchback Serpent | ◆◆ | Switchbacks (0.95) | 1.10 km | 1:59 | clear → clear → wind → clear → clear | dusk | finish #1 |
| 3 | 一本坂 The Long Straight | ◆◆ | Straight (0.06) | 1.50 km | 1:50 | clear → clear → clear → wind → clear | blue | finish #1 |
| 4 | 竹林ライン Bamboo Bends | ◆◆ | Twisty (0.78) | 1.30 km | 2:05 | wind → wind → fog → wind → clear | dawn | finish #2 |
| 5 | 高速林道 Forest Expressway | ◆◆◆ | Straight (0.15) | 1.70 km | 1:50 | clear → rain → rain → rain → clear | blue | finish #3 |
| 6 | 雨ノ蛇道 Rain Serpent | ◆◆◆ | Switchbacks (0.9) | 1.35 km | 2:03 | rain → rain → rain → fog → rain | night | finish #4; gear: Mud Spikes |
| 7 | 月見坂 Moon-Viewing Slope | ◆◆◆ | Flowing (0.3) | 1.60 km | 2:09 | clear → clear → wind → clear → clear | night | finish #5; gear: Helmet Light |
| 8 | 紅葉谷 Maple Valley | ◆◆◆ | Twisty (0.6) | 1.50 km | 2:15 | wind → clear → wind → wind → clear | dusk | finish #2 + #3 |
| 9 | 跳躍台 The Launchpad | ◆◆◆◆ | Flowing (0.35) | 1.40 km | 2:03 | clear → clear → wind → clear → clear | blue | finish #8; gear: DH Fork 200 mm |
| 10 | 雪見峠 Snow-Viewing Pass | ◆◆◆◆ | Twisty (0.72) | 1.50 km | 2:21 | snow → snow → fog → snow → snow | dawn | finish #8; gear: Studded Tyres |
| 11 | 霧ヶ峰 Fog Ridge | ◆◆◆ | Twisty (0.55) | 1.60 km | 2:14 | fog → fog → clear → fog → fog | blue | finish any of #6 / #7 |
| 12 | 銀嶺ストレート Silver Peak Straight | ◆◆◆◆ | Straight (0.1) | 2.00 km | 2:12 | snow → snow → snow → wind → snow | blue | finish #10; gear: Studded Tyres |
| 13 | 鉄道沿い Rail Line Run | ◆◆◆◆ | Flowing (0.4) | 1.70 km | 2:16 | clear → rain → clear → wind → clear | dusk | finish #9 |
| 14 | 星降る道 Starfall Road | ◆◆◆◆ | Twisty (0.68) | 1.90 km | 2:50 | clear → wind → clear → fog → clear | night | finish #7 + #11; gear: Helmet Light |
| 15 | 狐火ループ Fox-Fire Switchbacks | ◆◆◆◆◆ | Switchbacks (1) | 1.70 km | 2:58 | fog → wind → clear → fog → wind | dusk | finish any of #13 / #14 |
| 16 | 嵐ヶ岳 Storm Peak | ◆◆◆◆◆ | Flowing (0.5) | 2.20 km | 2:52 | wind → rain → rain → wind → rain | night | finish #12; gear: Mud Spikes |
| 17 | 千本鳥居 Thousand Gates | ◆◆◆◆ | Straight (0.2) | 1.90 km | 2:28 | clear → clear → wind → clear → clear | dawn | finish #15 |
| 18 | 大滑降 The Great Descent | ◆◆◆◆◆ | Straight (0.12) | 2.50 km | 2:51 | clear → wind → rain → fog → snow | blue | finish #16 + #17; gear: Carbon Frame |
| 19 | 鬼の背 Demon's Spine | ◆◆◆◆◆ | Switchbacks (0.92) | 2.10 km | 3:27 | wind → fog → wind → rain → wind | night | finish #18; gear: DH Fork 200 mm + Helmet Light |
| 20 | 天狗の道 Path of the Tengu | ◆◆◆◆◆ | Twisty (0.65) | 2.60 km | 3:41 | clear → rain → fog → wind → snow | dusk | finish #19; 15 trails cleared; gear: Carbon Frame + Studded Tyres |

The autopilot brakes at 30 m/s, so a real rider is faster. Trails #14–#20 run about 2:50–3:40 on autopilot.

### Gear shop (one-time purchases with banked points; saved in localStorage `mountainDrop.v5`)
| Gear | Cost | Effect | Gates |
|---|---|---|---|
| Trail Tyres | 600 | +12% grip everywhere | |
| Mud Spikes | 1,600 | +30% grip in rain/fog | #6, #16 |
| Studded Tyres | 2,400 | +40% grip on snow | #10, #12, #20 |
| 4-Piston Brakes | 900 | +40% braking | |
| DH Fork 200 mm | 2,800 | +35% landing tolerance | #9, #19 |
| Coil Shock | 1,400 | +15% landing tolerance | |
| Alloy Frame | 1,800 | −6% drag; grey frame | |
| Carbon Frame | 5,200 | −12% drag, +15% boost tank; black frame | #18, #20 |
| XL Boost Tank | 2,000 | +40% boost | |
| Helmet Light | 1,200 | lights the road on night trails | #7, #14, #19 |
| Carbon Full-Face | 1,500 | crash penalty halved; white helmet | |

- Shop controls: arrows/WASD select, Enter buys, Esc/G goes back to the hub. "reset save" asks for confirmation.
- The old v0.1–v0.4 shop levels are migrated into the new save.

### Unlocks and dev
- Finishing a trail marks it cleared and unlocks its branches: #1 opens #2 and #3, there are two branches that rejoin at #8, and so on.
- Gear gates are checked live, so buying the item opens the trail at once.
- **Dev unlock-all:** **Ctrl+Shift+U** toggles it (saved), or add **`?unlockall=1`** to the URL. Both the trail param and unlockall survive the CDN-fallback reload.

### Kept from v0.4
Y/H flips, the v4 rider and bike, all visuals, the error box, the CDN fallback, optional post FX, auto quality, file:// support and the JSR Classic radio default (private test only).

## v0.4 changelog (from Kris's v0.3.1 feedback)
- **Flips moved off W/S:**
  - **Y** = front flip and **H** = back flip, in the air only.
  - W/S stay pedal/tuck and brake on the ground. They are still the direction keys for J/K tricks and the balance keys for manuals, but they no longer flip.
  - Help moved from H to **F1** (or **?**).
  - The start screen, pause menu and help overlay tables are all updated. The start card now scrolls on short windows.
- **Character reference (Higgsfield nano_banana_2_1, 16:9, master concept as image reference):**
  - `concepts/05_rider_bike_ref.png`: turnaround with front, side and 3/4 views. Job `5ed22c46-5687-40d3-adc7-cb1d669d6fda`.
  - `concepts/05_rider_bike_ref_b.png`: rider and bike shown separately. Job `96312a64-5c1c-4a0a-a290-c2ac7c09b8b4`.
  - Both jobs are also in `concepts/jobs.txt`.
- **Rider rebuilt to match the ref** (faceted lathe forms instead of capsules):
  - olive jacket with a cyan zip stripe, a chest stripe and sleeve stripes, plus a collar;
  - backpack with straps and a sternum strap;
  - grey full-face helmet with chin bar, peak, a dark band and tinted goggles;
  - black gloves and cuffs, knee-pad sleeves and caps, charcoal pants, and shoes with soles.
  - The helmet, shoes and knee caps keep a forward orientation whatever the limb twist.
  - **New attack position:** hips back over the saddle, chest low, elbows out, and knees out. A **tuck** pose (W at speed, or while boosting) lowers the chest and head.
  - The same joint rig, so all tricks, the IK hands and feet on the bars and pedals, and the ragdoll still work.
  - A soft fill light rides with the player so the rider reads at night.
- **Bike rebuilt to match the ref:**
  - dark-slate DH frame with a kinked top tube, gusset and teal decal;
  - swingarm on a main pivot, and a coil shock with a real helical spring;
  - single-crown long-travel fork (silver stanchions, dark lowers, arch);
  - stem, riser bar, grips and brake levers; shaped saddle and dropper;
  - knobby tyres, dark rims, 24 spokes, rotors, cassette, derailleur, chain, chainring with guard, and platform pedals.
- **Environment pushed toward smooth stylized low-poly:**
  - **Terrain:** broader, softer landforms with less high-frequency noise, and smoother trail ground (fewer micro-bumps). It uses smooth gradient shading instead of hard facets, with low-frequency colour instead of per-face speckle, plus a mossy verge band.
  - **Trees:** cedars are 4 soft tiers with a dark-to-light gradient per tier. Maples and bushes are rounded blobs with soft gradients. Bushes are fewer, smaller and darker.
  - **Rocks:** 70 rounded boulders with gentle facets (down from 160 jagged ones), and they turn white in snow.
  - **Unchanged:** all the v0.3.1 robustness (optional post FX, jsDelivr fallback, error box, auto quality). Still works from file://.
- **Tests (headless, SwiftShader):**
  - **Flips:** holding Y for 0.35 s gives a flip of −1.85 rad (front), H gives +1.85 (back), and W/S give 0 in the air.
  - **Tricks and help:** J+W → Superman with no flip. F1 toggles help.
  - **Full run:** an autopilot run took 112.4 s with no errors.
  - **file:// boot:** normal, and with the addons blocked (it runs without post FX), both OK.

## v0.3.1 hotfix: "doesn't run" on Kris's Windows PC
- **Root cause (most likely):**
  - v0.3 loaded its post FX (bloom and grade) with 5 static imports from unpkg.com. These files are needed before the game can start, along with their own dependencies (about 8 extra CDN files).
  - In a browser, if any one of those downloads fails, the whole game script silently never runs. The start screen still shows, but DROP IN does nothing and nothing appears on screen.
  - v0.2 only needed three.js itself, which was probably already cached.
  - I reproduced this exactly by blocking the addon URLs on the v0.3 file. The likely culprits on the PC are an unpkg hiccup, an ad-blocker, or an antivirus web filter.
  - A GPU shader failure is still possible, and can't be tested without real hardware. If that is the cause, it now shows up in the error box.
- **Second bug found:**
  - The v0.3 moonlight (the directional "sun") line was accidentally commented out, so it was never added to the scene. It has been restored at half intensity, to keep the approved v0.3 look and shadows.
- **Fixes:**
  - Post FX now loads separately after the game has started. If it fails, the game runs without bloom or grade and shows a small warning.
  - three.js tries unpkg first. If that fails, it reloads once from jsDelivr (`?cdn=1`).
  - A boot guard shows a **red on-screen error box** for anything fatal, so Kris can screenshot it. It covers:
    - script errors and unhandled promise rejections;
    - the game script not starting, with a three.js reachability check;
    - WebGL not starting, with a hint about hardware acceleration;
    - shader compile errors, with the GPU log;
    - GPU context loss;
    - errors in the game loop.
  - Audio and radio start inside try/catch, so the start button always works. localStorage access is guarded.
  - Auto quality: if the frame rate stays under 30 fps for 3 s while riding, it first turns post FX off and drops the resolution to 1×. If it is still slow, it turns shadows off and lowers the resolution further. A warning explains each step, and G turns post FX back on.
- **Tests (headless Chromium, SwiftShader):**
  - **Opened as file://:** it boots and DROP IN starts a ride, JSR Classic plays, and no error box appears.
  - **JSR blocked:** it falls back to the synth.
  - **Addons blocked:** it runs without post FX and shows a warning; v0.3 died silently in the same test.
  - **unpkg blocked:** it reloads from jsDelivr automatically and runs.
  - **Both CDNs blocked:** the error box explains the problem.
  - **No WebGL:** the error box explains the problem.
  - **Over http:** it runs, and a full autopilot run took 111.5 s with no crashes or errors.

## v0.3 changelog (from Kris's v0.2 playtest)
- **Hold-to-boost:**
  - Shift (or I) drains the meter only while it is held, about 2.4 s for a full tank, and stops the moment you let go;
  - pressing on the beat gives PERFECT (half drain for 1 s);
  - the HUD meter is labelled "HOLD SHIFT", and an empty tank shows "BOOST EMPTY".
- **Controls remap:** the left hand stays on WASD and the right hand sits on J/K/L/I. Arrows act as WASD. The full map is below and appears on both the start screen and the pause menu.
  - The help overlay is now a short cheat sheet.
  - Drift moved to C, because Ctrl+W closes the browser tab.
- **Pause:** P, Esc or the on-screen ⏸ button (top left). The menu has Resume, Restart run, Controls and Mute.
  - Pausing freezes the simulation, suspends the Web Audio context and pauses the radio element.
  - Held keys are cleared when you pause.
- **Radio:**
  - the default is **JSR Live · Classic** (`jetsetradio.live/radio/stations/classic/`, checked against the site's `classic/~list.js`, 26 tracks), with no other station preselected;
  - it starts on the first click or keypress (start button, pause button, or any key);
  - if the stream fails (4 errors in a row, play() rejected, or no progress after 10 s), it falls back to the procedural Spirit FM 174 DnB.
  - The old "JSR Live can't be embedded" hint is gone.
- **Visuals (matched to `concepts/01_master.png`, also checked against 02–04):**
  - **Trail:** it is now a blue-hour asphalt mountain road, with white edge lines and wooden kickers.
  - **Roadside:** galvanised W-beam guardrails on about 72% of each side, with red/amber reflectors. They act as soft walls that bounce and spark, and drifting into one is a washout.
  - **Lights and props:**
    - sodium street lamps every 46 m, with light pools on the road, glow and bloom;
    - power poles with sagging wires;
    - Japanese road signs (急勾配, 落石, 40, curve, ♨);
    - glowing vending machines;
    - a brighter orange-red torii.
  - **Vegetation:** tiered dark-teal cedars, a few autumn maples, and roadside bushes, all instanced in 120 m chunks.
  - **Background:** 3 layers of misty ridges and about 1,600 valley town lights.
  - **Grade and lighting:**
    - cool moonlight key light with a blue hemisphere fill, and the fog colour matched to the concept's indigo;
    - the sky gradient has a horizon glow.
  - **Post FX:** half-res bloom, a grade pass (saturation plus a blue lift) and a vignette. G toggles post FX on and off (saved).
  - **Pixel ratio:** capped at 1.5.
  - **Weather:** each weather type sets its own palette:
    - rain makes the road wet, with brighter lamp pools;
    - snow turns the road and trees white.
  - **Rider:** olive jacket with a cyan stripe and a grey helmet, as in the concept.
  - **Outlines:** none, because the concepts have none.
- **Bug fix:** drift direction now comes from the steer key you hold. In v0.2, leftover steer could latch the wrong side, so the drift never washed out.

## v0.2 changelog (from Kris's feedback: stiff, slow, wants drifting and more tricks, wants JSR)
- **Feel:**
  - faster: cruising is about 40-55 km/h, boost reaches about 110 km/h, and the top speed is soft (no hard clamp);
  - steering eases in and springs back;
  - lean = atan(speed × turn rate / g), and the rider leans the other way;
  - camera: lag is shorter, it swings into turns, FOV kicks with speed (+12° on boost), speed lines, and shake at high speed;
  - torii fade out when the camera gets close, so the camera no longer clips through the beams.
- **Drift** (Ctrl or C + steer):
  - counter-steering sets the angle;
  - spray trail matches the weather;
  - angle × speed × time builds points and fills boost;
  - releasing cleanly gives a **Drift Bank** ×1.25;
  - going past about 75° (clear) or hitting something while drifting = **washout** crash;
  - rain and snow hold a wider, longer angle; clear has the most grip; sakura wind pushes the angle.
- **Tricks and combos:**
  - modifier + direction air tricks (J body / K bike), plus flips and spins;
  - ground tricks: manual, nose manual, bunny hop, manual hop;
  - **rail grinds** on 6 red shrine handrails;
  - THPS-style combo, details below;
  - combo HUD, trick flash and balance meter.
- **Rider and bike:**
  - jointed mid-poly rider: helmet with visor, torso with backpack, upper and lower arms and legs;
  - IK: feet follow the spinning pedals, and hands follow the bars;
  - mid-poly MTB: frame, rear shock, fork with stanchions, spoked wheels with disc rotors, cranks that spin with speed, a fork that steers;
  - each trick has its own pose; the ragdoll uses the new rig segments;
  - soft PCF shadows from the sun, whose shadow camera follows the player.
- **Radio:** a JSR Live station for private testing (details below). The radio panel now sits above the start screen, so you can tune in before the run.
- **Mountain:**
  - a **cliff jump** before the last torii: a 3 m lip, then a 7 m drop with a warning;
  - weather changes automatically by section: clear → wind → rain → fog → snow (key 0 = auto, 1-5 = manual).

## Controls (v0.4; v0.5 adds: hub arrows/WASD + Enter, G = gear shop, Esc = back, Ctrl+Shift+U = dev unlock-all)
- **A / D:** steer and lean. **W / S** on the ground: pedal (or tuck when fast) / brake. **Y / H:** front / back flip (air only). **Q / E:** 360 spins (air).
- **Space:** hold to charge, release to bunny hop or jump; also hops off a rail.
- **Shift or I:** hold to boost. Drains only while held; a press on the beat is PERFECT.
- **C + steer:** drift. Counter-steer sets the angle, release to bank, and past about 75° is a washout.
- **J + W/S/A/D:** body tricks: Superman / Can-Can / No-Hander / Table Top.
- **K + W/S/A/D:** bike tricks: X-Up / Barspin / Tailwhip / Turndown.
  - Hold tricks score per second held. Release before landing.
- **L (hold):**
  - on the ground: manual (L+W = nose manual), balance with W/S;
  - in the air: wide rail catch;
  - on landing: links a manual into the combo.
- **Rails:** land on one to grind, and balance with A/D.
- **P / Esc / ⏸:** pause. **R:** respawn. **M:** mute. **F1 or ?:** help. **G:** post FX. **1-5:** weather (clear, rain, snow, fog, wind). **0:** auto weather.
- **Arrow keys:** work the same as W A S D.

## Scoring (combo rules)
- Every trick in a combo adds +1 to the multiplier.
- A repeated trick is worth half each time it repeats.
- Air bonuses: flips 800 each, 360s 500 each, Big Air for airtime.
- Drifts, manuals and grinds keep the combo alive. Landing starts a 0.45 s window to link one of them.
- The combo banks into run points when it ends cleanly. A crash, a manual bail or falling off a rail loses it.
- A landing on the beat gives +1× and a little boost.
- Bigger combos give more boost (total ÷ 2600), and a banked combo scatters nearby spirits for one bar.
- Points still bank only at the finish. A crash costs 10% of unbanked points (minimum 150).

## JSR Live result (tested 2026-11-15, headless Chromium from localhost)
- **It plays.** The site has no live stream; it shuffles static MP3s per station. The game picks tracks the same way, from a snapshot of 18 stations' track lists (classic, future, ultraremixes, garage, turntablism, liquid, the gang stations and others), with a JSR bump every 5th track.
- Measured in the browser:
  - the `<audio>` element reached readyState 4 and currentTime kept advancing (233 s track) with no MediaError;
  - **Next** loaded and played the next track;
  - responses were `206 audio/mpeg`, with no hotlink block.
- **It can't be analysed or iframed.** There is no `Access-Control-Allow-Origin`, so it plays only through a plain `<audio>` element, with no crossOrigin attribute and no Web Audio routing. Spirits use the 174 BPM clock (the HUD shows "SYNC: 174 BPM CLOCK"). X-Frame-Options plus the CSP block iframes.
- The station is labelled **PRIVATE TEST · NOT SHIPPABLE** (it is a fan site re-hosting SEGA and licensed music). v0.3: Classic is the default station and starts on the first gesture. If a track fails 4 times in a row, play() is rejected, or there is no progress after 10 s, the radio switches to Spirit FM 174 (procedural DnB) and shows the reason. **OPEN JSR IN TAB** is still there.
- The shippable path is unchanged: our own CORS-hosted DnB stations, so the analyser can drive the spirits.

## What's in from v0.1, still true
- **Mountain:**
  - a 1.25 km procedural descent with 5 torii checkpoints;
  - konbini town and Mountain Shop at the bottom;
  - 12 kickers, 2 road crossings and a neon tunnel;
  - about 2,600 cedars.
- **Crash:**
  - 70 ms hit-stop and a verlet ragdoll rider plus a rigid bike;
  - red ink and cyan tears;
  - the music muffles, then a silent beat, then an ink wipe on the beat.
- **Spirits:**
  - koi ribbons that surge on kicks, flicker on snares and stretch with the bass;
  - each kick within 5.5 m below 40 km/h adds a hit, and the 3rd hit trips you.
- **Weather handling:**
  - rain: low grip;
  - snow: very low grip and floaty air;
  - fog: short view, spirits spawn closer;
  - sakura wind: gusts on the bar.
- **Shop:** tyres, brakes, boost tank and suspension upgrades.
- All tunables are in `T` at the top of the script.

## Tests run (headless, SwiftShader, fixed-step `MD.sim`)
- v0.5:
  - All 20 trails load (`?trail=N&unlockall=1&ride=1`) with no errors, and each completes a full autopilot run to the summary. Times: 1:50 (#3) to 3:41 (#20). The only crash in 20 runs was one on #19.
  - Unlock flow: a fresh save shows only #1 open. Finishing #1 shows "解放 UNLOCKED #2 #3".
  - Gear gate: #6 is ridden-but-locked until Mud Spikes is bought (bank 3000 → 1400), then opens. The save persists in localStorage.
  - Ctrl+Shift+U opens all 20 and toggles back.
  - Pause → RETURN TO HUB banks 50%.
  - Clicking a card navigates to `?trail=3&ride=1` with the intro.
  - Y/H flips still read −1.85/+1.85, with W/S at 0.
  - file:// works normally, with `?trail=15`, with post FX blocked, and with unpkg blocked (jsDelivr fallback keeps `?trail=12&ride=1`).
- **v0.3 (real keyboard events through Playwright):**
  - **Load and full run:** load has no errors. A full autopilot run took about 119 s of game time, with no crashes and no errors, and the guardrails didn't block the autopilot.
  - **Boost:** holding for 0.5 s drained 0.21, and on release the drain stopped at once. I boosts too.
  - **Drift and manual:** C+D drift: the angle went 0.41 → 1.11, counter-steer brought it to 0.88, and release gave Drift Bank. The L manual held.
  - **Tricks:** K+A Tailwhip and J+W Superman → a ×3 combo that banked 4,531 points.
  - **Pause:** P froze the game time and position; the audio context was suspended and the radio paused. Esc resumed both. The ⏸ button and Restart run both worked.
  - **Radio:** JSR Classic started on the first click (playing, currentTime advancing). With jetsetradio.live blocked, it fell back to the synth (mode `synth`, with a "play() failed" note).
- **v0.2:**
- **Full run:** an autopilot completed the course in 110 s of game time with no crashes and no console or page errors. The weather changed automatically at each torii.
- **Drift:** start, counter-steer, a clean bank (+combo), and a forced washout all worked.
- **Air combo:** Bunny Hop + Tailwhip + Superman + Big Air → ×5, banked. X-Up, Table Top, Barspin, Can-Can and No-Hander poses all checked.
- **Ground and rails:** a manual kept the combo alive. A rail grind caught and balanced.
- **Boost:** peaked at 117 km/h.

## Known gaps
- v0.5:
  - Picking a different trail reloads the page, so the radio restarts on DROP IN.
  - Generated trails are parameter-built, not hand-tuned. The kicker and rail spacing can feel samey, and difficulty is only roughly balanced.
  - Long trails (2–2.6 km) build bigger worlds. Speed was only checked under SwiftShader, so it needs a check on Kris's GPU.
  - Bailing to the hub mid-run banks 50%.
  - The autopilot crashed once on #19 (out of 20 full runs).
- **Performance:** not measured on real GPUs. Bloom, soft shadows, about 4,300 instanced plants and lamp pools all add cost. Tested only in SwiftShader. On a slow laptop, press G to turn post FX off.
- **Road vs. trail:** the trail now reads as an asphalt road, to match the concept. Krea should confirm this against the design doc's dirt-trail wording.
- **Guardrails:** they are soft walls on about 72% of each side. The gaps are random per 40 m block, so they aren't hand-placed.
- **Concept details not matched:**
  - lamps use light pools, not real point lights, so in rain there are no specular streaks;
  - no outlines and no hand-painted texture detail.
- **Drift key:** drift is C, not Ctrl, because Ctrl+W closes the tab.
- **Untested by hand:** feel at 60 fps still needs Kris's playtest.
- **Not built yet:**
  - no gamepad support;
  - the run is still about 2 minutes;
  - grinds still have no approach line.
- **JSR:**
  - needs internet and uses a snapshot of track lists;
  - **PRIVATE TEST · NOT SHIPPABLE.**

## Files
- `index.html` (v0.6) · `index_v5.html` · `index_v4.html` · `index_v3_1.html` · `index_v3_broken.html` · `index_v2.html` · `index_v1.html`
- `concepts/` (4 Higgsfield concepts)
- `shot_*.png` (v0.1)
- `shot_v2_*.png`:
  - 1: drift in rain
  - 2: air combo HUD
  - 3 and 3b: rider and bike
  - 4: JSR radio
  - trick_*: each trick pose
- `shot_v3_*.png`:
  - 1: clear, CP1 torii, lamps and guardrails
  - 2: rain spirit chase
  - 3_snow: snow, CP4 torii
  - pause: pause menu with the controls shown
  - compare: concepts 01 and 03 side by side with v0.3
  - 1_errorbox_demo: the v0.3.1 error box (both CDNs blocked on purpose)
- `shot_v4_*.png`:
  - 1: clear, CP1 torii
  - 2: snow, CP4 torii
  - rider_closeup: the reference sheet next to in-engine side, 3/4, tuck and Superman views
  - compare: concepts 01 and 03 side by side with v0.4
- `shot_v6.png`: v0.6 ride, Cedar Pass in sakura wind
- `shot_v5_*.png`:
  - hub: the Trail Hub mid-progression
  - shop: the gear shop
  - curvy: #2 Switchback Serpent at dusk
  - straight: #12 Silver Peak Straight in snow
- `concepts/05_rider_bike_ref*.png`: the character and bike reference sheets

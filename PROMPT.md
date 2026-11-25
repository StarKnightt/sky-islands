# Prompts

The prompts that produced this project, exactly as they were given to Claude Code, in order.

---

# Version 1: the first build

We have skills installed in ~/.claude/skills. Before you start, read and follow the ones that match this task (three.js, 3D, shaders, graphics, web audio). Use them, don't skip them.

Build a single index.html: floating sky islands at sunset that I can fly through. Three.js from a CDN is allowed. Everything else is generated in code: no images, no textures, no 3D models, no downloaded assets of any kind. Keep index.html under 100 KB.

The bar: when someone sees the first frame they should think "there's no way this is one html file". Beautiful first, and it must feel real. Nothing should look like a cheap game effect.

## The world
- A sea of soft clouds below, slowly drifting, catching the warm sunset light on top and going cool and shadowed underneath.
- 5 to 8 floating islands at different heights and sizes: grassy tops, rocky undersides that taper into hanging roots and rocks, a few trees that sway in the wind, one small cottage on one island.
- Waterfalls pouring off the island edges, falling down into the clouds. The water must flow and fall with real physics: a smooth sheet at the top that breaks into streams and spray as it falls, mist where it meets the clouds.
- Golden-hour sun low on the horizon, warm rim light on the islands, long soft shadows, distance haze and fog so far islands fade into the sky.
- A flock of birds that flies between the islands with natural flocking.

## Flying
- I control a small paper glider in third person. Mouse to steer, W/S for speed, smooth and floaty like gliding on wind. Gentle banking when turning.
- The camera follows smoothly with a little lag, never snapping.
- Start the experience on a cinematic opening shot that slowly reveals the islands and the sun, then hand control to me on click.

## Sound
- Synthesize ambient sound in code with Web Audio: soft wind that rises with speed, distant waterfall rush that gets louder near falls, occasional birds. No audio files. Starts on first click.

## Performance
- Smooth 60fps on an RTX 4060 at 1080p, and still runs on a normal laptop (scale quality down automatically if needed).

## Process
1. Build a first version.
2. Take screenshots from at least 8 different angles including the opening shot, close to a waterfall, above the clouds and looking toward the sun.
3. Act as a harsh art director: list everything that looks fake, flat, cheap or game-like. Fix it. Repeat this loop until a critic would genuinely believe it's a high-end demo. Use at least 6 rounds. Keep a short log in notes.md of what you fixed each round.
4. Check the final file size and that it runs with no console errors.

## Deliverables
- index.html (under 100 KB) and a README with how to run it.
- recording.mp4: a 45 to 60 second 1920x1080 60fps flythrough with the ambient sound, starting on the opening reveal, then flying between islands, past a waterfall up close, ending on a wide shot with the sun.
- thumbnail.png: the single most beautiful frame.
- Tell me the exact final size of index.html in KB.

---

# Version 2: first-person flight and realism

Good first version. Now take it to the next level. Fix these:

1. Remove the paper glider completely. The player is the camera itself: free first-person flight, like you are flying through the world with your own eyes. Smooth, floaty, cinematic movement.

2. Mouse must lock on click (Pointer Lock API). Mouse look like a real game, Esc to release. Keep W/S for speed, add A/D to strafe, Space/Shift to rise and fall.

3. Remove the small floating debris rocks. At video size they read as black dots and dirt specks. Remove them everywhere unless one is large and clearly a rock.

4. Make it much more realistic. Before changing anything, search the web for real reference photos and study them: golden hour sea of clouds from a plane window, real tall waterfalls falling off cliffs into mist, real sea stacks and cliff rock up close, real trees at sunset with backlight. Write in notes.md what you learned from each reference and what you changed because of it. References are for study only — everything in the file stays generated in code, no images embedded.
   Push on: cloud lighting and depth, rock detail and colour, waterfall shape and mist, tree silhouettes, atmospheric haze and scale. It should look like a still from a film, not a game.

5. Redo the cinematic recording like a real film trailer: slow, intentional camera moves, no player-style wobble. Shots: a slow reveal through the clouds rising up to the islands with the sun behind them, a close glide along a waterfall from top to the mist, a sweep past the cottage with its lit windows, a pass between two islands, and a final wide pull-back at sunset. Smooth easing, no sudden turns. 50-60 seconds, 1920x1080 60fps with the sound.

Run the art-director loop again with at least 6 rounds against the reference photos. Keep index.html under 100 KB. When done, tell me the new size, and give me a new recording.mp4 and thumbnail.png.

---

# Version 3: water, soil and rock, grass

We have skills installed in ~/.claude/skills. Read and follow the ones that match this task (three.js, shaders, water, terrain, foliage). Use them.

Version 2 is great. Clouds, sunset, sky and atmosphere are nailed — do NOT change them. You own this round.

GOAL: the water, the floating soil/rock and the grass must look real next to real photographs. That is the definition of done, not "looks better".

Focus only on:
1. Water physics: waterfalls and the pond/stream must feel like real water — the lip where water curves over the edge, the sheet thinning and breaking into ropes and droplets as it falls, white aerated water, spray that drifts with wind, mist that billows where it hits. Flowing water on top of the island should move with direction, foam and reflections.
2. Floating soil and rock: the undersides of the islands should look like real earth torn out of the ground — layered soil and roots on top, rock below, hanging roots, crumbling edges, moss and wet darker patches near the waterfalls. Real scale and detail when the camera is close.
3. Grass and ground: natural variation in colour, height and density, wind waves moving across it, small flowers/stones/dirt patches, grass that catches the golden backlight.

Process — gauntlet loop:
- First, find real reference photos online for each of the 3 focus areas (waterfall lip and spray, exposed soil cliffs with roots, sunlit grass meadows). Study them only; nothing gets embedded.
- Then loop: make your changes, capture close-up frames of each focus area, then spawn ONE fresh critic subagent (sequential only, never parallel — this is an RTX 4060) that compares your frames side by side with the reference photos and lists everything that still looks fake. Fix it. Repeat until the critic can't find anything major. At least 5 rounds. Log each round in notes.md.
- Keep 60fps at 1080p on the RTX 4060 and index.html under 100 KB.

When done:
- Re-record the cinematic trailer the same way as v2, but start it already out of the dark cloud (no dark murky opening), and add one slow close shot of a waterfall lip and one low shot gliding over the grass.
- New recording.mp4 (1920x1080 60fps with sound) and thumbnail.png.
- Create a public GitHub repo StarKnightt/sky-islands, push the project, enable GitHub Pages and give me the live link. Make an OG image from the best frame and set it as the repo's social preview. Add my support buttons to the README (Buy Me a Coffee + GitHub Sponsors) from assets/support/ in my profile repo StarKnightt/StarKnightt.
- Tell me the final size of index.html.

---

# The /goal condition for version 3

Every item in my v3 instructions above is done and proven: (1) a fresh critic subagent, comparing close-up frames side by side with the real reference photos, finds nothing major still looking fake in the waterfalls, the floating soil and rock, and the grass; (2) clouds, sky and sunset are unchanged from v2; (3) 60fps at 1080p on the RTX 4060 and index.html under 100 KB, both measured; (4) a new trailer recording.mp4 (1920x1080 60fps with sound, no dark opening, with the waterfall-lip and low-grass shots) and thumbnail.png exist; (5) the public GitHub repo StarKnightt/sky-islands is pushed, GitHub Pages is live with a working link, the OG image is set as the social preview, and the README has my support buttons. notes.md logs every critic round.

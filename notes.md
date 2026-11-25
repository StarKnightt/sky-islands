# Sky Islands — art-direction log

Each round: capture 8–13 fixed angles on the RTX 4060 (opening shot, close to a waterfall,
above the clouds, toward the sun, front-lit, underside, on the island, chase view), list what
looks fake, fix it, capture again. Renderer string is asserted on every run so nothing was
judged on a software rasteriser.

## Round 0 — first version
Working pipeline: baked atmosphere LUT, raymarched cloud sea, PCSS sun shadows, ballistic
waterfalls, boids, synthesized audio. It ran first try but looked like a tech demo:
pale daytime sky, milky haze, grey smoke-column waterfalls, blocky rock.

## Round 1 — "it looks like a game"
- Waterfalls were fat blurry mist columns with round puff sprites → cut mist size/opacity hard, finer spray.
- Rock looked like Minecraft dirt: derivative bump on trilinear noise gives per-voxel facets → C1 noise lookup.
- Sky read as noon: sun 7.5° → 4.6°, less white glare, pink-lit cirrus.
- Islands 300 m away were ghosts → fog density roughly halved.
- Grass blades were chunky triangles with giant diamond flowers → 3-segment thin blades, 770k of them, tiny flower heads.
- Leaves were huge diamonds, trees were lollipops → 6-vertex leaves, more clusters along the branches, bigger trees.
- Floating debris read as dirt on the lens → most of it removed, the rest kept close under the islands.
- Screen-door pattern in the sky from the light-shaft jitter → 9-tap blur of the shaft term.
- Undersides were black → warm bounce from the cloud sea in the ambient cube.

## Round 2 — clouds and water
- Clouds were grainy and stair-stepped → temporal reprojection with neighbourhood clamp, per-frame jitter,
  crossing bisection; clouds now full resolution on the top tier.
- Cloud shadows were muddy brown → less deep multi-scatter, bluer sky fill: warm tops, cool valleys.
- Impact mist was a white ball sitting on the deck → removed; the cloud height field now swells where each fall lands,
  so the mist is the same lit volume as the clouds.
- Falls ended in dotted particle lines → the sheet ribbon now runs all the way to the deck, droplets never go below 1 px.
- Rock still blocky at low frequency → bicubic (8-tap B-spline) lookups for the large scales.
- Cottage lamp glowed through 80 m of rock → hard range limit.

## Round 3 — rock and light
- Rock had cracked-mud cell outlines and contour-map veins → smooth strata + vertical joints + erosion streaks.
- Undersides were smooth melted cones → ridged displacement along the normal, denser meshes.
- Sun was a flat sticker → limb-brightened HDR disc, bloom no longer crushes it.
- Grass tips sparkled on far islands → glint fades with distance.
- Paper glider got crease shading and thin wingtip vortex ribbons.

## Round 4 — creatures, close-ups, direction
- Birds were black boomerangs → real wing planform, hinge-rotating flap, pale underside.
- Front-lit rock was flat orange clay → stronger relief, crevice shading also in direct light, greyer albedo.
- A NaN pixel bloomed into a black blob → guarded the bump normal and clamped the HDR input.
- Wingtip trails were orange laser beams → shorter, thinner, neutral, mostly visible when banking.
- Rock next to the camera was a blurry swirl → ridged mid/high-frequency relief.
- Authored the opening (sun hidden behind the hero island's trees, then cranes up) and the demo flight path.

## Round 5 — water surface, far field, sound
- Pond showed concentric rings (ripples were keyed to the mesh, not the world) → world-space ripples drifting with the flow,
  shallow transparent banks.
- Faint horizontal striations in far clouds: the empty-space skip re-synchronised the rays → jitter kept alive through the skip.
- Soundtrack was mostly sub-200 Hz rumble and the birds were inaudible (checked on a spectrogram) → band-limited low layer,
  airier wind, birds 3× louder.
- Real-time test with a real click, mouse and audio: 59.3 fps at 1080p, no console errors.

## Round 6 — shimmer and scaling
- Distant sun-lit rock crawled with blocky noise: the texel-snapping trick broke mip selection and sub-pixel relief
  fed the 2×2 derivative bump → plain mipmapped lookups, relief fades with pixel footprint.
- Camera banked ~20° with the glider → camera roll halved, bank softened.
- Lower tiers dropped whole islands' grass (instances were ordered per island) → shuffled instances; leaves and grass
  now thin uniformly, leaves grow to compensate, shadow map refreshes less often, tier drops faster when far too slow.

## What a critic could still pick at
- Trees are stylised leaf-card canopies, not photographic.
- Birds are fine as a flock but low-poly if one passes within a few metres of the camera.
- Waterfall droplets are stateless ballistic particles (gravity + linear drag + wind), not a fluid solve.

---

# Version 2 — reference study

References were looked at in the browser for study only. Nothing from them is in `index.html`.

| # | Subject | Photo (Wikimedia Commons) |
|---|---|---|
| R1 | Cloud deck at low sun, from above | `Summer above the clouds (Unsplash).jpg`, Liane Metzler, CC0 — plus the result grid for "clouds from above sunset aerial" |
| R2 | Tall waterfall off a cliff | `SaltoAngel4.jpg` (Angel Falls) |
| R3 | Layered cliff and sea stack in low warm sun | `The Old Man of Hoy, Orkney - geograph.org.uk - 8878.jpg` |
| R4 | Backlit trees | `Backlit tree (29175688128).jpg`, `Back light Tree.jpg` |

## What each reference showed

**R1 clouds.** At a low sun the deck is mostly in shade: deep teal-blue, with only crests and sun-facing
flanks lit, and those are cream-white, not orange. Clouds are darker than the sky at the horizon.
The texture is hundreds of small cells receding to a flat line, which is what makes it read as vast.
A thin haze layer sits above the deck and the sun fans visible rays across it. The horizon is a soft
bright band, not a hard edge. In the redder photos the tops go pink and the valleys go violet.

**R2 waterfall.** The water leaves the lip as a narrow bright jet and widens all the way down into a
soft cone several times wider than the lip. The edges are feathered and translucent, the core is
brighter. The lower half is veil, not strands. The rock directly behind and beside the fall is almost
black with vertical water stains; the dry sunlit rock next to it is saturated orange.

**R3 rock.** Thin, flat, horizontal beds stacked like slabs of different widths: the outline is stepped
and angular, with ledges, notches and overhangs, never rounded. Vertical joints cut the beds into
blocks. Low sun splits the colour hard: orange-red on lit faces, violet-blue in shade, with crisp cast
shadows. Grass sits on every ledge that is flat enough.

**R4 trees.** Backlit, the trunk and limbs are black and clearly visible right out to the edge of the
crown. Foliage is small leaves in thin, roughly horizontal sprays along the limbs, with sky showing
through gaps. Leaves glow where light passes through them; wood never does.

## What changed because of each reference

| Reference | Change in `index.html` |
|---|---|
| R1 deck mostly in shade, teal | Deep multi-scatter term cut to a fifth and the sky fill pushed blue, so valleys go teal and only crests are cream. |
| R1 many small cells | A second billow octave (37 m cells on top of the 100 m ones) and finer erosion, so the deck reads larger. |
| R1 clouds darker than the horizon sky | Far cloud now fades to a darker, bluer haze colour instead of the horizon colour, leaving a band under the bright sky. |
| R1 sun rays across a haze layer | A thin sunlit haze above the deck, shadowed by the islands (shadow map) and by cloud turrets (height field). |
| R2 narrow jet widening into a cone | Sheet width grows with fall time (about 3x by the deck); spray and mist emitters widen with it. |
| R2 feathered edge, bright core, veil low down | Lower half blends from torn streams to a filament veil with a Gaussian core and soft edges. |
| R2 black wet rock beside the fall | Stain under each lip is darker and runs the full height; wet rock no longer sparkles. |
| R3 stacked slabs, stepped outline | Island sides are displaced per bed (3.2 m beds of uneven thickness) with irregular joint blocks; meshes are denser. |
| R3 bed lines and block colour | Shader beds use the same bed function as the mesh: per-bed sandstone or grey, dark bed joints, vertical joints. |
| R3 orange lit / violet shade, crisp shadows | Bluer, stronger sky fill on shaded faces, less warm bounce, sun penumbra halved. |
| R3 grass on ledges | Moss follows the real surface slope, not the noisy bumped normal. |
| R4 black limbs out to the crown edge | 7-10 long limbs per tree plus twigs to every spray. |
| R4 thin layered sprays with gaps | Foliage is flattened sprays along the limbs (not balls), leaves 40% smaller, 30% of trees tall and narrow. |
| R4 leaves glow, wood does not | Stronger transmitted light through leaves, darker crown interior. |

## Version 2 art-direction rounds

Each round: capture fixed angles plus frames from every trailer shot on the RTX 4060, compare with R1-R4, fix.

**Round 1 - first pass against the references.** Slabs read as brickwork (regular joint blocks) -> correlated ledges and
irregular blocks. Chimney smoke was an orange comet -> small grey wisp. Mist was a tan search-light column -> a third of
the opacity. Wet rock threw yellow sparkles -> lower gloss.

**Round 2 - the trailer shots.** Close waterfall looked like a beige curtain with confetti -> veil keeps its filaments,
droplets are finer and no longer forced to 1 px (that made glittering columns at distance). Sun was a hard white ball at
long lenses -> half-degree disc with a tight glow. Flowers were paper scraps -> half the size and number. One island moved
in beside the hero island so the "pass between two islands" is a real canyon. Cloud pass was 22 ms at 1080p -> cheaper
height field for light samples, 0.75 resolution with the temporal resolve: 14 ms worst case.

**Round 3 - rock and shadow.** Cliff looked pixel-noisy (derivative bump fed by sub-pixel detail) -> bump only from the
smooth scales, fine detail in albedo. Faces grazing the sun had zebra shadow acne -> slope-scaled normal offset and a
soft terminator. Moss speckle -> driven by geometric slope.

**Round 4 - flying.** Real-time test with a real click: pointer lock engages, mouse look turns the view, W/S, A/D,
Space/Shift all move the camera, no console errors. Throttle was too abrupt -> gentler. Esc also calls exitPointerLock.

**Round 5 - film.** Added the sun-ray haze, bluer shade, split-toned grade, film grain, and a shallow depth of field on
the two close trailer shots (waterfall, cottage). Smoke puffs showed as balls through the blur -> many faint puffs.
Bed lines were ruled-paper regular -> uneven bed thickness in mesh and shader together.

**Round 6 - cleaning the sky.** The new haze left a faint woven pattern in the sky. Three attempts: aligned 3x3 box
(not enough), exact 6x6 box (blockier), then the real cause - the first sample along each ray carried most of the weight
and flipped with the jitter - fixed by spacing samples quadratically toward the camera. Distant birds enlarged so the
flock reads as birds.

## What a critic could still pick at (version 2)
- The last frames of the waterfall shot are inside the mist: soft and nearly featureless by design.
- Depth of field is a two-level blur, not a lens simulation; it is only used lightly.
- Rock beds are convincing at a distance; within a few metres the mesh steps are coarser than the shader lines.

**Round 7 - reviewing the recording itself.** The reveal shot started in a hollow of the deck, so it never went "through"
the clouds, and the sun disc showed through solid cloud (the march stopped at 2% transmittance and the disc is 1500x
brighter than the sky) -> the shot now starts 260 m down in solid cloud and leftover transmittance is rescaled to zero.
Re-recorded.

---

# Version 3 — water, floating soil and rock, grass

Scope: only those three. Sky, clouds, sun and atmosphere code is untouched (checked at the end by diffing those
sections against version 2 and comparing sky-only frames).

## Reference photos (Wikimedia Commons, study only, kept in `refs/`, never committed, nothing embedded)

| Area | File on Commons | Kept as |
|---|---|---|
| Water | `View of Lower Falls from the Brink of the Lower Falls Trail (54107912090).jpg` | `refs/water_lip_brink.jpg` |
| Water | `Aber Falls - The top of the falls as it tumbles over the edge - geograph.org.uk - 8165037.jpg` | `refs/water_fall_ropes.jpg` |
| Water | `Bowen Falls and mist backlit by the sun.jpg` | `refs/water_mist_backlit.jpg` |
| Water | `Seljalandsfoss at Sunset (15609793935) (2).jpg` | `refs/water_fall_sunset.jpg` |
| Soil | `Red Cliffs of Scarborough tree root erosion Jun07+ (2518262158).jpg` | `refs/soil_roots_a.jpg` |
| Soil | `Red Cliffs of Scarborough Tree Root Erosion-03+ (2518252492).jpg` | `refs/soil_roots_b.jpg` |
| Soil | `Red Cliffs of Scarborough Tree Root Erosion-04+ (2517435401).jpg` | `refs/soil_edge_turf.jpg` |
| Soil | `Red Cliffs of Scarborough Tree Root Erosion-02+ (2518250212).jpg` | `refs/soil_roots_close.jpg` |
| Grass | `Hay in backlight.jpg` | `refs/grass_backlit_close.jpg` |
| Grass | `Big Meadows sunset wildflowers (54386180446).jpg` | `refs/grass_meadow_sunset.jpg` |
| Grass | `Good Morning (215577125).jpeg` | `refs/grass_meadow_wide.jpg` |
| Grass | `A photo of evening sunset over the grass fields in Laaghalerveen; Drenthe, 2012.jpg` | `refs/grass_dirt_patches.jpg` |

## What the references showed

**Waterfall lip.** Upstream the water is dark green glass with standing ripples. It stays glassy over the curve of the
brink and only then turns white, first as streaks that start at fixed spots on the lip (every notch in the rock makes
its own rope). Rock shows dark through the thin water. Foam lines on the river follow the flow and run over the edge
without a seam.

**Falling water.** A small fall is not a sheet: within a few metres it is separate ropes with dark rock between them,
and the outer fringe is single drops drawn into short streaks. Wet rock beside it is nearly black, with vivid green
moss on both margins. Backlit mist glows and drifts sideways as a soft wedge, brightest toward the sun.

**Soil cliff.** Turf overhangs the edge. Below it the soil face is recessed, blocky and crumbly with vertical cracks,
darkest at the top (humus) and paler below. Pale woody roots sprawl out of the top and hang down, ending in bunches of
hair-thin rootlets. Small plants and stones sit in the face. There is a hard shadow under the overhang.

**Grass.** Up close: blades 1-2 cm wide in long arcs, bright yellow-green where the sun shines through them and dark
olive in shade, a bluish sheen on the ones facing the sky, dry straw-coloured blades mixed in. No ground visible.
At a distance: colour and height change in patches, seed heads and flowers catch the light as bright dots above the
blades, and worn ground shows as bare dirt with a soft edge.

## What changed because of them (first pass, before the critic rounds)

| Reference observation | Change in `index.html` |
|---|---|
| Glassy lip, white starts as streaks at fixed spots | The sheet has a stationary lip profile (ropes stay where the lip makes them) plus pulses in time; aeration grows from streaks; the first second is refractive glass tinted by depth. |
| Foam lines run over the edge without a seam | Stream and fall share one pattern keyed on lip position and the time each parcel crosses the lip. |
| Ropes, gaps, droplet streaks | Rope field from the lip profile and analytic noise; real holes between ropes; a droplet-streak fringe; two sheet layers; extra spray thrown off the lip. |
| Rock dark under shallow water, green tint with depth | Stream and pond refract the rendered scene behind them, absorb by path length, reflect the sky and (screen-space) the bank. Shoreline comes from the depth buffer. |
| Water cuts a notch | The channel rises to a rock sill at the lip with shoulders either side, so the sheet leaves the rock, not a floating plane. |
| Black wet rock, moss margins | Wet zone under each lip is streaky and darker; a moss band on its margins. |
| Turf overhang, recessed crumbly soil, shadow under the lip | Rim wall has twice the rows, an undercut profile, clod-scale displacement and its own ambient occlusion; the rim outline has bites taken out of it. |
| Humus over paler subsoil, cracks, stones | Soil horizons in the shader by depth below the turf: dark crumbly topsoil, ochre subsoil with vertical cracks and pebbles, rubble into rock; seepage stains on the rock below. |
| Pale woody roots, bunches of rootlets | Roots are thinner, gnarled, paler, with side roots; about 28,000 hair-thin hanging rootlets and drooping turf blades on the rim. |
| Blades in arcs, translucent, sheen, straw mixed in | A ring of 260,000 three-blade tufts follows the camera, rooted on a height map baked from the island mesh; curved blades with a midrib normal, transmitted light, sky sheen, seed stalks. |
| Patches of colour, height, bare dirt | One set of ground fields (dry, lush, dirt) drives blade colour, height and density and the ground under them, in JS and GLSL from the same noise texture. |
| Gusts cross a meadow as bands | Gust cells drift downwind; they bend blades and lighten them. |

## Version 3 critic rounds

Each round: capture 16 close-up frames (lip x2, falling sheet x3, stream, pond, soil band x2, wet rock, underside,
rim, grass x4) on the RTX 4060, hand them and the 12 reference photos to one fresh critic agent that has never seen
the scene, fix what it calls major, capture again. One critic at a time.

**Round 1.** Verdict: major issues in all three areas.
- Water: lit sheet "a tan fabric curtain", blocky noise on it, the lip "a bent strip" with a crease and a ruler-straight
  brink, a cream plank on the approach, the stream "a canal" with cut-out foam, no mist where the sheet breaks, the far
  fall a constant-width ribbon, the pond a perfect ellipse.
- Soil and rock: evenly spaced contour lines on all rock, the soil band one dimple pattern ("crocodile skin"), a
  ruler-straight turf edge, roots "sticks and wires", wet rock like gold foil, moss a blurry tint, underside without
  torn-earth character.
- Grass: close blades "flat paper strips", one uniform backlight colour, a monoculture, an even carpet, dirt patches
  "paved shapes", stones unlit blobs.
- Held up: the lip's side profile, ropes with rock between them, the pond reflection, island silhouettes, the meadow
  at a distance.

**Round 2 - fixes for round 1.**
- Water: stream and falling sheet now share one shading path (refractive glass tinted by depth, sky and screen-space
  reflection, foam fraction growing with fall time), so the crease and collar at the lip are gone. The brink follows
  the lip profile instead of a straight line. The "cream plank" was the stream bed poking through a channel too shallow
  for the mesh: channel deepened, sill kept under water, underwater surfaces lose their gloss. Ripples are six analytic
  sine waves travelling with the flow (no texel blocks). The stream meanders and changes width, the pond has an
  irregular outline, and the same channel function drives terrain, grass masks, wet banks and water meshes. Aerated
  water is brighter in its cores and sky-lit on its shaded sides; a soft veil surrounds the ropes; 500 mist puffs sit
  close around the sheet where it breaks; the sheet widens faster, frays more with height and sways with each pulse.
  Blocky value noise replaced by quintic two-octave sheared noise.
- Soil and rock: bed joints fade in and out along each bed and the beds undulate; the soil band is lit from three
  scales of lumps with its own occlusion and no rim gloss; the turf line is ragged in the mesh and in the shader, with
  five times the drooping blades; roots leave the wall sideways, wander, turn down and taper, with side roots and
  knobbly radius; rootlets hang in bunches; wet rock is dark and rough with tight glints; moss has a sharp edge, its
  own grain and relief; twice the hanging roots under the islands.
- Grass: tuft blades are half as wide, twist along their length and vary far more in height and tone; dead blades lie
  low; broadleaf rosettes, flower cups on stems and tall seed stalks are mixed in; static blades are thin near the
  camera, vary in height and colour by patch and thin out in poor ground; dirt has a ragged edge, lumps and stones;
  stones are lighter.

**Round 2 verdict.** Still major in all three, but the list changed.
- Water: the brink still read as "a flat card" with nothing under it; no near-white water at the lip, sunlit streaks
  "caramel and near-black"; streaks parallel and constant-width for the whole height; two hard-edged slabs inside the
  fall; pond and stream "a flat navy fill" with a clean vector waterline; comb-like aliasing on the stream at grazing
  angles; far falls do not atomise.
- Soil and rock: soil band "popcorn noise" with black shadows; smeared textures; roots "smooth tubes" with tyre-tread
  banding; rock cracks are uniform black lines forming a grid or a Voronoi net; moss "green airbrush"; turf line level
  with a "bead curtain" of rootlets.
- Grass: near blades flat straight polygons; flowers "pennants"; dirt patches striped plates with a vector outline;
  monoculture at mid distance; a picket-fence band where the tuft ring ends.
- Held up: backlit mist halo, sky and bank reflections in the stream, rope distribution at distance, island massing,
  the hair roots at the silhouette, mid-distance sunlit grass, the strength of the backlight.

**Round 3 - fixes for round 2.**
- Water: both sheet layers leave the same brink and launch slower, so the sheet hugs the rock instead of hanging off a
  shelf; aerated water is white from just below the lip, brighter in sun and in shade; the pattern is shorter clumps
  with bright leading heads instead of full-height streaks; the horizontal pulse term that made slabs is gone; the
  outline frays with height; the lower fall thins into mist. Ripple waves spread over more directions, vary in patches
  and each one fades below its pixel footprint (that was the comb aliasing). Grass no longer grows under water: nothing
  grows below the waterline plus 30 cm, the margin is wet gravel with stones, and blades lean over it. Mist puffs use
  tricubic noise (the square mosaic on the sheet was magnified noise texels).
- Soil and rock: drawn bed lines and the Worley crack net are replaced by jointed blocks: jittered box-shaped cells
  flattened into beds, each with its own facet tilt and tone and a thin joint. Moss sits only on faces that look up.
  Soil relief is gentler with fill light, clods and small stones in the albedo, smooth slumped patches and a sharper
  horizon above the rock. The rim sags in places so the turf line undulates. Roots start inside the soil, vary in
  radius by noise, and one in fourteen is trunk-thick. Rootlets are fewer, sit under the turf and curl.
- Grass: static blades are hidden within 6-12 m of the camera where the curved tufts take over (340,000 of them now);
  flowers are small daisy discs that face the camera; tussocks of taller blades and seed heads are scattered through
  the field; patch contrast is stronger; dirt has a blade-scale ragged edge and softer relief; litter colour under the
  blades.

**Round 3 verdict.** Still major in all three.
- Water: sunlit sheet "a cloth curtain" (I overshot the whitening: no gaps near the top); a hard seam where stream
  becomes fall; shaded lip a satin ribbon; a blue seam line and a square mosaic on the sheet; far fall constant width;
  pond "an opaque navy slab" with a uniform brown rim.
- Soil and rock: the joints read as "Voronoi crackle", lines drawn on lumpy forms, not shaded crevices; foil-like
  bumps; soil band "noise on a smooth surface"; combed streaks at the soil lip; moss a decal; roots "cables and rope".
- Grass: close blades still "paper cut-outs" without midrib or glow; daisies are "stickers" that face the camera and
  stay white in shade; bare flat ground showing; dirt patches striped.
- Held up: mid-height rope break-up with real gaps, stream reflections, cool shaded fall, backlit mist, island
  silhouettes, the dry meadow at distance, light-to-shadow transitions, seed heads.

**Round 4 - fixes for round 3.**
- Water: sheet shadows go through the soft shadow filter (the mosaic and the blue "seam" were shadow-map texels of thin
  roots); break-up starts at the lip, so there are ropes and gaps from the first metre; thick parts stay glassy longer
  and thin parts whiten first; the ripple normal carries over the brink and the sheet edge matches the stream edge there
  (no crease); twice the columns across the sheet; reflections march in more, finer steps; the wet margin has uneven
  height.
- Soil and rock: joints get a bevel instead of a drawn line and one layer of blocks per half bed whose length changes
  from bed to bed; moss is a raised cushion on up-facing rock; the soil band is made of clods (small jittered cells)
  with cavities and a few stones; roots kink more often; rootlets hang straight instead of combing sideways.
- Grass: blades have a midrib and a fold, the light through them is greener, the sheen is stronger; flowers are tilted
  discs in world space, lit and shadowed like everything else, and only grow in lush patches; ground under the grass
  has litter texture; dirt gets soft wrap lighting (the stripes were mesh facets under a 5 degree sun).

**Round 4 verdict.** Still major in all three.
- Water: shaded lip "long-exposure silk"; sunlit fall white from the first pixel, "combed fur"; still a straight seam
  at the brink; a dither pattern on the stream near the lip (my jittered reflection march); no rocks or turbulence at
  the brink; far fall constant width; pond "a flat navy cut-out"; foam "airbrushed".
- Soil and rock: the blocks still read as "cracked mud" with outlines, and the bevels catch light so the joints glow
  in shade; soil now "sandpaper"; roots "thick, dark, segmented tubes" and the hair-thin rootlets are missing; flat
  slabs up close; moss a uniform carpet, as heavy on dry sunlit rock as by the fall.
- Grass: shaded grass "a regular black stipple"; close blades still flat strips; dirt patches show the rock crack
  pattern; ground rocks smooth blobs.
- Held up: mid-fall ropes and droplets, stream sky reflection, wet mossy patch behind the fall, overhanging turf lip,
  thin silhouette roots, the golden seed-head field, bank grass with small flowers, backlit translucency colour.

**Round 5 - fixes for round 4.**
- Water: one aeration rule across stream and fall: thin water (low spots of the lip) turns white upstream of the
  brink, thick tongues stay dark glass past it, so there is no line at the edge; foam is crisp lace (cellular noise
  with hard cells) instead of soft streaks, on the stream and in the upper fall; two rocks sit in the brink where the
  lip profile is lowest; reflections bisect their hit instead of jittering (that was the dither); pond bed is pale so
  the shallows read, with stones and leaning blades round it; veil is wider and mist denser lower down.
- Soil and rock: blocks are four times larger and told apart by facet and tone only; joints are a hairline of
  occlusion with no bevel (nothing left to catch light in shade); loose rocks take their facets from the mesh; moss
  is rarer, prefers shade and wet rock, and has no cell pattern; soil clods range from fist to boulder size with
  rills and a pebbly lens; roots are pale grey-tan, weave in and out of the soil face before hanging free, and the
  trunk-thick ones are gone; hair rootlets are back as tight irregular bunches.
- Grass: shaded blades and the ground under them sit at the same tone; deeper fold, more sky sheen, more dead blades
  and a wider spread of blade widths in the tufts; the jointed-rock occlusion no longer leaks onto dirt.
- Performance: one bicubic lookup made trilinear, 300,000 tufts instead of 340,000, 12% fewer static blades, cheaper
  jittered shadow taps on water.

**Round 5 verdict.** Still major in all three; several items are the opposite of the round before (soft foam was
"airbrushed", crisp foam is "cow-print"; small blocks were "cracked mud", large ones are "low-poly").
- Water: foam two-tone with black holes; a bright cell net on the crest (the lace was in the bump map); shaded lip "a
  glass cylinder"; brink rocks do not disturb the water; sheet still "a flat scrolling card", same width top to
  bottom; pond "a matte slab" with a pale "frosted rim" of constant width; distant mist "coloured gas".
- Soil and rock: large flat facets with one "melted-wax" bump; joints still "pen lines"; roots "a few bent pipes",
  bone-white; soil "sandpaper"; "comb-tooth" rootlet bundles; a pale ribbon along the turf edge; moss a tint with no
  thickness.
- Grass: sprigs on dirt "in rows" (the low-discrepancy tuft sequence shows when thinned); bare ground a smooth sheet;
  blades near the bank "plastic ribbons" (those were the old static blades leaning over the water).
- Held up: stream reflections, short-shutter streak length and warm backlit white water, the arc of the lip, the
  layered read of turf, topsoil, subsoil and rock, underside mass and lighting, golden backlight through the blades,
  long tree shadows, the grass silhouette on the rim.

**Round 6 - fixes for round 5.**
- Water: foam is a soft-edged density again but built from three scales (cells, strands, grain), and holes in it are
  thin veil rather than dark glass; the lace is out of the bump map (that was the bright net on the crest); a wider
  rope scale so the sheet has thick and thin zones; cores more opaque; the back layer falls further behind and
  droplets fly further off the plane; each brink rock gets a foam collar and a white wake; reflections step evenly
  along the ray on screen (so thin trunks are not skipped) and bisect the hit; the pond is a bowl with a long shallow
  margin over a smooth dark bed; mist is wispier.
- Soil and rock: two sizes of block (the small ones only near the camera), granular relief instead of wormy ridges,
  joints as soft recesses; soil clods carry the look (cavities, per-clod tone and facets) and the grain steps back;
  the grass colour stops at the turf edge (the pale ribbon was the lip face shaded as lawn); roots are mid grey-brown,
  taper faster, start inside the soil and fork into side roots and twigs; each rootlet curls its own way. Moss
  cushions as instanced geometry were tried and removed: they read as lily pads.
- Grass: tufts sit at random positions (the rows were the low-discrepancy sequence showing through), shrink into
  dirt; dirt uses the soil clods, has pebbles lying on it and is fully soft-lit; the old static blades that lean over
  the water also give way to tufts near the camera.

**Round 6 verdict.** Still major in all three.
- Water: foam still "camouflage" (blotches punched into white); now no glassy zone at the brink; the brink "a
  ruler-straight weir" seen from the front; the sheet flares into a bell with feathered edges; streaks read as a long
  shutter; pond "a flat tar-coloured polygon"; ripples "generic game water" with nothing showing the flow; foam
  collars "plates the rocks sit on".
- Soil and rock: soil "sandpaper" for the third time; roots "pasted tubes" with a squared-off cap; a pale ribbon
  along the turf edge; "polka-dot grass sprigs" on the soil face; rock "melted wax" with smeared texture, "pen-line"
  cracks and wet rock as "gold glitter".
- Grass: one stiff blade type near the camera; no thatch; dirt patches "carpet cut-outs" with black pebbles;
  "crew-cut" height.
- Held up: exposure and colour of sunlit and shaded white water, stream sky reflection, droplet dashes at the sheet
  edge, layer order and colour of the soil profile, warm key and blue shade on rock masses, root silhouettes at mid
  distance, the tawny seed-head slope, backlit rim glow, bank grass with small flowers, grass drooping over the cut
  edge.

Two of those turned out to be bugs, found by switching layers off one at a time:
- The "sprigs on the soil face" were the camera-following tufts. Their height map is bilinear, so across the rim it
  interpolated between the turf and the cliff below and planted tufts half way down the face. Tufts now need a fully
  inside texel.
- The "sandpaper" was the clod cells: I had multiplied their coordinates by a fast-varying noise to vary clod size,
  which scrambles the cell hash into per-pixel speckle. Cells now use a fixed scale.

**Round 7 - fixes for round 6** (besides the two bugs above).
- Water: the lip profile now also sets height and presence: low spots carry no water (rock nubs split the sheet into
  tongues) and the surface dips where it is thin, so the brink is not level; foam is built from flow-aligned
  filaments, not cells; thick tongues are visible green glass with a sky sheen for the first second, then tear into
  white; the sheet stays narrow and its silhouette is made of ropes instead of a feathered fade; beads inside it are
  crisper; the stream carries fine ripples drawn out along the flow; foam round the brink rocks is broken up; spray
  droplets are fainter.
- Soil and rock: no drawn joints left at all, smaller facet tilt, a crisp grit layer so the rock is as sharp as the
  soil above it, less stretched streak noise; wet rock darker with a smooth sheen; loose stones get sky fill; roots
  taper to a point where they leave the soil (no open pipe end), curve more smoothly and no longer run level along
  the rim.
- Grass: more seed stalks of mixed height standing above the mass, 5% of field blades half again as tall; the wet
  margin is soft-lit like dirt; rim blades thin out right at the lens.

**Round 7 verdict.** Still major in all three: pond "a flat cut-out"; "stray flat grey quads" (stones I had made too
bright); sunlit fall "an alpha card" with no dense core; rock "ghost copies" on the fall face (the brink rocks seen
through the sheet); a screen-door pattern on the water (my jittered shadow taps); soil "glossy marbled noise" with
no real relief under a 5 degree sun; roots "bent pipes"; giant flat blades right at the lens; a faceted boulder.
Held up: the curve of the water over the brink, stream reflection, warm and cool water colour, soil layer order at
mid distance, blades draping over the edge, thin hanging rootlets, backlit mid-ground grass, the golden slope, seed
stems.

**Round 8 - fixes for round 7.** Stones back to a normal exposure, small ones faceted and boulders smooth; fixed
(unjittered) shadow taps on water; no sheet in front of the brink rocks; a denser white core behind the ropes; grain
kept at distance so far falls are not silky; mist also lit by the sky; soil clods shadow each other under the low sun
(one more cell lookup toward the sun) and their outlines are gone; no tufts within 80 cm of the lens; all bump relief
now comes from tricubic lookups (trilinear noise showed its texels as a checker on a boulder).

**Round 8 verdict.** Still major in all three, grass "closest of the three; mid-distance grass passes". Water: the
sheet still "a constant-width textured ribbon" with a streak texture and hairline strip edges, the brink "a dead-level
weir", a stipple pattern near the lip, pond "flat matte paint". Soil and rock: soil now "bark mulch" (the clod shadows
were too hard), dashed crack lines (cell-search errors), rock "low-poly facets with a blurry texture" under crisp
soil, roots "smooth pale noodles", rim "a level cake layer". Grass: near blades "flat paper strips", a faceted boulder,
dirt patches "round dark discs".
Held up: stream reflection and Fresnel, the glassy roll-over seen side-on, droplet speckle, sunlit versus shaded water
colour, grass drooping over the rim, underside lighting and silhouette, the root-mat idea, mid-distance grass
translucency and clumping, long shadows across the meadow, seed stalks against the sky.

After round 8: exact cell search (the dashed lines were pixels assigned to the wrong block), no alpha left at the
sheet's strip edge, moderate widening, softer clod shadows.

**Round 9 - fixes for round 8.**
- Rock: the dotted "stitches" were a stepped value (each block's random height) fed into the derivative bump; it is
  out. Close relief now comes from one extra tricubic lookup at 1 cm texels, so rock is as crisp as the soil above it
  without texel artefacts. Loose rocks are one calm block each.
- Soil: matte fine earth with clods only as soft relief, stronger vertical rills, softer clod shadows.
- Dirt: wider, thinner transition into grass.
- Water: flow ripples broader and gentler (they read as scanlines); each tongue turns white at its own height; ropes
  with gaps persist all the way down the veil; the pond margin is dark wet mud instead of a pale ring. Pond and bank
  frames are now taken from eye level (1.2-1.3 m), where water reflects; the earlier ones looked steeply down into it.
- Grass: tuft blades about 40% narrower, a visible crease, blade-to-blade variation in how much light comes through,
  slimmer seed heads.

**Round 9 verdict.** Still major in water and in soil and rock. Grass: "blades and backlight pass; the ground
substrate and slope coverage do not". Water: lip "a glass tube" then an abrupt switch to white; sheet "a rectangle of
scrolling noise" with a dark outline; far falls "straight pipes"; stream "a still pond right up to the drop"; a
fine-line moire; rocks in the water "pasted on". Soil and rock: close soil "wrinkled leather" with a satin sheen; rock
"one-scale procedural marble" with a streak fan under the island; roots "wires"; "orange-peel" bump.
Held up: the pond as a whole (reflection, tone, Fresnel), droplet streaks at the sheet edge, warm and cool fall
colour, short-shutter streak length, soil layering with grass draping over it, root colour, rim grass against the
clouds "close to photographic", golden translucency, seed stalks, sunlit meadow patches.

**Round 10 - fixes for round 9.** Sheet edges are white veil instead of dark glass (the outline); softer lace at the
brink; thin foam lines run down the whole stream with the flow; five ripple waves instead of six, each fading earlier
against its pixel footprint (the moire); stones standing in water are dark and wet at the waterline; the stretched
erosion noise is isotropic (it fanned out on the underside); crisp rock relief comes and goes in patches; soil has no
specular at all and carries millimetre grit, as do bare ground and the wet margin. Performance: roots were 2.9
million triangles and are now about a third of that; stones use a coarser mesh; close-range rock and soil detail is
skipped for distant pixels; far views draw 40% of the field blades, wider.

**Round 10 verdict.** Still major in all three. Water: brink "ruler-straight" from the front and "a bent sheet of
plastic" from the side; residual moire above the lip; a black unlit facet under the lip; the sheet "one constant-width
ribbon"; sunlit white water "flat cream paint" without blown-out cores or a cool shaded side; the stream "a dead
mirror right up to the brink". Soil and rock: soil "swirled noise" that goes blurry up close; roots "isolated smooth
tubes"; a straight soil-to-rock seam; "orange-peel" bump; far rock blotchy; nothing torn on the underside. Grass: bare
ground "a smooth blurry surface"; slopes and banks expose the mesh; the largest foreground blades are flat strips.
Held up: pond and bank water (navy tone, dusk gradient, distorted bank and tree reflections), droplet streaks, the
shaded fall, the foam ring round a brink rock, the turf overhang and dark topsoil band, low-sun response on the soil
face, island silhouettes, backlit glow and rim light, long tree shadows, rim grass against the clouds.

Between rounds 10 and 11 one more cause was found by switching things off: the "moire" on the stream was the three
shortest of the six ripple waves, pure sines seen at a grazing angle. The stream now uses three waves.

**Round 11 - fixes for round 10.** Deeper notches in the brink and wider gaps, so the sheet leaves as separate
tongues that only merge two to five seconds down; the sheet widens faster with the drop; white water runs from
blown-out cores to cool sky-lit sides; wet rock under the lip is dark instead of black; soil has less low-frequency
swirl, twice the grain contrast and pebbles, and bare ground and the wet margin get the same grain and pebbles.

**Round 11 verdict (the build that was recorded and published).** Still major in all three; grass is "the strongest
area; close backlit grass holds up, while banks, dirt transitions and the wide meadow do not".
- Water: a residual hatch pattern on the stream above the shaded lip; straight polygon edges at the brink seen from
  the side; the water goes over "as a thin glassy ribbon" with no visible thickness; the dissolve to white is soft;
  the sunlit sheet is still "a vertically stretched blotch-noise texture" of one blob size; the column keeps its
  width inside a feathered glow; nothing on the stream says it is moving; the fall does not wet or shadow the cliff.
- Soil and rock: rock "swirly marbled" with "melted folds" and one "reptile-skin" bump, no joints or ledges; mesh and
  stretch artefacts in the closest soil frame; soil still soft lumps rather than crumbly earth; roots "hoses or
  cables"; moss a tint; turf, soil and rock stack "like a layer cake".
- Grass: sparse upright blades on stream banks with ground showing; bare ground a smooth plane; the wide meadow reads
  as one olive carpet where coverage thins on slopes.
- Held up: the pond (colour, dusk reflection, broken bank reflections); the shaded fall, "the most convincing fall
  frame"; droplet streaks; the turf overhang with warm raking light; warm against blue on the rock masses; backlit
  translucency, height variation and depth in close grass, which "genuinely sit close to" the reference; rim-lit
  blades against the sky; the dry golden foreground that "reads as a real hay meadow at dusk"; long shadows.

## Where version 3 stands

Eleven critic rounds, one fresh critic each time, never two at once. No round ended with "no major issues" in any
area, so the stated definition of done is not met. What moved, going by which items the critics stopped raising and
which they listed under "holds up":

- Passed at least once and stayed passed: pond and still-water reflections from eye level, the shaded waterfall,
  droplet streaks, the turf overhang and soil layering at mid distance, island massing and lighting, close backlit
  grass, rim grass against the sky, the front-lit dry meadow.
- Removed for good: the crease and "cream plank" at the lip, blocky noise and mosaics on the sheet, the dark outline
  and ghost rocks, contour-line rock, drawn and dashed joints, sandpaper and mulch soil, stick and pipe roots with
  open ends, sprigs on the cliff face, paper-strip blades and striped blades, sticker flowers, row-planted tufts,
  the picket-fence band, three separate moire and dither patterns.
- Still failing every round: the falling sheet as a single ribbon (it needs real 3D ropes and thrown water, not a
  textured strip with particles); visible water thickness at the brink; structured rock (every joint system tried
  read as either lines or cells, and its absence reads as marble); crumbly soil relief (needs real displacement, the
  mesh is too coarse); a root hierarchy; moss with thickness; a ground layer of thatch and litter; denser grass on
  steep banks.

Those last items are geometry, not shading. Within one file and a 60 fps budget they would need: a particle-rope
waterfall, tessellated or parallax-displaced rim soil, a block-fracture pass on the island mesh, and a second grass
population keyed to slope.

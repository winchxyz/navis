# Reddit post

Live demo: https://winchxyz.github.io/navis/
Source: https://github.com/winchxyz/navis

Suggested subreddits, in order: **r/threejs**, **r/webgl**, **r/proceduralgeneration**,
**r/gamedev** (Screenshot Saturday), **r/InternetIsBeautiful**.
Attach `screenshots/nave.png` as the image; put the links in the body, not the title.

---

## Title options

1. `I built a flooded Gothic cathedral in three.js — one HTML file, zero assets. The rose window is a spotlight cookie.`
2. `Flooded cathedral in three.js: no models, no textures, no audio files. Everything is generated at load.`
3. `[three.js] Real-time flooded cathedral — marble, caustics, stained glass and the reverb are all procedural`

I'd lead with **1** on r/threejs and **2** on r/proceduralgeneration.

---

## Body

I wanted to see how far you can get in a browser with **nothing loaded from disk** —
no models, no textures, no HDRIs, no audio files. One HTML file, one CDN import of
three.js, and everything else generated at startup.

**Demo:** https://winchxyz.github.io/navis/ · **Source:** https://github.com/winchxyz/navis

Click once for sound. WASD to walk in, P for photo mode, H hides the panel.

A few things I learned building it:

**The rose window is a spotlight cookie, not geometry.** This was the decision that
made the whole scene work. Instead of modelling stained glass as a transmissive
material, I draw the window once to a 2048² canvas and hang it on `SpotLight.map`.
That single object gives you three separate systems for free: coloured patches on the
columns, a coloured source to tint the caustics, and a coloured source for the
volumetric shafts. Modelling it as glass would have given you one of the three and
cost more.

**Vertical light shafts are nearly invisible with a normal phase function.** My
volumetric raymarch uses Henyey-Greenstein with `g = 0.65`, which is right for the
rose — you're looking almost straight into those beams. But the daylight coming down
through the hole in the vault is a *vertical* shaft, and you basically always view
that one across its axis, where a forward-peaked phase gives you almost nothing —
about forty times less than the forward peak. It was completely invisible until I
gave that light its own, nearly isotropic phase (`g = 0.12`) and its own gain.
Surface lighting and in-scatter want very different numbers once the phase is flat.

**A drop is a rising pitch, not a falling one.** My first drip sound was a sine
sweeping downward, and it sounded exactly like a blaster from a 1980s arcade game.
What you actually hear when water lands is the bubble trapped by the impact, and as
that bubble shrinks its resonance goes **up**. Short broadband tick for the strike,
then a sine rising ~15% over 45 ms. Completely different sound, one sign flip.

**Water sound is not surf.** Same lesson from the other direction: a broad band of
filtered noise reads as a beach no matter what you do to it. Indoors you want a
narrow, low bed that's almost inaudible, plus sparse individual laps scheduled
against it — and mostly you're hearing the reverb tail. The reverb impulse response
is generated too: twenty discrete early reflections off the arcade, then a
four-second stone tail.

**Triplanar mapping for a building made of mixed geometry.** The columns are lathes,
the walls are extrusions, the ribs are tubes — three completely different UV
conventions. Any single texture scale looks stretched on two of them. Projecting the
marble from world coordinates instead makes the veining run continuously from the
floor, up a column and across a rib. The relief has to be projected the same way; I
sampled the normal map off the mesh UVs at first and at grazing angles the mismatch
between where the veins *were* and where the bumps *were* lit up as glowing cracks.

**The refraction is screen space, so everything submerged actually shows.** The scene
is drawn once without the water into a buffer that packs colour in RGB and linear
depth in alpha, then the water is drawn last on its own layer and samples that copy.
Packing both into one target means one blit and — more importantly — no feedback
loop, since the water writes into the scene target while reading a copy rather than
the depth attachment it's bound to. The distance light travelled through water falls
straight out of the depth difference.

Other bits: quadripartite vault webs are Coons patches over cubic-Bézier half-arches;
caustics are three layers of tileable Voronoi where the `F2 − F1` ridge term is what
gives you filaments instead of bubbles; the marble is domain-warped turbulence pushed
through a sine with integer coefficients so it tiles; there's a ninety-second
dusk→noon→dusk cycle on a sine so the loop seam never shows.

Panel has about seventy controls and six art-directed presets if you want to pull it
apart. Settings persist in localStorage.

Happy to answer anything.

---

## First comment (post it yourself, right after)

Technical detail I left out of the post, for anyone curious about the waterline:

The camera path deliberately crosses the surface, and for a while that looked
completely broken — a hard band would swallow half the frame. It turned out to be a
wave crest two centimetres high passing over the eye: at that grazing angle the
reflection UVs stretch enormously and the half-resolution reflection buffer smears
across the screen. The fix is to flatten the Gerstner displacement in a small disc
around the camera, and only while the eye is within ±0.35 m of the surface, where you
couldn't resolve the detail anyway.

Underwater, the surface seen from below is split at the critical angle: everything
above the water arrives through a cone about 48.7° wide (Snell's window) and outside
it the surface mirrors the flooded nave back at you.

---

## Notes before you post

- Post in the morning US Eastern on a weekday; r/threejs is small and slow, so
  timing matters less there than on r/webgl.
- Reddit strips markdown headers in some clients — the body above uses bold leads
  instead of headers on purpose.
- If someone asks about performance, the honest answer is that it was never measured
  on a specific GPU. Say what's true: four quality tiers, the panel has an FPS meter,
  and reflection resolution plus volumetric resolution are the two cheapest dials.
- Expect "why not path tracing" — the answer is that this deliberately holds a
  constant frame with no accumulation, so the first frame is the final frame.

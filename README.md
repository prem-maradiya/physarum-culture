# Physarum Culture

A live GPU simulation of up to two million slime-mould agents growing real transport networks, in a single HTML file with no build step and no dependencies.

**[Open the live culture →](https://prem-maradiya.github.io/physarum-culture/)**

*Physarum polycephalum* is one cell with no brain and no neurons, yet it finds the shortest path through a maze and — fed oat flakes laid out as Japanese cities — grows a network close to the Tokyo rail map. Every vein on screen is emergent. Nothing is drawn.

## Controls

**Drag on the field to place food** and watch the colony rewire toward it.

| Key | Action |
| --- | --- |
| `Space` | Pause / resume |
| `R` | Re-inoculate the dish |
| `H` | Hide the panel for a clean full-screen view |

Six behaviour presets (Veins is the canonical model; Nebula blooms outward; Dendrite goes crystalline), five stains, and four inoculation patterns. Every parameter of the agent model is exposed as a slider.

## The model

Each agent follows the rule from Jeff Jones' 2010 paper: sample the chemical trail at three points ahead, turn toward the strongest, step forward, deposit a little trail of your own. The trail diffuses and evaporates. That is the entire program — run it a million times a frame and transport networks fall out.

## How it runs fast

- **One draw call does two jobs.** A single `POINTS` pass advances every agent *and* rasterises its deposit: the new state is captured by transform feedback into a ping-ponged VBO, while the same vertex emits `gl_Position` so the deposit blends into the trail texture. The CPU never touches an agent after seeding.
- **Agents sense the pre-blur field and deposit into the post-blur one.** Sampling a texture you are currently rendering into is undefined behaviour; splitting the read and write across the ping-pong pair avoids it, at the cost of a one-frame-stale gradient that is invisible in motion.
- **Four half-texel-offset samples ride the bilinear unit** to produce a weighted 3×3 diffusion kernel for the price of four fetches instead of nine. The first version was fill-rate bound; this roughly halved the per-pixel cost.
- **Deposit is normalised** against population and persistence, so the *Trail density* slider means the same brightness whether 50,000 agents are alive or 2,000,000. The display shader exposes against that same known mean, so no combination of settings blows the image out.
- **Half-float trail** (`RGBA16F`) where the card can render to it, so faint pheromone survives instead of quantising to zero. Falls back to `RGBA8`.

Measured on an integrated AMD Radeon (Ryzen APU): 47 fps at 400,000 agents, against a 143 fps display-only baseline. Discrete cards handle the two-million ceiling comfortably.

## Requirements

WebGL2. Transform feedback is core WebGL2, so no extensions are strictly required; `EXT_color_buffer_float` is used when present for the higher-precision trail. The page detects a missing context and says so rather than showing a black screen.

## Running it locally

It is one static file with everything inlined.

```bash
git clone https://github.com/prem-maradiya/physarum-culture.git
```

Then open `index.html` in a browser. No server, no install, no build.

## Credits

- Jeff Jones (2010), *Characteristics of pattern formation and evolution in approximations of Physarum transport networks* — the three-sensor agent model.
- Tero et al. (2010), *Rules for biologically inspired adaptive network design* — the Tokyo rail experiment.

## License

MIT — see [LICENSE](LICENSE).

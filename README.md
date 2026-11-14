# Image to Three.js miner (claude_r5 H2c)

Entry point: `main.py` -> `async generate(image, seed, witness) -> str | None` (the competition witness runtime).
The active plan is `ACTIVE_PLAN` in `miner404/versions.py`.

## Base and changes

Base: [SomPeer/copperbellows@d481cfc](https://github.com/SomPeer/copperbellows/tree/d481cfc14057e25d5618a76098ecc54343639f30)
(its code is identical to the round-4 winner TiltGilt/VertexLoom@2385ec1; only the plan differs).

Plan relative to the round-4 winner's:

- same 13 gated rewrite steps, fallbacks, `judge_pick` gates, `replan`, `judged_tilt` and `outline_fit_last`;
- SEARCH/REPLACE patch edits on rewrite steps 8-13 (as the round-4 winner), full-module rewrites on steps 1-7;
- 6 Sol-high drafts as 2 plain + 2 proportions + 2 inventory (parts, counts, colours listed before building);
- no draft merge, no tone pass;
- two deterministic finishing steps after the judged tilt, each kept only if the renderer accepts it (ideas from
  duongnd18/abc-3d-champ round5 @ a4bf284, re-written): `texfix` gives linear + mipmap sampling to DataTextures the model
  left at NearestFilter, and `stretch` scales each shorter bounding-box axis by up to x1.25 (never past 0.999 of the cube;
  round bodies keep a round cross-section; modules naming a spout are left as they are);
- `routed=True`: one Sol-low call (request 9995) labels the photo; upright furniture/structures skip both the photo
  tilt and the judged tilt, tall photos skip the photo tilt (idea from duongnd18/abc-3d-champ@04d5fdc, re-written in
  `miner404/route.py`). A refused or failed label call leaves the base behaviour.

Code changes: `miner404/route.py` (new), `miner404/tilt.py` (`photo_is_tall`), `miner404/pipeline.py` (router call,
both tilt gates, and HTTP 504 `render_timeout` from the current renderer counted as a render timeout so a too-heavy
module is sent back to be made lighter; `stretch` / `texfix` finishing), `miner404/versions.py` (plan). Everything else is the base verbatim.

## Local results vs the round-4 winner (VertexLoom)

Local S1 replica judge (z-ai/glm-5.3-flash, upstream stage-1 prompts). Margin = mean over 5 judge seeds of
(wins - losses) / prompts; generation on a renderer built from the subnet Pod's renderer-runtime stage (30 s preview limit).

| Set | H2c margin | H2b | H1.1 |
|---|---|---|---|
| 16 dev prompts (round 4) | +20.0% | +13.8% | +1.2% |
| 16 random holdout prompts (round 4) | +17.5% | +11.2% | +21.2% |
| All 32 | +18.8% | +12.5% | +11.2% |

Budget per prompt in those runs: at most 54 calls and 4.31M credits (limits 60 / 5M); the finishing steps add no model calls.

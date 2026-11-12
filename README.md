
# Charlie-s-Agents
Group Project
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PYTHONUNBUFFERED=1 PORT=8501
EXPOSE 8501
HEALTHCHECK CMD python -c "import urllib.request;urllib.request.urlopen('http://127.0.0.1:8501/_stcore/health')"
CMD streamlit run charlies_agents/app.py --server.port=${PORT} --server.address=0.0.0.0 --server.headless=true

# Charlie's Agents: agentic GEOINT tabletop exercise

**UNCLASSIFIED // FICTIONAL TRAINING CONTENT.** This is a training simulation for INTA 689. It cannot task real satellites, aircraft or sensors, and it holds no real operational data.

This is a multi-agent web application. A **deterministic rules engine** owns every fact and ruling. Three AI roles work around it, each in a separate context and each with **its own toolbox**. They make real LLM calls with real function calling:

| Role | Real tools it can call | What it can never do |
|---|---|---|
| **Blue** advisor | `get_situation`, `coverage_check`, `list_options`, `pass_geometry`, `get_evidence`, `comms_status`, `review_plan` / `review_report` → `submit_challenge`; on request `check_plan` → `submit_plan` | Task anything; raise a concern the engine did not find, or drop one it did; tell the trainee which decision to take; see the engine's rating of the evidence (its tools do not expose it) |
| **Red** adversary | `get_observations`, `get_menu`, `check_action` → `submit_actions` | See Blue's plan or evidence. Its toolbox is built **only from the filtered observation snapshot** and holds no engine handle, so the firewall is structural |
| **White Cell** | `get_adjudication`, `get_injects`, `get_rules` → `submit_narration` | Change a ruling or omit a failure. Its text is model-written, so the engine's own rulings are always printed beneath it and are the authority |

Each agent runs a bounded loop (at most 8 steps): the model calls tools, the engine executes them and returns the results, and the model must finish with a `submit_*` tool whose arguments the engine validates. A rejected submission goes back to the model as a tool error and it tries again. Every model step and tool call (name, arguments, result digest, latency) is written to the hash-chained audit log and shown in the **Agent tool trace** tab.

**The trainee decides first.** The trainee commits to their own collection plan and their own draft report. Blue then *challenges* them with questions built from the engine's list of weaknesses. Blue does not hand over an answer, and the engine's assessment of the evidence is hidden until the after-action review.

---

## 1. Use the deployed app (nothing to install)

1. Open the URL your instructor or team lead gives you in any modern browser and enter the class passcode.
2. On the landing page, type **your name** and press **Start new exercise**. The exercise options (seed, orbits, model) are folded away; the defaults are fine. The rules are not a page you must read first: they sit in the **Rules & orbits** tab for reference throughout the exercise.
3. **Briefing.** An adjutant opens the channel and the J2, the collection manager, the J6 and Blue brief you, each in their own voice (recorded neural text-to-speech clips; the text types out in step with the voice; the **Voice** button turns it off). Press Space for the next line, or skip it. The exercise starts when you press **Take the watch**.
4. The exercise runs for **four turns** of six hours each. You start with three requirements. The White Cell injects one or two more, unannounced and never more than one at a time; which ones and when differs from one exercise to the next. The **Exercise** tab is laid out as steps along its top: **Requirements**, **1 · Plan**, **2 · Blue & collection gate**, **3 · What happened this turn** and **4 · Reports**. Each opens on its own, and the tab moves you to the next step by itself (planning opens on the plan; after you commit, the gate; after you approve, what happened):
   1. **Requirements and Plan.** The **Requirements** step is the **requirements board**: one tile per requirement with what is being looked for, where, its priority, a countdown to its deadline, what collection is available or tasked, the evidence held and any report filed. Click a tile to open its detail (the full question, the options, each piece of evidence with its finding, the standing report); the map then dims the other requirements' options and the option list puts that requirement's first. Click it again to clear. On the map, every collection option is an icon: satellites, Reapers, the Triton, the Global Hawk and the commercial archive image. Click an icon to add it to your plan or take it out; hover for its risk, timing, its **image quality** (on its own line, against the quality the requirement needs) and whether it can identify what is there; once an asset is in your plan for one requirement, its icons for the others are greyed out, because one aircraft or pass cannot serve two places at once; drag the time bar to see where the satellites and their coverage footprints are. You may task every asset that is ready, but each one then stands down, so what you use now you do not have next turn. If a base has lost an aircraft, a **Request to move an aircraft** box appears. Write your reasoning and press **Commit my plan and hear Blue's challenge**.
   2. **Decide (the collection gate).** Read Blue's questions. You may ask to see Blue's own alternative plan. Then task your plan, task Blue's, task nothing, or choose **Revise my plan**: that sends you back to the map with your plan as it was, and when you commit the new version Blue questions it again (and can draft a fresh alternative to it). You can go round as often as you like; nothing is tasked until you press **Approve tasking**. Your first version is scored as your own plan. Red's move, the engine's rulings and the narration then run without further clicks.
   3. **Watch and report.** The **What happened this turn** step opens with the results, the narration and the comms picture. The **Operations map** tab plays an animated replay of the turn (at ½×, 1×, 2× or 4× speed) (satellites crossing with their footprints and imaging, aircraft flying out from their bases, anything turned back, damaged or shot down, jamming and other interference, channels going down, and your reports travelling to the consumer or failing to). On the **4 · Reports** step, draft a report for each requirement that is due: say **what the evidence shows** (confirmed, consistent but not identified, not a threat, or something else that is a threat), your confidence and the channel. Press **Submit draft for Blue's challenge**, revise if you wish, then file it. Press **End turn**.
5. When the exercise is complete, open **Outbrief**: a score out of 100, how each decision was assessed, alternative courses of action, and a replay of every turn back to back.

Your session is saved after every step. The address bar holds your session code (`?s=…`): a refresh brings you back to where you were, and **Resume a saved exercise** on the landing page offers a list of every saved exercise (name, where it stopped, when it was saved, its code), or takes a code typed in.

Instructors: switch on **Instructor view** in the sidebar to see Red's provenance and tool calls, pause, override or rule on contested results, read the outbrief mid-game, and compare all saved sessions (with scores) in one table. Keep it off on trainee screens; it is not yet protected by its own passcode.

### What each choice costs (the published rules, in the *Rules & orbits* tab)

| Choice | Rule |
|---|---|
| Tasking | No limit: you may task every asset that is ready. Each asset then **stands down**: a Reaper for 1 turn (2 from the austere strip at Osprey Point), the Triton and the Global Hawk for 2 turns, a national satellite for 1 turn |
| Image quality | Each requirement states the quality it needs to say **what** is there, on four tiers that follow the public imagery rating scale (NIIRS): low (coarser than 2.5 m), medium (1 to 2.5 m), high (0.4 to 1 m), very high (finer than 0.4 m). Medium tells large ship types apart; small craft, aircraft on an apron and equipment on a pad need high. Only optical imagery at or above that tier identifies. Radar shows size, shape, number and movement, never what an object is; the exception is a question about whether something large has moved, which radar can settle |
| Sensor quality | Reaper video turret very high; national and commercial optical satellites very high overhead, one tier worse at a low look angle; radar satellites and the Global Hawk radar imagery only; the Triton radar tracks only; the commercial archive low (about 3 m) |
| MQ-9A Reaper (three, at two bases) | Sharp optical, but must orbit inside the defences: full engagement risk (25 to 35% in the defended sectors). Reaches only the sectors within range of its base |
| MQ-4C Triton | Stand-off surface-search radar: tracks ships through cloud at lower risk (about 10%), cannot identify them |
| RQ-4B Global Hawk | Stand-off imaging radar: sees through cloud at lower risk, cannot identify |
| An aircraft that is hit | Either **shot down** (lost for the rest of the exercise) or **damaged** (it returns and is in repair for 2 turns) |
| Moving aircraft | When a base has lost an aircraft (shot down or in repair), you may request that a Reaper from another base move there. The move takes the turn; one move per turn |
| A satellite Red interrupts | Fails this turn and stays offline for the following turn |
| Commercial satellite, new order | Always available and never stands down, but the imagery reaches you **two turns** after the pass, so it is stale for most requirements, and the vendor may bump the order (20%) |
| Commercial archive image | Arrives this turn, but it was taken last turn, is low quality (about 3 m, cannot identify), counts as a low-confidence source and may be cloud-covered. One per turn |
| Optical collection | Every option shows its chance of being clouded out, however small |
| Reporting early | Allowed in any turn up to the deadline; the newest report stands, but every version is scored for calibration. The courier arrives a turn after it is sent, so it is only on time when sent early |
| Withhold | Final. Closes the requirement with no answer. Only at the deadline |
| Escalate | Spends one of 2 escalations. Moves the deadline one turn; next turn, assets standing down for routine turnaround may be tasked against that requirement. It is then answered late. Escalating in turn 4 opens one extension turn |

### Requirements that are not what they seem

A requirement's question is the J2's best guess. In **every** exercise at least one requirement (one or two, chosen by the seed) is not what its question assumes: either it is no threat at all, or OPFOR is doing something else that is still a threat. Nothing on screen marks which one. You find out only when a collection that can identify what is there succeeds and you read its finding; your report then has to say what is there, not what was expected. The outbrief's **What was really there** section reveals every requirement's ground truth, whether your collection established it, and whether your report got it right.

### Communications

Each channel now has a bearer and a set of ways it can fail. When Red or a scheduled event takes a channel down, the engine picks the cause with a seeded roll and the app reports what happened, what it stops and when it is expected back (for example: the undersea fibre trunk severed by a dredger, uplink jamming keyed on traffic volume, a network intrusion that forced J6 to isolate the enclave, a trawler jamming the relay's ground terminal, navigation spoofing that knocks the protected waveform out of sync, a courier flight held by a live-fire closure). The text lives in `comms_channels` in the scenario file.

### How the outbrief scores you (100 points)

| Component | Points | What it measures |
|---|---|---|
| Reporting defensibility | 30 | Share of requirements whose standing report or withholding the evidence and channels supported |
| Requirements answered | 20 | A usable answer on time; priority-1 requirements count double; partial and late count for less |
| Collection planning | 20 | Half: turns tasked without an avoidable weakness that was visible at the time. Half: taskings that returned usable evidence |
| Confidence calibration | 15 | Share of filed report versions whose confidence, citations and gaps matched the evidence |
| Independent judgment | 10 | How your own first drafts would have scored before Blue challenged them |
| Asset stewardship | 5 | Aircraft brought home; escalations spent only where justified |

85+ is highly defensible, 70+ defensible, 50+ partly defensible. The scores, the defensibility rulings and the alternative courses of action are computed by the engine from the audit log. The White Cell model only words a short spoken summary, and the engine rejects that summary if it misstates the score or band.

## 2. Run or deploy it yourself

You need Python 3.10+ and an API key for any OpenAI-compatible endpoint (OpenAI, OpenRouter, Anthropic's OpenAI-compatible API, Nous; hosts are allow-listed in `guardrails.py`).

```bash
unzip charlies_agents_app_v7.zip && cd charlies_agents
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env          # then edit .env: CA_API_KEY, CA_BASE_URL, CA_MODEL, APP_PASSCODE
deploy/start.sh               # → http://localhost:8501  (set PORT in .env to change it)
deploy/start.sh --public      # → also prints a public https://<random>.trycloudflare.com URL
nohup deploy/supervise.sh > runs/supervise.log 2>&1 &   # optional: auto-restart app + tunnel; URL in runs/public_url.txt
```

`--public` needs the free [`cloudflared`](https://github.com/cloudflare/cloudflared/releases) binary on your PATH, or set `CLOUDFLARED=/path/to/cloudflared` in `.env`. No Cloudflare account is required. The URL changes each time the tunnel restarts.

**Docker** (for Render, Fly.io, Railway, Cloud Run, or any VM):

```bash
docker build -t charlies-agents .
docker run -p 8501:8501 --env-file .env charlies-agents
```

**Streamlit Community Cloud:** push this folder to GitHub, create an app with main file `charlies_agents/app.py`, and paste the `.env` values into *Secrets* as `KEY = "value"` lines.

### Configuration (`.env`)

| Variable | Purpose |
|---|---|
| `CA_API_KEY` | API key for the model endpoint |
| `CA_API_KEY_FILE` | Alternative: a file holding the key (re-read on every call, so rotating keys keep working) |
| `CA_BASE_URL` | e.g. `https://api.openai.com/v1`, `https://openrouter.ai/api/v1` |
| `CA_MODEL` | Must support function calling, e.g. `gpt-4o-mini`, `anthropic/claude-sonnet-4.5` |
| `APP_PASSCODE` | Passcode gate for a public URL (strongly recommended) |
| `CA_MAX_CALLS_PER_SESSION` | Cap on LLM calls per exercise (kept across a refresh or resume); after the cap, labelled scripted fallbacks take over |
| `PORT` | Port for the app (default 8501). Each copy only manages the instance on its own port |

Without a key the app still runs end to end on **labelled** scripted fallbacks, so the engine stays testable without an API key (PRD 10.3).

### Verify

```bash
.venv/bin/python -m pytest -q                       # 100 tests, no key needed
.venv/bin/python -m charlies_agents.evaluate --llm  # ten-vignette battery against the live model
.venv/bin/python -m charlies_agents demo --llm --orbits cached   # terminal walkthrough (scripted stand-in trainee)
.venv/bin/python -m charlies_agents play --name "Your Name"      # play in the terminal; --resume CODE continues a session
```

### Cost and speed

A full five-turn game played through the browser with live agents (Claude Sonnet 4.5) took about 13 minutes and made 67 model steps and 103 tool calls. Each Blue challenge took 8 to 14 seconds; Red's move, adjudication and narration together took about 20 seconds per turn. `CA_MAX_CALLS_PER_SESSION` caps the spend per session.

## How a turn runs

Before turn 1 the app plays the briefing (`web/briefing.js`, text in the scenario file's `briefing` block); it is logged as a `BRIEFING` record and can be replayed from the sidebar.

Engine phases: `PIR_ISSUE -> COVERAGE -> PLANNING -> APPROVAL -> RED_MOVE -> ADJUDICATION -> PRODUCT -> next turn`. The engine refuses any action taken out of phase. The steps with no decision in them (opening the turn, the coverage check, Red's move, adjudication, narration) advance by themselves; with **Hold after Red's move** switched on, the instructor can stop before adjudication to strike a Red action.

Both human gates fail closed. If the trainee has not approved a plan, nothing is tasked. If a requirement that is due has no decision, the turn cannot end.

## Map from PRD requirement to code

| PRD requirement | Where it lives | Proven by |
|---|---|---|
| Story 1, coverage check from the engine, not the model | `engine.coverage_check` / `options_for` | `test_denied_airborne_infeasible...` |
| Story 2, options cite real catalog entries and name the binding constraint | `engine.validate_plan`, `tools.BlueTools.submit_plan` | `test_invented_option_rejected`, `test_loop_executes_tools...` |
| Story 3 / MH1, approval gate with rationale | `engine.approve_plan`, `record_product` | `test_collection_gate...`, `test_product_gate...`, `test_plan_provenance_logged` |
| Story 4 / Decision 4, four end states; justified withholding scores as defensible | `scoring.score_report`, `score_pir` | `test_justified_withholding...`, `test_escalate_and_withhold_score_differently` |
| Sec. 8.2, automation bias: the trainee reasons first, Blue challenges | `session.challenge_plan` / `challenge_report`, `agents.validate_challenge` | `test_blue_alternative_is_locked_until_trainee_commits`, `test_live_blue_challenge...must_ask_not_tell` |
| Sec. 8.2, score judgment, not rubric-copying | engine rating withheld from the page and from Blue's tools until the AAR | `test_blue_tools_do_not_expose_the_engines_rating`, `test_full_exercise_through_ui` |
| Story 5 / MH3, Red firewall | `engine.red_snapshot`, `validate_red_action`; separate `AgentSession`s | `test_red_snapshot_invariant_to_hidden_blue_plan`, `test_red_cannot_cite...`, V10 |
| Story 6, AAR with causal attribution, score and alternative courses of action | `scoring.build_aar`, `final_assessment`, `collection_alternatives`, `reporting_alternatives` | `test_final_assessment_scores_and_suggests_alternatives`, `test_debrief_text_must_match_the_engines_score` |
| Story 7, what-if replay against the recorded Red posture | `engine.replay_turn` | `test_whatif_replay...` |
| Story 8, provenance, pause, override | `engine.instructor_*`, CLI `:provenance` | CLI |
| MH2, deterministic adjudication (seeded hash rolls) | `engine._roll`, `adjudicate` | `test_identical_seed_identical_outcomes`, battery |
| MH4, full audit trail, hash-chained JSONL | `audit.AuditLog` | `test_audit_log_tamper_detected` |
| MH5, honest degradation | `agents.AgentSession.ask` + fallbacks | `test_model_outage_uses_labelled_fallback...` |
| MH7, unclassified-data discipline | `guardrails.py` | `test_guardrails_refuse_operational_content` |
| Sec. 6.3, ten vignettes plus a manual baseline | `vignettes.py`, `evaluate.py` | `test_ten_vignette_battery_passes` |

## Honest limits

* The rubric's "defensible" sets were written by the builder, before any model ran. The PRD (10.5) asks for independent review of the ten expected outcomes, and the team should do that review.
* **The scoring weights and bands are the builder's judgment**, not a validated instrument. They are in `scoring.WEIGHTS` and `BANDS` and should be reviewed by the team and tuned after real trainee runs.
* **Collection alternatives are hindsight.** Each turn is re-adjudicated on its own against Red's recorded moves and the weather and engagement rolls as they fell. The outbrief separates these from the weaknesses that were knowable when the trainee tasked.
* The "baseline" arm of `evaluate.py` is a naive scripted player. It is **not** a human manual baseline. Real trainee runs are still required.
* A stand-in trainee (actor `trainee(auto)`) drives `demo` and the battery. Its results show that the gates hold and that the challenge carries enough information to fix a bad draft. They do not show that people exercise judgment.
* The game-balance numbers (stand-down lengths, 3 turns of repair, the share of hits that destroy an aircraft, 2 escalations, 25% base engagement risk and the per-sector and per-aircraft multipliers, the commercial delays) are first guesses. They sit in `data/scenario_scs.json`. In 40 scripted games per policy, tasking every ready asset every turn averaged 76 of 100 and lost 1.3 aircraft a game; one identifying collector per requirement averaged 82 and lost 0.55; commercial imagery alone averaged 62; tasking nothing 50. Those are scripted stand-ins, not people.
* **Aircraft are real types, simplified.** The MQ-9A, MQ-4C and RQ-4B Block 40 are uncrewed aircraft the United States operates in the Indo-Pacific. Their numbers, bases, sensors, speeds, turnaround times and survivability here are simplified for the exercise and come from no operational source. The bases and all places are invented.
* **The injected requirements are drawn from a pool of five** written into the scenario file, and the hidden variants are written there too. The seed picks which arrive, when, and which are not as assumed; the White Cell model only words the inject, from the question alone, and never sees the truth. They are not invented by the model during play.
* For one pooled question (whether large shipping has left harbour) radar alone can establish the truth. For every other requirement only sharp optical imagery can.
* The map replay is an illustration of the engine's rulings. Sortie routes are straight lines at a nominal cruise speed, and the moment of a hit or shoot-down within the turn is chosen for the animation; only the result comes from the engine. Satellite footprints are the area the satellite could image at that moment, computed from its altitude and minimum look angle; the imaged swath has a true width and a nominal length. The communications links are schematic: the courier line is bowed and some nodes are nudged apart so they can be read, and the time a report takes to travel is cosmetic. With the scripted pass schedule (orbits off) the satellite tracks are schematic. The Hawaii inset is not to the main map's scale, the three links are spread along its frame so they can be told apart, and the cable's landing point is nudged clear of the Guam base icon. Take-off times of the aircraft from Guam (hours before the turn opens) are set for the animation; the engine does not model transit time, only whether a sortie collects, is turned back, is damaged or is lost.
* Some map effects have been checked only against hand-made test data, not seen in a recorded game: laser dazzle, satellite and downlink jamming, a bumped commercial pass, radio jamming of a channel, a courier closure, and a target revealed as harmless.
* On the 3D globe at default zoom, sector names shorten to their letter and the suspected-target icons and parked aircraft are hidden until you zoom in. Touch input on the map is untested.
* Blue's questions are bounded by the engine's list of weaknesses. A weakness the engine has no rule for is one Blue will not raise.
* The trainee's written rationale is recorded but not assessed, and there is no decision clock.
* The rubric penalises over-confidence and avoidable silence. It does not penalise under-confidence: a report at low confidence on strong evidence scores as defensible.
* The White Cell's narration is checked for omitted failures, overstated losses and blaming a comms outage for an imagery delay, not for every embellishment. The engine's own rulings are always printed beneath it.
* One browser tab per session. Two tabs playing the same session code at once will fork the saved log.
* The instructor view is a plain toggle. Anyone who finds it can see Red's moves and the outbrief mid-game.
* Exercises saved by an earlier version still open. One that was in progress keeps the rules it was started under (tasking slots, a national allowance, the P-8) and can be played to the end. One saved before the turn record existed shows an outbrief but has no animated replay or what-if. Earlier saves have no hidden ground truth, so their outbrief has no *What was really there* section.
* Each satellite follows the **orbit** of a real public satellite as geometry only. That says nothing about the real satellite's tasking, availability or performance, and resolution classes are notional.

## Recorded results (in `runs/`)

| Run | File | Result |
|---|---|---|
| Automated tests | `tests/` | 100 pass, no key needed |
| Ten-vignette battery, live tool-using agents | `v7_battery_llm.txt`, `.json` | 10 of 10 pass all checks; 148 tool calls; no fallbacks; 1 submission bounced by the engine (Blue's alternative plan put a priority-2 requirement ahead of a priority-1 one); 0 Red violations; the naive baseline is defensible in 1 of 10 |
| Scripted demo, seed 7 | `v7_demo_seed7.txt` | Five turns; stand-in trainee scores 82 of 100; one requirement was not what the question assumed, and it was established and reported correctly |
| Scripted demo with orbits | `v7_demo_orbits_seed7.txt`, `v7_replay_orbits_seed7.html` | Five turns; 82 of 100; the HTML file is the animated replay and opens offline |
| Browser game, live agents | `v7_browser_game_live.txt` | Landing page, briefing, then five turns plus one escalation turn, planned by clicking map icons (four taskings in turn 1). The requirements board drew on every turn, and clicking a tile focused the map on that requirement. A mid-turn refresh lost nothing. Two injected requirements worded by the model, both a different threat from the one asked about: one was identified and reported correctly, the other was reported as consistent without an identifying collection, and the outbrief said so. One escalation, a Reaper shot down, the primary network cut and the alternate jammed. 67 model steps, 98 tool calls; 78 of 100; audit chain intact; no rubric terms visible during play |

## Nice-to-have features built (PRD 4.2 backlog a, b, c)

### (a) Live satellite orbits: `orbits.py`
* `--orbits live` fetches current TLEs from CelesTrak. `--orbits cached` uses the bundled snapshot in `data/tle_cache.json`; refresh it with `python -m charlies_agents.orbits`. `--orbits off` keeps the precomputed fictional windows.
* Skyfield propagates each orbit across the six-hour turns. A window exists when the satellite rises above 30° (EO) or 25° (SAR) elevation over a sector. EO also needs the sun above 10° at the target. Resolution drops one class below 55° elevation.
* Proxies: NEO-1 uses WORLDVIEW-3, NSAR-1 uses SENTINEL-1C, CEO-2 uses PLEIADES NEO 3, CSAR-3 uses ICEYE-X2.
* The four sectors sit on deliberately open-ocean points, rounded to one decimal, so none falls on a real feature.
* Guardrails from PRD 5.5 and 6.1:
  * Provenance (NORAD number, TLE epoch, hash, source URL, fetch time) is recorded and written into the audit log at session start.
  * A TLE older than 14 days is flagged **stale** and that asset falls back to its precomputed windows, with the fallback labelled. A TLE older than 3 days is flagged "aging".
  * If the network fails, the cached TLE is used and the failure is reported.
* Determinism still holds: windows are a pure function of the TLE snapshot and the exercise start, and both are logged. Tested in `tests/test_orbits.py`.
* Example: `python -m charlies_agents demo --orbits cached --viz runs/coverage_3d.html`

### (b) Streamlit interface: `app.py`
`streamlit run charlies_agents/app.py`. It uses the same Session, gates and audit log as the CLI. Tabs: Exercise, Operations map, Agent tool trace, Outbrief, Audit log, and Rules & orbits. `tests/test_ui.py` plays full exercises through the UI headlessly: the briefing gate, four turns end to end, the step tabs, the revise-and-rechallenge loop at the gate, the saved-exercise list, a refresh mid-turn, early reporting and escalation, map clicks, the comms explanation, and an old save.

### (c) Operations map with 3D globe and animated replay: `mapdata.py`, `viz.py`, `web/tacmap.js`
One workspace replaces the earlier separate 3D and regional-map tabs. It is a plain canvas view with no libraries and no network requests.
* **Planning:** every option is a clickable icon grouped by requirement; selected options draw their route or pass; a time bar moves the satellites along their tracks. The **Theatre / Globe** toggle switches between the flat map and a rotatable globe with orbits at true relative altitude.
* **Coverage footprints:** each satellite carries a circle showing the ground it can image at that moment, moving with it in planning and in the replay; the strip it actually images is drawn when it collects. Toggle with the **Footprints** button.
* **Geography:** the operating area is the western third of the map. **Anvil Main** is drawn at its true position on Guam, near the eastern edge, with open ocean between; the Triton and the Global Hawk fly the five-to-six-hour leg from there, so they are launched before a turn opens and are already inbound when it starts. The **theater intelligence center** is on Oahu, about 8,400 km east of the operating area, so it is drawn in a small Hawaii inset with its own scale.
* **Communications:** the three reporting channels are drawn as links from the forward cell to the theater intelligence center in the Hawaii inset (the cable landing near Guam on the way), each crossing into the inset with a break mark and its distance. An outage shows on the link where and when it happens (a cable cut, jamming, spoofing, a network intrusion, a courier closure), and each report is shown travelling along its channel and arriving, arriving late, or failing. Toggle with the **Comms** button.
* **Replay:** after adjudication the map plays the turn: satellites transit and image, sorties fly from their bases, orbit, collect and return, or are turned back at a denied sector, damaged (they limp home trailing smoke), or shot down; re-basing flights are shown; interference with a satellite is drawn on it; and a suspected target changes its marker when a collection identifies it. Controls: play, pause, speed (½×, 1×, 2×, 4×), scrub, and a clickable event feed. Red's own moves stay hidden from the trainee until the exercise ends.
* **Outbrief:** all turns back to back, with a **Download replay** button that writes one self-contained HTML file.
* `mapdata.py` builds everything the view draws from the engine's state and its per-turn record. The view makes no rulings.

### (e) Requirements board: `boarddata.py`, `web/pirboard.js`

The **Requirements** step of the Exercise tab is a dashboard of the requirements instead of a table: a header strip with the turn, the clock and counters (open, due now, reported, closed, assets ready, escalations left, channels down), then one tile per requirement. A tile shows a drawn icon for what is being looked for (it changes when a collection identifies it), the sector, the priority, a segmented countdown to the deadline, the collection state, badges for evidence held and in the pipeline, and the next action. Tiles are buttons: click, or use Tab, the arrow keys and Enter. The selected requirement opens a detail panel and becomes the focus of the map and the option list. Like the map, the board is plain JavaScript with no libraries and no network requests, and it is given only what the trainee is already allowed to see: the engine's rating of the evidence and the hidden ground truth are not in its data.

### (f) Landing page and logo: `web/logo.svg`, `web/logo_mark.svg`

The first screen is the logo, two sentences on what the exercise is and who the three AI agents are, a name box and **Start**. The logo is original artwork drawn as SVG (three trench-coated agents with terminal-prompt faces, in the Blue, Red and White Cell colours, inside a satellite orbit ring) and is embedded in the page, so nothing is fetched from outside.

### (d) Briefing: `web/briefing.js`
Pixel-art portraits drawn in code (no image files), typed-out lines, and an inset map that highlights the sectors being discussed. The script is in the scenario file. Every line has a voice clip in `web/audio/` (MP3, 11 clips, about 3 minutes in all, made with Microsoft Edge neural voices through `edge-tts`; one voice per speaker; `audio/manifest.json` records the voice, rate and text of each clip). The typing keeps pace with the clip. The clips were generated in advance, so the app makes no speech requests at run time; browsers block sound until the page has been clicked, so if the first line is silent, the first press of Next plays it. If the briefing text is changed, the clips must be regenerated.

### What real geometry changed
With true orbits, one national EO pass sweeps several sectors within minutes, so a single interruption can fail several collections at once. The scripted schedule spreads the passes out and hides that single point of failure. This is a good discussion point for the class.

streamlit>=1.40
plotly>=5.20
skyfield>=1.48
numpy
pandas
pytest

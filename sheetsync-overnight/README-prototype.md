# SheetSync Overnight: prototype

Two pages live here. Northline Freight is fictional and every email, load and number on both pages is synthetic.

| Page | What it is | Link |
|---|---|---|
| `index.html` | **The mockup, the main prototype.** It opens on the pipeline view: six synthetic emails on one fixed route, three that work and three that stop. A second tab, Whole night, plays seven emails from the two test nights. Nothing is connected and it is not scored. | https://claude.ai/artifact/4iQwwbqRDcKKa2SHYULVV1 |
| `under-the-hood.html` | **The scored version.** All 30 synthetic emails against the answer key; the 96.7% right calls (right decision, row and value) come from here. | https://claude.ai/artifact/9wZL8BQF5P875W4i3TdFRB |

GitHub Pages copy: https://ranvir01.github.io/Vibe/sheetsync-overnight/

## The mockup (`index.html`)

Built from `mockup.src.html` by `node _build/build-mockup.mjs` (which also writes `_build/out/mockup.artifact.html`). One self-contained file, no external JavaScript, runs from `file://` and as a Claude Artifact.

**Method.** One of the four methods in the course matrix: **LLM-Based Automated Workflow** (fixed steps, one AI step, not an agent). A strip at the top lights this one and greys out Rules-Based (If-Then), Machine Learning (Predict Y using X) and LLM-Based Autonomous Agent.

**Pipeline view (opens first).** Seven stations, left to right, with the tag each would use:

1. **Gmail** (connector, read-only): flags new mail and attachments.
2. **Drive** (connector): saves the attachment to one folder.
3. **PDF reader** (code): text from the PDF; OCR for scans, with a quality flag.
4. **Claude** (AI step): one decision (new row / fix a cell / nothing to do / ask a person) plus fixed fields.
5. **Check** (code): every value copied from the source must be in the email or PDF text, and the load ID must be a valid sheet ID.
6. **Sheets** (MCP connector): a new row or one cell edit, with the Drive link, then a read-back. MCP is a standard plug that lets the routine edit the sheet.
7. **Digest** (7:00).

Six scenario chips and a **Play all** button. A token moves station to station and each station shows its result in plain words. A stop scenario puts a red stop marker on Claude, where the ask is decided; the check still shows its own result after it, in amber ("Code agrees"), as the second line of defence; Sheets and Digest show "Not reached."; the page then shows "Asks a person: <reason>" and moves the item into **Waiting for you**.

| # | Email | Result on screen |
|---|---|---|
| s1 works | M015 rate confirmation PDF | new row NL-1074 with a Drive link; the check finds the copied values in the PDF |
| s2 works | M005 delivered, POD attached | the reader shows "Proof of delivery: no text needed."; fix a cell: In Transit becomes Delivered |
| s3 works | M003 newsletter | nothing to do, no write |
| s4 stops | M028 scanned rate confirmation | the reader flags a low-quality scan; Claude asks a person (load ID illegible) |
| s5 stops | M023 "NL-1O71" (letter O, not zero, marked in red) | Claude finds the likely row (NL-1071) but asks a person; the check agrees: not a sheet ID |
| s6 stops | M004 invoice for NL-1039 | money stays with a person: Claude asks a person (G1); the check agrees |

**Honesty.** The decisions are the recorded AI run (`llm/cached-llm-run.json`, right on all six); the build fails if any of the six disagrees with the answer key. The PDF reader and the check run for real in the page: they match the recorded AI fields against the synthetic email and PDF text. Only copied fields are checked (load ID, shipper, origin and destination codes, pickup date and time); derived fields such as status show "set by the decision". The scored run did not include the check. The check is the pilot design step and can only turn a write into ask a person. The header reads "Mockup · nothing connected · synthetic data".

**Whole night view (second tab).** Four lanes, **Inbox, AI step, Sheet, Morning digest**. **Play the night** plays seven Northline Freight emails in the order they arrived (M002 appointment moved, M003 newsletter, M004 invoice, M005 delivered, M015 new load, M024 portal login, M026 two loads in one email) and ends on the **7:00 morning digest**. Two more tabs show the same flow for two hand-written fictional businesses: **Larkfield Home Comfort (fictional)**, an HVAC jobs sheet, and **Tidewell Goods Co. (fictional)**, an orders sheet. A full play is about 50 s at speed 1.

The rows, emails and decision text are the real synthetic files (`data/inbox_messages.csv`, `data/master_sheet.csv`, the answer key, `llm/cached-llm-run.json`), read by the build. The synthetic shipper was renamed "Harborline Partners" in the data files and both pages on 2026-10-02 because the earlier name read too close to a real brand; every score was re-run and is unchanged. No savings, minutes or ROI claims.

**URL parameters.** `?view=night` opens the Whole night tab (`?case=` and `?business=freight|hvac|wholesale` also open it) · `?present=1` (1920 x 1080 canvas) · `?present=1&frame=1` (1920 x 904 capture mode: no header, method strip, notes or chips) · `?scenario=s1` to `s6` (one scenario, settled) · `?autoplay=1` (in the pipeline view: Play all) · `?speed=N`. Play all takes about 77 s at speed 1.

**Automation hook** (`window.__demo`). Night view, unchanged: `ready` · `play({speed})` · `state()` · `select(id)` · `reset()` · `steps` · `showStep(i)` · `total()`. Pipeline view: `scenarios` (id s1 to s6, `n`, `message_id`, `works`, `caption`, `short`, `stopAt`) · `showScenario(i)` · `playScenario(i, {speed})` · `playScenarios({speed})` · `scenarioResult(i)` · `view()` · `setView(v)` · `total('pipe')`.

**Used by:** the video screen of the deck, between slide 2 and slide 3. `node _build/build-demo-narrated.mjs` records the flowing frame mode (`index.html?present=1&frame=1`), one `__demo.playScenario(i, {ms})` per narration line, with a Kokoro voice reading the six "Narration k of 6" lines of the script. It writes `03-presentation/media/demo-narrated.mp4` (and `.webm`, `.json`, `.srt`, poster), 39.7 s, and `05-video/SheetSync-Demo-Narrated.mp4`. The fallback pictures of the video screen: `node _build/capture-steps.mjs` opens the same frame mode at 1920 x 904, calls `showScenario` for each id in `03-presentation/media/steps/steps.json` (s1 to s6) and writes `step-s1.png` to `step-s6.png`. It fails if any text is cut off. Appendix A2 of the deck is the silent Play all: `node _build/record-demo.mjs 1` writes `05-video/demo.mp4`, `demo.webm` and `poster.png`; `node _build/record-demo.mjs 1 night` writes `05-video/demo-night.mp4`.

**Checks:** `node _build/shoot-mockup.mjs` (screenshots of both views at 1920, 1440 and 390 px; frame-mode text 24 px or more; no horizontal scroll on phones; the six decisions equal the recorded run).

---

## Under the hood (`under-the-hood.html`)

The scored version: one self-contained page (`under-the-hood.html`) that plays an overnight pass: synthetic inbox in, a current spreadsheet Master out, with human gates. Northline Freight is fictional and every message, row, label and number on the page is synthetic.

## Open it

- Double-click `under-the-hood.html` (it runs from `file://`, no server, no build).
- Or serve the folder: `python3 -m http.server 8765` → `http://127.0.0.1:8765/under-the-hood.html`.
- Claude Artifact (scored version): `https://claude.ai/artifact/9wZL8BQF5P875W4i3TdFRB`

The only network request is the Google Fonts stylesheet (IBM Plex Sans / Mono, with system fallbacks). If that request fails (offline, a strict network), the page falls back to the font files in `fonts/` next to it: an inline `@font-face` block sits between `<!--__LOCALFONTS_START__-->` and `<!--__LOCALFONTS_END__-->` right after the Google Fonts link; the artifact build strips that block, the pack and the GitHub Pages copy keep it. Nothing else leaves the page unless you choose the live LLM engine and paste your own key.

## What you are looking at

Header → controls → KPI tiles → the eight-step strip → three panes (Overnight inbox · Closed circle · Master sheet) → the selected message and its routing record → Morning digest · Engine comparison · Run log.

**Run overnight pass** walks the predetermined steps:

| # | Step | Kind |
|---|---|---|
| 1 | Overnight mail arrives in the company inbox | input |
| 2 | Scheduled pass starts (nightly, ~20:30 local); fetch one inbox; de-duplicate on message id | deterministic |
| 3 | Extract attachment text (rate confirmations) | deterministic |
| 4 | Classify the message; extract a new load or match to ≤ 1 Master row; choose create / patch / quiet / escalate; score confidence | **LLM** |
| 5 | Apply: append a row, patch cells, or do nothing; confidence < 0.7 → escalate | deterministic |
| 6 | Verify the write landed (re-read the row; not a semantic check) | deterministic |
| 7 | Write the morning digest and run log (created · patched · quiet · waiting on you) | deterministic |
| 8 | Morning read of the live Master; clear the "waiting on you" queue | human |

Human gates the system never performs: **G1** consequential sends & money · **G2** human-presence checks (login, 2FA, captcha, payment rails; the run stops and waits) · **G3** changing the workflow's rules or scope. Policy: reversible and in scope → automated; irreversible, outward-facing or scope-changing → gated.

## The three engines (same input, same schema)

- **Rules only**: a generic keyword/regex baseline, written blind before the data existed and frozen. Shows what deterministic code alone gets. The page ships a placeholder engine with the same interface; the build step (`_build/inject.mjs`) swaps in `rules/rules-engine.js`.
- **LLM (cached run)**: the default. Claude ran the production prompt (`llm/classify-match-prompt.md`) over every message during the build: three independent passes, majority vote; stored in `llm/cached-llm-run.json` and replayed so the demo runs offline with no key. Label, verbatim: *"LLM outputs: Claude via Claude Code, 3 independent passes, majority vote (N unanimous), generated DATE. The same model family wrote and labeled the data, so this accuracy is an upper bound."* (N and DATE come from the data block.)
- **LLM (live, your key)**: the same prompt and JSON schema (`llm/output-schema.json`, embedded verbatim in the page) sent to `https://api.anthropic.com/v1/messages` from the browser with a key you paste. Off by default, never auto-runs, costs your API credits (about 30 short calls per pass, typically under a dollar at current API pricing; `claude-haiku-4-5` is the cheaper option next to the default `claude-opus-5-5`). The key lives in a closure for the session only; "Forget key" clears it. Any failure (no key, network, non-200, bad JSON, 10-s timeout) falls back to the cached record, tagged `cached-fallback`, with a visible notice.

After every pass the page also scores the other two engines silently, so the comparison table is always complete; the engine you actually ran is highlighted.

## Routing record (one per message)

```json
{ "message_id": "M001", "action": "create | patch | quiet | escalate", "row_id": "R01 or null",
  "fields": { "load_id": null, "shipper": null, "lane": null, "status": null, "appt_local": null, "eta_note": null, "notes_append": null },
  "confidence": 0.0, "gate": "G1 | G2 | G3 | null", "rationale": "one sentence" }
```

`fields` is null for quiet/escalate; for create/patch only the changed keys are set. No money fields exist in the record or the Master. Step 5 is deterministic: any non-escalate record with confidence below 0.7 becomes an escalation (`thresholded: true`), both in the page and in `_build/score.mjs`.

## Metrics (computed live from the data block; never typed in)

- **Routing accuracy** = share of messages where the predicted action equals gold and (gold `row_id` is null or the predicted `row_id` equals gold).
- **Field accuracy** = on gold create/patch messages, share of gold-specified `status`, `appt_local`, `load_id` values reproduced exactly.
- **Gate precision** = on gold escalations with a gate, share where the predicted gate matches.
- Silent misses (auto-handled where gold says escalate, the dangerous error) and over-escalations (escalate where gold says auto: safe, but it costs a human a look).
- Morning lag for auto-handled events ≈ 0 min is a synthetic framing, not a measurement.

## Data and the stated limitation

Synthetic overnight messages (attachment text already extracted), a synthetic Master, gold labels and a cached LLM run power the demo. The same model family wrote and labeled the data, so the cached-LLM accuracy is an upper bound. Production would read the real company mailbox under access controls and write to the real sheet through its API (cell edits and appended rows only). The page never reads `data/` or `rules/` directly; `_build/inject.mjs` embeds them between the markers `<script id="ssdata">`, `<!--__RULES_START__-->…<!--__RULES_END__-->` and `<!--__CIRCLE_START__-->…<!--__CIRCLE_END__-->`.

## What else you can click (v2.1)

- **Show me:** after a pass, a row of beat buttons appears above the detail cards: Create · Patch + verify · G1 sends & money · G2 login · G3 rule change · Ambiguous · Quiet. Each one selects the first message of that kind in the current results, scrolls the matching Master row (or the item in the digest's waiting list, or the inbox card for Quiet) into view, highlights it for about 1.5 s and opens its routing record. Beats that do not occur in the run are disabled. The row stays visible in presentation mode so it can be used for the live click. (`__demo.beat(name)`)
- **Drill: drop one write.** A checkbox in the controls (hidden in presentation mode). When armed, the first write of the run (create or patch) is silently not applied to the working Master. Step 6 re-reads the row, finds the mismatch, restores the row's prior values, logs a coral "verify FAILED → restored" line, puts a "verify ✗ restored" chip on the row, and moves that message to "waiting on human" with the rationale suffix "(verify failed: write did not land; row restored, human review)". The waiting KPI and the digest include it; the engine-comparison cell for that message carries the drilled action with a "drill" tooltip. Routing accuracy is unchanged (the routing decision was right; the write was dropped), and the silent comparison table always scores the raw engine outputs. The provenance caption shows "Drill armed: one write will be dropped on purpose." while armed. Reset un-arms it. (`__demo.drill(on)`)
- **Run again (same night).** After a completed pass the primary button reads "Run again (same night)" (Reset stays separate). A second pass without reset de-duplicates on message id at step 2: every already-applied message routes to `quiet` with the rationale "Already applied in a previous pass (de-duplicated on message id)" and source `dedupe`; no Master changes, no verify writes, no model call. The digest reads "Nothing new. Overnight: 0 created · 0 patched · N quiet · 0 waiting on you" and the status line "Second pass ended quiet. Quiet is a finished pass." The first pass's accuracy and comparison table stay on screen (`state().passes` counts the passes). Changing the engine, the night or the drill makes the next run a fresh pass. `__demo.run()` behaves the same way when called twice without `reset()`.
- **Undo from version history.** The Morning digest lists every created and patched row with an "Undo" button and an "Undo all" at the bottom of that list. Undo restores the row's prior values (or removes a created row), writes an audit line to the run log ("undo R03 by Ops Manager: status Delivered → In Transit"), moves the message to "waiting on human" with the suffix "(undone at morning read)", moves the created/patched and waiting KPIs accordingly, and adds a before/after entry to the "Version history" list under the digest, which stays until Reset. (`__demo.undo(message_id)`; `__demo.undo()` undoes the last write)
- **Outside the gates.** A KPI tile with four counters that are zero by construction (sends 0 · payments 0 · logins 0 · rule changes 0: the workflow has no code path that sends mail, moves money, logs in or edits its own rules; those messages wait for a human) plus "verified writes N/N" (writes that passed verify over writes attempted; N−1/N in the drill). The digest footer repeats the same five values.

## Controls, keys and URL parameters

Keyboard: `R` run (after a pass: run again, same night) · `X` reset · `1` / `2` / `3` engine (rules · cached · live) · `↑` / `↓` select message · `G` toggle gold labels · `U` undo the last write · `D` toggle the drill (not in presentation mode) · `?` help · `Esc` close.

URL parameters: `?engine=rules|cached|live` · `?night=1|2|all` · `?speed=1|2|4|8` · `?gold=0` · `?present=1` (presentation mode: larger type, caption bar, demo-only controls hidden) · `?autoplay=1` (with `present`, plays the scripted timeline once fonts are ready).

Automation hook (`window.__demo`): `ready` (Promise) · `play({speed})` · `run(engine, night, speed)` · `select(message_id)` · `caption(text)` · `highlight(selector|null)` · `reset()` · `beat(name)` · `undo(message_id?)` · `drill(on?)` · `state()` → `{done, running, engine, counts, accuracy, comparison, records, …, passes, rerun, selected, runLabel, drill, history, versions, safety, beats}`. The `play()` timeline and its ten captions are unchanged from v2.

Smoke test: `node _build/inject.mjs && node _build/smoke-prototype.mjs` (headless Chromium via Playwright; checks the markers, every inline script with `node --check`, the three engines, the live-mode mock and fallback, the beats, the drill, run again, undo, the safety counters, the local fonts block, presentation autoplay with the unchanged captions, and the 400-px layout).

## Honesty rules this page follows

- Inspired by a workflow the author runs for his own small freight operation. Everything shown is synthetic; no production ROI or measured minutes saved is claimed.
- Internal Facing × LLM-Based Automated Workflow: the human sets the plan; some steps involve an LLM. Not an autonomous agent: it does not plan its own course, pick its own tools, send mail, move money, or change its rules.
- No real company names, people, emails, rates, drivers, paths or vendor names appear. The entity is "Northline Freight (fictional)".
- Every number on the page is computed from the embedded data block at run time.
- Sheet hygiene first. Money stays human.
- Closed-circle diagram adapted from the course's cycle diagram.

# SheetSync Overnight: prototype

One self-contained page (`index.html`) that plays an overnight pass: synthetic inbox in, a current spreadsheet Master out, with human gates. Northline Freight is fictional and every message, row, label and number on the page is synthetic.

## Open it

- Double-click `index.html` (it runs from `file://`, no server, no build).
- Or serve the folder: `python3 -m http.server 8765` → `http://127.0.0.1:8765/`.
- Claude Artifact: `https://claude.ai/artifact/4iQwwbqRDcKKa2SHYULVV1`
- GitHub Pages: `https://ranvir01.github.io/Vibe/sheetsync-overnight/`

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

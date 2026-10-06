# SheetSync Overnight: capstone prototype (synthetic data)

Open `index.html` (or the GitHub Pages URL for this folder). It is a self-contained prototype of one nightly routine for a fictional small trucking company (a carrier with its own trucks and drivers). Nothing is connected. Everything is synthetic; there is no real company data. Built for the FIN579 Capstone Sprint (AI Fundamentals for Business, Foster School of Business, UW).

**The prototype (`index.html`).** Six synthetic emails go through the routine one app at a time, drawn as the real apps: a Gmail inbox, a Drive folder, a PDF viewer, the Claude call and its answer, a check log, a Google Sheets grid, and the 7:00 digest on a phone. Three work on their own (a rate confirmation becomes a new row, a delivery email fixes a cell, a newsletter needs nothing). Three stop and wait for a person (a smudged scan, a load ID with a letter O, a $2,450 invoice). Press Play all, or use the arrow keys. Of the four methods in the course matrix, this is an LLM-Based Automated Workflow: fixed steps, one AI step, not an agent. The decisions are the recorded AI run.

**Whole night view (`whole-night.html`).** Seven emails from one test night in four lanes, then the 7:00 morning digest.

**Scored version (`under-the-hood.html`).** Plain rules versus the recorded AI run on all 30 synthetic emails, checked against the answer key.

Publishing: Settings, then Pages, then Deploy from a branch, then this branch and `/ (root)`. The pages need no build step and make no network calls, except the optional bring-your-own-key live mode on the scored version.

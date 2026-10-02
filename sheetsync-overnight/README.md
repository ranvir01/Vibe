# SheetSync Overnight: capstone prototype (synthetic data)

Open `index.html` (or the GitHub Pages URL for this folder). It is a simple, self-contained mockup of one nightly routine for a fictional small trucking company (a carrier with its own trucks and drivers). Nothing is connected. Everything is synthetic; there is no real company data. Built for the FIN579 Capstone Sprint (AI Fundamentals for Business, Foster School of Business, UW).

**Scenarios (the default view).** One fixed route: Gmail (read-only), Google Drive (saves the PDF), a PDF reader (text, or OCR for scans), Claude (the one AI step), a check (code: every copied value must be in the source), Google Sheets through an MCP connector (a new row or one cell edit), and a 7:00 digest. Of the four methods in the course matrix, this is an LLM-Based Automated Workflow: fixed steps, one AI step, not an agent. Six synthetic emails play on the route: three that work (a rate confirmation becomes a new row, a delivery email fixes a cell, a newsletter needs nothing) and three that stop and ask a person (a smudged scan, a load ID with a letter O, a $2,450 invoice). The decisions are the recorded AI run. The check is the pilot design: it runs for real on the synthetic text here, but the scored run did not include it.

**Whole night (second tab).** The earlier four-lane view: seven emails from the two test nights, then the 7:00 morning digest, plus two hand-written examples from other businesses.

`under-the-hood.html` is the scored version behind the mockup: plain rules versus the recorded AI run on 30 synthetic emails, checked against the answer key.

Publishing: Settings, then Pages, then Deploy from a branch, then this branch and `/ (root)`. The pages need no build step and make no network calls except Google Fonts (offline copies in `fonts/` as fallback) and the optional bring-your-own-key live mode on the scored version.

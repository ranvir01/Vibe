# SheetSync Overnight: capstone prototype (synthetic data)

Open `index.html` (or the GitHub Pages URL for this folder). It is a simple, self-contained mockup of one nightly routine for a fictional small trucking company (a carrier with its own trucks and drivers): overnight emails go through Inbox, one AI step, the spreadsheet and a 7:00 morning digest. Seven made-up emails show every outcome (new row, fix a cell, nothing to do, ask a person), and two hand-written examples show the same flow in other businesses. Nothing is connected; short notes say what each step would connect to in a pilot. Everything is synthetic; no real company data. Built for the FIN579 Capstone Sprint (AI Fundamentals for Business, Foster School of Business, UW).

`under-the-hood.html` is the scored version behind the mockup: plain rules versus the recorded AI run on 30 synthetic emails, checked against the answer key.

Publishing: Settings, then Pages, then Deploy from a branch, then this branch and `/ (root)`. The pages need no build step and make no network calls except Google Fonts (offline copies in `fonts/` as fallback) and the optional bring-your-own-key live mode on the scored version.

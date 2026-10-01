# SheetSync Overnight: capstone prototype (synthetic data)

Open `index.html` (or the GitHub Pages URL for this folder). It is a single self-contained page: an overnight inbox → spreadsheet "Master" workflow for a fictional small freight brokerage, with one LLM step, three human gates, gold-label scoring, "Show me" buttons, a verifier drill, undo and a morning digest (v2.1). Everything is synthetic; no real company data. Built for the FIN579 Capstone Sprint (AI Fundamentals for Business, Foster School of Business, UW).

Publishing: commit this folder, then Settings → Pages → Deploy from a branch → root. The page needs no build step and makes no network calls except Google Fonts (with the offline copies in `fonts/` as fallback) and the optional bring-your-own-key live mode.

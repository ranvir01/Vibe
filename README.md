# CSV Column Stats

A deterministic, single-file web app (`index.html`). Upload a CSV and it lists every column's name, whether the column is numeric, and whether it has missing values. Pick a numeric column to get its mean, its sample standard deviation (n − 1) and its population standard deviation (n).

- Plain JavaScript with no libraries, no server, no network calls and no AI at runtime. The same file always gives the same numbers.
- The page runs a built-in self-test on load (91 checks against `test.csv`, whose values were worked out by hand).
- Rules: missing = blank, `NA`, `N/A`, `NaN`, `null`, `None`, `#N/A`. A column is numeric only when every non-missing value is a plain number.

## Expected results for `test.csv`

| Column | Type | Missing | n | Mean | SD sample | SD population |
|---|---|---|---|---|---|---|
| id | Numeric | 0 | 5 | 3.0000 | 1.5811 | 1.4142 |
| price | Numeric | 0 | 5 | 30.0000 | 15.8114 | 14.1421 |
| score | Numeric | 2 | 3 | 4.0000 | 2.0000 | 1.6330 |
| city | Text | 1 | — | — | — | — |
| code | Text (mixed: 3 of 5) | 0 | — | — | — | — |
| note | Text | 0 | — | — | — | — |
| single | Numeric | 4 | 1 | 42.0000 | n/a | 0.0000 |
| sci | Numeric | 0 | 5 | 20.8000 | 44.3122 | 39.6341 |

## Make it public (GitHub Pages)

1. Make sure the repository is public (Settings → General → Danger Zone → Change visibility).
2. Settings → Pages → Source: **Deploy from a branch**, then pick the branch that holds `index.html`, folder `/ (root)`, and save.
3. After a minute or two the app is live at `https://ranvir01.github.io/Vibe/`.

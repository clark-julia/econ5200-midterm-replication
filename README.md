# Replicating Hansen (2015), "Punishment and Deterrence: Evidence from Drunk Driving"

## The paper and its headline claim

- **Paper:** Hansen, Benjamin. 2015. "Punishment and Deterrence: Evidence from Drunk Driving." *American Economic Review* 105(4): 1581-1617.
- **Identification strategy:** Regression discontinuity. Drivers stopped with a blood alcohol content (BAC) of 0.08 or higher face harsher punishment (DUI), and 0.15 or higher harsher still (aggravated DUI). I compare drivers just above and just below each cutoff.
- **Headline claim:** Crossing the 0.08 cutoff lowers recidivism by about 2.1 percentage points (Table 3, Panel A, Bandwidth 0.05)
- **My replication:** -0.0222 (SE 0.0041, clustered by BAC) against the paper's -0.021, there is a gap of ~5.7%.

## Reproduce the headline table

Python 3.11 or newer. From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python reproduce.py
```

`python reproduce.py` is the one command. Once `MY_PAPER` is set in `download_data.py`,
it downloads the data if `data/` does not have it, runs `notebooks/01_positioning_and_replication.ipynb` and then
`notebooks/02_robustness.ipynb` top to bottom in a fresh kernel, and prints the headline
table from `output/headline_table.csv`. Executed copies of the notebooks, with their
outputs, go to `output/executed/`.

- **Expected runtime:** TODO: minutes on your machine, measured from a clean clone (the empty template runs in under a minute).

## Data: source and licence

- **Source:** hansen_dwi.dta` from the `scunning1975/mixtape` repository on GitHub
  (URL set in `download_data.py`). It contains 214,558 Washington State drunk-driving stops from 1999 to 2007. The underlying data were collected by Benjamin Hansen for the original paper.
**Included or downloaded:** Downloaded on the first run (about 9.5 MB) and not committed to the repository, following the template default. The mixtape repository is under an MIT-style licence that permits copies if its notice is kept.
- The five files from `scunning1975/mixtape` (castle, card, lee, hansen, abortion) are in a repository under an MIT-style licence that permits copies if its notice is kept. The underlying data were collected by the original authors.
- The Dehejia-Wahba files on `users.nber.org/~rdehejia` state no licence. Download them; do not commit them.

## Repository map

| Path | What it is |
|---|---|
| `reproduce.py` | The one command: runs the notebooks in name order, prints the headline table |
| `download_data.py` | Data URLs for the six papers; `python download_data.py --paper card` fetches one into `data/` |
| `notebooks/01_positioning_and_replication.ipynb` | Sections 1 and 2: positioning notes, the replication, the theirs-vs-mine table, the discrepancy log. **No AI.** |
| `notebooks/02_robustness.ipynb` | Section 3: four robustness exercises. AI allowed, logged in `ai-appendix.md` |
| `output/headline_table.csv` | The theirs-vs-mine table, written by notebook 01 |
| `output/` | Every table and figure the report uses |
| `report/` | The report PDF |
| `ai-appendix.md` | P.R.I.M.E. entries and full transcripts for Sections 3 to 6, and the no-AI statement for Sections 1 and 2 |
| `AUDIT.md` | How to run the peer audit you file on a classmate's repository |
| `requirements.txt` | Pinned package versions |


## Use of AI

Sections 1 and 2 were completed without AI.

Section 3 I used AI to help interpret results, fix syntax and build some code to help organize results.
Section 4 I used AI to help edit my writing and asked if my explainations were strong. 
entries in [`ai-appendix.md`](ai-appendix.md). 

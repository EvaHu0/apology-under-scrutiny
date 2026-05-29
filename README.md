# apology-under-scrutiny

Reddit audience response to corporate apologies across competence and integrity crises — a computational comparative analysis.

*MACS 30200 Final Proposal, University of Chicago, Spring 2026.*

This repository accompanies a four-case proposal on how online audiences respond to corporate apologies. The full design compares competence-failure cases against integrity-violation cases using a validated dual-method sentiment pipeline, LDA topic modeling, and image-repair-tactic close reading. **What is implemented here is the CrowdStrike pilot** — the competence-failure case that establishes feasibility, validates the dual-method sentiment design, and recovers the pre/post topic-shift pattern the proposal predicts.

## Pilot at a glance

The pilot analyzes Reddit discussion of the July 19, 2024 CrowdStrike outage and CrowdStrike's subsequent apology. Three findings carry the methodological argument:

- **Single-method sentiment is fragile.** VADER and Twitter-RoBERTa agree on the sentiment label of only ~47% of comments. Treating either model as ground truth on its own would have produced a misleading picture, which is the motivation for the dual-method design used across the full study.
- **The sentiment shift is method-dependent under scrutiny.** A full-sample pre/post test is significant for VADER, but a matched-sample bootstrap (379 comments per group to correct the pre/post size imbalance, 200 trials) shows that result is not robust for VADER while it holds far more often for RoBERTa. Details in `data/stats_v2.json` and `figures/matched_sample_robustness.png`.
- **Topics shift from incident to accountability.** LDA (k = 6, selected by c_v coherence) recovers a move from immediate-incident discussion before the apology toward responsibility- and trust-oriented discussion after it.


## Data and collection pipeline

Comments were retrieved from the public **PullPush.io** API (a Reddit archive mirror), paginating backwards through time on the `before` parameter. The pilot collection parameters, logged in `stats.json`, are:

- **Case:** CrowdStrike (competence failure)
- **Query:** `CrowdStrike`
- **Subreddits:** `technology`, `sysadmin`, `news`
- **Window:** ~24 hours before the crisis through ~30 days after the apology (exact UTC timestamps for the crisis, the apology, and the collection bounds are in `stats.json` under `collection`)
- **Pre/post split:** comments are assigned to `pre` or `post` relative to the apology timestamp (pre = 379, post = 3,389)
- **Volume:** 3,873 raw comments → 3,768 after filtering

Preprocessing drops `[deleted]`/`[removed]`/empty bodies and very short comments. Each retained comment is scored by both VADER (lexicon) and `cardiffnlp/twitter-roberta-base-sentiment` (transformer).

**Pseudonymization.** Author identifiers are replaced with a truncated SHA-256 hash (`author_hash`); no raw usernames appear in any released file. Comment text (`body`) is included so the sentiment and topic results are inspectable and reproducible. All comments originate from the public PullPush mirror and can be re-fetched from their original IDs; please observe Reddit's and PullPush's terms when reusing the text.

## Setup

```bash
git clone https://github.com/EvaHu0/apology-under-scrutiny.git
cd apology-under-scrutiny
pip install -r requirements.txt
```

The transformer step (`pilot_crowdstrike.ipynb`) downloads the RoBERTa weights on first run and benefits from a GPU but does not require one.

## Reproducing the results

- **`notebooks/pilot_crowdstrike.ipynb`** runs the full pipeline and regenerates `data/comments_hashed.csv`, `data/stats.json`, and the four v1 figures. The data-collection cells call PullPush live, so re-running reconstructs the corpus from scratch.
- **`notebooks/pilot_crowdstrike_v2.ipynb`** loads the saved `comments_hashed.csv` and produces the matched-sample and extended-stopword robustness outputs (`data/stats_v2.json`, the two v2 figures). This notebook currently loads the CSV through a Colab upload widget; running it outside Colab requires replacing that cell with a local `pd.read_csv("data/comments_hashed.csv")`.

## Viewing the notebooks

GitHub's in-browser notebook preview intermittently fails with an "An error occurred" page even for valid notebooks; this is a known issue on GitHub's rendering service and is independent of the files here. If the preview does not load, view the notebooks through nbviewer:

- pilot: https://nbviewer.org/github/EvaHu0/apology-under-scrutiny/blob/main/notebooks/pilot_crowdstrike.ipynb
- robustness (v2): https://nbviewer.org/github/EvaHu0/apology-under-scrutiny/blob/main/notebooks/pilot_crowdstrike_v2.ipynb

Both notebooks also render correctly on download in any local Jupyter, JupyterLab, or VS Code environment.

## Citation

If you use this code or data, please cite the accompanying proposal:

> Hu, E. (2026). *Audience Responses to Corporate Apologies on Reddit.* MACS 30200, University of Chicago.

## License

MIT — see `LICENSE`.

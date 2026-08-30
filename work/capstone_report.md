# Capstone Report — Structured Content Archetypes in Search Performance

- **Author:** Zuzanna Hibner
- **Lane:** Structured Content Archetypes in Search Performance
- **Repo:** https://github.com/zmhibner-gif/ml_internship
- **Date:** August 2026

## 0. Abstract

Five sentences, written last, placed first: question → data → method → headline result →
what the output is for. This is the top of your deployed paper.

## 1. Problem framing

This project asks whether search-performance data can be used to group pages into actionable content archetypes.

The unit of analysis is one page aggregated over one month. The main output is a cluster assignment that describes a page's search-performance pattern. Each cluster is translated into a content action such as improve CTR, improve visibility, monitor, or protect.

The analysis supports content-review decisions by helping editors distinguish between pages that perform poorly for different reasons. A wrong recommendation could waste editorial time or lead to unnecessary changes to a page that is already performing appropriately.

Data analysis helps because the dataset contains a large number of pages with different combinations of visibility, ranking position, and click activity. Clustering can identify recurring patterns across these pages that would be difficult to review manually.

## 2. Data safety

The project uses the FlyRank ML Internship warehouse hosted on Hugging Face. The analysis uses March 2026 data from the fact_content_daily_performance table.

The unit of analysis is one page aggregated over March 2026. The five features used are:

- impressions
- CTR
- average search position
- days with impressions
- days with clicks

Rows without available Google Search Console data are excluded, as well as pages with zero total impressions. This produces a development dataset of 176,738 pages.

Client and content hash IDs are kept only for grouping and identifying pages. They are not used as model features. Label-derived variables such as trend_direction and trend_pct are deliberately excluded, as are product flags and future-month data.

No client names, domains, URLs, private queries, credentials, or other client-identifying information are included in the public analysis or outputs.

## 3. Baseline
Before clustering, I built a transparent rule-based baseline that identifies visible pages with lower-than-expected CTR for their average search position.

Two signals were checked before creating the rule. Mean CTR decreased as average search position became worse, while pages with more impressions were increasingly likely to receive clicks. These checks supported using search position together with CTR and applying a minimum visibility requirement.

The baseline flags pages with at least 500 impressions whose CTR is below the average CTR of their position bucket. Pages are ranked using an estimated missed-click opportunity:

Baseline score = impressions × CTR gap

where CTR gap is the difference between the average CTR for the page's position bucket and the page's actual CTR.

For the capstone comparison, the same baseline rule was applied to the held-out test pages. It produced two groups and achieved a silhouette score of 0.215, compared with 0.377 for the four-cluster K-Means model on the same test set.

## 4. Model / analysis

I use K-Means clustering because the project aims to group pages into recurring search-performance archetypes rather than predict a predefined outcome.

The model uses five features:

impressions
CTR
average search position
days with impressions
days with clicks

Client and content hash IDs are left out on purpose because they identify pages rather than describe their performance. Product flags, future-month data, and label-derived fields are also excluded.

Impressions, CTR, and average position are log-transformed because they are strongly skewed. All five features are then standardized so that no single feature dominates the K-Means distance calculation.

I compare solutions from 2 to 6 clusters on the training data. K = 4 gives the highest silhouette score, 0.404, so four clusters are selected.

Target / proxy definition: There is no target label; the page archetypes are inferred directly from the five search-performance features.

## 5. Evaluation

The data is split into training and test sets by client, with 20% of clients held out for testing. Grouping by client prevents pages from the same client appearing in both sets and gives a more realistic check of whether the clustering patterns generalize to unseen clients.

The final evaluation uses 38,428 held-out test pages. Because this is an unsupervised task with no true class label, accuracy and precision are not appropriate. I use silhouette score to measure how clearly the resulting groups are separated.

On the same held-out pages:

- Week 4 baseline: silhouette score = 0.215
- K-Means, 4 clusters: silhouette score = 0.377

The higher test silhouette suggests that K-Means produces more distinct search-performance groups than the simple baseline rule. The test score is also reasonably close to the training silhouette of 0.404, suggesting that the cluster structure transfers to unseen clients.

**Error analysis**

The clusters are not perfectly separated and should not be interpreted as true categories. Some pages do not match the typical profile of their cluster, for example pages in a low-visibility cluster that still have relatively high impressions. This happens because K-Means assigns pages using all five features together rather than one fixed threshold.

The Week 4 baseline also highlights a limitation of the clustering comparison: it is designed specifically to find low-CTR opportunities, while K-Means separates several different performance patterns. The silhouette comparison therefore measures group separation, not whether either approach causes better content performance.

## 6. Interpretation

The four clusters represent distinct search-performance archetypes:

- Cluster 0 — Low visibility: pages typically receive very few impressions, appear on only a small number of days, and usually receive no clicks.
- Cluster 1 — Visible, weak engagement: pages are shown consistently throughout the month and receive more impressions, but the typical page still receives no clicks.
- Cluster 2 — Low volume, high CTR: a small group of pages with relatively few impressions but much stronger CTR and some click activity.
- Cluster 3 — High visibility, active: pages receive thousands of impressions, rank relatively well, and receive clicks on many days.

The largest group is Cluster 1, which suggests that persistent visibility without strong click activity is a common pattern in the dataset. Cluster 2 is much smaller than the others, showing that high CTR at low volume is a relatively rare archetype.

A useful result is that the baseline and clustering do not identify exactly the same pages. The baseline mainly flags pages in Clusters 1 and 3, while clustering also separates low-visibility and high-CTR page types that the baseline does not target. This suggests that clustering adds descriptive structure beyond the single CTR-focused rule.

## 7. Recommendation

The clustering output is translated into four content actions:

- IMPROVE_CTR: review titles, descriptions, search intent, and content relevance for pages with strong visibility but weak click performance.
- IMPROVE_VISIBILITY: focus on pages with relatively strong CTR but limited exposure, for example by improving internal linking, topical coverage, or discoverability.
- MONITOR: avoid immediate intervention on low-activity pages and collect more data before deciding whether they need stronger changes.
- PROTECT: preserve high-visibility pages with regular click activity and make changes cautiously.

Pages are ranked within each action group rather than across all actions, because priority means something different for each archetype. For IMPROVE_CTR, pages are ranked by estimated missed-click opportunity. The other groups are ranked by impressions to highlight the most visible pages within each archetype.

A FlyRank editor could use the output as a review queue: start with the highest-ranked pages in the action that matches the current content goal, then check the individual page context before making changes.

Confidence is highest at the archetype level, where the clusters show clear differences in visibility and click activity. Confidence is lower for individual page recommendations because the model does not include search intent, SERP features, page content, or other contextual factors.

These recommendations are therefore decision-support, not automatic instructions. They indicate which pages deserve attention and what type of review may be useful, but they do not prove that a specific content change will improve performance.

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas). If you claim a sealed or
holdout evaluation, two things must be committed: the cell/script that builds the sealed
frame, and the metrics file it produced — "evaluated once, blind" should be checkable from
your repo, not taken on faith.

## 9. Acknowledgments & data credit

One short section at the bottom of the deployed paper: "Built on the FlyRank ML Internship
dataset" **linking to https://flyrank.ai**. Crediting your data source is standard research
practice — and it's on the capstone's required-section list, so a paper without it isn't done.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.

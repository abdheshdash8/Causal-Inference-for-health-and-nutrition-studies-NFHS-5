# Causal Learning for Strategic Policy Design in Health and Nutrition
### Based on NFHS-5 | IIT Delhi — Master's Thesis Project

**Author:** Abdhesh Dash · Department of Mathematics · Indian Institute of Technology Delhi  
**Supervisor:** Dr. Rahul Singh · Co-supervisor: Dr. Biplab Paul  
**Target Journal:** Statistics and Computing (Springer Nature)

---

## Overview

This project develops a **data-driven causal learning framework** applied to India's fifth National Family Health Survey (NFHS-5, 2019–21), which covers 131 standardised health and nutrition indicators across all 36 states and union territories.

Standard public health analysis of NFHS data stops at correlation and association. This work goes further: it asks *what would actually happen to health outcomes if a specific indicator were changed by policy intervention?* That question requires causal inference, not just statistics.

The framework answers it in three stages:

1. **Graph clustering** — group 131 indicators into thematically coherent clusters using hierarchical, Louvain, Leiden, and Spectral methods, with evidence-based parameter selection (elbow diagnostics, silhouette scores, NMI consensus ranking).

2. **Causal structure learning** — apply the PC algorithm within each cluster with bootstrap edge-confidence estimation, then screen recovered edges through a three-criterion filter (directional asymmetry ≥ 0.10, undirected-edge fraction < 0.30, partial-correlation residual ≥ 0.35) to separate genuine causal directions from confounding artefacts.

3. **Causal effect estimation** — estimate Average Treatment Effects (ATEs) along 10 validated causal pathways using four complementary estimators (IPW, DR Learner, T-Learner, X-Learner), stratified by rural/urban area type and per-capita GSDP income tertile.

The stratified analysis reveals critical heterogeneity invisible to pooled inference — including a sign reversal in one pathway between rural and urban strata that is a direct instance of Simpson's paradox.

---

## Repository Structure

```
.
├── notebooks/
│   ├── nfhs_main.ipynb                  # Full pipeline: data → clustering → discovery → ATE
│
├── data/
│   ├── NFHS5_key_indicators.xlsx        # Raw NFHS-5 state-level key indicators (131 vars)
│   └── gsdp_states_2020_21.csv          # Per-capita GSDP by state (RBI, 2020-21)
│
├── plots/
│   ├── clustering/
│   │   ├── hierarchical_n_clusters.png  # Merge-height elbow + cohesion curve diagnostics
│   │   ├── mod_vs_res.png               # Louvain/Leiden resolution sweep
│   │   ├── nmi_heat_map.png             # Pairwise NMI between four clustering methods
│   │   ├── heirarchical_cluster_1.png   # Full dendrogram (k=32)
│   │   ├── louvain.png                  # Louvain community graph
│   │   ├── leiden.png                   # Leiden community graph
│   │   └── spectral_clustering.png      # Spectral clustering result
│   │
│   ├── causal_discovery/
│   │   ├── boostrap_thres_analysis.png  # Bootstrap threshold sensitivity sweep
│   │   ├── distribution_of_edge_conf.png # Bootstrap confidence distribution
│   │   ├── direction_asymm_distribution.png # Asymmetry KDE + CAUSAL/REJECT split
│   │   ├── three_criterion_screening.png # Criterion pass-rate bars + asymmetry hist
│   │   ├── cluster_level_causal_dag_3.png # Inter-cluster causal DAG
│   │   └── intra_cluster_graph.png      # Intra-cluster DAG example
│   │
│   ├── ate_estimation/
│   │   ├── forest_plot.png              # Pooled ATE forest plot (all 4 estimators)
│   │   ├── ate_heatmap.png              # ATE heatmap (pathway × estimator)
│   │   ├── cate_heterogeneity_plot.png  # State-level CATE violin plots
│   │   ├── forest_plot_area.png         # Rural vs Urban stratified forest plot
│   │   ├── forest_plot_income.png       # Low/Moderate/High GSDP forest plot
│   │   └── heat_map_income.png          # Income-stratified median ATE heatmap
│   │
│   └── correlation/
│       ├── line_plot_corr_1.png         # Overall correlation rank plot
│       ├── line_plot_area_corr_1.png    # Area-stratified correlation rank plot
│       ├── line_plot_sgdp_corr_1.png    # Income-stratified correlation rank plot
│       └── line_plot_partial_corr.png   # Partial vs stratified correlation
│
├── report/
│   ├── sn-article_fixed.tex             # Main paper (Springer sn-jnl format)
│   ├── sn-jnl.cls                       # Patched Springer journal class file
│   ├── ref.bib                          # Complete BibTeX bibliography (46 entries)
│   └── sn-mathphys-num.bst              # Springer bibliography style
│
└── README.md
```

---

## The 10 Causal Pathways

After multi-stage screening of 131 indicators, **10 policy-relevant causal pathways** were validated. These cover child nutrition, immunisation, maternal health, family planning, gender-based violence, and non-communicable disease prevention.

| # | Pathway | Mechanism | Expected ATE Direction |
|---|---------|-----------|----------------------|
| 1 | Solid Food+BF (6–8m) → Diarrhoea ORS Treatment | IYCF → immune function → care-seeking | Positive |
| 2 | Family Planning (Any) → BCG Vaccination | Shared PHC delivery platform | Positive |
| 3 | Pregnant Women Anaemia → Exclusive Breastfeeding | Maternal iron → lactation capacity | Negative |
| 4 | Unmet Need for Spacing → Neonatal Tetanus Protection | Inter-birth interval → TT schedule completion | Negative |
| 5 | Physical Violence → Household Decision-Making | Violence → women's agency erosion | Negative |
| 6 | Women Tobacco Use → Child Overweight | Prenatal tobacco exposure → metabolic disruption | Positive |
| 7 | First-Trimester ANC → Home Births to Facility | Early care-seeking → care trajectory | Positive |
| 8 | Postnatal Care → Vitamin A Supplementation | Shared PNC delivery platform | Positive |
| 9 | Women Elevated BP → Public-Facility C-section | Hypertensive pregnancy → obstetric emergency | Positive |
| 10 | Family Planning (Any) → Unmet Need for Spacing | FP adoption → exit from unmet-need pool (collider simplification) | Negative |

---

## Methods

### Data
- **NFHS-5 (2019–21)**: 131 key indicators, 36 states/UTs, stratified by Total/Rural/Urban and Low/Moderate/High per-capita GSDP
- **RBI GSDP data (2020–21)**: per-capita Gross State Domestic Product at 2011–12 constant prices

### Graph Clustering
- **Methods evaluated**: Hierarchical (Ward/Average linkage), Louvain, Leiden, Spectral
- **Parameter selection**: merge-height elbow + within-cluster cohesion curve for *k* (hierarchical); modularity + silhouette sweep for resolution γ (Louvain/Leiden); silhouette sweep for *k* (Spectral)
- **Method selection**: combined rank across Silhouette score, Modularity Q, and mean pairwise NMI — hierarchical clustering (k=32) selected
- **Correlation thresholds** (Cohen 1988): |r| ≥ 0.70 strong, 0.50–0.70 moderate, 0.30–0.50 weak

### Causal Structure Learning
- **Algorithm**: PC algorithm (causal-learn) applied within each of 32 clusters + inter-cluster meta-graph
- **Bootstrap aggregation**: B=100 resamples per cluster; edge confidence = fraction of boots where edge is oriented in the same direction
- **Three-criterion screening**:
  - Directional asymmetry α = P(A→B) − P(B→A) ≥ **0.10** — threshold justified by right trough in KDE of asymmetry distribution
  - Undirected-edge fraction < **0.30** — threshold at natural gap in bidir-fraction distribution  
  - Partial-correlation residual |ρ<sub>AB·Z</sub>| ≥ **0.35** — threshold at antimode of partial-r KDE, separating confounded from residual associations
- **Stage 4 rule-based filter**: removes severity-pair sub-indicators, gender-mirror pairs, FP sub-method pairs, and temporally implausible directions
- **Stage 5 ranking**: combined rank of domain score + orientation stability + partial-r residual

### ATE Estimation
Four estimators are used to span the space of identification assumptions:

| Estimator | Relies on | Implementation |
|-----------|-----------|---------------|
| IPW (Inverse Probability Weighting) | Propensity model only | DoWhy |
| DR Learner (Doubly Robust) | Either propensity OR outcome model | EconML LinearDRLearner |
| T-Learner | Outcome models only (separate per group) | CausalML BaseTLearner |
| X-Learner | Propensity-weighted cross-imputation | CausalML BaseXLearner |

**Consistency criterion**: ATE classified as *Consistent* if ≥ 3 of 4 estimators agree on sign.  
**Stratified estimation**: full pipeline re-run on Rural/Urban strata and Low/Moderate/High GSDP tertiles.

---

## Key Findings

### Pooled ATEs
- **FP → BCG**: large positive effect (+15–25 pp) — shared PHC platform delivers both FP counselling and childhood vaccination
- **FP → Unmet Spacing**: robustly negative (−3 to −5 pp) — mechanically guaranteed by the definitional structure of NFHS unmet-need decomposition
- **Pregnant Anaemia → EBF**: negative (−8 to −15 pp) — maternal iron deficiency impairs lactation
- **Postnatal Care → Vitamin A**: positive (+6 to +12 pp) — shared home-visit delivery platform

### Area-Stratified Heterogeneity
Stratification revealed heterogeneity completely hidden in pooled estimates:
- Effects are systematically larger in **rural strata** for nutrition and vaccination pathways — rural areas have lower baselines and greater marginal returns to health-system engagement
- The **FP→BCG pathway** shows evidence of a sign reversal between strata, analogous to Simpson's paradox, driven by differential PHC infrastructure between rural and urban areas

### Income-Stratified Heterogeneity  
- **Diminishing marginal returns** pattern: FP→Spacing effect is largest in Low-GSDP states and smallest in High-GSDP states, consistent with higher baseline unmet need in poorer states
- **Causal effect saturation** in Low-GSDP states: anaemia effects are weakest in the poorest states where child undernutrition is overdetermined by multiple concurrent causes beyond maternal anaemia alone

---

## Setup and Usage

### Requirements
```
python >= 3.10
causal-learn
dowhy
econml
causalml
scikit-learn
scipy
pandas
numpy
networkx
python-louvain
igraph
leidenalg
matplotlib
seaborn
statsmodels
```

### Install
```bash
pip install causal-learn dowhy econml causalml python-louvain igraph leidenalg
pip install scikit-learn scipy pandas numpy networkx matplotlib seaborn statsmodels
```


### Run order
```
1. nfhs_main.ipynb            # all code in one notebook
```

---

## Compiling the Paper

The LaTeX paper uses the **Springer `sn-jnl` class** with the `sn-mathphys-num` bibliography style.

```bash
cd report/
pdflatex sn-article_fixed
bibtex   sn-article_fixed
pdflatex sn-article_fixed
pdflatex sn-article_fixed
```

**Note on `sn-jnl.cls`:** The included `sn-jnl.cls` is patched to remove the forced `threeparttable` wrapping from the `table` environment, which conflicts with `\resizebox` in the original Springer distribution. Use this patched version — do not replace it with the unpatched version from the Springer download.

---

## Identification Assumptions

All causal estimates are observational and rely on standard identifiability assumptions:

- **Consistency**: the observed outcome equals the potential outcome under the received treatment
- **Conditional ignorability**: no unmeasured confounders after conditioning on per-capita GSDP and women's literacy
- **Positivity**: every state has non-zero probability of being in both treatment groups at the state median

These assumptions **cannot be tested from the data alone**. Results should be interpreted as *estimated treatment effects under standard identifiability assumptions*, not as proven causal quantities. Unmeasured state-level confounders (governance quality, historical health infrastructure, political economy) cannot be ruled out. All estimates are ecological (state-level) and should not be attributed to individual-level mechanisms without further microdata analysis.

---

## Citation

If you use this codebase, data analysis, or findings, please cite:

```bibtex
@article{dash2025causal,
  author  = {Dash, Abdhesh and Singh, Rahul and Paul, Biplab},
  title   = {Causal Learning for Strategic Policy Design in Health and
             Nutrition (Based on {NFHS}-5)},
  journal = {Statistics and Computing},
  year    = {2026},
  note    = {Under review}
}
```

---

## Data Sources

| Dataset | Source | Access |
|---------|--------|--------|
| NFHS-5 Key Indicators | International Institute for Population Sciences (IIPS) | [dhsprogram.com](https://dhsprogram.com/data) |
| Per-capita GSDP 2020–21 | Reserve Bank of India Statistical Handbook | [rbi.org.in](https://www.rbi.org.in/Scripts/AnnualPublications.aspx) |

NFHS-5 data are publicly available and free to use for research purposes. Per-capita GSDP figures are from the public RBI Statistical Publication and are in the public domain.

---

## License

Code in this repository is released under the **MIT License**. See `LICENSE` for details.  
The paper manuscript (`report/sn-article_fixed.tex`) is © the authors; all rights reserved pending journal publication.  
NFHS-5 data are subject to the DHS Program data use agreement.

---

## Acknowledgements

This work was carried out in the Department of Mathematics, IIT Delhi. The authors thank the International Institute for Population Sciences (IIPS) and ICF for making the NFHS-5 data publicly available, and the developers of causal-learn, DoWhy, EconML, and CausalML for their open-source implementations.

# Ethiopia forced-displacement pilot: destination choice modeling evidence pack (BGM)

**Author:** Arturo de Nieves (UNHCR Senior Operations Officer & IHEID Senior Fellow in Residence, incoming)  
**Contact:** [denieves@unhcr.org]  
**Purpose:** Demonstrate predictive performance, robustness, and methodological credibility of a gravity-style BGM for **camp destination choice** under forced displacement, and propose integration into the team’s modeling stack.

## 1) Problem and ML framing
We model **household → camp destination** as a **dyadic choice** problem. For each refugee household \(i\), the observed camp \(d^\*\) is treated as the positive choice \(y=1\) and we sample \(k_{neg}=10\) alternative camps as negatives \(y=0\). We evaluate held-out performance using **GroupShuffleSplit by household id (hid)** to prevent within-household leakage. Primary metric: **AUC**.

**Dataset summary (pilot):** 3,627 refugee households; 5 origins; 19 camps; 39,897 dyads per run (3,627 positives + 36,270 negatives).

## 2) Model specification (no OD-pair memorization)
We explicitly exclude any origin×destination pair dummy (which can memorize OD propensity). The main specification is a gravity-style logistic model with origin and camp fixed effects plus interpretable structural terms:

\[
P(y_{i,d}=1)=\sigma\Big(
\alpha_{o(i)}+\gamma_{d}
+\beta_E dE_{o(i),d}
+\beta_C dC_{o(i),d}
+\beta_S dS_{o(i),d}
+\theta_1 \log(1+\text{diaspora\_share}^{LOO}_{o(i),d})
+\theta_2 \log(1+\text{diaspora\_count}^{LOO}_{o(i),d})
\Big)
\]

**Terms (interpretation):**
- \(\alpha_o\): **origin fixed effects** (baseline origin pattern)
- \(\gamma_d\): **camp fixed effects** (baseline camp pull/assignment propensity)
- \(dE,dC,dS\): **capital-space distances** (compatibility/friction between origin and camp)
- diaspora terms: **leave-one-out** (LOO) network concentration features (path dependence) to avoid trivial leakage

## 3) Capital-space construction and distance
We construct household-level proxies for **Economic (E)**, **Cultural (C)**, and **Social (S)** “capitals” using interpretable survey-derived blocks (economic anchored in assets/food/nonfood; C and S from roster/household composition proxies). We define:
- **Origin centroid** = mean household vector by origin  
- **Camp centroid** = mean household vector by observed camp  

Distances are computed using **Mahalanobis** distance between origin and camp centroids within E/C/S blocks:
\[
d_{\text{maha}}(o,d)=\sqrt{(\mathbf{x}_o-\mathbf{x}_d)^\top \mathbf{V}^{-1}(\mathbf{x}_o-\mathbf{x}_d)}
\]

## 4) Headline results (uniform negatives)
**Uniform negatives (kneg=10), mean±sd across 10 seeds:**
- **FE + BGM:** AUC **0.882 ± 0.004**
- **FE + BGM + diaspora(LOO):** AUC **0.906 ± 0.004**

## 5) Choice-set robustness (Table 1)
Paste the full Table 1 below.

| Choice set regime | Specification | Mean AUC (sd) |
|---|---:|---:|
| Uniform negatives | FE + BGM | 0.882 (0.004) |
| Uniform negatives | FE + BGM + diaspora (LOO) | 0.906 (0.004) |
| Destination-neighbor hard negatives | FE + BGM + diaspora (LOO) | 0.872 (0.004) |
| Destination-neighbor hard negatives | + camp_pop | 0.872 (0.004) |
| Origin-diaspora hard negatives | FE only | 0.662 (0.010) |
| Origin-diaspora hard negatives | FE + diaspora (LOO) | 0.829 (0.005) |
| Origin-diaspora hard negatives | FE + BGM + diaspora (LOO) | 0.829 (0.005) |
| Origin-capital hard negatives | FE + BGM | 0.889 (0.002) |
| Origin-capital hard negatives | FE + BGM + diaspora (LOO) | 0.893 (0.004) |

**Mechanism takeaway:**  
- If alternatives are built from diaspora prominence, diaspora saturates signal (expected).  
- If alternatives are built from capital proximity, BGM dominates and diaspora adds incremental lift.

## 6) Interpretability: top coefficients (seed=2)
Paste top coefficients below (from exports).

Uniform negatives (FE+BGM+diaspora):
- log_diaspora_share_lo: 2.728
- log_diaspora_od_lo: 0.439
- dC: -0.054
- dS: 0.032

Origin-capital hard negatives (FE+BGM+diaspora):
- log_diaspora_share_lo: 3.199
- log_diaspora_od_lo: 0.589
- dC: 0.561
- dS: 0.171

## 7) One figure (ROC)
Insert one ROC curve figure here (recommended: uniform negatives, FE+BGM+diaspora, seed=2), with AUC annotated.

## 8) Governance and reproducibility stance
- This pack includes results and model specification.  
- **Raw microdata are not distributed.**  
- Full reproduction requires controlled access to microdata and the full pipeline.

**Code walkthrough + repo access available for collaborators (contact Arturo).**

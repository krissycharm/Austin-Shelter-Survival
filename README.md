# Which Shelter Animals Wait Longest?
**A Survival Analysis of 11+ Years of Austin Animal Center Data (2013–2026)**

🔗 **Live Interactive Report:** [krissycharm.github.io/Austin-Shelter-Survival](https://krissycharm.github.io/Austin-Shelter-Survival/)

---

## Overview
Animal shelters operate with finite kennel space, staff capacity, and medical budgets. Every additional day an animal remains in care increases operational costs and heightens behavioral and health risks.

Using **163,653 cleaned dog and cat stays** from the Austin Animal Center (October 2013 – May 2025)—plus **16,989 newer-schema stays** (May 2025 – November 2026) for out-of-schema validation—this project investigates which animal characteristics and intake conditions drive length of stay and shelter outcomes, and tests whether high-risk animals can be flagged at intake so staff can intervene early with foster placement, medical triage, or targeted adoption marketing.

---

## Key Findings

* **Dogs and cats exit the shelter through very different pathways:** Roughly half of both species are adopted (50% of dogs, 51% of cats), but **25% of dogs are reclaimed by their owners** (versus 4% of cats), while **38% of cats are transferred** to rescue partners (versus 22% of dogs).
* **Dogs are adopted faster initially, but cats catch up:** Within 7 days of arrival, 21.9% of dog stays end in adoption compared to 12.9% of cat stays. By day 90, cumulative adoption shares converge at 47.4% (cats) and 46.2% (dogs).
* **Arrival condition is the strongest predictor of mortality:** In the first two weeks, injured and sick animals die or are euthanized at **13x to 17x the rate** of animals arriving in normal condition. Nearly **28.7% of injured cats** and **21.9% of sick cats** die or are euthanized, compared to **12.4%** and **10.8%** of dogs, and under 3% of animals arriving in normal condition.
* **Age reshapes exit pathways:** Animals aged 8 weeks to 1 year are adopted fastest. Among dogs over 5 years old, lower adoption shares (24.4%) largely reflect high owner-reclaim rates (46.1%) rather than longer kennel waits alone. Most neonatal and nursing animals exit via partner transfers (68.9% of cats, 53.0% of dogs).
* **Predictor effects shift over the course of a stay:** Schoenfeld residual diagnostics confirm that nearly every intake factor changes in strength over time. Stratified piecewise Cox models show that puppies under 8 weeks adopt slowly in days 0–14 (`HR = 0.75`), but among those still waiting past day 56, they adopt nearly **3x faster** (`HR = 2.95`) once they reach adoptable age.
* **Intake risk scoring vs. simple triage rules:** A multivariable logistic regression model trained on pre-2022 intakes achieves an **AUC of 0.83** on unseen 2022–2025 stays (`n = 36,140`) when predicting death or euthanasia within 14 days. At the same alert volume (~14.3% of intakes), the model catches **66.1%** of poor outcomes versus **64.4%** caught by a zero-model rule flagging every injured or sick arrival.
* **Cross-schema generalization (May 2025 – November 2026):** Evaluated on `16,989` eligible companion-animal stays following Austin's May 2025 schema overhaul, the pre-2022 model maintains a **0.79 AUC** (`0.79` dogs, `0.78` cats) and continues outperforming condition-only triage (`27.1%` vs. `23.0%` recall; `15.9%` vs. `14.9%` precision) despite lower health-condition tagging rates in the newer intake system.

---

## Operational Takeaways

1. **Front-load feline medical triage:** Because injured and sick cats experience more than double the mortality rate of injured and sick dogs—and continue facing elevated risk past day 14—immediate veterinary assessment and specialized foster routing for compromised cats offer the highest life-saving leverage.
2. **Start with a simple condition-based rule before deploying complex scoring:** Flagging all injured and sick arrivals requires zero predictive infrastructure and captures nearly two-thirds of 14-day mortality events in standard intake workflows. Predictive scoring adds the most value when veterinary capacity is severely constrained and staff can only intervene on the top 5% highest-risk intakes (`18.5%` precision vs. `11.1%` base triage).
3. **Target kennel-space interventions to adult cats and post-hold dogs:** Senior dogs have low adoption shares largely because nearly half are reclaimed by owners within days; resources aimed at reducing long stays are better directed toward adult cats (1+ years) and unnamed stray dogs still waiting past day 14.

---

## Data Source & Scope
Data is sourced from the [City of Austin Open Data Portal](https://data.austintexas.gov/) (Austin Animal Center Intakes and Outcomes).

* **Primary Cohort (Legacy Schema):** October 1, 2013 – May 5, 2025 (`163,653` cleaned stays: `94,397` dogs and `69,256` cats, including `911` right-censored stays still in care at cutoff).
* **Out-of-Schema Validation Cohort:** May 2025 – November 14, 2026 (`19,547` raw intakes; `16,989` eligible live dog and cat stays with at least 14 days of follow-up).
* **Exclusions:** Non-companion species (wildlife, birds, livestock), owner-requested euthanasia intakes, `DOA`/`Dead` arrivals, and unmatched legacy records open for over one year (treated as missing discharge logs).

---

## Methodology

1. **Stay Reconstruction:** Matched each animal intake record chronologically to its subsequent outcome record to handle repeat visits (`cluster(animal_id)` used in survival models for robust standard errors).
2. **Competing-Risks Cumulative Incidence:** Estimated daily cumulative incidence curves (`tidycmprsk`) across all four major exit pathways—adoption, owner reclaim, rescue transfer, and death/euthanasia—accounting for right-censored stays still in care on May 5, 2025.
3. **Piecewise Cause-Specific Cox Models:** Fit separate models for dogs and cats using strictly intake-only predictors (`age_group`, `condition_model`, `sex_fixed`, `intake_type`, `has_own_name`, and `era`). Because Schoenfeld residual diagnostics indicated non-proportional hazards, models were stratified into three stay phases (`Days 0–14`, `Days 15–56`, and `Day 57 onward` for adoption; `Days 0–14` and `Day 15 onward` for mortality).
4. **Early Risk Flagging & Out-of-Schema Validation:** Trained a binary logistic regression model on pre-2022 stays to predict 14-day mortality/euthanasia, evaluating ROC AUC, decile calibration, and precision/recall against heuristic triage rules on both 2022–2025 legacy arrivals and mapped 2025–2026 new-schema arrivals.

---

## Tools & Packages
* **Language:** R
* **Key Libraries:** `tidyverse`, `janitor`, `lubridate`, `survival`, `ggsurvfit`, `tidycmprsk`, `broom`
* **Publishing:** Quarto (`analysis.qmd` + `custom.scss` rendered to self-contained HTML on GitHub Pages)

---

## Future Work
* Parse free-text breed and size descriptors to capture morphological effects on adoption speed.
* Explore multi-state competing-risks models to estimate expected kennel-days saved per early foster or medical intervention.
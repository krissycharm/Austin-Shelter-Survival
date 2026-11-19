# Which Shelter Animals Wait Longest?
**A Survival Analysis of 11+ Years of Austin Animal Center Data (2013–2025)**

## Overview
Animal shelters operate with finite kennel space, staff capacity, and medical budgets. Every additional day an animal remains in care increases operational costs and heightens behavioral and health risks. 

Using over **163,000 dog and cat stays** from the Austin Animal Center (October 2013 – May 2025), this project investigates which animal characteristics and intake conditions drive length of stay and shelter outcomes—and tests whether high-risk animals can be flagged at intake so staff can intervene early with foster placement, medical care, or targeted marketing.

---

## Key Findings

* **Dogs and cats exit the shelter through very different pathways:** Roughly half of both species are adopted (50% of dogs, 51% of cats), but **25% of dogs are reclaimed by their owners** (versus 4% of cats), while **38% of cats are transferred** to rescue partners (versus 22% of dogs).
* **Dogs are adopted faster initially, but cats catch up:** Within 7 days of arrival, 22% of dog stays end in adoption compared to 13% of cat stays. By day 90, cumulative adoption shares converge at ~46%–47%.
* **Arrival condition is the strongest predictor of mortality:** In the first two weeks, injured and sick animals die or are euthanized at **13x to 17x the rate** of animals arriving in normal condition. Nearly **29% of injured cats** and **22% of sick cats** die or are euthanized, compared to 12% and 11% of dogs, and under 3% of animals arriving in normal condition.
* **Age reshapes exit pathways:** Animals aged 8 weeks to 1 year are adopted fastest. Among dogs over 5 years old, low adoption shares (24%) are largely driven by high owner-reclaim rates (46%). Most neonatal and nursing animals exit via partner transfers (69% of cats, 53% of dogs).
* **Predictor effects shift over the course of a stay:** Proportional hazards tests confirm that almost every intake factor changes in strength over time. For example, kittens and puppies under 8 weeks are adopted slowly during their first two weeks, but among animals still waiting past day 56, puppies are adopted nearly **3x faster** than the reference group once they reach adoptable age.
* **Intake risk scoring vs. simple triage rules:** A logistic regression model trained on pre-2022 intakes achieves an **AUC of 0.83** on unseen 2022–2025 arrivals when predicting death or euthanasia within 14 days. At the same alert volume (~14% of intakes), the model catches **66.1%** of poor outcomes compared to **64.4%** caught by a simple rule flagging all injured or sick animals—demonstrating that while intake data ranks risk well, medical condition at arrival captures most of the actionable signal.

---

## Data Source & Scope
Data is sourced from the [City of Austin Open Data Portal](https://data.austintexas.gov/) (Austin Animal Center Intakes and Outcomes).

* **Primary Analysis Window:** October 1, 2013 – May 5, 2025 (`163,653` cleaned stays: `94,397` dogs and `69,256` cats, including `911` right-censored stays still in care at cutoff).
* **Holdout Dataset (Schema Change):** May 2025 – November 2026 (`~19.5k` rows). Because the City of Austin updated its schema and animal ID format in May 2025, newer records are separated for out-of-sample validation.
* **Exclusions:** Non-companion species (wildlife, birds, livestock), owner-requested euthanasia intakes, and unmatched records open for over one year (treated as missing outcome records).

---

## Methodology

1. **Stay Reconstruction:** Matched each animal intake record to its chronologically subsequent outcome record to handle repeat visits (`animal_id` clustered in modeling).
2. **Cumulative Incidence & Survival Curves:** Estimated time-to-outcome curves accounting for right-censored stays still in the shelter on May 5, 2025.
3. **Time-Varying Cause-Specific Cox Models:** Fit separate models for dogs and cats using only features known at intake (`age_group`, `condition`, `sex/spay-neuter status`, `intake_type`, `has_own_name`, and `pre/post-2020 era`). Because Schoenfeld residual diagnostics indicated non-proportional hazards, models were stratified into three stay phases:
   * **Phase 1:** Days 0–14
   * **Phase 2:** Days 15–56
   * **Phase 3:** Day 57 onward
4. **Early Risk Flagging:** Trained a binary logistic regression model on pre-2022 stays to predict 14-day mortality/euthanasia, evaluating calibration, decile lift, and precision/recall against heuristic triage baselines on 2022–2025 stays.

---


# NBA Pre-Game Fatigue Risk Modeling

## Overview

This project builds a **pre-game NBA fatigue risk model** using **publicly available data**.  
The objective is to quantify how **cumulative workload and schedule density** influence a player’s likelihood of **underperforming relative to their own baseline**, using only information available **before tipoff**.

Rather than attempting to predict raw performance directly—which is extremely noisy at the single-game level—the model focuses on **risk stratification**: identifying contexts where fatigue-related underperformance is more likely.

---

## Key Questions

- How does rolling workload (minutes over recent games) affect efficiency risk?
- Does schedule density (games played in short time windows) compound fatigue effects?
- Can fatigue risk be estimated pre-game without leaking same-game information?
- What are the practical limits of fatigue modeling with public NBA data?

---

## Data

**Source:** `nba_api` (NBA Stats API)  
**Granularity:** Player-game level

**Core inputs include:**
- Minutes played
- Field goal attempts, free throw attempts, turnovers
- Game dates and schedule context
- Team pace (optional contextual control)

All features are constructed using **only historical information** relative to each game (no look-ahead bias).

---

## Feature Engineering

### Pre-Game Workload & Schedule Features
- Rolling minutes (last 3 games)
- Rolling minutes (last 5 games)
- Games played in last 7 days
- Back-to-back indicator
- Days since last game
- Rolling usage proxy (shot attempts + free throws + turnovers per minute)
- Team pace (contextual control)

All features are **lagged**, ensuring they are available prior to tipoff.

---

## Target Definition

To isolate fatigue effects from player skill, performance is measured **relative to each player’s baseline**:

- True Shooting % (TS%) is computed from box score data.
- Each player’s prior baseline TS% is estimated using an expanding historical mean.
- **TS_DELTA = TS% − prior baseline TS%**

A binary fatigue outcome is defined as:
BAD_GAME = 1 if TS_DELTA ≤ −0.05


This framing removes player-specific efficiency differences and focuses on **context-driven underperformance risk**.

---

## Modeling Approach

### Primary Model
- **Model:** Logistic Regression
- **Purpose:** Pre-game fatigue risk classification

Logistic regression is used for:
- Stability and interpretability
- Robust ranking of risk
- Avoidance of overfitting in high-noise outcomes

### Evaluation Strategy
- Time-based train/test split (chronological)
- Metrics:
  - ROC AUC
  - Average Precision (PR AUC)

The emphasis is on **ranking and stratification**, not exact prediction.

---

## Results Summary

Typical model performance:
- ROC AUC ≈ 0.56–0.58
- Average Precision meaningfully above base rate

Key findings:
- Rolling workload over the last 3–5 games is the strongest fatigue signal
- Schedule density amplifies workload effects
- Back-to-backs matter primarily through cumulative load
- Fatigue risk increases at moderate-to-high workload levels and plateaus at extremes due to coaching intervention and player heterogeneity

This behavior is consistent with real-world NBA decision-making.

---

## Visualization Highlights

The project includes fatigue risk curves illustrating:
- Risk vs rolling minutes
- Interaction between workload and back-to-back games
- Risk vs games played in the last 7 days

These plots highlight **threshold effects** rather than deterministic predictions.

---

## Interpretation & Limitations

### What This Model Is
- A **relative fatigue risk ranking tool**
- Designed for **pre-game decision support**
- Focused on interpretability and realism

### What This Model Is Not
- A precise predictor of single-game efficiency
- A causal physiological fatigue estimator
- A replacement for tracking, biometric, or travel data

Single-game TS% is extremely noisy; even internal team models with richer data rarely achieve high predictive accuracy.

---

## Practical Use Case

The model can be used to:
- Flag players in elevated fatigue contexts
- Compare relative fatigue risk across upcoming games
- Inform rotation or rest considerations

Example framing:
> “This game presents higher fatigue-related underperformance risk relative to recent baseline.”

---

## Why This Project Matters

This project demonstrates:
- Proper handling of temporal leakage
- Awareness of selection bias (minutes as endogenous)
- Realistic expectations for sports analytics performance
- Translation of noisy models into actionable insight

The workflow mirrors how fatigue models are applied in professional basketball environments using public data constraints.

---

## Future Improvements

- Train on multiple seasons to reduce variance
- Use rolling performance targets (e.g., 3-game averages)
- Incorporate opponent defensive context
- Extend to shot-level modeling for finer signal

---

## Tech Stack

- Python
- pandas, numpy
- scikit-learn
- nba_api
- matplotlib


# Airline Customer Satisfaction: Touchpoint Drivers & Segment Analysis

## Question

Which service touchpoints most affect customer satisfaction, and how does that differ across passenger segments (class, travel type, loyalty)? Built as practice for a CX/Insights analyst application — this uses a public Kaggle dataset, not real airline data.

## Data

[Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction) (Kaggle), combined train + test files: 129,880 survey responses covering demographics, travel details, 14 service ratings (0–5 scale), delay times, and an overall satisfaction label ("satisfied" / "neutral or dissatisfied").

The dataset doesn't name a specific airline. This is satisfaction, not NPS or Customer Effort Score — the data doesn't have either.

## Method

**Cleaning:**
- Combined train and test files into one dataset (no model is being deployed, so the train/test split serves no purpose here).
- A rating of 0 on any of the 14 service touchpoints was treated as "not applicable" rather than a genuine low score, and excluded from that touchpoint's calculations only (the passenger's row is kept; other ratings from that passenger are unaffected). Touchpoints with the highest share of 0s (Departure/Arrival time convenient, Ease of Online booking, Inflight wifi) are also the ones where this distinction matters most.
- Dropped 393 rows (0.3%) missing Arrival Delay — too small a share to justify imputing.
- Final analysis dataset: 129,487 rows.

**Touchpoint satisfaction:** top-2-box score (share of 4s and 5s) per touchpoint, using only the valid ratings for that touchpoint.

**Drivers:** logistic regression predicting overall satisfaction from all 14 touchpoint ratings, arrival delay, and controls for class, travel type, and loyalty. Reported as marginal effects — percentage-point change in probability of satisfaction per 1-point increase in a rating — not raw log-odds, because log-odds isn't something a manager can act on. Model uses 119,204 rows (the ones with a complete set of touchpoint ratings). This subset skews slightly Economy-heavy, since Economy passengers are more likely to have at least one "not applicable" rating.

Departure Delay was dropped from the final model — it was highly collinear with Arrival Delay (VIF ≈ 14.6), which was distorting both coefficients. Arrival Delay alone is used as the delay measure.

## Key findings

**Digital touchpoints are the strongest levers, and currently the weakest-performing ones.** Online boarding (+6.8 percentage points per rating point), inflight wifi (+5.8pp), and online booking (+2.9pp) are the three strongest drivers of satisfaction — and also three of the four lowest-rated touchpoints (31–51% top-2-box). Cabin-comfort touchpoints (seat comfort, food, entertainment) barely move satisfaction by comparison.

**Two touchpoints show a negative coefficient that isn't a genuine negative effect.** "Departure/Arrival time convenient" and "Gate location" both come out negative in the model. For time-convenience, that's a confound: personal travellers rate this touchpoint more generously on average but are far less satisfied overall for unrelated reasons, so the model wrongly credits some of their dissatisfaction to this touchpoint. Gate location's negative effect doesn't have that same explanation and is small anyway — it looks like a genuinely negligible driver, not a data artifact.

**Personal travel, not class, is the biggest satisfaction gap.** Personal travellers are satisfied 10.1% of the time versus 58.4% for business travellers — a gap that barely narrows even for personal travellers flying Business class (11.7%, vs 8.7% in Economy). This gap persists in the regression after controlling for every touchpoint rating, meaning it isn't explained by worse service — something about the personal-travel experience itself (price sensitivity, expectations, trip purpose) drives it, and this dataset can't say what.

**Loyalty and travel type mostly describe the same passengers, not two separate populations.** 99% of "Disloyal" customers in this dataset are business travellers — so a loyalty-focused fix and a business-traveller-focused fix would likely reach the same people. Loyalty should be read within travel type, not as an independent segment.

**Delay hurts business travellers' satisfaction more than personal travellers'** (a 14-point drop from no-delay to 60+ minutes late, vs an 8-point drop for personal travellers) — mostly because personal travellers start from such a low satisfaction base that there's little room left to fall. Delay-recovery resources protect more satisfaction when aimed at business travellers.

## Recommendations

1. Fix the digital front door first — online boarding and inflight wifi reliability are the highest-leverage, currently-weakest touchpoints.
2. Investigate personal travel as its own problem, not a class problem — the driver model has hit its ceiling here; a qualitative follow-up (exit interviews, a targeted survey question) is the logical next step.
3. Report loyalty satisfaction within travel type, not as a separate KPI, given the near-total overlap between "disloyal" and "business traveller" in this data.
4. Prioritize delay-recovery resources for business travellers if resources are limited.
5. Treat Economy/Eco Plus satisfaction as largely a digital-touchpoint and passenger-mix problem, not a cabin-comfort problem — the touchpoint scores are similar across both classes.

## Repo contents

- `notebooks/` — Python cleaning and analysis (pandas, statsmodels)
- `dashboard/` — Power BI file (.pbix) and PDF export, 3 pages: Overview, Drivers, Segments
- `data/processed/` — cleaned dataset and driver-effects table used by the dashboard (raw Kaggle files are not included — download from the link above)

## Limitations

- Pseudo R² is 0.64, which is high for a real-world CX model. That's probably because the "satisfaction" label was partly derived from these same ratings when the dataset was built, rather than coming from something fully independent like "would fly again." Treat this as a demonstration of method, not a benchmark for real-world predictive strength.
- The rows excluded from the driver model (8% of the cleaned data, for having at least one missing touchpoint rating) skew slightly toward Economy passengers, so the driver model's sample isn't perfectly representative of the full dataset.

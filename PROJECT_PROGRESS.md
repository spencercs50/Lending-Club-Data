# Lending Club Project Progress Log

**Update:** June 25, 2026

### Current Phase
- Phase 1: Univariate Analysis (In Progress)

### Work Completed Today
- Ran Chi-square tests on categorical variables (population + sample)
- Ran Pearson correlations on numeric variables
- Created `defaulted_tf` and `delinquency_tf` target variables
- Investigated missing `dti` values

### Key Decisions
- Decided not to fill missing `dti` with 0
- Chose to analyze full `dfa_data` instead of just samples
- Keeping both delinquency and default targets for now

### Next Steps
- Finish interpreting numeric correlations
- Start bivariate analysis (interactions)
- Begin feature selection for modeling

### Insights So Far
- `sub_grade` is by far the strongest predictor
- `inq_last_6mths` and `loan_amnt` look promising among numeric variables

### June 28, 2026 – Univariate Analysis & Feature Selection Reflections

Today I spent time digging into the Pearson correlations and Chi-square results from both the full population and the 500k sample. I focused less on just collecting the numbers and more on trying to understand what they actually mean in a real lending context.

**Key Questions I Mullled Over:**
- How much of `int_rate`’s strong correlation (≈0.20) is truly predictive vs it simply reflecting Lending Club’s own internal risk grading? Since interest rate is partly an *output* of their underwriting process, there’s a real risk of information leakage.
- Are some statistically significant variables (like `dti`) actually too weak in practice to carry meaningful weight in the final model?
- How do I balance raw statistical strength with business realism, and avoid building something that just copies Lending Club’s past (flawed) decisions?

**Main Conclusions So Far:**
- Strongest signals: `sub_grade`, `inq_last_6mths`, `int_rate`, and FICO range variables.
- I will likely drop `dti`, `annual_inc`, `revol_bal`, and `revol_util`. Their correlations are too low to be primary features.
- I plan to train multiple model versions (with and without `int_rate`) to better understand its real contribution versus the risk of circularity.

**What I Learned Today:**
- Extremely small p-values (showing as 0.0) are common with large datasets. They confirm statistical significance, but the correlation coefficient is what tells you the actual strength of the relationship.
- I'm repeatedly asking "Would this variable have been available at the time of the lending decision, or is it contaminated?”

This session reinforced the importance of thinking critically about data leakage and business context rather than just chasing higher numbers. I feel like I’m building a more thoughtful and defensible approach.



Questions I now need to answer for feature selection, and justifying why I am keeping or removing features from the model later on.

#### Statistical Strength:
How strong is the correlation / association with the target (defaulted_tf or delinquency_tf)?

- **Categorical variables**: `sub_grade` shows the strongest association with both defaults and delinquencies by a wide margin.

- **Numeric variables**: The strongest relationships are:

    **Population Defaults**: `inq_last_6mths` (0.083), `pub_rec` (0.032), `loan_amnt` (0.021)  
    **Population Delinquency**: `inq_last_6mths` (0.082), `pub_rec` (0.033), `loan_amnt` (0.027)  
    **Population Adjusted**: `int_rate` (0.199), `term` (0.084), `revol_util` (0.066)
    **Sample Adjusted**: `int_rate` (0.238 / 0.245), `term` (0.097 / 0.107), `dti` (0.078)

The first list of variables I'll use for a first ML algorithm will be:
`sub_grade`
`inq_last_6mths`
`loan_amnt`
`int_rate`
`term`
`revol_util`

I may decide to use fico ranges (`fico_range_low`, `fico_range_high`). They have a strong negative correlation.

**Are the relationships statistically significant?**  
Yes. Even though many of the correlation values are comparatively lower than my past class exercises, all tested variables show extremely small p-values (essentially 0.0), meaning the relationships are statistically significant.

**Does it still hold when comparing full population vs sample?**  
Yes, the patterns are highly consistent. The top variables (`int_rate`, `inq_last_6mths`, `term`, `sub_grade`) remain strong across both the full population and the 500k sample. This gives me confidence that the signals are stable and not just artifacts of sampling.

#### Business / Domain Relevance:
**How do the chosen variables make intuitive sense from a lending / credit risk perspective?**
`sub_grade` is one of the strongest categorical predictors. It directly reflects Lending Club’s own risk assessment of the borrower, so it makes sense that it would be strongly associated with default and delinquency outcomes.

`revol_util` - A very useful indicator for a few reasons. It is one of the earliest signals of higher risk borrowing behavior prior to any risk scores, inquiries, grades, etc. High utilization often indicates financial stress or living close to the edge, while low utilization suggests more breathing room. My intuition is that the high-utilization cases far outweigh the rare “excellent manager of credit” cases.

`loan_amnt` - Helps reveal LC's risk appetite in hard dollar amounts. Since I don't have direct insight into LC's capital position or policies, loan amount will act as a proxy signal for their willingness to take risk. 

`inq_last_6mths` - Strong behavioral signal. It captures recent credit seeking activity from borrowers. If my MLA underperforms because of inq_last_6mths, it should still be a valuable indicator about LC's performance as well. 

`int_rate` - Interest rates were a tougher decision that I originally assumed. Statistically it is one of the strongest predictors, but I’m cautious because it is partly an *output* of LC's underwriting process. There’s a risk it could introduce leakage or cause the model to simply learn LC’s past grading decisions rather than discover independent risk signals. I plan to test versions both with and without it.

`term` - In my mind, the term variable is a signal for the speed at which LC was planning to recover money, pressure they wanted to levy against borrowers. Longer terms (60mo) stretch repayment, lower monthly payments for the borrower, but increase overall risk for LC due to the loan being outstanding for longer time periods.



**Would this information realistically be available at the time of the lending decision?**
Most of the variables I selected are **pre-decision** data points — meaning they would have been available to Lending Club at the time they evaluated the borrower’s application.

- `sub_grade`, `inq_last_6mths`, `fico_range_high/low`, and `revol_util` are all pulled from the borrower’s credit profile and application data **before** approval.
- `loan_amnt` is the amount requested by the borrower, so it’s known upfront.
- `term` is part of the loan offer. Decided before funding.
- `int_rate` is mostly set before the loan is granted, although minor adjustments can happen right before funding.

The nuance is that the entire accepted loans dataset (`dfa_data`) is inherently post-decision and only contains loans that at one point or another, were approved. 

This gives me reasonable confidence that the features are appropriate for predictive modeling, while still acknowledging the post-approval nature of the final dataset.


#### Data Quality & Reliability:

**Are the variables stable over time, or does it behave very differently by year?**
In looking at the default/delinquency rates by year, there is some variation in terms of rate change. But the overall trend is continually increased and then plateaus moderately. Then, defaults/delinquencies begin to trail off in 2016 onward. 

However, that trend will likely not signal easy predictability, or stability for LC. In my year_summary, even though I created delinquencies and defaults as overlapping variables, there is a significant uptick with late payments of shifting time windows. So although the numbers themselves follow a trend, it does not spell stability for LC. 

For modeling, it means I cannot assume a variable's predictive power will remain the same across the entire data set. 

### 7.9.2026 My Own Prediction on Lending Club's Underwriting Issues

My current hypothesis is that while my first model is underperforming on raw accuracy metrics, the specific “gap” it reveals — particularly in the false positives — is actually highlighting a major weakness in Lending Club’s underwriting. 

I believe we’re seeing evidence that Lending Club was routinely approving higher-risk loans (D-grade and below, high revol_util, recent inquiries, elevated interest rates) that carried more risk than their eventual performance suggested. This pattern appears to have been common across multiple years, pointing to an overly aggressive lending strategy that prioritized volume over tighter risk controls.

**What I would have done differently (as a business strategy):**

Lending Club should have adopted a much more disciplined, risk-first approach, especially after the 2008–2009 period. Instead of chasing rapid growth through looser standards, I would have:

- Significantly tightened credit criteria starting around 2010, rejecting a higher percentage of subprime applications (D-grade and below).
- Raised interest rates more aggressively on borrowers to better price the risk.
- Been quicker to move struggling accounts into collections or charge-off earlier rather than letting late payments drag on for months.
- Implemented stricter ongoing monitoring (more inquiries, utilization limits, etc.) to catch problems before they snowballed.

In short, they got caught up in growth-at-all-costs and paid for it. A more conservative strategy like what traditional banks (Chase, JPMorgan, etc.) typically follow would have likely produced fewer bad loans and better long-term stability, even if it meant slower short-term growth.













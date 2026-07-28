# CTR-OSF-Data-Dashboard
Dashboard visualization and experimental design of OSF Experimental Data testing how varieties of certain headlines warrant more webpage clicks

# Do Question Marks Hurt Click-Through Rates? An A/B Test Analysis of 100K+ Upworthy Headlines

## Overview
A causal analysis of the [Upworthy Research Archive](https://osf.io/jd64p/) — 
105,551 headline variants across 22,743 randomized A/B tests run by Upworthy 
between 2013–2015. This project combines SQL, NLP feature engineering, and 
statistical hypothesis testing to identify which headline characteristics 
actually drove clicks — and which "significant" effects were too small to matter.

**[→ View the interactive dashboard on Tableau Public](https://public.tableau.com/app/profile/dash.kwan/viz/CTR-OSFUpworthyResearch/Dashboard1?publish=yes)**

![Dashboard screenshot](CTR_OSF_dashboard.png)

## The Question
Does the *language* of a headline, not just the story, causally affect 
click-through rate? And can standard statistical significance mislead you 
about which effects are worth acting on?

## Methodology

### Data
- Source: Upworthy Research Archive (confirmatory split), 105,551 packages 
  across 22,743 tests, Jan 2013–Apr 2015
- Each test = multiple headline variants for the same article, randomly 
  shown to different visitors — real randomization, not observational data

### Defining Treatment/Control
The dataset has no built-in treatment/control label — headlines within a 
test just compete against each other. Rather than an arbitrary split (e.g., 
"first created = control"), treatment/control was defined **per engineered 
NLP feature**:
- `has_question` — presence of "?"
- `has_number` — presence of a digit
- `sentiment` — VADER compound score, bucketed negative/neutral/positive

This makes each comparison a real hypothesis test ("does X feature affect 
CTR?") rather than an arbitrary label, while still inheriting the real 
randomization within each test.

### Pipeline
1. **SQL (SQLite)** — raw packages loaded into a `packages` table; NLP 
   features engineered separately into `headline_features`, joined on 
   `package_id`. Aggregation, stratification, and window-function ranking 
   done in SQL, not pandas.
2. **NLP (Python, VADER)** — sentiment scoring and regex-based feature 
   extraction on headline text
3. **Statistics (Python, statsmodels/scipy)** — two-proportion z-tests, 
   effect size, and 95% confidence intervals for each feature — not just 
   p-values
4. **Visualization (Tableau)** — trend and comparison dashboards

## Key Findings

| Feature comparison | p-value | Absolute Δ CTR | Relative lift | 
|---|---|---|---|
| No question mark vs. question mark | <0.001 | 0.205 pts | **15.1%** |
| Negative vs. positive sentiment | <0.001 | 0.119 pts | **8.0%** |
| Has number vs. no number | <0.001 | 0.015 pts | 1.0% |

**All three effects were statistically significant** — expected, given 
sample sizes in the hundreds of millions of impressions gave enormous 
statistical power. But effect size tells a different story: **question 
marks in headlines were associated with a 15% relative drop in CTR**, 
a meaningful and actionable effect. Presence of a number, while 
technically significant, had a negligible ~1% effect — a good example 
of statistical significance without practical significance.

The has_question effect held up consistently when stratified by 
sentiment (negative, neutral, and positive headlines all showed the 
same direction of effect), strengthening confidence this isn't an 
artifact of headline tone.

**Positive headlines received the most impressions of any sentiment 
group, despite having the lowest CTR** — a case where volume and 
quality of engagement diverge.

## Recommendation
The strongest, most actionable finding here is straightforward: **avoid 
question-mark framing in headlines.** A 15% relative lift in CTR is large 
enough to justify a real editorial guideline, not just a footnote — and it 
held up consistently across negative, neutral, and positive headlines, 
suggesting it's a robust effect rather than a quirk of a few viral outliers.

Sentiment is a secondary, more nuanced lever. Negative-leaning headlines 
outperformed positive ones by about 8% in CTR — but positive headlines 
still drove the most total impressions of any group. That split matters 
for how a team acts on it: if the goal is maximizing *reach*, positive 
framing already works well; if the goal is maximizing *engagement rate* 
per headline shown, leaning negative or urgent is the better lever. This 
is a case where the "best" answer depends on which metric a team is 
actually optimizing for — worth surfacing explicitly rather than picking 
one number to report.

The presence of numbers in a headline is not a reliable lever. It cleared 
statistical significance only because of the dataset's enormous sample 
size — a ~1% relative lift is not something I'd recommend building an 
editorial rule around.

More broadly: this project is a useful reminder that "statistically 
significant" and "worth acting on" are different questions, especially 
at scale. All three features here were significant by p-value; only two 
were actually meaningful in size.

## Limitations & Next Steps

**Limitations**
- Treatment/control was defined by engineered headline features rather 
  than a built-in experimental label, since the raw data doesn't 
  distinguish "treatment" from "control" within a test. This is a 
  defensible substitute, but it means cross-test comparisons (e.g., 
  comparing a negative headline in one test to a positive headline in a 
  different test) are more observational than a true randomized 
  comparison, even though within-test comparisons retain real 
  randomization.
- Sentiment was scored using VADER, a lexicon-based tool tuned for 
  general text and social media — it may not perfectly capture the 
  specific emotional register Upworthy headlines were written in (e.g., 
  moral outrage, inspiration), which a more specialized or fine-tuned 
  model might resolve.
- The dataset spans 2013–2015; headline strategies and reader behavior 
  online have changed substantially since then (mobile-first reading, 
  different platform algorithms, general "clickbait fatigue"), so these 
  specific effect sizes shouldn't be assumed to generalize to today's 
  media environment without re-testing.
- This analysis used the confirmatory data split only; results were not 
  cross-validated against the exploratory or holdout splits, which the 
  original dataset was explicitly designed to support.

**Next Steps**
- Extend the NLP feature set beyond question marks/numbers/sentiment — 
  headline length, presence of a named person or organization, 
  first-person vs. second-person phrasing, and use of superlatives are 
  all plausible additional levers worth testing with the same pipeline.
- Validate the has_question and sentiment findings against the 
  exploratory or holdout splits to confirm they aren't specific to the 
  confirmatory subset.
- A natural extension would be a predictive model (e.g., logistic 
  regression on the engineered features) to estimate CTR for a new, 
  unseen headline before publication — treated as a genuinely separate 
  follow-on project rather than an extension of this one, since 
  predictive modeling and causal inference call for different design 
  and evaluation standards.

## Tools
Python (pandas, VADER, statsmodels, scipy) · SQLite · Tableau

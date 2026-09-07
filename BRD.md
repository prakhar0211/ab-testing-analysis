# Business Requirements Document: A/B Testing Evaluation Framework

## Business Problem

Teams running product experiments often make ship/no-ship decisions based on
a single p-value, without verifying that the underlying randomization was
sound, without distinguishing statistical significance from practical
significance, and without a repeatable process for generating stakeholder-
ready summaries. This creates two risks: shipping changes based on false
positives (or broken experiment mechanics), and spending unnecessary manual
effort writing up results for every test that concludes.

## Business Objective

Provide a repeatable, defensible framework for evaluating A/B test results
that:
1. Verifies the experiment's randomization was valid before trusting any
   result (Sample Ratio Mismatch check)
2. Produces a statistically sound significance test and confidence interval,
   not just a pass/fail p-value
3. Translates the statistical output into a plain-language, stakeholder-ready
   recommendation automatically, reducing manual reporting time
4. Alerts the relevant channel/team as soon as a result is available, with
   the recommendation already framed correctly (ship / do not ship)

## Scope — In

- A hypothesis-testing pipeline covering: SRM check (chi-square goodness of
  fit), pooled two-proportion Z-test, unpooled confidence interval on the
  effect size
- Validation of the pipeline against a synthetic dataset with a known
  ground-truth effect, before trusting it on real data
- Application of the validated pipeline to a real-world experiment dataset
- Automated generation of an executive summary from the statistical output,
  via the Claude API
- Automated delivery of that summary to a Slack channel, with the
  alert style (significant vs. non-significant) determined by the actual
  result

## Scope — Out

- Sequential/continuous monitoring of a test while it is still running
  (this framework evaluates a test at a defined endpoint, not mid-flight)
- Multi-variant (A/B/n) testing beyond a single control/treatment comparison
- Automatic rollout/deployment of the winning variant (the framework
  produces a recommendation; a human decides whether to act on it)
- Segment-level (e.g., device type, new vs. returning user) breakdowns of
  the result

---

# User Story

**As a** product or business stakeholder awaiting an experiment result,
**I want** an automated, statistically validated summary of whether a
tested change should ship,
**so that** I can make a fast, defensible decision without manually
re-deriving the statistics or waiting on an analyst to write up the result.

## Acceptance Criteria

- The system verifies the experiment's group split matches the intended
  allocation (SRM check) before producing any recommendation. If the SRM
  check fails, no ship/no-ship recommendation is generated, and this is
  flagged explicitly.
- The system computes a two-proportion Z-test and reports the p-value.
- The system computes a 95% confidence interval on the difference in
  conversion rates, using an unpooled standard error, separate from the
  pooled standard error used for the significance test.
- If p < 0.05, the system generates a "significant result" summary and
  alert (visually distinct — e.g., a warning-style header/emoji in the
  Slack message).
- If p ≥ 0.05, the system generates a "non-significant result" summary and
  alert, explicitly recommending against shipping, rather than omitting a
  recommendation or defaulting to a positive framing.
- The generated summary states the observed lift, the p-value, and the
  confidence interval in plain language, not just as raw statistical output.
- The pipeline has been verified to produce correct outputs on both a
  significant and a non-significant real input before being considered
  production-ready.

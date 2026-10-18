# Black-Box Optimisation Capstone

Documentation:
- [Datasheet](DATASHEET.md): the query history and function evaluations
- [Model card](MODEL_CARD.md): the optimisation approach

## Project overview

Eight unknown functions, 2D to 8D. One query per function per week, 13 total. Goal: find the maximum. No formulas, no gradients, no retries — every query has to be worth a week.

This is what real optimisation usually looks like: tuning a model that takes days to train, running an experiment that costs actual money, testing a recipe you can only bake once. The skill being trained is not "run more experiments" but "decide what is worth measuring next" — and defending that decision before knowing how it turns out. That's the transferable part: surrogate modelling, reasoning under uncertainty, and making expensive decisions with sparse data.

## Inputs and outputs

Inputs are vectors in the unit cube, dimensionality 2 to 8 depending on the function. Each function starts with an initial dataset (`initial_inputs.npy`, `initial_outputs.npy`, between 10 and 40 points) that grows by one observation per week.

Output is a single scalar per query — no gradient, no noise estimate, no diagnostics. Scales vary wildly between functions, from values clustered near zero to four-digit outputs.

Submissions are hyphen-separated coordinates with six decimals: `0.123456-0.654321`.

## Challenge objectives

The objective is to maximise all eight functions within the given query budget. The main constraints are the limit of 13 queries per function, a response delay of one week per query, the strictly sequential nature of the process, and the unknown structure of the functions. The functions differ considerably in character. Some are smooth and unimodal, some are noisy, some are high-dimensional with sparse data coverage, and one is almost entirely flat with the signal confined to a small region. A uniform strategy across all functions would waste queries. An important part of the challenge is therefore to assess, per function and per week, what kind of problem is actually being solved, and to allocate the queries accordingly.

## Technical approach

Bayesian optimisation, built on Ax and BoTorch. A Gaussian Process surrogate per function, refit from scratch on the full history every week. On top of that, several acquisition functions run side by side each round — from information-seeking to greedy — and their proposals are compared before anything is submitted.

Which proposal wins is a per-function decision, not a global policy, and it shifts as the data grows. The weekly loop: merge new results, refit, generate proposals, inspect (posterior mean and uncertainty plots, expected values at candidate points), decide, submit. Settings are tuned per function and revisited weekly rather than fixed globally.

For the mostly-flat function, standard BO has nothing to work with — the surrogate is blind on a zero landscape. That one runs a plain space-filling search instead until it produces a signal worth modelling.

Decisions and their reasons are kept in a running journal — the short version is in the weekly discussion posts, the long version is not in this repo on purpose.

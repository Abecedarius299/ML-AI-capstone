# Black-Box Optimisation Capstone

Capstone project of the Imperial College Professional Certificate in Machine Learning and AI. All 13 rounds are submitted. Only the final (round 13) submission is scored.

## In short

Imagine eight machines, each with a few unknown dials. Once a week I can set the dials of each machine and get back one number, and the goal is to make that number as high as possible. With only 13 tries per machine, every try has to count. I used a statistical model that estimates from the results so far where high values might be, and how unsure it is. Early tries were used to learn, later ones to improve on the best. All eight machines ended above their starting data, and the hardest one, almost zero everywhere, revealed its peak in week 10.

## Documentation

- [Logbook](LOGBOOK.md): every query per function and round, with the method, the result and the reasoning
- [Datasheet](DATASHEET.md): the query history and function evaluations
- [Model card](MODEL_CARD.md): the optimisation approach

## Repository structure

```
README.md
LOGBOOK.md
DATASHEET.md
MODEL_CARD.md
requirements.txt
data/
  initial_data/function_1 ... function_8/   initial inputs and outputs provided by the course
optimisation_sorted.ipynb   refits the models and proposes the next query per function
analysis.ipynb              plots, checks and benchmarking
```

The **optimisation notebook** ([optimisation_sorted.ipynb](optimisation_sorted.ipynb)) is the weekly loop: it loads the data, fits a Gaussian Process per function, generates the candidate points of the different acquisition functions (KG, EI, UCB and the geometric search) and prints the query strings for the portal. It also contains the Function 1 shape models. The final cell picks the best observed point per function, which was the round 13 submission.

The **analysis notebook** ([analysis.ipynb](analysis.ipynb)) looks at the data: it prints the readings per function, plots Function 1 (including an interpolation of the exponent of its readings), and contains the benchmarking, i.e. the comparison of KG and EI and of different acquisition schedules on standard test functions.

Both notebooks load the initial data from `data/initial_data/` and add the weekly results in their data cell, one block per round. To run them, install the packages in `requirements.txt`. Note that the acquisition optimisation is partly random and no seeds were fixed, so a rerun can propose slightly different points than the ones submitted. The submitted points are in the logbook.

## Project overview

Eight unknown functions, 2D to 8D. One query per function per week, 13 in total. Goal: find the maximum. No formulas, no gradients, no retries: every query has to be worth a week.

This is what real optimisation usually looks like: tuning a model that takes days to train, running an experiment that costs actual money, testing a recipe you can only bake once. The skill being trained is not "run more experiments" but "decide what is worth measuring next", and defending that decision before knowing how it turns out. That is the transferable part: surrogate modelling, reasoning under uncertainty, and making expensive decisions with sparse data.

## Inputs and outputs

Inputs are vectors in the unit cube, dimensionality 2 to 8 depending on the function. Each function starts with an initial data set (`initial_inputs.npy`, `initial_outputs.npy`, between 10 and 40 points) that grows by one observation per week.

The output is a single scalar per query: no gradient, no noise estimate, no diagnostics. Scales vary wildly between functions, from values clustered near zero to four-digit outputs.

Submissions are hyphen-separated coordinates with six decimals: `0.123456-0.654321`.

## Challenge objectives

The objective is to maximise all eight functions within the given query budget. The main constraints are the limit of 13 queries per function, a response delay of one week per query, the strictly sequential nature of the process, and the unknown structure of the functions. The functions differ considerably in character. Some are smooth and unimodal, some are noisy, some are high-dimensional with sparse data coverage, and one is almost entirely flat with the signal confined to a small region. A uniform strategy across all functions would waste queries. An important part of the challenge is therefore to assess, per function and per week, what kind of problem is actually being solved, and to allocate the queries accordingly.

## Technical approach

Bayesian optimisation, built on Ax and BoTorch. A Gaussian Process surrogate per function, refit from scratch on the full history every week. On top of that, several acquisition functions run side by side each round, from information-seeking to greedy, and their proposals are compared before anything is submitted.

Which proposal wins is a per-function decision, not a global policy, and it shifts as the data grows. The weekly loop: merge new results, refit, generate proposals, inspect (posterior mean and uncertainty plots, expected values at candidate points), decide, submit. Settings are tuned per function and revisited weekly rather than fixed globally.

For the mostly flat function (Function 1), standard BO has nothing to work with: the surrogate is blind on a zero landscape. It therefore started with a plain space-filling search. Once the tiny readings showed a trend in their exponent, I fitted simple shape models on the exponents and triangulated the peak, which returned 1.45 in round 10 after nine rounds of values close to zero.

As only the final submission is scored, rounds 11 and 12 were used to gather information, and round 13 resubmitted the best observed point of each function.

## Results

| Function | Dimensions | Best initial | Best found | Round |
|---|---|---|---|---|
| 1 | 2 | 7.7e-16 | 1.4497 | 10 |
| 2 | 2 | 0.6112 | 0.6128 | 4 |
| 3 | 3 | -0.0348 | -0.0010 | 6 |
| 4 | 4 | -4.0255 | 0.5518 | 5 |
| 5 | 4 | 1088.86 | 8662.41 | 6 |
| 6 | 5 | -0.7143 | -0.1771 | 4 |
| 7 | 6 | 1.3650 | 3.3224 | 4 |
| 8 | 8 | 9.5985 | 9.9896 | 9 |

These are the best points found in the explored regions, not proven global maxima. Function 2 is noisy, so its resubmitted point may return a different value.

## Decisions

The decisions and their reasons are in the [logbook](LOGBOOK.md), one table per function. The short version is also in the weekly discussion posts. My private working journal is not in this repo.

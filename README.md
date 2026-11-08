# Black-Box Optimisation Capstone

Capstone project of the Imperial College Professional Certificate in Machine Learning and AI. All 13 rounds are submitted. Only the final (round 13) submission is scored.

Documentation:

- [Logbook](LOGBOOK.md): every query per function and round, with the method, the result and the reasoning
- [Datasheet](DATASHEET.md): the data, per function, and how it was handled
- [Model card](MODEL_CARD.md): the optimisation approach, its performance, limitations and trade-offs

## Non-technical explanation of the project

Imagine eight machines, each with a few unknown dials. Once a week I can set the dials of each machine and get back one number, and the goal is to make that number as high as possible. With only 13 tries per machine, every try has to count. I used a statistical model that estimates from the results so far where high values might be, and how unsure it is. Early tries were used to learn, later ones to improve on the best. All eight machines ended above their starting data, and the hardest one, almost zero everywhere, revealed its peak in week 10.

## Data

The data was provided by the course through the capstone portal: eight unknown functions, 2D to 8D, each with a small initial data set (`initial_inputs.npy`, `initial_outputs.npy`, between 10 and 40 points). Every week I submitted one query per function and the portal returned the function value about a week later, so each function grew by 13 points over the project. No external data or sources were used.

Inputs are vectors in the unit cube. The output is a single scalar per query: no gradient, no noise estimate, no diagnostics. Scales vary wildly between functions, from values clustered near zero to four-digit outputs. Submissions are hyphen-separated coordinates with six decimals: `0.123456-0.654321`.

The initial data is in `data/initial_data/`. The weekly queries and results are in the data cell of both notebooks (one block per round) and in the [logbook](LOGBOOK.md). The data set is small (under 1 MB), so it is stored directly in this repository. Details per function are in the [datasheet](DATASHEET.md).

## Model

Bayesian optimisation, built on Ax and BoTorch. A Gaussian Process (GP) surrogate per function, refit from scratch on the full history every week. On top of that, several acquisition functions run side by side each round, from information-seeking to greedy: Knowledge Gradient (KG), Expected Improvement (EI) and Upper Confidence Bound (UCB), plus a geometric "battleship" search that does not use the GP. Their proposals are compared before anything is submitted.

I chose a GP because it works well with very few data points, gives an uncertainty with every prediction, and does not need gradients. With one evaluation per week, the cost of fitting a model is negligible compared to the cost of a wasted query.

Which proposal wins is a per-function decision, not a global policy, and it shifts as the data grows. The weekly loop: merge new results, refit, generate proposals, inspect (posterior mean and uncertainty plots, expected values at candidate points), decide, submit.

For the mostly flat function (Function 1), standard BO has nothing to work with: the surrogate is blind on a zero landscape. It therefore started with the geometric search. Once the tiny readings showed a trend in their exponent, I fitted simple shape models on the exponents and triangulated the peak, which returned 1.45 in round 10 after nine rounds of values close to zero.

As only the final submission is scored, rounds 11 and 12 were used to gather information, and round 13 resubmitted the best observed point of each function.

### Code

- [optimisation_sorted.ipynb](optimisation_sorted.ipynb) is the weekly loop: it loads the data, fits a GP per function, generates the candidate points of the acquisition functions and prints the query strings for the portal. It also contains the Function 1 shape models. The final cell picks the best observed point per function, which was the round 13 submission.
- [analysis.ipynb](analysis.ipynb) looks at the data: it prints the readings per function, plots Function 1 (including an interpolation of the exponent of its readings), and contains the benchmarking, i.e. the comparison of KG and EI and of different acquisition schedules on standard test functions.

To run them, install the packages in `requirements.txt`. The acquisition optimisation is partly random and no seeds were fixed in the weekly loop, so a rerun can propose slightly different points than the ones submitted. The submitted points are in the logbook.

## Hyperparameter optimisation

There are two kinds of hyperparameters.

**GP hyperparameters** (length scale per input dimension, output scale, noise level) are not tuned by hand. Ax/BoTorch fits them every week by maximising the marginal likelihood of the data, with the outputs standardised.

**Settings of the search** were chosen by me:

- Acquisition optimisation: number of restarts and raw samples (default 10 and 1024 for KG, 20 and 1024 for UCB), and 128 fantasies for KG. I noticed that some recommendations changed when rerunning the code, therefore I increased these per function (up to four times) until reruns gave stable points.
- UCB trade-off between mean and uncertainty: beta = 2.0.
- Battleship search: 1,000,000 random candidates; the point with the largest distance to all existing points and to the borders wins.
- The schedule of when to use which acquisition function. This was the most important choice, and the one I optimised most explicitly: I simulated the remaining rounds on standard test functions with known optimum, compared several schedules with five repeated runs each, and chose to alternate between information-seeking (KG) and exploiting (EI) rounds. A single run first pointed to a different schedule; only the repeats gave a stable answer.
- Function 1 shape models: the shape level (round, rotated or bent Gaussian) was chosen by leave-one-out error, with a margin of 0.10 in favour of the simpler shape. For the triangulation the distance metric was quadratic (power 2), which the data also selected when the power was left free (1.8 to 2.1).

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

All eight functions improved on the initial data. These are the best points found in the explored regions, not proven global maxima. Function 2 is noisy, so its improvement may be noise, and its resubmitted point may return a different value.

What I learned:

- One strategy for all functions would have wasted queries. The main work was to find out per function what kind of problem it is: a narrow peak (Function 4), a corner (Function 5), a noisy band (Function 2) or a flat field with one hidden peak (Function 1).
- Even readings that look like zero can carry information. For Function 1 the size of the tiny values pointed to the peak.
- Every fixed choice should be checked each round against its recent results. On Function 6 I kept EI for three rounds below the best before switching.
- The scoring rule changes the best strategy. With only the final round counting, earlier rounds can take risks.

## Decisions

The decisions and their reasons are in the [logbook](LOGBOOK.md), one table per function. The short version is also in the weekly discussion posts. My private working journal is not in this repo.

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

# Model card: BBO approach

## Overview

The approach is a sequential Bayesian optimisation with a Gaussian Process (GP) surrogate per function and a set of acquisition functions that are compared every round. For one function a purely geometric search is used instead. It is built on Ax and BoTorch. This card describes the version used in round 10.

## Intended use

The approach is suitable for maximising expensive black-box functions with few dimensions (here 2 to 8), bounded inputs and very few possible evaluations, where every evaluation has to be justified. Typical real world examples would be tuning a model that takes days to train, or an experiment that is costly to run.

It should not be used for high-dimensional problems or large evaluation budgets, where a GP gets slow and simpler methods do the job. It is also not suited for functions with jumps or isolated spikes, as the GP assumes a smooth function. In safety-critical settings it should only propose points, with a human making the final decision.

## Details

For each function a GP is refit on the full history every week. Based on the GP, four candidate points are generated from the same data:

- Knowledge Gradient (KG): picks the point that is expected to improve the final recommendation the most, i.e. it values information.
- Expected Improvement (EI): picks the point most likely to beat the current best, i.e. it exploits.
- Upper Confidence Bound (UCB): optimistic exploration, with a tunable trade-off between mean and uncertainty.
- "Battleship": does not use the GP at all and picks the point with the largest distance to all existing points and the borders.

Before submitting, the candidates are compared using plots of the GP mean and uncertainty and the expected values at the candidate points.

The approach changed over the rounds. In the first rounds mostly KG and UCB were used to learn the structure of the functions. Then I noticed that some recommendations changed when rerunning the code, as the acquisition optimisation is partly random. I therefore increased restarts and samples per function until reruns gave stable points. I also tested different schedules of switching between the acquisition functions on standard benchmark functions with repeated runs, and chose to alternate between information-seeking and exploiting rounds. In the later rounds the decision is made per function: functions with a single clear peak stay on EI, a function where I am confident the maximum is found gets its best point resubmitted, and the mostly flat Function 1 stays on the geometric search, since the GP has nothing to model there.

## Performance

The main metric is the best observed value per function compared to the best value in the initial data. The number of rounds that produced a new best value is a secondary measure.

| Function | Best initial | Best found | Round of best |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
| 6 | | | |
| 7 | | | |
| 8 | | | |

These are the best points found in the explored regions, not proven global maxima.

## Assumptions and limitations

The GP assumes that the functions are smooth, i.e. that nearby points have similar values, and that this holds in the same way everywhere in the input space. Narrow peaks therefore get smoothed out. In the later rounds I additionally assume that each function has one dominant peak near the current best point. If a function has a second, better region somewhere else, the approach will not find it. Single evaluations are treated as exact, as no noise information is available.

Two failure modes came up in practice. On Function 1 the GP is blind, and a simple model I fitted to the expected shape of the function gave a confident prediction that turned out wrong. And as the acquisition optimisation is partly random, an unchecked rerun can turn a good recommendation into a poor one.

The main constraint is the budget of one query per function per round. With that, especially in 6D and 8D, it is not possible to tell whether the result is the global maximum or only the best point in the region that was explored.

## Ethical considerations

The data, the notebook and the decision rules make the strategy traceable and reproducible. The exact points are not fully reproducible though, as no random seeds were fixed and reruns can differ slightly. For an exact reproduction the seeds and the library versions would need to be recorded.

Being transparent about the assumptions and failure modes allows others to judge when the approach can be transferred to a real problem and when not. In a real setting I would add fixed seeds, a log of every decision and a human approval before each expensive evaluation.

I think the current level of detail is sufficient for this card. A table per function and round with the chosen method and the result would make it fully auditable, but that is kept in my private decision log.

# Model card: BBO approach

## Model Description

The approach is a sequential Bayesian optimisation with a Gaussian Process (GP) surrogate per function and a set of acquisition functions that are compared every round. For Function 1, which is almost entirely flat, a geometric search and simple shape models are used instead. It is built on Ax and BoTorch. This card describes the final version (round 13).

**Input:** the full history of one function: the initial data provided by the course plus one query and its result per round. Inputs are vectors in the unit cube with 2 to 8 dimensions, outputs a single scalar.

**Output:** one proposed query point per function and round, formatted for the portal with six decimals (e.g. `0.123456-0.654321`). In the final round the output is the best observed point of each function.

**Model Architecture:** for each function a GP is refit on the full history every week (Ax/BoTorch default GP with one length scale per input dimension, hyperparameters fitted by maximum marginal likelihood). Based on the GP, four candidate points are generated from the same data:

- Knowledge Gradient (KG): picks the point that is expected to improve the final recommendation the most, i.e. it values information.
- Expected Improvement (EI): picks the point most likely to beat the current best, i.e. it exploits.
- Upper Confidence Bound (UCB): optimistic exploration, with a tunable trade-off between mean and uncertainty.
- "Battleship": does not use the GP at all and picks the point with the largest distance to all existing points and the borders.

Before submitting, the candidates are compared using plots of the GP mean and uncertainty and the expected values at the candidate points. The final choice per function is mine.

The approach changed over the rounds. In the first rounds mostly KG and UCB were used to learn the structure of the functions. Then I noticed that some recommendations changed when rerunning the code, as the acquisition optimisation is partly random. I therefore increased restarts and samples per function until reruns gave stable points. I also tested different schedules of switching between the acquisition functions on standard benchmark functions with repeated runs, and chose to alternate between information-seeking and exploiting rounds. In the later rounds the decision is made per function: functions with a single clear peak stay on EI, and a function where I am confident the maximum is found gets its best point resubmitted.

Function 1 needed a different approach, since it returns practically zero almost everywhere and the GP has nothing to model there. It started with the geometric search. Then I noticed that the exponent of the tiny readings (e.g. 1e-58, 1e-12, 1e-10) shrinks steadily towards one region. I therefore fitted simple shape models on the exponents instead of the values: a Gaussian blob in rounds 8 and 9, and in round 10 a triangulation of the peak from the three highest readings. Round 10 returned 1.45 after nine rounds close to zero. Rounds 11 and 12 tested a Gaussian fit and plain EI around it, both returned less.

Only the final submission is scored. Rounds 11 and 12 were therefore used to gather information, and in round 13 the best observed point of each function was resubmitted.

### Intended use

The approach is suitable for maximising expensive black-box functions with few dimensions (here 2 to 8), bounded inputs and very few possible evaluations, where every evaluation has to be justified. Typical real world examples would be tuning a model that takes days to train, or an experiment that is costly to run.

It should not be used for high-dimensional problems or large evaluation budgets, where a GP gets slow and simpler methods do the job. It is also not suited for functions with jumps or isolated spikes, as the GP assumes a smooth function. In safety-critical settings it should only propose points, with a human making the final decision.

## Performance

The main metric is the best observed value per function compared to the best value in the initial data, measured on the values returned by the portal over the 13 rounds. The number of rounds that produced a new best value is a secondary measure.

| Function | Best initial | Best found | Round of best |
|---|---|---|---|
| 1 | 7.7e-16 | 1.4497 | 10 |
| 2 | 0.6112 | 0.6128 | 4 |
| 3 | -0.0348 | -0.0010 | 6 |
| 4 | -4.0255 | 0.5518 | 5 |
| 5 | 1088.86 | 8662.41 | 6 |
| 6 | -0.7143 | -0.1771 | 4 |
| 7 | 1.3650 | 3.3224 | 4 |
| 8 | 9.5985 | 9.9896 | 9 |

All eight functions improved on the initial data. The clearest gains are Function 1 (from practically zero to a peak), Function 4 (from negative to positive values) and Function 5 (eight times the initial best). Function 2 improved only marginally, and as it is noisy, the difference to the initial best may be noise. New best values came in rounds 4, 5, 6, 9 and 10, so most of the progress happened in the first half, and the later rounds mainly confirmed the peaks.

These are the best points found in the explored regions, not proven global maxima. The round 13 results were not yet returned when this card was written.

## Limitations

The GP assumes that the functions are smooth, i.e. that nearby points have similar values, and that this holds in the same way everywhere in the input space. Narrow peaks therefore get smoothed out. In the later rounds I additionally assume that each function has one dominant peak near the current best point. If a function has a second, better region somewhere else, the approach will not find it. No noise information is given and no point was evaluated twice before round 13, so the noise level could only be estimated by the GP, not measured. The Function 1 shape models add a further assumption: a single smooth peak, with the exponent of the readings falling roughly with the squared distance to it.

Two failure modes came up in practice. On Function 1 the GP is blind, and a simple model I fitted to the expected shape of the function gave a confident prediction that turned out wrong. And as the acquisition optimisation is partly random, an unchecked rerun can turn a good recommendation into a poor one.

The main constraint is the budget of one query per function per round. With that, especially in 6D and 8D, it is not possible to tell whether the result is the global maximum or only the best point in the region that was explored.

## Trade-offs

- **Exploration against exploitation.** Every KG round gives up a likely good value for information. As only the final round is scored, this was cheap here. If every round counted, the same schedule would cost points.
- **Model assumptions against robustness.** The Function 1 shape models found the peak within three rounds (rounds 8 to 10), but when their assumptions were wrong (round 5, and round 8 which was ten orders of magnitude below its prediction) the query was largely wasted. In round 12 I therefore chose plain EI over adding further model assumptions.
- **Stability against computing time.** More restarts and raw samples make the proposals stable between reruns, but make each run slower. KG is the slowest, as it simulates many possible outcomes (fantasies) for each candidate.
- **Smoothness against narrow peaks.** The GP performs well where the function changes slowly or rises towards a border (Functions 3, 5 and 8). It performs worse on narrow peaks, e.g. Function 4, where refinements 0.001 to 0.005 away from the best point all returned lower values, and Function 1, where the GP could not see the peak at all.
- **Per-function judgement against reproducibility.** Deciding per function and per round gave better results than one fixed rule, but it relies on my judgement, which is harder to reproduce than code. The logbook documents each decision to make up for that.

## Ethical considerations

The data, the notebooks, the logbook and the decision rules make the strategy traceable and reproducible. The exact points are not fully reproducible though, as no random seeds were fixed and reruns can differ slightly. For an exact reproduction the seeds and the library versions would need to be recorded.

Being transparent about the assumptions and failure modes allows others to judge when the approach can be transferred to a real problem and when not. In a real setting I would add fixed seeds, a log of every decision and a human approval before each expensive evaluation.

I think the current level of detail is sufficient for this card. The full table per function and round, with the chosen method, the result and the reasoning, is in the [logbook](LOGBOOK.md), which makes the decisions auditable.

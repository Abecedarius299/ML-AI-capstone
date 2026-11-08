# Datasheet: BBO capstone data set

The data set was created as part of the black-box optimisation (BBO) capstone project of the Imperial College Professional Certificate in Machine Learning and AI. The task is to find the maximum of eight unknown functions with only one evaluation per function and week. The data set documents every evaluation, so that each query decision can be traced back to the data that was available at that point in time.

This datasheet covers all eight functions. Where the answers differ per function, they are given per function.

## Function overview

1. **Which function does this datasheet describe?** All eight, Function 1 to Function 8 (the functions have no other names).
2. **What real-world scenario does each function simulate?** See the table, as described by the course.
3. **Dimensionality of the input:** 2D to 8D, see the table.
4. **Initial data points:** 10 to 40, see the table.
5. **What does the output represent?** See the table. In all cases higher is better.

| Function | Scenario (course description) | Input | Initial points | Output |
|---|---|---|---|---|
| 1 | Detecting contamination sources (e.g. radiation) in an area | 2D | 10 | Signal strength, practically zero away from a source |
| 2 | A black-box machine learning model | 2D | 10 | Noisy log-likelihood |
| 3 | Drug discovery, a mix of three compounds | 3D | 15 | Adverse reactions, negated so that higher is better |
| 4 | Placing products across warehouses, approximated by a model | 4D | 30 | Score of the placement |
| 5 | Chemical process | 4D | 20 | Yield |
| 6 | Cake recipe with five ingredients | 5D | 20 | Combined score of flavour, consistency, calories, waste and cost (negative, higher is better) |
| 7 | Tuning six hyperparameters of a machine learning model | 6D | 30 | Performance score of the model |
| 8 | Tuning eight hyperparameters of a machine learning model | 8D | 40 | Performance score of the model |

## Nature of the data

1. **Structure of the initial data set.** Each function has two NumPy arrays: `initial_inputs.npy` with shape (n, d) and `initial_outputs.npy` with shape (n,), where n is the number of initial points and d the dimension: (10, 2), (10, 2), (15, 3), (30, 4), (20, 4), (20, 5), (30, 6) and (40, 8). All inputs lie in the unit cube [0, 1]. The output scales differ a lot between functions, from values very close to zero (Function 1) to four digit numbers (Function 5). No noise estimate is given.

2. **How the data set evolves.** Each round adds one point per function, so after 13 rounds each function has 13 more points. The weekly queries and results are added in the data cell of both notebooks (one block per round) and listed in the [logbook](LOGBOOK.md). The sampling is adaptive, not random:
   - Rounds 1 to 4: exploration while comparing acquisition functions, including probes at the borders.
   - Rounds 5 to 10: alternating between information-seeking (KG) and exploiting (EI) rounds, adapted per function.
   - Round 11: information gathering (KG), as only the final round is scored.
   - Round 12: a last exploitation test (EI) on all functions.
   - Round 13: the best observed point of each function was resubmitted.

   The queries therefore concentrate around the best points found, and large parts of the input space are only covered by the initial points. This gets worse with higher dimensions: in the 6D and 8D functions a few dozen points barely cover the space. The data set is a record of the search, not a representative sample of the functions.

3. **Noise.** No noise information is given, and before round 13 no point was evaluated twice, so noise could not be measured directly. Function 2 is described as noisy, and its data agrees: a query 0.036 away from its best point returned 0.17 lower. I therefore did not trust single high readings on Function 2 and treated its small improvement over the initial data as uncertain. The other functions behaved consistently and were treated as practically exact.

4. **Shape of each function**, based on plots of the data, the GP mean and uncertainty, and how well the GP predictions matched the returned values:
   - Function 1: flat (practically zero) almost everywhere, with one narrow peak near (0.64, 0.63). The exponent of the tiny readings falls smoothly towards the peak.
   - Function 2: noisy. Good values appear in a band around x1 = 0.7 rather than at one clear peak.
   - Function 3: smooth. The best values are at x1 = 1 (upper bound), all readings are negative and the best is close to zero.
   - Function 4: unimodal with a narrow peak near (0.42, 0.38, 0.41, 0.43) and steep drops (down to -7) away from it.
   - Function 5: rises steadily towards the corner where all inputs are at their upper bound, i.e. unimodal with the maximum at the border.
   - Function 6: unclear. No query after round 4 improved on the best, which suggests either a narrow peak or several local optima.
   - Function 7: smooth with several local optima. The data was consistent with a known benchmark function (see the logbook).
   - Function 8: smooth and flat at the top. All my queries returned values between 9.7 and 10, so the differences between good points are small.

## Your optimisation strategy

1. **Methods used.** Bayesian optimisation with a Gaussian Process (GP) surrogate (Ax/BoTorch) and the acquisition functions Knowledge Gradient (KG), Expected Improvement (EI) and Upper Confidence Bound (UCB). In addition a geometric space-filling search ("battleship") without a model, and for Function 1 manual reasoning with simple shape models.

2. **Why this method.** Each evaluation costs a week, there are very few points and at most 8 dimensions, which is the setting where GPs work best: they need few points, give an uncertainty with every prediction, and do not need gradients. Per function:
   - Function 1: the GP is blind on a zero landscape, therefore geometric search first, then shape models fitted on the exponent of the readings.
   - Function 2: noisy, therefore no aggressive exploitation of single high readings.
   - Functions 4 and 5: a single clear peak or trend, therefore EI once it was visible.
   - Functions 6, 7 and 8: higher dimensions and sparse data, therefore mostly KG to learn more about the landscape.

3. **Exploration and exploitation.** KG for information, EI for exploitation, UCB as a comparison. The schedule (alternating KG and EI rounds) was chosen by simulating the remaining rounds on standard test functions with five repeated runs per schedule. On top of that, the choice was adapted per function, and in the end by the scoring rule: as only round 13 counts, rounds 11 and 12 could be spent on information and tests without risk.

4. **Changes over the weeks.**
   - Reruns gave different points, so I increased restarts and raw samples per function until they were stable.
   - Function 4 was treated as converged after two refinements in a row returned lower values, and its queries were used elsewhere.
   - On Function 6, EI returned below the best three rounds in a row, so it was switched back to KG.
   - Function 1 went from geometric search to a round Mexican hat model (rejected after it returned zero), a Gaussian blob, a triangulation (which found the peak in round 10), a Gaussian fit and finally plain EI.
   - Once it was clear that only the final submission is scored, the last rounds were planned around that.

## Data handling and preprocessing

1. **Rescaling inputs.** No. The inputs are already in the unit cube for all dimensions, so no rescaling was needed. Ax/BoTorch standardises the outputs internally when fitting the GP.

2. **Surrogate models.** A GP per function, refit every week. For Function 1 also simple parametric models fitted with least squares: a Mexican hat, a Gaussian blob in three levels of shape (round, rotated ellipse, bent ellipse), a quadratic triangulation and a symmetric Gaussian.

3. **Preprocessing for the surrogates.**
   - GP: the Ax/BoTorch default, with one length scale per input dimension. Its hyperparameters, including the noise level, are fitted by maximum marginal likelihood every week. No manual kernel choice.
   - Function 1 models: fitted on log10 of the absolute value of the readings (i.e. their exponent), as the raw values span from about 1e-124 to 1.45. The blob level was chosen by leave-one-out error, with a margin of 0.10 in favour of the simpler shape.
   - The stored data stays unchanged.

4. **Outliers.** No points were removed. Very low readings (e.g. -7 on Function 4, the edges of Function 2) were kept, as they show where not to look. The tiny readings of Function 1 were not treated as zero either: their exponent turned out to be the most useful information in the whole data set.

## Weekly iteration and learning

1. **How new data changed my understanding.** Function 1 changed from "flat, nothing to learn" to a function whose tiny values point to the peak. Function 5 turned out to have its maximum at the corner rather than inside. Function 4 showed how narrow its peak is. Function 2 showed its noise. Function 6 never became clear.

2. **Local optima.** Detected when refinements around the best point kept returning lower values (Function 4, rounds 6 to 8), or when exploitation got stuck below the best (Function 6, rounds 5 to 7). On Function 5 another corner (x3 = 0) returned a strong 4256, i.e. a second high region, but well below the best corner. Probes far from the best points returned lower values on all functions, which supports, but does not prove, that the best points are global maxima.

3. **Most informative queries.**
   - The tiny Function 1 readings of rounds 5, 8 and 9, whose exponents pointed to the peak.
   - Boundary points: the corner of Function 5 and x1 = 1 on Function 3.
   - Failed predictions, e.g. the Mexican hat model on Function 1 in round 5, which ruled out a whole class of shapes.

4. **What I would do differently.**
   - Look at the exponent of the Function 1 readings from the start; the peak could have been found several rounds earlier.
   - Plan for "only the final round is scored" from the beginning.
   - Fix random seeds from round 1.
   - Repeat one point on Function 2 early to measure its noise.
   - Record the method of every query in a structured file from day one, not only in the notebook.

## Performance and results

| Function | Best output | Input vector | Confidence that it is near the global maximum |
|---|---|---|---|
| 1 | 1.4497 | `0.639566-0.625206` | Medium. The peak was found from a smooth trend, but it is narrow: 0.015 away the value drops to a third, so the true top may be a bit higher. |
| 2 | 0.6128 | `0.705807-0.548955` | Low. Noisy, and only marginally above the initial best (0.6112). |
| 3 | -0.0010 | `0.999999-0.604294-0.475172` | High. Very close to zero, which is likely the upper limit for negated adverse reactions. |
| 4 | 0.5518 | `0.418584-0.379256-0.406197-0.429294` | Medium to high. All refinements around it returned lower, all probes elsewhere far lower. |
| 5 | 8662.41 | `0.999999-0.999999-0.999999-0.999999` | High within the allowed inputs: the values rise steadily towards this corner, which cannot be exceeded. |
| 6 | -0.1771 | `0.469319-0.386405-0.637791-0.692818-0.155377` | Low. No improvement after round 4, and the landscape is not understood. |
| 7 | 3.3224 | `0.201690-0.150011-0.476874-0.275332-0.311652-0.657300` | High. It matches the optimum of the benchmark function the data was consistent with. |
| 8 | 9.9896 | `0.129074-0.186300-0.147630-0.185138-0.783370-0.494732-0.229729-0.780314` | Medium. The top is flat and 8D is poorly covered, so a slightly better point may exist. |

**Alignment with expectations.** Mostly yes. Function 5 behaved like a typical yield with one maximum, Function 2 was noisy as described, Function 1 behaved like a single source with a signal confined to its surroundings, and Function 8 had the flat plateau typical for hyperparameter tuning. Function 6 did not: for a recipe I expected a clear optimum inside the range, which I could not find.

## Ethical, practical and general considerations

1. **Relation to real-world applications.** This is what real optimisation usually looks like: tuning a model that takes days to train, running an experiment that costs actual money, testing a recipe you can only bake once. The skill is to decide what is worth measuring next, and to defend that decision before knowing how it turns out.

2. **Limitations of the synthetic functions.** The functions have no evaluation cost apart from time, no constraints and, apart from Function 2, practically no noise. Real problems usually have all three. Also, a synthetic function can turn out to be a known benchmark (Function 7), which is a shortcut that would not exist in a real problem.

3. **Scaling to more serious or expensive problems.** For more expensive problems, yes: the computing time of the GP is negligible compared to an expensive evaluation, and the more expensive each evaluation, the more a careful choice pays off. For many more dimensions or thousands of evaluations, no: the GP gets slow and its uncertainty becomes less useful. For a serious application I would add fixed seeds, a log of every decision, noise handling and a human approval before each evaluation.

4. **Risks for a future user.**
   - The sampling is biased towards the explored maxima, so the data should not be used to estimate the overall shape of a function, or to train flexible models such as neural networks, for which it is far too small.
   - Values close to zero (Function 1) look like no information, but are not.
   - Single readings of Function 2 should not be trusted.
   - The best observed points are not proven global maxima.
   - Reruns of the notebooks can give slightly different points, as no seeds were fixed.

## Distribution and maintenance

The data set is available in this repository. The initial data was provided by the course, the query history is shared for review and educational purposes. I maintained the data set and updated it after each round. The capstone is finished, therefore the data set is final once the round 13 results are added.

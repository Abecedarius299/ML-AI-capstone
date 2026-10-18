# Datasheet: BBO capstone data set

## Motivation

The data set was created as part of the black-box optimisation (BBO) capstone project of the Imperial College Professional Certificate in Machine Learning and AI. The task is to find the maximum of eight unknown functions with only one evaluation per function and week. The data set documents every evaluation, so that each query decision can be traced back to the data that was available at that point in time.

## Composition

The data set contains input-output pairs for eight functions. Each input is a vector in the unit cube [0, 1], each output a single scalar value. The functions differ in dimensionality and in the size of the initial data:

| Function | Dimensions | Initial points |
|---|---|---|
| 1 | 2 | 10 |
| 2 | 2 | 10 |
| 3 | 3 | 15 |
| 4 | 4 | 30 |
| 5 | 4 | 20 |
| 6 | 5 | 20 |
| 7 | 6 | 30 |
| 8 | 8 | 40 |

Each round adds one point per function. The initial data is stored as NumPy arrays (`initial_inputs.npy`, `initial_outputs.npy`), the weekly results are appended in the notebook. The output scales differ a lot between functions, from values very close to zero to four digit numbers. No noise estimate is given.

The data set has clear gaps. The weekly queries concentrate around the best points found, so large parts of the input space are only covered by the initial points. This gets worse with higher dimensions: in the 6D and 8D functions a few dozen points barely cover the space. Function 1 is almost entirely zero, therefore most of its points carry no information about where the maximum might be.

## Collection process

The initial points were provided by the course. Every week one new query per function was chosen with the approach described in the [model card](MODEL_CARD.md) and submitted through the course portal, with six decimals per coordinate. The portal returns the function value about a week later. The project runs over 13 rounds.

The sampling is adaptive, not random. In the early rounds the queries were chosen to learn about the functions, in the later rounds increasingly to improve on the best point. The data set is therefore a record of the search, not a representative sample of the functions.

## Preprocessing and uses

The raw values are stored unchanged. When fitting the Gaussian Process, Ax/BoTorch standardises the outputs internally. No other transformations are applied.

Intended uses are reviewing and reproducing the query decisions, and comparing other sequential optimisation strategies on the same history. Inappropriate uses are estimating the overall shape of a function, since the sampling is biased towards the explored maxima, and training flexible models such as neural networks, for which the data set is far too small. Results should also not be generalised to functions outside this challenge.

## Distribution and maintenance

The data set is available in this repository. The initial data belongs to the course provider, the query history is shared for review and educational purposes. I maintain the data set and update it after each round until the end of the capstone. After that it will not be updated.

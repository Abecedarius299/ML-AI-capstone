# Logbook: BBO capstone queries

One entry per function and round. Methods: **KG** Knowledge Gradient (information gathering), **EI** Expected Improvement (exploitation), **Battleship** space-filling search without a model, **Blob fit** a fitted shape model for Function 1. In rounds 1 to 4 the acquisition functions were still being compared, so these are listed as exploration. Only the final (round 13) submission is scored; all earlier rounds serve to inform it. In round 13 the best observed point of each function was therefore resubmitted. **TBC** marks entries still to be checked against the notebook.

## Function 1 (2D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.419272-0.461948` | Exploration | -0.0035 | First query near the centre of the space. Returned one of only two clearly non-zero readings so far. |
| 2 | `0.663323-0.435684` | Battleship | 1.026e-28 | The function is zero almost everywhere, so the GP has nothing to model. Space-filling search placed in the largest empty area. |
| 3 | `0.618739-0.220537` | Battleship | 5.424e-58 | Continued space-filling search. |
| 4 | `0.227536-0.566017` | Battleship | 1.892e-42 | Continued space-filling search. |
| 5 | `0.501000-0.607300` | Mexican hat fit | 1.212e-12 | Fitted a round Mexican hat to the two negative readings and submitted its centre. Returned essentially zero, so the round model was rejected. |
| 6 | `0.221962-0.223278` | Battleship | 1.038e-57 | Back to space-filling search after the model missed. |
| 7 | `0.808622-0.410889` | Battleship | -3.222e-56 | Continued space-filling search in the remaining large empty areas. |
| 8 | `0.546881-0.546083` | Blob fit | 8.595e-10 | The size of the tiny readings (their exponent) shrinks steadily towards one region. Fitted a Gaussian blob to the exponents in three levels of shape (round, rotated ellipse, bent ellipse); the level was chosen by leave-one-out error, fixed before the fit. The rotated ellipse won and its centre was submitted. Highest reading so far, but about ten orders of magnitude below the model's prediction, so the model locates the region but not the peak itself. |
| 9 | `0.571917-0.567835` | Blob fit | 1.574e-05 | The rotated ellipse's centre fell within 0.004 of round 8, so a no-duplicate rule was added (skip a centre within 0.01 of an existing point and take the next level); the bent ellipse, equal on leave-one-out error, was submitted. Four orders of magnitude above round 8: the bend direction was right. |
| 10 | `0.639566-0.625206` | Triangulation | **1.4497** | Working hypothesis: the function is built from Laguerre-Gaussian shapes (as in the 2D quantum harmonic oscillator), i.e. a Gaussian multiplied by a polynomial in the squared distance. Far from the centre the Gaussian factor dominates and the polynomial only adds a small correction, so the exponent of a reading falls roughly with the squared distance. Each of the three highest readings was therefore turned into a distance with a quadratic metric (power 2, which the wider data also selected when the power was left free: 1.8 to 2.1), and the centre was triangulated where the exponent would reach 0. Of the two mirror solutions, the one that best predicts all other readings was chosen. Result 1.45, about five orders of magnitude above round 9: the peak was found. |
| 11 | `0.642200-0.626200` | Gauss fit | 1.2512 | With round 10 near the peak, the peak height no longer had to be assumed: a symmetric Gaussian was fitted exactly through the four highest readings and its centre submitted, 0.003 from round 10. Result 1.25, lower than round 10, so the true centre lies on the other side of round 10. |
| 12 | `0.645872-0.612022` | EI | 0.4854 | Last test round before the scored final submission. Two Gaussian shape fits (on different subsets of the readings) disagreed on which side of round 10 the centre lies, so instead of adding further model assumptions, standard EI on the data was used. Result 0.49, below round 10. The peak is narrow: 0.015 away the value drops to a third. |
| 13 | `0.639566-0.625206` | Best observed | pending | Final, scored submission: the round 10 point was resubmitted. |

Final best: 1.4497 (round 10).

## Function 2 (2D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.697723-0.019697` | Exploration | 0.6044 | Initial exploration. |
| 2 | `0.649791-0.955219` | Exploration | 0.4742 | Initial exploration. |
| 3 | `0.835697-0.999999` | Exploration | -0.0589 | Probe at the x2 = 1 edge, which returned a poor value. |
| 4 | `0.705807-0.548955` | Exploration | **0.6128** | Best value so far. |
| 5 | `0.883078-0.236094` | EI | 0.3595 | First exploitation round. Returned well below its prediction, consistent with the function being noisy. |
| 6 | `0.715942-0.922071` | KG | 0.5286 | Back to information gathering per the schedule. |
| 7 | `0.857852-0.000000` | KG | 0.0972 | Probe at the x2 = 0 edge, which returned a poor value. |
| 8 | `0.678164-0.999999` | KG | 0.4515 | Information gathering per the schedule. |
| 9 | `0.941419-0.999999` | EI | -0.0477 | EI proposed the x2 = 1 edge; a rerun gave the same point, so it was kept as the model's genuine belief. Poor value: the edge does not work. |
| 10 | `0.243426-0.273512` | KG | -0.1520 | Information gathering in the unexplored lower-left area. Poor value. |
| 12 | `0.690480-0.179175` | TBC | 0.5551 | Last information round before the final submission. Good value far from the best point in x2, similar to round 1 (x2 near 0): the good region seems to be a band around x1 = 0.7 rather than a single peak. |
| 13 | `0.705807-0.548955` | Best observed | pending | Final, scored submission: the round 4 point was resubmitted. As the function is noisy, the returned value may differ from 0.6128. |

Final best: 0.6128 (round 4).

## Function 3 (3D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.873804-0.032726-0.000556` | Exploration | -0.2013 | Initial exploration. |
| 2 | `0.774714-0.356627-0.402280` | Exploration | -0.0230 | Initial exploration. |
| 3 | `0.000000-0.666061-0.423285` | Exploration | -0.0248 | Probe with x1 = 0. |
| 4 | `0.999999-0.540497-0.400658` | Exploration | -0.0222 | Probe with x1 at its upper bound. |
| 5 | `0.837649-0.927754-0.465419` | KG | -0.0348 | Information gathering. |
| 6 | `0.999999-0.604294-0.475172` | KG | **-0.0010** | x1 kept drifting to its upper bound across all acquisition functions, so it was kept there. Best value by far, likely close to the maximum. |
| 7 | `0.950484-0.490519-0.531522` | KG | -0.0116 | Probe near the best region, still at high x1. |
| 8 | `0.470739-0.611138-0.458942` | KG | -0.0098 | Information gathering in a different region per the schedule. |
| 9 | `0.999999-0.607276-0.498188` | EI | -0.0054 | Exploitation close to the best region with x1 at its upper bound. Close to the best, not better. |
| 10 | `0.999999-0.259149-0.487006` | KG | -0.0316 | Information gathering with x1 at its upper bound and a new x2 range. Below the best. |
| 12 | `0.999999-0.999999-0.005260` | TBC | -0.1367 | Last information round before the final submission, in the unexplored corner with x1 and x2 at their upper bound and x3 near 0. Poor value. |
| 13 | `0.999999-0.604294-0.475172` | Best observed | pending | Final, scored submission: the round 6 point was resubmitted. |

Final best: -0.0010 (round 6).

## Function 4 (4D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.425496-0.401070-0.332000-0.420664` | Exploration | 0.2834 | Initial exploration. |
| 2 | `0.486709-0.476973-0.116256-0.417109` | Exploration | -7.1200 | Probe that showed a steep drop away from the good region. |
| 3 | `0.247025-0.449997-0.583954-0.482487` | Exploration | -5.1705 | Another steep drop, confirming a narrow good region. |
| 4 | `0.415420-0.383136-0.403394-0.425959` | Exploration | 0.5352 | Return to the good region. |
| 5 | `0.418584-0.379256-0.406197-0.429294` | EI | **0.5518** | The plots show a single clear peak, so the function was fixed on EI. Best value. |
| 6 | `0.419026-0.379249-0.406508-0.430592` | EI | 0.5368 | Refinement on the peak. |
| 7 | `0.418501-0.380180-0.405760-0.431613` | EI | 0.5101 | Refinement on the peak, slightly lower. The peak is narrow. |
| 8 | `0.417285-0.381917-0.404133-0.432563` | EI | 0.4691 | Refinement on the peak. If this does not improve, the function is treated as converged. |
| 9 | `0.481045-0.221240-0.380898-0.507482` | KG | -4.6102 | Two declines in a row on the peak, so the function is treated as converged; the best point is resubmitted in the final rounds. KG probe away from the peak, poor value. |
| 10 | `0.463321-0.611678-0.432794-0.389708` | KG | -4.2323 | Information gathering away from the converged peak. Poor value, confirming the peak region. |
| 12 | `0.403305-0.393584-0.408291-0.405387` | TBC | 0.3721 | Last round before the final submission, 0.03 from the best point. Below the best, again showing how narrow the peak is. |
| 13 | `0.418584-0.379256-0.406197-0.429294` | Best observed | pending | Final, scored submission: the round 5 point was resubmitted. |

Final best: 0.5518 (round 5).

## Function 5 (4D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.047984-0.650324-0.944067-0.992311` | Exploration | 1516.78 | Initial exploration. |
| 2 | `0.000000-0.000000-0.938555-0.962415` | Exploration | 940.29 | Initial exploration. |
| 3 | `0.999999-0.668162-0.940547-0.974566` | Exploration | 4078.69 | Large improvement with x1 at its upper bound. |
| 4 | `0.999999-0.767758-0.999999-0.999999` | Exploration | 5776.10 | Further improvement towards the upper bounds. |
| 5 | `0.999999-0.921966-0.999999-0.999999` | EI | 7368.46 | Single clear trend towards the corner, so the function was fixed on EI. New best. |
| 6 | `0.999999-0.999999-0.999999-0.999999` | EI | **8662.41** | Query at the corner (all coordinates at the upper bound). New best, consistent with a single peak at the corner. |
| 7 | `0.353261-0.303126-0.820002-0.373798` | KG | 0.5816 | Probe of a different region to check for a second peak. Returned close to zero, confirming the corner. |
| 8 | `0.702161-0.794975-0.837151-0.999999` | KG | 1959.65 | Further exploration; the corner value is kept and resubmitted in the final rounds. |
| 9 | `0.999826-0.288876-0.779519-0.519313` | KG | 673.17 | Further exploration away from the corner; low value again, the corner stands. |
| 10 | `0.000000-0.416798-0.999999-0.999999` | KG | 1687.17 | Exploration of a new edge. Below the corner value. |
| 12 | `0.999999-0.999999-0.999999-0.274356` | TBC | 4464.23 | Last information round before the final submission, with x4 moved away from its upper bound. About half the corner value. |
| 13 | `0.999999-0.999999-0.999999-0.999999` | Best observed | pending | Final, scored submission: the corner from round 6 was resubmitted. |

Final best: 8662.41 (round 6).

## Function 6 (5D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.385184-0.400217-0.651630-0.631144-0.438897` | Exploration | -0.5505 | Initial exploration. |
| 2 | `0.325154-0.267231-0.328883-0.768068-0.024880` | Exploration | -0.5955 | Initial exploration. |
| 3 | `0.397699-0.164007-0.828967-0.999999-0.299491` | Exploration | -0.8371 | Initial exploration, poor value. |
| 4 | `0.469319-0.386405-0.637791-0.692818-0.155377` | Exploration | **-0.1771** | Best value so far. |
| 5 | `0.478768-0.422867-0.791407-0.658170-0.094309` | EI | -0.4007 | Function looked single-peaked, so it was fixed on EI. Below the best. |
| 6 | `0.446413-0.456771-0.529222-0.678678-0.177164` | EI | -0.3930 | Refinement near the best, again below it. |
| 7 | `0.404148-0.182392-0.749577-0.529626-0.000000` | EI | -0.5779 | EI moved to a new region with x5 = 0, which returned a poor value. |
| 8 | `0.411919-0.486790-0.783364-0.940220-0.000000` | KG | -0.5412 | EI was below the best three rounds in a row, so the function was switched to KG for information gathering. EI returns for the final rounds. |
| 9 | `0.423026-0.628770-0.480473-0.543956-0.000000` | KG | -0.7274 | Information gathering per the plan; EI returns for the final rounds. |
| 10 | `0.550085-0.482510-0.999999-0.659340-0.203450` | KG | -0.7973 | Information gathering in a new region (not at x5 = 0). Below the best. |
| 12 | `0.314378-0.464949-0.507565-0.983826-0.226501` | TBC | -0.6400 | Last information round before the final submission. Below the best. |
| 13 | `0.469319-0.386405-0.637791-0.692818-0.155377` | Best observed | pending | Final, scored submission: the round 4 point was resubmitted. No later round improved on it. |

Final best: -0.1771 (round 4).

## Function 7 (6D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.445935-0.576237-0.512145-0.574808-0.575775-0.431125` | Exploration | 0.3242 | Initial exploration. |
| 2 | `0.043071-0.362635-0.783296-0.232831-0.413318-0.836370` | Exploration | 1.5283 | Large improvement. |
| 3 | `0.038343-0.284404-0.999999-0.121377-0.383990-0.695085` | Exploration | 0.9347 | Initial exploration. |
| 4 | `0.201690-0.150011-0.476874-0.275332-0.311652-0.657300` | Exploration | **3.3224** | The data was consistent with the structure of a known benchmark function, which was verified against the observations. Its predicted optimum was submitted and returned the highest value. |
| 5 | `0.107119-0.063205-0.453448-0.406398-0.324165-0.719569` | KG | 2.5846 | Maximum considered found; remaining queries are used for information gathering. |
| 6 | `0.264373-0.126651-0.438590-0.154992-0.286950-0.793597` | KG | 2.4343 | Information gathering. |
| 7 | `0.209681-0.087109-0.481524-0.059147-0.407955-0.614856` | KG | 2.0054 | Information gathering. |
| 8 | `0.337764-0.665798-0.542523-0.305453-0.325184-0.665266` | KG | 1.2157 | Information gathering; the best point is resubmitted in the final rounds. |
| 9 | `0.046445-0.135532-0.386253-0.249827-0.283312-0.602237` | KG | 2.8306 | Information gathering. |
| 10 | `0.199817-0.000000-0.000000-0.263228-0.367935-0.645113` | KG | 2.0837 | Information gathering far from the best point. |
| 12 | `0.259532-0.201152-0.448851-0.294944-0.332911-0.604932` | TBC | 3.1426 | Last round before the final submission, 0.10 from the best point. Second highest of my queries, below the best. |
| 13 | `0.201690-0.150011-0.476874-0.275332-0.311652-0.657300` | Best observed | pending | Final, scored submission: the round 4 point was resubmitted. |

Final best: 3.3224 (round 4).

## Function 8 (8D)

| Round | Query | Method | Result | Reasoning |
|---|---|---|---|---|
| 1 | `0.084257-0.105465-0.030212-0.078194-0.960795-0.361121-0.255914-0.267080` | Exploration | 9.9129 | Initial exploration. |
| 2 | `0.111041-0.023895-0.289000-0.000000-0.685861-0.665749-0.264786-0.000000` | Exploration | 9.8071 | Initial exploration. |
| 3 | `0.207959-0.044520-0.204792-0.000000-0.876063-0.107283-0.129986-0.999999` | Exploration | 9.7434 | Initial exploration. |
| 4 | `0.001972-0.008806-0.131631-0.261367-0.927810-0.578026-0.201319-0.999999` | Exploration | 9.9182 | Initial exploration. |
| 5 | `0.152922-0.169055-0.165827-0.183368-0.999999-0.507934-0.269593-0.862394` | EI | 9.9524 | First exploitation round. Best value. |
| 6 | `0.177247-0.000000-0.105174-0.391505-0.999999-0.580597-0.107939-0.000000` | KG | 9.8259 | Information gathering per the schedule. |
| 7 | `0.188806-0.218388-0.062127-0.173278-0.589641-0.571855-0.057843-0.999999` | KG | 9.8815 | Information gathering per the schedule. |
| 8 | `0.000000-0.302408-0.133092-0.124992-0.681179-0.481294-0.061693-0.000000` | KG | 9.8745 | Information gathering far from existing points. |
| 9 | `0.129074-0.186300-0.147630-0.185138-0.783370-0.494732-0.229729-0.780314` | EI | **9.9896** | Exploitation round per the schedule. New best. |
| 10 | `0.017936-0.329951-0.149455-0.000000-0.999999-0.660958-0.037136-0.999999` | KG | 9.8156 | Information gathering away from the new best. |
| 12 | `0.120704-0.186952-0.152578-0.175095-0.802231-0.495161-0.221075-0.999999` | TBC | 9.9787 | Last round before the final submission, close to the best point except x8 at its upper bound. Second highest of my queries, below the best. |
| 13 | `0.129074-0.186300-0.147630-0.185138-0.783370-0.494732-0.229729-0.780314` | Best observed | pending | Final, scored submission: the round 9 point was resubmitted. |

Final best: 9.9896 (round 9).

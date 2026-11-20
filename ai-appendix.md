# AI Appendix

## Sections 1 and 2

Sections 1 and 2 were completed without AI.

Name: Julia Clark
Date: 11/19/2026

Using AI in Sections 1 or 2 is an integrity violation. Do not edit the statement above; if it
is not true, talk to the instructor before you submit.

## How to file an entry

Sections 3 to 6 allow AI as a documented co-pilot under P.R.I.M.E. File one entry for each
piece of work you used it on, under the section it belongs to. Paste the full exchange: every
prompt as you sent it and every reply as it came back, including what you rejected. A summary
is not a transcript. If you cannot explain a line of what the model gave you, you cannot
submit it.

If you did not use AI in a section, write "No AI used in this section." under its heading.

Copy this block for each entry:

```
### Entry: TODO what it was for (for example, Exercise 1, not-yet-treated comparison)

- Model and version: TODO
- Date: TODO

**[Prep]** TODO: the role you gave the model.
**[Request]** TODO: what you asked for.
**[Iterate]** TODO: the constraints you added, and the version-2 prompt.
**[Mechanism Check]** TODO: what you asked it to explain, and whether the explanation held.
**[Evaluate]** TODO: how you checked the output against your own data or results.

**Full transcript**

TODO: paste the exchange here, unedited.

**What I changed, and why:** TODO, one line per change.
```

## Section 3: Robustness

### Exercise 1

TODO

### Exercise 2
```
### Entry: Outside factors
- Model and version: Claude Sonnet 5.5
- Date: 11/18/2026
**[Prep]** Pretend you are replicating Hansen (2015) study on drink driving and writing a report.
**[Request]** I did a placebo test at 0.05 on the hansen data and it produced a negative coefficient that was very close to my actual cutoff but it was not signifigant. What else could contribute to this pattern other than the treatment effect?
  
 Answers: A negative coefficient at the 0.05 placebo cutoff that is close in magnitude to the actual 0.08 estimate could reflect several things other than the treatment effect.

One possibility is that there is an underlying relationship between BAC levels and recidivism that is not entirely attributable to the legal threshold. For example, drivers with different BAC levels may differ in characteristics that influence their likelihood of reoffending. Even without a treatment effect, these differences could produce a negative estimated discontinuity.

Another possibility is sampling variation. The placebo estimate may be negative simply due to random variation in the sample, particularly if there are fewer observations around the 0.05 cutoff. The fact that the estimate is not statistically significant means there is insufficient evidence to conclude that a true discontinuity exists at that threshold.

There could also be differences in the distribution or measurement of BAC readings across the cutoff. If observations are unevenly distributed or there are patterns in how BAC values are recorded, this could influence the estimated coefficient.

I would compare the sample sizes, standard errors, and distribution of BAC readings around the two cutoffs before drawing conclusions. The similar negative coefficients are worth noting, but the lack of statistical significance at 0.05 means the placebo result does not provide strong evidence of a separate discontinuity.

**[Evaluate]** I think this is a good answer and gives lots of possible explainations however, i think it is too unknown that I did not include any of these in my analysis.
```
### Exercise 3

### Exercise 4
```
### Entry: Donut size
- Model and version: Claude Sonnet 5.5
- Date: 11/18/2026
**[Prep]** Pretend you are replicating Hansen (2015) study on drink driving and writing a report.
**[Request]** For the donut RD exercise, what would be reasonable exclusion ranges around the 0.08 cutoff to test? What would provide a meaningful sensitivity analysis when keeping the other variables the same?
**[Iterate]** I originally used a 0.05 bandwidth and here is the data set. I also want to keep the same kernel, and clustering approach 

Answer: I would recommend testing donut widths of 0.001, 0.002, and 0.003 around the 0.08 cutoff. These provide progressively larger exclusion windows while remaining relatively narrow compared with the original bandwidth of 0.05.

I would also include a baseline specification with no donut (0.000) so that the results can be directly compared against the original model. This would allow you to evaluate whether excluding observations immediately around the cutoff substantially changes the estimated treatment effect.

Before settling on these values, I would examine the distribution of BAC readings around 0.08 to determine how many observations would be removed at each exclusion level. If the larger donut removes too many observations, it could reduce precision and make the results harder to interpret.

I would keep the remaining specification choices constant, including the cutoff, bandwidth, kernel, and covariates, so that changes in the estimated effect can be attributed primarily to the donut width.

**[Evaluate]** I tested the model at the different donut widths and compared the numbers to eachother. I used 0.00, 0.001, 0.002 and 0.003. 
```
```
## Entry: Organize and interpret results
**[Prep]** Pretend you are replication Hansen (2015) and conducting different exercises to check the robustness of the result
**[Request]** I have this chunk of code completed and need help presenting the results in a table and drawing conclusions. Can you help with the syntax for organizing the data. 
**[Iterate]** Don't change any of my code, use my variable names and give the code to make an organized output. 

import numpy as np
import pandas as pd
from rdrobust import rdrobust

# My spec: the replication settings (Table 3, h = 0.05, rectangular kernel) plus a donut
MY_SPEC = {"cutoff": 0.08, "h": 0.05, "kernel": "uniform", "donut": 0.002, "sample": None}

RV       = "bac_min"                          # lower of the two readings (low_score / 1000)
Y        = "recidivism"
CONTROLS = ["male", "white", "aged", "acc"]
CLUSTER  = RV                                 # SEs clustered by BAC value, as in Section 2

def run_spec(df, spec):
    d = df.query(spec["sample"]) if spec["sample"] else df
    d = d.dropna(subset=[Y, RV, "year"] + CONTROLS)
    d = d[(d[RV] - spec["cutoff"]).abs() >= spec["donut"]]       # the donut
    yr = pd.get_dummies(d["year"], prefix="yr", drop_first=True).astype(float)
    covs = pd.concat([d[CONTROLS], yr], axis=1)                  # controls + year fixed effects
    kw = dict(c=spec["cutoff"], p=1, kernel=spec["kernel"], covs=covs, cluster=d[CLUSTER])
    if spec["h"] is not None:
        kw["h"] = spec["h"]
    res = rdrobust(d[Y], d[RV], **kw)
    n_window = int(res.N_h[0] + res.N_h[1])                      # stops inside the bandwidth
    return res, n_window
win = dwi[(dwi[RV] - MY_SPEC["cutoff"]).abs() <= 0.01]
print(win[RV].round(3).value_counts().sort_index())

Answer: 
Here is the rest of the code to organize the data: 

rows = []
for dn in [0, 0.001, MY_SPEC["donut"], 0.003]:
    res, n = run_spec(dwi, {**MY_SPEC, "donut": dn})
    rows.append({
        "donut": dn,
        "estimate": res.coef.iloc[0, 0],     # conventional estimate
        "se": res.se.iloc[0, 0],             # conventional SE, clustered by BAC
        "robust p": res.pv.iloc[2, 0],       # robust p-value, same type as Exercises 1-3
        "N in bandwidth": n,
    })
donut_table = pd.DataFrame(rows)
print(donut_table.round({"donut": 3, "estimate": 4, "se": 4, "robust p": 3}))
base = donut_table.loc[donut_table["donut"] == 0].iloc[0]
main = donut_table.loc[donut_table["donut"] == MY_SPEC["donut"]].iloc[0]
dropped = int(base["N in bandwidth"] - main["N in bandwidth"])
print(f"Stops dropped at donut {MY_SPEC['donut']}: {dropped:,} "
      f"({dropped / base['N in bandwidth']:.1%} of {int(base['N in bandwidth']):,} in the bandwidth)")
print(f"Change in estimate vs baseline: {main['estimate'] - base['estimate']:+.4f}")
print(f"Donut SE / baseline SE: {main['se']:.4f} / {base['se']:.4f}")

Your results suggest that the estimated deterrence effect is fairly robust to excluding observations immediately around the 0.08 cutoff. As you increase the donut width, the estimated effect remains close to the original estimate, although the standard errors increase and statistical significance weakens.
1. What the BAC distribution tells you
Your first table shows the number of observations at each BAC value between 0.070 and 0.090.

A few things stand out:
- There are 998 observations at exactly 0.080.
- The number of observations generally increases as BAC rises above 0.080.
- There is no obvious, dramatic spike at exactly 0.080 compared with neighboring values.
This is useful for examining whether there might be unusual bunching around the cutoff. However, the counts alone do not establish whether drivers manipulated or sorted around the threshold. They also reflect the underlying distribution of BAC measurements.
2. What the different donut widths tell you
Your RD estimates after excluding increasingly wide intervals around 0.08.
Donut width	Estimate	SE	Robust p-value	N
0.000	-0.0222	0.0042	0.008	93,899
0.001	-0.0222	0.0042	0.013	92,901
0.002	-0.0231	0.0045	0.035	90,877
0.003	-0.0240	0.0049	0.057	88,739

There are three important findings:

A. The estimated effect barely changes.

Your baseline estimate is -0.0222. At a donut width of 0.002, it becomes -0.0231, a difference of only 0.0009.

Even after excluding observations within 0.003 of the cutoff, the estimate is -0.0240. This suggests that the negative discontinuity is not being driven entirely by observations immediately around 0.08.

B. Standard errors increase as the donut gets wider.

The standard error increases from 0.0042 to 0.0049. This is expected because removing observations reduces the sample available for estimation, making the estimate less precise

**[Evaluate]** I ran the given code in the block and it created a organized list of the results. The interpretation was helpful in understanding what it means in terms of the different donut sizes. 
```

    
## Section 4: Does the finding hold?
```
### Entry: Grammer editing
- Model and version: Claude Sonnet 5.5
- Date: 11/19/2026

**[Prep]** Pretend you are replicating Hansen (2015) study on drink driving and writing a report.
**[Request]** Can you make sure my grammer is good and that the sentences make sense. Also let me know if there are any areas where I can explain more.
**[Iterate]** Don't rewrite it, just show me places I can improve. Only give me edits from section 3, 4 and my conclusion.
**[Evaluate]** I read what claude said and made sure it made sense and followed along with what i was saying in my paper before going ahead with the suggested edits. 

"Patterns across the paper

Informal register. “Basically,” “a bit,” “a little,” “story,” and “you can’t just compare” show up repeatedly, and contractions appear in Section 4. For a graded paper, pick one register and keep it. I’d drop second person (“you”) and swap “story” for “argument” or “claim.”
“If” vs. “whether.” “Asks if it depends,” “to see if,” and “and if the evidence holds” should all be “whether.”

Section 3

The heading “Robustness: where you go beyond” is leftover template text. Rename it.
“Each of these four exercise below” should be “exercises.”
The Exercise 4 preview sentence runs on. Add a comma: “the readings closest to the cutoff, which are more likely to be recorded incorrectly.”
Exercise 1: “Hansen was right that...” overclaims for a robustness check. Say the result doesn’t depend on his bandwidth. “Looks small next to the estimate” is ambiguous. “Are also not a clean test” reads better as “are not a clean test either.”
Exercise 2: “I will treat the 0.05 placebo as inconclusive” should be present tense, matching the rest.
Exercise 3: “robust p-0.203” has a hyphen where “=” belongs. “The effect still exists” is vague. “Remains” is better.
Exercise 4: “N, the full sample left after the donut” is awkward. “The difference of about 0.001 which is smaller...” needs a comma before “which.” “If readings were rounded or recorded incorrectly removing them...” needs a comma after “incorrectly.” “It did not” has an unclear referent.

Section 4 and Conclusion

Headings are inconsistent (“Section 1: Positioning,” “Section 2. Direct Replication,” “Section: 4. Does...”). Standardize them.
“On the methods. I think...” is a fragment-style opener. Fold it into the sentence.
“Still, rdrobust warned me...” has a muddled contrast, since the warning is about your analysis and the preceding sentence is about Hansen. “Too confident” is better as “too small” or “overstate precision.”
Consistency: Section 2 says the controls are not driving the gap, but the Conclusion says you traced it to “sample and control differences.” Make these agree.
“Where the penalty also raises future penalties” is repetitive. Try “where crossing the cutoff also raises penalties for future offenses.”
The Conclusion cuts off at “methods designed for discrete.” That may be a paste issue, but finish the sentence."

What I changed: I changed a lot of these grammer and wording edits to make my paper sound stronger and easier to read.
```

## Section 5: Peer audit

TODO

## Section 6: Your own repository

TODO

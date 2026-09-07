# `binspwc`

The `binspwc` function in `Python/binsreg/src/binsreg/binspwc.py` implements data-driven pairwise group comparisons for binscatter estimators. It tests whether two subgroup curves are equal or ordered under a chosen estimation method, while supporting automatic bin selection, covariate adjustment, robust inference, and subgroup-specific or shared binning.

## Overview

`binspwc` is designed for:

- pairwise comparison of binscatter curves across groups
- least-squares, quantile-regression, or GLM-based estimation
- one-sided or two-sided group comparison tests
- covariate adjustment via `w`
- cluster-robust and heteroskedastic-robust inference
- data-driven bin selection when `bynbins` is not fixed by the user

It is the pairwise-comparison companion to `binsreg`, `binsqreg`, `binsglm`, and `binstest`.

## Function signature

```python
binspwc(
    y,
    x,
    w=None,
    data=None,
    estmethod="reg",
    dist=None,
    link=None,
    quantile=None,
    deriv=0,
    at=None,
    nolink=False,
    by=None,
    pwc=None,
    testtype="two-sided",
    lp=np.inf,
    bins=None,
    bynbins=None,
    binspos="qs",
    pselect=None,
    sselect=None,
    binsmethod="dpi",
    nbinsrot=None,
    samebinsby=False,
    randcut=None,
    nsims=500,
    simsgrid=20,
    simsseed=None,
    vce=None,
    cluster=None,
    asyvar=False,
    dfcheck=(20, 30),
    masspoints="on",
    weights=None,
    subset=None,
    numdist=None,
    numclust=None,
    estmethodopt=None,
    **optmize,
)
```

## Inputs and parameters

### Main data inputs

- `y`: outcome variable. Can be a vector or a column name when `data` is supplied.
- `x`: regressor of interest. Can be a vector or a column name when `data` is supplied.
- `w`: optional control variables. Can be a vector, matrix, or a list of column names when `data` is supplied.
- `data`: optional pandas DataFrame containing the variables used in the model.
- `by`: group indicator for subgroup comparison.
- `at`: value(s) of `w` at which the estimated function is evaluated.
- `weights`: optional observation weights.
- `subset`: optional subset selector for observations.

### Estimation method and functional form

- `estmethod`: estimation method for the underlying binscatter fit.
  - `"reg"` for least squares
  - `"qreg"` for quantile regression
  - `"glm"` for generalized linear models
- `dist`: GLM family distribution when `estmethod="glm"`.
- `link`: link function used with `dist`.
- `quantile`: target quantile for quantile-regression comparisons.
- `nolink`: if `True`, reports the function on the inverse-link scale instead of the conditional mean.
- `deriv`: derivative order of the regression function.

### Pairwise comparison setup

- `pwc`: pairwise-comparison polynomial specification. If `True` or `None`, the default is typically `(1, 1)` unless degree/smoothness selection is requested.
- `testtype`: comparison type. `"two-sided"` tests equality, `"left"` tests `mu_1(x) <= mu_2(x)`, and `"right"` tests `mu_1(x) >= mu_2(x)`.
- `lp`: norm used for the test statistic. Default is `np.inf`.

### Binning and model selection

- `bins`: tuple `(p, s)` specifying the piecewise polynomial degree and smoothness used for data-driven partition selection.
- `bynbins`: number of bins applied to each subgroup. If `True` or `None`, bin counts are selected automatically.
- `binspos`: bin placement rule. Default is `"qs"` for quantile-spaced bins; `"es"` and manual knot locations are also supported.
- `binsmethod`: method for selecting bin count. Default is `"dpi"`; `"rot"` is an alternative.
- `nbinsrot`: initial number of bins used for DPI selection.
- `pselect`, `sselect`: candidate values used when selecting polynomial degree and smoothness.
- `randcut`: random subsample cutoff used in selection procedures for large samples.
- `samebinsby`: if `True`, forces a common binning structure across groups.
- `numdist`: number of distinct `x` values used to speed up selection.
- `numclust`: number of clusters used to speed up selection.

### Inference and robustness

- `nsims`: number of simulations used for confidence band or test approximation.
- `simsgrid`: grid resolution used in the supremum approximation.
- `simsseed`: random seed for reproducible simulations.
- `vce`: variance estimator. Defaults to the method-specific choice.
- `cluster`: cluster ID used to compute cluster-robust standard errors.
- `asyvar`: if `True`, ignores uncertainty from control variables when computing the standard error of the nonparametric component.
- `dfcheck`: minimum effective sample size checks.
- `masspoints`: rule for handling repeated values in `x`.

### Optimizer options

- `estmethodopt`: optional arguments passed to the underlying GLM or quantile-regression optimizer.
- `**optmize`: additional optimizer options passed through to the fitting backend.

## Return value

The function returns:

- `tstat`: matrix of pairwise comparison results. Each row corresponds to a group pair and includes the test statistic and group indices.
- `pval`: vector of p-values for all pairwise comparisons.
- `imse_v_rot`, `imse_b_rot`, `imse_v_dpi`, `imse_b_dpi`: IMSE selection constants.
- `options`: option metadata and sample-size diagnostics, including group sizes and selected bin counts.

## Implementation notes

The function:

1. extracts variables from `data` when column names are supplied
2. reshapes and filters inputs, including `subset` handling and missing-value removal
3. determines the estimation method and variance estimator
4. validates pairwise-comparison and binning settings
5. selects subgroup-specific or common bins as requested
6. computes pairwise binscatter comparison statistics and p-values
7. returns the results and diagnostics

If `bynbins` is not specified, `binsregselect` is used to choose the binning in a data-driven way.

## Example usage

```python
import numpy as np
from binsreg.binspwc import binspwc

x = np.random.uniform(size=500)
y = np.sin(x) + np.random.normal(size=500)
t = (np.random.uniform(size=500) > 0.5).astype(int)

out = binspwc(y, x, by=t)
print(out)
```

A more complete example with controls and a two-sided comparison:

```python
import pandas as pd
from binsreg.binspwc import binspwc

df = pd.DataFrame({
    "y": [0.5, 1.2, 2.0, 3.1, 4.0, 5.5],
    "x": [0.1, 0.4, 0.8, 1.2, 1.8, 2.5],
    "w1": [1.0, 0.5, 1.2, 0.8, 1.7, 2.0],
    "group": ["A", "A", "B", "B", "B", "A"],
})

result = binspwc(
    y="y",
    x="x",
    w="w1",
    data=df,
    by="group",
    testtype="two-sided",
    pwc=(1, 1),
    nbynbins=10,
)
```

## Notes

- `testtype="left"` and `testtype="right"` require `lp=np.inf`.
- `dist` and `link` apply when `estmethod="glm"`.
- If `data` is supplied, strings can be used for `x`, `y`, `w`, `by`, `cluster`, and `weights`.
- `samebinsby=True` forces a common binning structure across subgroups.

## See also

- `binsregselect`: data-driven binscatter partition selection
- `binsreg`: least-squares binscatter estimation and plotting
- `binsqreg`: binscatter quantile regression
- `binsglm`: binscatter generalized linear models
- `binstest`: binscatter-based hypothesis testing

## References

The implementation follows the binscatter methods and testing framework developed in:

- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2024)
- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2026)

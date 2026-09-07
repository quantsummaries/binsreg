# `binstest`

The `binstest` function in `Python/binsreg/src/binsreg/binstest.py` implements binscatter-based hypothesis tests for parametric specification and shape restrictions in a regression function. It is designed for testing whether the conditional mean (or other estimands) follows a particular parametric form or satisfies a nonparametric shape restriction, while allowing data-driven bin selection, robust variance estimation, and inference based on binscatter approximations.

## Overview

`binstest` is used to evaluate hypotheses such as:

- a parametric model is correctly specified
- a function satisfies a shape restriction such as monotonicity, boundedness, or a one-/two-sided null boundary condition
- a nonparametric binscatter fit is statistically distinguishable from a simpler model

The function wraps binscatter estimation with test statistics computed from local polynomial approximations, robust bias correction, and simulation-based confidence bands or supremum-norm tests when needed. It is typically paired with `binsreg`, `binsregselect`, `binsqreg`, `binsglm`, and `binspwc`.

## Function signature

```python
binstest(
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
    testmodel=None,
    testmodelparfit=None,
    testmodelpoly=None,
    testshape=None,
    testshapel=None,
    testshaper=None,
    testshape2=None,
    lp=np.inf,
    bins=None,
    nbins=None,
    pselect=None,
    sselect=None,
    binspos="qs",
    binsmethod="dpi",
    nbinsrot=None,
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
- `weights`: optional observation weights.
- `subset`: optional subset selector to use a restricted sample.
- `cluster`: optional cluster identifiers for cluster-robust inference.

### Estimation method and functional form

- `estmethod`: estimation method for the underlying binscatter fit. Accepted values are:
  - `"reg"` for least-squares regression (default)
  - `"qreg"` for quantile regression
  - `"glm"` for generalized linear models
- `dist`: distribution for GLM estimation; examples include `"Gaussian"`, `"Binomial"`, `"Gamma"`, and `"Poisson"`.
- `link`: link function associated with `dist`, such as `"identity"`, `"logit"`, `"log"`, or `"probit"`.
- `quantile`: target quantile for quantile-regression tests. Must be in `(0, 1)`.
- `deriv`: derivative order of the regression function being tested or estimated. Defaults to `0`.
- `at`: evaluation point(s) for `w` at which the function is evaluated. Common choices are `"mean"`, `"median"`, or `"zero"`, or a vector/data frame of covariate values.
- `nolink`: if `True`, the non-link-transformed index is reported instead of the conditional mean.

### Parametric model specification tests

The tests in `binstest` target departures from parametric models or restrictions on the estimated nonparametric function.

- `testmodel`: specifies the parametric model used in the specification test. If it is a tuple `(p, s)`, the test uses a piecewise polynomial of degree `p` with `s` smoothness constraints. If `True` or `None`, the default is typically `(1, 1)` unless degree/smoothness selection is requested via `pselect` or `sselect`.
- `testmodelparfit`: evaluation grid and fitted values for a parametric benchmark model(s) to be tested against.
- `testmodelpoly`: degree of a global polynomial model to be tested against.
- `lp`: norm used for the test statistic. The default is `np.inf`, which corresponds to the supremum-norm version. Other positive values `p >= 1` are allowed.
- `pselect`: candidate polynomial degrees for selecting the test model.
- `sselect`: candidate smoothness choices for selecting the test model.

### Shape restriction tests

- `testshape`: general shape restriction test specification, using a piecewise polynomial of degree `p` and smoothness `s`.
- `testshapel`: vector of left-side null boundary values for hypotheses of the form `H0: sup_x mu(x) <= a`.
- `testshaper`: vector of right-side null boundary values for hypotheses of the form `H0: inf_x mu(x) >= a`.
- `testshape2`: vector of null boundary values for two-sided tests of the form `H0: sup_x |mu(x) - a| = 0`.

### Binning and model selection

- `bins`: tuple `(p, s)` specifying the piecewise polynomial degree and smoothness used for data-driven partition selection.
- `nbins`: number of bins for partitioning. If `True` or `None`, it is selected by `binsregselect` in a data-driven way whenever appropriate.
- `binspos`: bin placement rule. Default is `"qs"` for quantile-spaced bins, with `"es"` available for evenly spaced bins.
- `binsmethod`: method to select the number of bins. Default is `"dpi"` (direct plug-in). `"rot"` is an alternative rule-of-thumb selector.
- `nbinsrot`: initial number of bins used for the DPI selector.
- `randcut`: subsample cutoff used in selection procedures for large samples.
- `numdist`: number of distinct `x` values used to speed up selection computations.
- `numclust`: number of clusters used to speed up selection computations.

### Inference and robustness

- `nsims`: number of Monte Carlo draws used to approximate the distribution of the test statistic or confidence band.
- `simsgrid`: number of evaluation points within each bin used for the supremum approximation in confidence bands or tests.
- `simsseed`: random seed for reproducible simulation draws.
- `vce`: variance estimator. Options include `"const"`, `"HC0"`, `"HC1"`, `"HC2"`, and `"HC3"`; the default is `"HC1"` when unspecified.
- `asyvar`: if `True`, the standard error calculation omits uncertainty due to control variables.
- `dfcheck`: minimum effective sample size checks used to guard against overfitting or weak identification. Default is `(20, 30)`.
- `masspoints`: rules for handling repeated values in `x` (`"on"`, `"noadjust"`, `"nolocalcheck"`, `"off"`, `"veryfew"`).
- `estmethodopt`: optional arguments to the optimizer used by GLM or quantile-regression methods.

## Return value

`binstest` returns a dictionary-like collection of results, including the outputs of the different hypothesis tests and the auxiliary selection quantities used to construct the tests.

Typical entries include:

- `testshapeL`: results for left-sided shape restriction tests, including `val`, `stat`, and `pval`
- `testshapeR`: results for right-sided shape restriction tests, including `val`, `stat`, and `pval`
- `testshape2`: results for two-sided shape restriction tests, including `val`, `stat`, and `pval`
- `testpoly`: results for polynomial specification tests, including `val`, `stat`, and `pval`
- `testmodel`: results for parametric model comparison tests, including `val`, `stat`, and `pval`
- `imse_v_rot`: variance constant in the IMSE criterion for ROT selection
- `imse_b_rot`: bias constant in the IMSE criterion for ROT selection
- `imse_v_dpi`: variance constant in the IMSE criterion for DPI selection
- `imse_b_dpi`: bias constant in the IMSE criterion for DPI selection
- `options`: a record of the options used, together with sample-size diagnostics such as `n`, number of distinct `x` values, number of clusters, and selected number of bins

## Implementation notes

The function performs the following main steps:

1. validates and prepares the data (`x`, `y`, `w`, cluster, weights, subset)
2. drops missing values in all relevant inputs
3. determines the estimation method and variance estimator
4. sets up the testing tasks for parametric specification and/or shape restrictions
5. selects the partitioning and polynomial/smoothness choices when `nbins`, `pselect`, or `sselect` are unspecified
6. computes the binscatter-based test statistics and p-values
7. returns the results and diagnostics for interpretation or plotting

A key feature of the implementation is that if the binning scheme is not specified by the user, it relies on the companion function `binsregselect` to choose the bins in a data-driven, IMSE-optimal way.

## Example usage

```python
import numpy as np
from binsreg.binstest import binstest

x = np.random.uniform(size=500)
y = np.sin(x) + np.random.normal(size=500)

out = binstest(y, x, testmodelpoly=1)
print(out)
```

A more complete example with controls and a parametric model test:

```python
import pandas as pd
from binsreg.binstest import binstest

df = pd.DataFrame({
    "y": [0.5, 1.2, 2.0, 3.1, 4.0, 5.5],
    "x": [0.1, 0.4, 0.8, 1.2, 1.8, 2.5],
    "w1": [1.0, 0.5, 1.2, 0.8, 1.7, 2.0],
})

result = binstest(
    y="y",
    x="x",
    w="w1",
    data=df,
    testmodel=(1, 1),
    nbins=10,
    level=95,
)
```

## Notes

- The default behavior is data-driven and robust, especially for large samples.
- `testmodel`, `testshape`, and related restrictions can be used to evaluate parametric adequacy or shape restrictions using binscatter approximations.
- The supremum-norm (`lp = np.inf`) is especially important for one-sided shape restriction tests.
- When `estmethod="glm"`, `dist` and optionally `link` must be specified appropriately.
- If `data` is supplied, strings can be used for `x`, `y`, `w`, `cluster`, and `weights`.

## See also

- `binsregselect`: data-driven binscatter partition selection
- `binsreg`: least-squares binscatter estimation and plotting
- `binsqreg`: binscatter quantile regression
- `binsglm`: binscatter generalized linear models
- `binspwc`: pairwise comparison of binscatter estimators

## References

The implementation follows the binscatter methods and testing framework developed in:

- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2024)
- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2026)

The method is designed for shape-restricted and specification-testing procedures in nonparametric regression using binscatter methods with robust bias correction.

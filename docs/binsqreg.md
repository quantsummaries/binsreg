# `binsqreg`

The `binsqreg` function in `Python/binsreg/src/binsreg/binsqreg.py` implements data-driven binscatter quantile regression with robust inference and plotting. It estimates conditional quantiles of `y` given `x`, optionally adjusting for controls `w`, and supports binscatter-style points, fitted lines, confidence intervals, confidence bands, subgroup analysis, and automatic bin selection.

## Overview

`binsqreg` is designed for:

- flexible quantile regression with binscatter-style partitioning
- plot construction with dots, fitted lines, confidence intervals, and uniform confidence bands
- covariate adjustment via `w`
- subgroup analysis via `by`
- heteroskedastic-robust and cluster-robust inference
- data-driven bin selection when `nbins` is not fixed by the user

It is the quantile-regression companion to `binsreg` and is commonly used alongside `binsregselect`, `binstest`, `binsglm`, and `binspwc`.

## Function signature

```python
binsqreg(
    y,
    x,
    w=None,
    data=None,
    at=None,
    quantile=0.5,
    deriv=0,
    dots=None,
    dotsgrid=0,
    dotsgridmean=True,
    line=None,
    linegrid=20,
    ci=None,
    cigrid=0,
    cigridmean=True,
    cb=None,
    cbgrid=20,
    polyreg=None,
    polyreggrid=20,
    polyregcigrid=0,
    by=None,
    bycolors=None,
    bysymbols=None,
    bylpatterns=None,
    legendTitle=None,
    legendoff=False,
    nbins=None,
    binspos="qs",
    binsmethod="dpi",
    nbinsrot=None,
    samebinsby=False,
    randcut=None,
    pselect=None,
    sselect=None,
    nsims=500,
    simsgrid=20,
    simsseed=None,
    vce="nid",
    cluster=None,
    asyvar=False,
    level=95,
    noplot=False,
    dfcheck=(20, 30),
    masspoints="on",
    weights=None,
    subset=None,
    plotxrange=None,
    plotyrange=None,
    **optimize,
)
```

## Inputs and parameters

### Main data inputs

- `y`: outcome variable. Can be a vector or a column name when `data` is supplied.
- `x`: regressor of interest. Can be a vector or a column name when `data` is supplied.
- `w`: optional control variables. Can be a vector, matrix, or a list of column names when `data` is supplied.
- `data`: optional pandas DataFrame containing the variables used in the model.
- `quantile`: quantile to estimate, strictly between 0 and 1.
- `at`: value(s) of `w` at which the estimated function is evaluated. Common settings are `"mean"`, `"median"`, or `"zero"`.
- `deriv`: derivative order to estimate. `0` estimates the function itself.
- `weights`: optional observation weights.
- `subset`: optional subset selector for observations.

### Plotting controls

- `dots`: binscatter dots specification. Typical usage is `dots=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint. If `dots=True`, a default binscatter is used.
- `dotsgrid`: number of evaluation points per bin for plotting dots.
- `dotsgridmean`: whether to include the bin-mean evaluation point in the plotted dots.
- `line`: line overlay specification, used for a smoothed fitted line. The syntax is `line=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint.
- `linegrid`: grid resolution used for evaluating the line.
- `ci`: pointwise confidence interval specification.
- `cigrid`: grid resolution used for confidence interval construction.
- `cigridmean`: whether to include bin-mean confidence intervals.
- `cb`: uniform confidence band specification.
- `cbgrid`: grid resolution used for confidence band construction.
- `polyreg`: optional global polynomial regression to overlay.
- `polyreggrid`: grid resolution for the polynomial fit.
- `polyregcigrid`: grid resolution for polynomial confidence intervals.
- `noplot`: if `True`, suppresses plot creation while still returning plotting data.
- `plotxrange` and `plotyrange`: x- and y-axis ranges for the plot.

### Subgroup analysis and legend styling

- `by`: grouping variable for subgroup analysis. Separate quantile curves are estimated for each group.
- `bycolors`, `bysymbols`, `bylpatterns`: styling controls for subgroup series.
- `legendTitle`: title shown in the plot legend.
- `legendoff`: if `True`, hides the legend.
- `samebinsby`: if `True`, forces a common set of bins across groups.

### Binning and model selection

- `nbins`: number of bins. If omitted or set to `True`, the number of bins is selected automatically.
- `binspos`: binning position rule. Default is `"qs"` for quantile-spaced bins; `"es"` is also supported.
- `binsmethod`: selection method for bin count. Default is `"dpi"` (direct plug-in); `"rot"` is an alternative.
- `nbinsrot`: initial number of bins used in constructing the DPI selector.
- `pselect`, `sselect`: candidate values used when selecting the polynomial degree and smoothness constraints.
- `randcut`: random subsample cutoff used in selection procedures for large samples.

### Inference and robustness

- `nsims`: number of simulations used for confidence band construction.
- `simsgrid`: grid resolution used in the supremum approximation for confidence bands.
- `simsseed`: random seed for reproducible simulations.
- `vce`: variance estimator. The default is `"nid"`.
- `cluster`: cluster ID used to compute cluster-robust standard errors.
- `asyvar`: if `True`, ignores uncertainty from control variables when computing the standard error of the nonparametric component.
- `level`: nominal confidence level for intervals/bands, expressed as a percentage.
- `dfcheck`: minimum effective sample size checks used to guard against overfitting or weak identification.
- `masspoints`: rule for handling repeated values (mass points) in `x`.

### Optimizer options

- `**optimize`: optional arguments passed to the `statsmodels` QuantReg optimizer.

## Return value

The function returns:

- `bins_plot`: a `ggplot` object suitable for display or saving.
- `data_plot`: a nested list of plotting datasets, one per subgroup if `by` is used.

The plotting data may include DataFrames for:

- `dots`: fitted dot estimates
- `line`: fitted line estimates
- `ci`: pointwise confidence intervals
- `cb`: uniform confidence band
- `poly`: global polynomial regression fit
- `polyci`: polynomial regression confidence intervals
- `data_bin`: binning structure summary
- `imse_v_rot`, `imse_b_rot`, `imse_v_dpi`, `imse_b_dpi`: IMSE selection constants
- `cval_by`: critical values for confidence bands by group
- `options`: option metadata and sample-size diagnostics

## Implementation notes

The function:

1. extracts variables from `data` when column names are supplied
2. reshapes and filters inputs, including `subset` handling and missing-value removal
3. determines subgroup structure and plotting defaults
4. validates the requested binning, polynomial, and inference settings
5. selects bins and polynomial degrees when needed
6. computes quantile-regression-based binscatter estimates and associated uncertainty
7. returns plotting objects and diagnostic data

If `nbins` is not specified, `binsregselect` is used to choose the binning in a data-driven way.

## Example usage

```python
import numpy as np
from binsreg.binsqreg import binsqreg

x = np.random.uniform(size=500)
y = np.sin(x) + np.random.normal(size=500)

out = binsqreg(y, x, quantile=0.5)
print(out)
```

A more complete example with controls and subgroup analysis:

```python
import pandas as pd
from binsreg.binsqreg import binsqreg

df = pd.DataFrame({
    "y": [0.5, 1.2, 2.0, 3.1, 4.0, 5.5],
    "x": [0.1, 0.4, 0.8, 1.2, 1.8, 2.5],
    "w1": [1.0, 0.5, 1.2, 0.8, 1.7, 2.0],
    "group": ["A", "A", "B", "B", "B", "A"],
})

result = binsqreg(
    y="y",
    x="x",
    w="w1",
    data=df,
    by="group",
    quantile=0.5,
    dots=(0, 0),
    line=(1, 1),
    ci=(1, 1),
    cb=(1, 1),
    nbins=10,
)
```

## Notes

- `quantile` must be strictly between 0 and 1.
- `samebinsby=True` forces a common binning structure across subgroups.
- If `data` is supplied, strings can be used for `x`, `y`, `w`, `by`, `cluster`, and `weights`.
- `vce` defaults to `"nid"` for quantile-regression inference.

## See also

- `binsregselect`: data-driven binscatter partition selection
- `binsreg`: least-squares binscatter estimation and plotting
- `binsglm`: binscatter generalized linear models
- `binstest`: binscatter-based hypothesis testing
- `binspwc`: pairwise comparisons of binscatter estimators

## References

The implementation follows the binscatter methods and testing framework developed in:

- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2024)
- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2026)

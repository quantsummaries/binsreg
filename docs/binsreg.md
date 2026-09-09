# `binsreg`

The `binsreg` function in `Python/binsreg/src/binsreg/binsreg.py` implements data-driven binscatter least squares regression with robust inference procedures and plotting. It estimates the conditional mean relationship between an outcome `y` and a regressor of interest `x`, optionally adjusting for control variables `w`, with binscatter-style partitioning and inference based on the methods discussed in Cattaneo, Crump, Farrell, and Feng.

## Overview

`binsreg` is designed for:

- flexible nonparametric estimation of the mean relationship between `y` and `x`
- binscatter plots with points, fitted lines, confidence intervals, and uniform confidence bands
- covariate adjustment via `w`
- subgroup analysis via `by`
- clustered and heteroskedastic-robust inference
- data-driven bin selection when `nbins` is left unspecified

It is the core least-squares implementation in the `binsreg` package and is often paired with `binsregselect`, `binsqreg`, `binsglm`, `binstest`, and `binspwc`.

## Function signature

```python
binsreg(
    y,
    x,
    w=None,
    data=None,
    at=None,
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
    vce="HC1",
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
)
```

## Inputs and parameters

### Main data inputs

- `y`: outcome variable. Can be a vector or a column name when `data` is supplied.
- `x`: regressor of interest. Can be a vector or a column name when `data` is supplied.
- `w`: optional control variables. Can be a vector, matrix, or a list of column names when `data` is supplied.
- `data`: optional pandas DataFrame containing the variables used in the model.
- `at`: value(s) of `w` at which to evaluate the estimated function. Common settings are `"mean"`, `"median"`, or `"zero"`.
- `deriv`: derivative order to estimate. `0` estimates the function itself; higher values estimate derivatives.

### Plotting controls

- `dots`: binscatter dots specification. Typical usage is `dots=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint. The `dots=(p, s)` option chooses the polynomial form used for the fitted binscatter estimate over `x`: `x` is the variable being binned and evaluated, while `y` is the variable being estimated/conditioned on. So `dots` is about modeling `E[y | x, w]` (or a local polynomial approximation to it). If `dots=True`, a default binscatter is used.
- `dotsgrid`: number of evaluation points per bin for plotting dots.
- `dotsgridmean`: whether to include the bin-mean evaluation point in the plotted dots.
- `line`: line overlay specification, used for a smoothed fitted line. The syntax is `line=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint. This is not a global linear model; it fits a piecewise polynomial over the bins. For example, `line=(3, 3)` fits a cubic piecewise polynomial with smoothness 3 across the bins, which can appear nonlinear. A linear overlay would use `line=(1, 1)` or `line=(0, 0)` depending on the desired smoothness.
- `linegrid`: grid resolution used for evaluating the line.
- `ci`: pointwise confidence interval specification. The syntax is `ci=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint used to construct the interval. This is not a generic `"confidence interval"` flag; it defines the local polynomial approximation whose uncertainty is being summarized. For example, `ci=(3, 3)` builds intervals around a cubic piecewise polynomial with smoothness 3, which can look curved or wavy. If you want a simpler interval around a less flexible fit, use a lower degree such as `ci=(1, 1)` or omit `ci` entirely.
- `cigrid`: grid resolution used for confidence interval construction.
- `cigridmean`: whether to include bin-mean confidence intervals.
- `cb`: uniform confidence band specification. The syntax is `cb=(p, s)`, where `p` is the polynomial degree and `s` is the smoothness constraint used to form the band. This is a global uncertainty band over the whole estimated curve, not a single-point interval. For example, `cb=(3, 3)` builds a uniform confidence band around a cubic piecewise polynomial with smoothness 3, which can look smooth but still nonlinear. A simpler band is obtained with lower-order choices such as `cb=(1, 1)`.
- `cbgrid`: grid resolution used for confidence band construction.
- `polyreg`: optional global polynomial regression to overlay.
- `polyreggrid`: grid resolution for the polynomial fit.
- `polyregcigrid`: grid resolution for polynomial confidence intervals.
- `noplot`: if `True`, suppresses plot creation while still returning plotting data.
- `plotxrange` and `plotyrange`: x- and y-axis ranges for the plot.

### Subgroup analysis and legend styling

- `by`: grouping variable for subgroup analysis. This is the option used to estimate and plot separate binscatter curves for different groups in the data. For example, `by="gender"` fits one curve for each gender and plots them on the same axes. Unlike `cluster`, this is not about dependence in the variance calculation; it is about estimating separate regression curves for different subpopulations.
- `bycolors`, `bysymbols`, `bylpatterns`: stylings for subgroup series.
- `legendTitle`: title shown in the plot legend.
- `legendoff`: if `True`, hides the legend.
- `samebinsby`: if `True`, forces a common set of bins across groups.

### Binning and model selection

- `nbins`: number of bins. If omitted or set to `True`, the number of bins is selected automatically.
- `binspos`: binning position rule. Default is `"qs"` for quantile-spaced bins; alternatives include `"es"` or a user-supplied vector of knot locations.
- `binsmethod`: selection method for bin count. Default is `"dpi"` (direct plug-in); `"rot"` is an alternative.
- `nbinsrot`: initial number of bins used in constructing the DPI selector.
- `pselect`, `sselect`: candidate values used when selecting the polynomial degree and smoothness constraints for the estimated binscatter.
- `randcut`: random subsample cutoff used in selection procedures for large samples.

### Inference and robustness

- `nsims`: number of simulations used for confidence band construction.
- `simsgrid`: grid resolution used in the supremum approximation for confidence bands.
- `simsseed`: random seed for reproducible simulations.
- `vce`: variance estimator. Options include `"const"`, `"HC0"`, `"HC1"`, `"HC2"`, and `"HC3"`; default is `"HC1"`.
- `cluster`: cluster ID used to compute cluster-robust standard errors. This adjusts the variance estimator for dependence among observations within each cluster, such as repeated measurements or units in the same county, school, or firm. It does not split the sample into separate curves; it only changes how uncertainty is estimated.
- `asyvar`: if `True`, ignores uncertainty from control variables when computing the standard error of the nonparametric component.
- `level`: nominal confidence level for intervals/bands, expressed as a percentage (default `95`).
- `weights`: optional observation weights.
- `subset`: optional subset selector for observations.
- `dfcheck`: minimum effective sample size checks used to guard against overfitting or weak identification.
- `masspoints`: rule for handling repeated values (mass points) in `x`, including options such as `"on"`, `"off"`, and `"veryfew"`.

## Return value

The function returns a `bins_plot` object and a `data_plot` structure.

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
- additional diagnostic entries such as `imse_v_rot`, `imse_b_rot`, `imse_v_dpi`, `imse_b_dpi`, and `cval_by`

In addition to the plotted information, the returned metadata stores options and sample-size diagnostics such as:

- total sample size by group
- number of distinct `x` values by group
- number of clusters by group
- number of bins by group
- degree and smoothness choices for each plot element

## Example usage

```python
import numpy as np
from binsreg import binsreg

x = np.random.uniform(size=500)
y = np.sin(x) + np.random.normal(size=500)

out = binsreg(y, x)
print(out)

# or, if you need more detailed object access
plot_obj, plot_data = out
```

A more complete example with `data` and controls:

```python
import pandas as pd
from binsreg import binsreg

df = pd.DataFrame({
    "y": [0.5, 1.2, 2.0, ...],
    "x": [0.1, 0.4, 0.8, ...],
    "w1": [1.0, 0.5, 1.2, ...],
    "group": ["A", "A", "B", "B", ...],
})

binsreg(
    y="y",
    x="x",
    w="w1",
    data=df,
    by="group",
    dots=(0, 0),
    line=(1, 1),
    ci=(1, 1),
    cb=(1, 1),
    nbins=20,
    level=95,
)
```

## Notes

- The function is designed to work well for large samples where data-driven bin selection is beneficial.
- For full inference, the recommended practice is to specify a sufficiently large number of simulations and grid points, especially when constructing confidence bands.
- When `by` is specified, a common binning structure can be forced with `samebinsby=True`.
- If `data` is provided, the function accepts variable names as strings for `x`, `y`, `w`, and `by`.

## See also

- `binsregselect`: data-driven binscatter bin selection
- `binsqreg`: binscatter quantile regression
- `binsglm`: binscatter generalized linear models
- `binstest`: binscatter-based hypothesis testing
- `binspwc`: pairwise group comparisons of binscatter estimators

## References

The implementation follows the binscatter methods described in:

- Cattaneo, Crump, Farrell, and Feng (2024): "On Binscatter"
- Cattaneo, Crump, Farrell, and Feng (2026): "Nonlinear Binscatter Methods"

These methods underlie the robust inference and confidence-band logic used within `binsreg`.

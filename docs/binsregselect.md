# `binsregselect`

The `binsregselect` function in `Python/binsreg/src/binsreg/binsregselect.py` implements data-driven IMSE-optimal binning and partition selection for binscatter estimation. It chooses the number of bins, and in some modes the polynomial degree and smoothness, to support downstream estimation and inference in `binsreg`, `binsqreg`, `binsglm`, `binstest`, and `binspwc`.

## Overview

`binsregselect` is designed for:

- IMSE-based selection of the number of bins
- optional degree and smoothness selection for binscatter estimators
- quantile-spaced or evenly spaced bin placement
- clustered, weighted, and subset-aware selection
- handling repeated values (mass points) in `x`

It is the common selection utility used by the package’s estimation and testing routines.

## Function signature

```python
binsregselect(
    y,
    x,
    w=None,
    data=None,
    deriv=0,
    bins=None,
    pselect=None,
    sselect=None,
    binspos="qs",
    nbins=None,
    binsmethod="dpi",
    nbinsrot=None,
    simsgrid=20,
    savegrid=False,
    vce="HC1",
    useeffn=None,
    randcut=None,
    cluster=None,
    dfcheck=(20, 30),
    masspoints="on",
    weights=None,
    subset=None,
    norotnorm=False,
    numdist=None,
    numclust=None,
)
```

## Inputs and parameters

### Main data inputs

- `y`: outcome variable. Can be a vector or a column name when `data` is supplied.
- `x`: regressor of interest. Can be a vector or a column name when `data` is supplied.
- `w`: optional control variables. Can be a vector, matrix, or a list of column names when `data` is supplied.
- `data`: optional pandas DataFrame containing the variables used in the model.
- `deriv`: derivative order of the regression function.
- `weights`: optional observation weights.
- `subset`: optional subset selector for observations.
- `cluster`: optional cluster identifiers.

### Binning and selection

- `bins`: tuple `(p, s)` specifying the piecewise polynomial degree and smoothness used for IMSE selection.
- `pselect`, `sselect`: candidate degree and smoothness values used when selecting bins.
- `binspos`: bin placement rule. Default is `"qs"` for quantile-spaced bins; `"es"` is also supported.
- `nbins`: number of bins. If `True` or `None`, the function selects the number of bins automatically.
- `binsmethod`: selection method for bin count. Default is `"dpi"`; `"rot"` is an alternative.
- `nbinsrot`: initial number of bins used in constructing the DPI selector.
- `useeffn`: effective sample size used for selection.
- `randcut`: random subsample cutoff used in selection procedures for large samples.
- `norotnorm`: if `True`, uses a uniform density instead of a normal density for ROT selection.
- `numdist`: number of distinct `x` values used to speed up selection.
- `numclust`: number of clusters used to speed up selection.

### Inference and robustness

- `simsgrid`: grid resolution used in supremum-type approximations.
- `savegrid`: if `True`, returns the evaluation grid.
- `vce`: variance estimator. Options include `"const"`, `"HC0"`, `"HC1"`, `"HC2"`, and `"HC3"`.
- `dfcheck`: minimum effective sample size checks.
- `masspoints`: rule for handling repeated values in `x`.

## Return value

The function returns a collection of selection outputs and diagnostics, including:

- `nbinsrot_poly`, `nbinsrot_regul`, `nbinsrot_uknot`: ROT-selected bin counts
- `nbinsdpi`, `nbinsdpi_uknot`: DPI-selected bin counts
- `prot_poly`, `prot_regul`, `prot_uknot`: ROT-selected polynomial degrees
- `pdpi`, `pdpi_uknot`: DPI-selected polynomial degrees
- `srot_poly`, `srot_regul`, `srot_uknot`: ROT-selected smoothness constraints
- `sdpi`, `sdpi_uknot`: DPI-selected smoothness constraints
- `imse_v_rot`, `imse_b_rot`, `imse_v_dpi`, `imse_b_dpi`: IMSE constants
- `int_result`: intermediate selection results, including degree/smoothness grids and candidate bin counts
- `options`: option metadata and sample-size diagnostics
- `knot`: selected or user-specified knot locations
- `data_grid`: optional grid data frame when `savegrid=True`

## Implementation notes

The function:

1. extracts variables from `data` when column names are supplied
2. reshapes and filters inputs, including `subset` handling and missing-value removal
3. validates binning, degree, and smoothness inputs
4. determines the effective sample size, accounting for mass points and clusters
5. optionally subsamples large data using `randcut`
6. computes IMSE-based ROT and DPI selection criteria
7. returns the chosen binning and supporting diagnostics

When `masspoints="on"`, the routine may cluster at the mass-point level to stabilize the selection procedure.

## Example usage

```python
import numpy as np
from binsreg.binsregselect import binsregselect

x = np.random.uniform(size=500)
y = np.sin(x) + np.random.normal(size=500)

out = binsregselect(y, x)
print(out)
```

A more complete example with controls:

```python
import pandas as pd
from binsreg.binsregselect import binsregselect

df = pd.DataFrame({
    "y": [0.5, 1.2, 2.0, 3.1, 4.0, 5.5],
    "x": [0.1, 0.4, 0.8, 1.2, 1.8, 2.5],
    "w1": [1.0, 0.5, 1.2, 0.8, 1.7, 2.0],
})

result = binsregselect(
    y="y",
    x="x",
    w="w1",
    data=df,
    binsmethod="dpi",
    binspos="qs",
)
```

## Notes

- `binspos="qs"` uses quantile-spaced bins; `binspos="es"` uses evenly spaced bins.
- `binsmethod="dpi"` is the default IMSE-optimal selector; `binsmethod="rot"` is the rule-of-thumb alternative.
- If `data` is supplied, strings can be used for `x`, `y`, `w`, `cluster`, and `weights`.

## See also

- `binsreg`: least-squares binscatter estimation and plotting
- `binsqreg`: binscatter quantile regression
- `binsglm`: binscatter generalized linear models
- `binstest`: binscatter-based hypothesis testing
- `binspwc`: pairwise comparisons of binscatter estimators

## References

The implementation follows the binscatter methods and selection framework developed in:

- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2024)
- Cattaneo, M. D., Crump, R. K., Farrell, M. H., and Feng, Y. (2026)

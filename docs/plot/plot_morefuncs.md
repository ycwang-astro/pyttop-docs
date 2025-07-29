---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Additional Plotting Functions
PyTTOP provides [basic plotting functions](plot_single.md#built-in-plotfunc) such as `plot`, `scatter`, `hist`, etc., which can be used standalone or passed to `data.plots()`. In addition to these, PyTTOP also offers some higher-level plotting functions designed for more specific use cases.

This page demonstrates typical usage patterns of these plotting functions through examples. For a complete list of supported arguments, refer to the [API reference](../api/plotfuncs).

## `refline`
[`refline()`](../api/plotfuncs.rst#pyttop.plot.refline) plots horizontal and/or vertical reference lines to mark specified `x` and `y` values. Typical usage is shown in the examples below:
```{code-cell}
from pyttop.plot import refline
import numpy as np
import matplotlib.pyplot as plt

xs = np.linspace(-.5, 1.5, 100)
ys = np.exp(xs)
plt.plot(xs, ys)

# Annotate the point (1, e) on the curve
refline(x=1, y=np.e, 
        ytxt='e', # custom label for the y value
        marker='s', style='axis',
        )

# Add a vertical line at x=0 with label positioned 75% up the y-axis
refline(x=0, 
        ypos=.75, # vertical label position (fraction of axis height)
        xfmt='.0f', # format x label with no decimal
        linestyle='--', color='k',
        );
```


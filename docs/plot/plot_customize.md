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

# Advanced Plot Control and Customization
The data and subsets defined [here](plot_subsets) will be used for demonstration on this page.

```{code-cell}
:tags: [remove-cell]
from pyttop import get_example
from pyttop.table import Data, Subset
import matplotlib.pyplot as plt

data = get_example('P1')

data.subset_group_from_values('obj_class', group_name='class')
data.subset_group_from_ranges(
    'x', [[-20, -10], [-10, 0], [0, 10], [10, 20]], # 4 bins specified for x
    group_name='x_bins',
    )
data.subset_group_from_ranges(
    'z', [[-40, 0], [0, 40]],
    group_name='z_bins',
    )
subset_A = data.get_subsets(name='obj_class=A')
subset_xbin = data.get_subsets('x_bins/x(0-10)')
```

## Setting labels for columns and subsets
You can set the labels for columns using the `set_labels()` method so that they will be used for automatic label generation:
```{code-cell}
data.set_labels(x='My label for $x$')

data.plots('scatter', cols=('x', 'y'), subsets=subset_A);
```
The labels of subsets can be specified during their definition, as described [here](../subset/subset_def.md#name-expression-and-label). They can also be modified by directly changing the `label` property:
```{code-cell}
subset_A.label = 'Class A object'
data.plots('scatter', cols=('x', 'y'), subsets=subset_A);
```
<!-- plt.scatter(data['x'][np.array(subset_A)], data['y'][np.array(subset_A)], marker='+') -->

The automatic label generation can be turned off by setting `autolabel=False`:
```{code-cell}
data.plots('scatter', cols=('x', 'y'), subsets=subset_A, autolabel=False);
```

```{note}
The automatic generation of axis labels (e.g., x and y labels) covers typical scenarios and should work reliably in most cases. However, it *might* not be accurate in some *rare* situations. It is always advisable to review what you are plotting and manually adjust axis labels if needed. 
```
 <!-- cannot guarantee correctness in some rare cases. being aware of what you are plotting, and what axis labels you should use. -->

## Handling figures and axes
By default, the `plots()` method returns the Matplotlib [Figure](https://matplotlib.org/stable/users/explain/figure/figure_intro.html) and [Axes](https://matplotlib.org/stable/users/explain/axes/axes_intro.html), allowing you to make further adjustments, such as changing axis limits, labels, and more.
```{code-cell}
fig, ax = data.plots('scatter', cols=('x', 'y'))
fig, ax
```

```{code-cell}
fig, axes = data.plots('scatter', cols=('x', 'y'), arraygroups='x_bins')
fig, axes
```

You can also pass Figure and/or Axes objects to the `fig` and `axes` arguments of `plots()`, specifying where the plot should be made. For example:
```{code-cell}
my_fig = plt.figure(figsize=(5, 5))
data.plots(
    'scatter', cols=('x', 'y'), arraygroups='x_bins',
    fig=my_fig);
```

```{code-cell}
fig, axes = plt.subplots(4, 2, figsize=(6, 10))
left_panels = axes[:, 0]
data.plots(
    'scatter', cols=('x', 'y'), arraygroups='x_bins',
    axes=left_panels)
plt.tight_layout()
```

## The axis callback
The `plots()` method accepts a callback function that is executed after plotting on each axis. This can be useful if you need to make the same adjustments to every axis. For example:
```{code-cell}
def add_text(ax): # an axis callback function takes a Matplotlib axis as input
    ax.text(20, -5, 'text')

data.plots(
    'scatter', cols=('x', 'y'), arraygroups='z_bins',
    ax_callback=add_text
    );
```

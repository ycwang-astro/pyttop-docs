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

```{code-cell}
:tags: [remove-cell]
from astropy import conf
conf.max_lines = 13
```

# Making Plots Given Subsets
It is common to plot only a subset of a dataset (e.g., when only some rows have good data quality) or to compare plots of several subsets (e.g., when comparing different populations). This page explains how to plot a single subset or multiple subsets, as well as how to plot different subsets on different axes.

For demonstration, an example dataset will be used:
```{code-cell}
from pyttop import get_example
from pyttop.table import Data, Subset
import matplotlib.pyplot as plt

data = get_example('P1')
data.t
```
Some subsets will be defined for demonstration (see the [documentation for subsets](../subset/subset) for explanations of the methods used in the code below):
```{code-cell}
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
data.subset_summary()
``` 

## In a single axis
### Given one or several subsets
You can specify a single or several subsets to plot using the `group`, `subsets`, or `paths` arguments, which correspond to the `group`, `name`, and `path` arguments in the `get_subsets()` method, as describied [here](../subset/subset_use.md#retrieving-subsets). Below are several usage examples:
```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), kwcols={'c': 'z'},
    s=2,
    subsets='obj_class=B',
    );
```
In the above code, a single subset is specified by its name using the `subsets` argument. The subset label (`'B'`) is automatically shown in the axis title. 
```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), 
    s=2,
    paths=['class/obj_class=A', 'x_bins/x(10-20)'],
    );
```
In the above code, two subsets are specified using the `paths` argument. In this case, the subset labels are automatically included in the legend.

The `paths` also accepts [special subsets](../subset/subset_use.md#special-subsets):
```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), 
    s=2,
    paths=['$eval:obj_class=="A"', '$eval:(10 < x) & (x < 20)'],
    );
```
This can be used to quickly explore the data without explicitly defining the subsets.

Both `paths` and `subsets` accepts `Subset` objects. For example:
```{code-cell}
print(subset_A, subset_xbin, subset_A & subset_xbin)
data.plots(
    'scatter', 
    cols=('x', 'y'), kwcols={'c': 'z'},
    s=2,
    subsets=subset_A & subset_xbin,
    );
```
As shown, the [intersection set](../subset/subset_use.md#subset-calculations) of the two subsets is specified, and the resulting subset label (shown in the title) is automatically generated.

### Given a subset group
One can also specify a subset group instead of individual subsets:
```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), 
    s=2,
    group='class',
    );
```
In the above code, a subset group is specifed, so all subsets within the given group are plotted.

By default, `plots()` plots all subsets in the `'default'` group, which includes one subset (named `'all'`) containing all rows if no other subsets are added to the `'default'` group.
```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), 
    s=2,
    # group='default',
    );
```

## In multiple axes (subplots)
### Given a subset group
To compare different subsets by plotting them in various axes (subplots), set the `arraygroups` argument to the group name:
```{code-cell}
data.plots(
    'hist', cols='y',
    arraygroups='x_bins',
    );
```
As shown, each subplot displays the data for a specific subset, with the subset label included in the titles.

### Given several subset groups
You can also specify two subset groups for `arraygroups`, which will result in an array of subplots:
```{code-cell}
data.plots(
    'hist', cols='y',
    arraygroups=('x_bins', 'z_bins'),
    )
plt.tight_layout()
```
For example, the upper left panel plots the data that satisfies both $-20 < x < -10$ and $-40 < z < 0$.

The `arraygroups` can be combined with the [aformentioned methods](#in-a-single-axis) to overplot subsets:
```{code-cell}
data.plots(
    'hist', cols='y',
    plotgroups='class', # `plotgroups` is an alternative to the `group` argument
    arraygroups=('x_bins', 'z_bins'),
    )
plt.tight_layout()
```
In this example, the data in each panel is further separated into the two classes, `'A'` and `'B'`.

## Miscellaneous
### Global selection
You can set the `global_selection` argument to a subset or its path so that only rows within the specified subset are plotted. You can also provide a list of subsets, in which case the intersection set of these subsets will be used.
```{code-cell}
data.plots(
    'hist', cols='y',
    arraygroups=('x_bins', 'z_bins'),
    global_selection=subset_A, 
    # some other valid examples: 
    # global_selection='class/obj_class=A', 
    # global_selection=['class/obj_class=A', 'x_bins/x(0-10)'], # the intersection set of them
    )
plt.tight_layout()
```
As seen in the example above, the global selection is indicated in the topmost title (`suptitle` in matplotlib). 

The global selection will be automatically indicated in a reasonable manner, as shown in the examples below:
```{code-cell}
data.plots(
    'hist', cols='y',
    group='class',
    global_selection=['z_bins/z(-40-0)', 'x_bins/x(0-10)'], # the intersection of them
    );
```
```{code-cell}
data.plots(
    'hist', cols='y',
    subsets=subset_A,
    global_selection=['z_bins/z(-40-0)', 'x_bins/x(0-10)'], # the intersection of them
    );
```

### Passing different arguments for different subsets
You can pass different arguments to the plot function (i.e., set different options) for each subset that is overplotted on the same axis. This is done using the `iter_kwargs` argument, as demonstrated in the examples below:

```{code-cell}
data.plots(
    'hist', cols=['y'], histtype='step',
    plotgroups='class', arraygroups='z_bins',
    iter_kwargs={'linestyle': ['-', '--'], # options for subsets 'A' and 'B' in the 'class' group, respectively
                 'linewidth': [2, 1]},
    )
plt.tight_layout()
```

```{code-cell}
data.plots(
    'scatter', 
    cols=('x', 'y'), kwcols={'c': 'x'},
    subsets=('x(-10-0)', 'x(10-20)'),
    iter_kwargs={
        's': [10, 2],
        'marker': ['v', '^'],
        },
    );
```
As demonstrated in the example above, when using the built-in `scatter` plot function, different subsets overplotted on the same axis share the same colorbar, making their color coding comparable.


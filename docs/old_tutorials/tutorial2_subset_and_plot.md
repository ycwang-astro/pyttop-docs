---
jupytext:
  formats: ipynb,md:myst
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

# Tutorial on subsets and plotting

<!-- # Subsets and plots -->

This notebook will demostrate how to use row subsets and make plots with this package.

Updated for version 0.3.0.

+++

## Data I/O
Let's import necessary packages and prepare our data first:

```{code-cell} ipython3
from pyttop.table import Data, Subset
import matplotlib.pyplot as plt
import numpy as np
```

```{code-cell} ipython3
data = Data('samples/catalog_p1.csv', name='data')
data.t
```

`data.t` is an `astropy.table.Table` object. 
For more information on data input/output, see [tutorial 1](tutorial1_matching).

+++

## Subsets
Sometimes we need to specify row subsets of the dataset `data`. For example, maybe you would like to study records that satisfies both $z>0$ and $x>0$. The traditional method is selecting these data with indexing:

```{code-cell} ipython3
my_selected_subset = data[(data['z']>0) & (data['x']>0)]
```

This may be troublesome if multiple subsets (and even the intersection, union and complementary sets of them) are involved in your study.

In this package, various methods of specifying subsets are demostrated below:

### Specifying subsets from selection criteria
To specify a row subset, you need to input a selection criteria (e.g. $z>0$), which all records in this subset should satisfy.

To define a subset of `data` that satisfies $z>0$ AND $x>0$:

```{code-cell} ipython3
subset1 = data.add_subsets(Subset((data['z']>0) & (data['x']>0), name='subset1'))
```

You may also directly input a string:

```{code-cell} ipython3
subset1 = data.add_subsets(Subset('(z>0) & (x>0)', name='subset1'))
```

The method will try to automatically convert the string into expressions like `(data.t['z']>0) & (data.t['x']>0)`. However, you should check the output to make sure the inferred logical expressions are correct.

One last way to specify the subset is inputting a function:

```{code-cell} ipython3
selection = lambda t: (t['z']>0) & (t['x']>0)
subset1 = data.add_subsets(Subset(selection, name='subset1'))
```

The input function `selection` takes an `astropy.table.Table` object as input, and returns the boolean array (i.e. the value of expression `(t['z']>0) & (t['x']>0)`).

`subset1` is a `Subset` object, which contains the information of the selection, rather than the full data table:

```{code-cell} ipython3
subset1
```

To extract the dataset of this subset ($z>0$ AND $x>0$), input the name (`'subset1'`, as setted with `name='subset1'`) to `data.subset_data`:

```{code-cell} ipython3
subset1_data = data.subset_data('subset1') # subset1_data is an pyttop.table.Data object
subset1_data.t
```

Alternatively, you may also convert a `subset1` object to a boolean array:

```{code-cell} ipython3
subset1_table = data[np.array(subset1)]
subset1_table
```

You can also define multiple subset at a time:

```{code-cell} ipython3
subset2, subset3, subset4 = data.add_subsets(
    Subset('(x<0) | (z<0)', name='subset2'), # x<0 OR z<0
    Subset('~(y<5)', name='subset3'), # NOT (y<5)
    Subset(np.isin(data['obj_class'], ['A', 'B']), name='subset4'), # obj_class in {'A', 'B'}
    )
```

### Convenient methods for subset definition
There are also two convenient methods to define subsets:

```{code-cell} ipython3
data.add_subsets(
    Subset.by_range(x=[0, 5], y=[0, np.inf]), # (x>0) & (x<5) & (y>0) 
    Subset.by_value('obj_class', 'A'), # obj_class == 'A'
    )
```

When using the above methods, the names are automatically set (see below and `help(Subset.by_range)`, `help(Subset.by_value)` for details).

### Summary of subsets

+++

Now let's take a look at all the subsets defined so far:

```{code-cell} ipython3
data.subset_summary()
```

The columns of the above table are:
- **group**: The name of the subset group, which will be introduced in the next section.
- **name**: The name of the subset, which can be manully set by `Subset(<...>, name=<...>)`.
- **size**: The size of this subset.
- **fraction**: The fraction of this subset relative to the whole dataset `data`.
- **expression**: A hint of the expression of the selection that specifies this subset. This can be manually set by `Subset(<...>, expression=<...>)`
- **label**: The string that is used for this subset in plots (e.g. label in legends and titles; see the plotting section for examples). This can be manually set by `Subset(<...>, label=<...>)`.

We can also see that there is always a subset called "all", which consists of all records in the dataset (that is, the whole dataset).

<!-- concept of [group](#Subset groups) will be introduced -->

Now let's clear all subsets defined above (the subset "all" is never deleted):

```{code-cell} ipython3
data.clear_subsets()
data.subset_summary() 
```

## Subset groups
This package introduces the concept of a "subset group", which is a group of subset. This is extremely useful when you need groups of related subsets. 

For example, you may want to divide the dataset according to the value of `obj_class`. You can define subsets in a new subset group called 'class':

```{code-cell} ipython3
data.add_subsets(
    Subset.by_value('obj_class', 'A'), # obj_class == 'A'
    Subset.by_value('obj_class', 'B'), # obj_class == 'B'
    group='class',
    )
```

This can be easily done with a convenient method of `data`:

```{code-cell} ipython3
# data.subset_group_from_values('obj_class', group_name='class') 
# -> ValueError: A subset group with name "class" already exists.
data.subset_group_from_values('obj_class', group_name='class', overwrite=True) 
```

Since a group named "class" already exists, we have to pass `overwrite=True`, though this is not necessary if group "class" has not been defined before.

In some other situations, you may want to define a group of subsets with bins of some quantity:

```{code-cell} ipython3
data.subset_group_from_ranges(
    'x', [[-20, -10], [-10, 0], [0, 10], [10, 20]], # 4 bins specified for x
    group_name='x_binning', overwrite=True, # optional arguments
    )
```

We can understand the outcome of the above codes by checking the `subset_summary()`:

```{code-cell} ipython3
data.subset_summary()
```

By default, if you do not specify the group name (e.g. `group` when calling `data.add_subsets()`), the group name will be "default". 

To extract the dataset of a subset in a non-default group, use one of the expressions below:

```{code-cell} ipython3
class_A_data = data.subset_data('class/obj_class=A')
class_A_data = data.subset_data(group='class', name='obj_class=A')
```

## Making single plots
### Quick reference
**Note**: From version 0.3.0, the previously defined `plot()` is unified with `subplot_array()` (introduced later), and it is recommended to use the new method, `plots()`, for most cases. However, for backward compatibility, `plot()` and `subplot_array()` can still be used.

The simplest way to make a plot with `pyttop.table.Data` is:

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'y'], s=2)
# equivalent to:
# plt.scatter(data.t['x'], data.t['y'], s=2)
```

Where:
- `'scatter'` is the type of the plot. Supported types are: `'plot', 'scatter', 'hist', 'hist2d', 'errorbar'`, which corresponds to `plt.plot, plt.scatter, ...`
- `cols=['x', 'y']` specifies the column names of the dataset to be used for the plot.
- `s=2` is a keyword argument passed to the plot function (in this case, `plt.scatter`).

To make a plot comparing different subsets in a subset group (say, `'class'`):

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'y'], groups='class', s=2)
```

Note that in the above figure, the legend is the labels of the subsets, as shown in the output of `data.subset_summary()`.

To make a plot comparing different subsets:

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'y'], s=2,
          paths=['x_binning/x(0-10)', 'x_binning/x(-20--10)'],
          )
```

To make a plot with columns in the dataset as keyword arguments of the plot function:

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'z'], kwcols=dict(c='y'), s=2, cmap='jet')
# equivalent to:
# s = plt.scatter(data.t['x'], data.t['z'], c=data.t['y'], s=2, cmap='jet')
# plt.colorbar(s)
```

We can see that a colorbar is automatically added when color (the argument `c` for `scatter()`) is specified. To disable this, add `autobar=False` in the above call of `data.plots()`. For more information on `'scatter'` plot in `pyttop`, see [tutorials for scatter plot.](tutorial3_plots#Scatter-plot).

To require that all data on the plot are elements of one specific subset, provide a global selection:

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'y'], groups='class', global_selection='x_binning/x(0-10)', s=4)
```

We can see that only those with $x\in(0, 10)$ are shown on the plot. The global selection is automatically written in the title.

+++

### Advanced features
<!--For simple tasks, the above should be enough. If you would like more control on plotting-->
#### Labels of columns
By default, the labels on the axes are the column names. If you want to adjust the label of the columns, you can execute:

```{code-cell} ipython3
data.set_labels(x='My label of $x$', y='My label of $y$')
```

The setted labels can be accessed with

```{code-cell} ipython3
data.col_labels
```

Now the plot becomes

```{code-cell} ipython3
data.plots('scatter', cols=['x', 'y'], groups='class', global_selection='x_binning/x(0-10)', s=4)
```

```{code-cell} ipython3
data.col_labels = {} # clear the labels
```

#### Custom plot function
This part is for advanced users. If you are new to this package, you may skip this part for now. For more advanced use and details, see the [advanced tutorial for matching and plotting](tutorialA1_advanced_match_plot).

Sometimes you need more control on the plot. In general, you can input a function (rather than strings such as `'scatter'`) to `data.plot` (note: not `data.plots`).

```{code-cell} ipython3
fig, ax = plt.subplots(1, 1)
def my_plot(x, z, **kwargs):
    s = ax.scatter(x+10, z**2, **kwargs)
    ax.set_xlabel('$x+10$')
    ax.set_ylabel('$z^2$')
    cax = plt.colorbar(s)
    cax.set_label('$y$')
data.plot(my_plot, cols=['x', 'z'], kwcols=dict(c='y'), s=2, cmap='jet',
          autolabel=False, # we don't want to use automatically generated labels/titles this time
          )
```

## Subplot arrays
**Note**: From version 0.3.0, the previously defined `subplot_array()` is unified with `plot()`, and it is recommended to use the new method, `plots()`, for most cases. However, for backward compatibility, `plot()` and `subplot_array()` can still be used.
### Basic usage
The concept of "subplot array" is one of the hightlights of this package.
To understand this concept, let's take a look at an example first:

```{code-cell} ipython3
data.subset_group_from_ranges('x', [[-20, -10], [-10, 0], [0, 10], [10, 20]]) # the default name of the group is 'x'
data.subset_group_from_ranges('z', [[-40, 0], [0, 40]]) # the default name of the group is 'z'
data.clear_subsets('x_binning') # delete subset group 'x_binning'
data.subset_summary()
```

```{code-cell} ipython3
data.plots(
    'hist', cols=['y'], histtype='step', linewidth=1.3, # note: kwcols is also supported
    plotgroups='class', # subset group for comparison within each subplot
    arraygroups=['z', 'x'], # subset group for comparison between each subplot
    )
plt.tight_layout()
```

As we have seen in this example, the subset group `'x'` has 4 subsets, and group `'z'` has 2 subsets. By passing the argument `arraygroups=['z', 'x']`, the 2 subset groups are used to generate a $4\times 2$ subplot "array". The different rows have different selections of $x$, while different columns have different selections of $z$. By passing the argument `plotgroups='class'`, the 2 subsets in the subset group named `'class'` are plotted within each panel (subplot).

+++

Sometimes you may want to generate a $1\times n$ subplot "array":

```{code-cell} ipython3
data.plots(
    'hist', cols=['y'], histtype='step', linewidth=1.3, # note: kwcols is also supported
    plotgroups='class', # subset group for comparison within each subplot
    arraygroups=['x'], # subset group for comparison between each subplot
    )
plt.tight_layout()
```

Since there are too many columns, we can try to warp this $1\times n$ array:

```{code-cell} ipython3
data.plots(
    'hist', cols=['y'], histtype='step', linewidth=1.3, # note: kwcols is also supported
    plotgroups='class', # subset group for comparison within each subplot
    arraygroups=['x'], # subset group for comparison between each subplot
    autobreak=True, # automatically warp
    )
plt.tight_layout()
```

Now the $1\times 4$ subplot array is changed into a $2\times 2$ array.

<!--Though not demostrated, `data.subplot_array` also accepts keyword arguments `kwcols`, `global_selection`, which are similar to `data.plot`.-->

+++

### Advanced usage
**Note**: This part is for advanced users. If you are new to this package, you may skip this part for now.

#### Callback 
Sometimes you just want to do some operations on the axis of each subplot (after the plot is made):

```{code-cell} ipython3
def callback(ax): # input a axis
    ax.set_xlim(-10, 15) # set xlim for each subplot

data.plots(
    'hist', cols=['y'], histtype='step', linewidth=1.3, 
    plotgroups='class', arraygroups=['x'], autobreak=True, 
    ax_callback=callback, 
    )
plt.tight_layout()
```

In the above example, 2 histrograms are made for each subplot, thus the function called `'hist'` is called twise for each subplot. However, the `callback` function is called only once for each subplot.

#### Custom plot function
You can also define a custom plot function, similar to that for `data.plot`. However, This function should input an axis (e.g. a `matplotlib.axes._subplots.AxesSubplot` object), and return a function that is suitable for input of `data.plot`.

```{code-cell} ipython3
def my_plot_func(ax): # input an axis
    def hist(*args, **kwargs):
        ax.hist(*args, **kwargs)
        ax.set_xlim(-10, 15)
    return hist

data.plots(
    my_plot_func, cols=['y'], histtype='step', linewidth=1.3, 
    plotgroups='class', arraygroups=['x'], autobreak=True, 
    )
plt.tight_layout()
```

#### Iterative keyword arguments for plot function
If the keywoard arguments (to be passed to the plot function) are different for each subset in ``plotgroups``, you may set them by `iter_kwargs`:

```{code-cell} ipython3
data.plots(
    'hist', cols=['y'], histtype='step',
    plotgroups='class', arraygroups=['z'], # group 'class' consists of two subsets, 'A' and 'B'.
    iter_kwargs={'linestyle': ['-', '--'], # kwargs for subset 'A', 'B' respectively
                 'linewidth': [2, 1]},
    )
plt.tight_layout()
```

#### Handling axes of the figure
`data.subplot_array` returns figure and axes of the generated figure, similar to calling 
`fig, axes = plt.subplots(<...>).`

You can also generate your figure and axes in advance, and input the axes on which you would like to make plots to `data.subplot_array`:

```{code-cell} ipython3
fig, axes = plt.subplots(4, 2, figsize=(6, 10))
left_panels = axes[:, 0]
fig, axes = data.plots( # data.subplot_array itself returns fig, axes
    'hist', cols=['y'], histtype='step', linewidth=1.3, 
    plotgroups='class', arraygroups=['x'], 
    axes = left_panels, # make plots on the left panels 
    )
plt.tight_layout()
```

```{code-cell} ipython3

```

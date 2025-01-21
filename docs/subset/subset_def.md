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


# Defining Subsets and Subset Groups
One can define subsets and groups of subsets in PyTTOP, as we will cover in this section.

## Defining subsets
To use the subset feature, `Subset`, let us import it from `pyttop.table`:
```{code-cell}
from pyttop.table import Data, Subset

d = Data(name='example')
d['x'] = [1, 2, 3, 4, 5]
d['object type'] = ['meal', 'meal', 'tea', 'coffee', 'tea']
```


### From expressions
The most straightforward way to define a subset is by using the `add_subsets()` method and specifying an expression:
```{code-cell}
sub1 = d.add_subsets(Subset('x > 3'))
print(sub1)
```
In the above example, we have defined a subset named `'x > 3'` in our `Data`. A subset can be retrieved using `get_subsets()`:
```{code-cell}
d.get_subsets('x > 3')
```

```{tip}
In principle, you may instantiate a subset outside `add_subsets()`, e.g. `sub1 = Subset('x > 3')`. However, this does not evaluate the provided expression (or function, as in the next section), and is not considered a subset of any `Data`. You will have to add it to `Data` with `d.add_subsets(sub1)` to make it a real subset of your `Data`.
```

Multiple subsets can be added simultaneously. For example:
```{code-cell}
:tags: [remove-output]
sub1, sub2 = d.add_subsets(
    Subset('x > 3'),
    Subset('$(object type) == "tea"')
    )
```

```{tip}
You can use any expression that can be evaluated by `Data.eval()`: see documentation [here](../basics/operations.md#evaluating-expressions) for details.
```


### From functions
A subset can also be defined given a function. For example:
```Python
sub1 = d.add_subsets(
    Subset(lambda t: t['x'] > 3)
    )
```
The above example defines a subset where `d.t['x'] > 3` using a lambda function. For more complex situations, you can define a function:
```Python
def select_func(t): # astropy.table.Table -> boolean array
    select = t['x'] > 3
    return select # this should be boolean

sub1 = d.add_subsets(
    Subset(select_func)
    )
```
Technically, the function should take an `astropy.table.Table` as input, and returns a *boolean* array (i.e., an array where the elements are either `True` or `False`).

### From array-like objects
Finally, you can directly provide a boolean array, or simply a sequence of `True` or `False` values. For example:
```Python
sub1, sub1_1, sub1_2 = d.add_subsets(
    Subset(d['x'] > 3),
    Subset([False, False, False, True, True]),
    Subset([0, 0, 0, 1, 1]), # the values will be converted to boolean values
    )
```

### Name, expression, and label
Subsets have several properties, including `name`, `expression`, and `label`. 

The **name** is used to access a subset:
```{code-cell}
sub1 = d.get_subsets('x > 3') # gets a subset named 'x > 3'
```
The character '/' (slash) is not allowed in a subset name.

The **expression** provides a hint of how the subset have been generated. If a subset has been [defined with an expression](#from-expressions), the default `expression` property is simply the expression:
```{code-cell}
sub1.expression
```

The **label** is the text use to indicate the subset in your plots. When you make a plot for data in a certain subset, the subset label can be automatically added in legends or plot titles (see [documentation for plot making](../plot/plot) and [tutorials](../start/Tutorials) for examples). 

The properties can be manually set when defining a subset:
```Python
d.add_subsets(
    Subset('x >= 3', name='xgeq3', expression='x >= 3', label='$x \\geq 3$'))
```

### Convenience methods
There are also convenient ways to define a subset, as illustrated by examples below:
```Python
d.add_subsets(
    Subset.by_range(col1=[0, 1], col2=[0, np.inf]) # defines a subset requiring `(0 < col1 < 1) & (col2 > 0)`
    Subset.by_value('col1', 'value') # defines a subset requiring the value of the column named 'col1' equals 'value'
    )
```

## Subset groups
By default, the subset is added to the "default" subset group when calling `add_subsets()`. Alternatively, a subset can be added to a specified **subset group**. For example:
```Python
d.add_subsets(
    Subset('x > 3'),
    Subset('$(object type) == "tea"'),
    group='mygroup',
    )
```
The above example adds two subsets to a group named 'mygroup' (creating it if it does not already exist).

```{note}
When a subset is created from a masked array (e.g., created by evaluating columns with masked values), the masked values are NEVER considered elements of the subset. See [this page](../caveats/subset_mask) for discussions, demonstrations and caveats.  
```

### Convenience methods for creating groups of subsets
A subset group can be particularly useful when there are several subsets that are related to each other. PyTTOP provide convenience methods to quickly creating groups of subsets. For example, the below code defines a subset group named 'x', which consists of 2 subsets, `0 < x < 3` and `3 < x < 5`:
```{code-cell}
d.subset_group_from_ranges(column='x', ranges=[[0, 3], [3, 5]])
```
This can be used to divide a column of continuous value into several bins.

A subset group can also be generated for each unique value in a column. This is typically useful when a column represents a classification. For example, the following example creates a group named 'object type' with several subsets, each representing a specific value of 'object type'.`
```{code-cell}
d.subset_group_from_values('object type')
```

## Managing subsets and groups
### Summary of subsets
To get a summary of all subsets defined for a `Data`, call the `subset_summary()` method (or the abbreviation, `subsum()`):
```{code-cell}
d.subset_summary()
```
As can be seen, this returns a table with each row representing a subset.
For reference, the columns of the table are summarized below:
- **group**: The name of the subset group.
- **name**: The name of the subset.
- **size**: The size of this subset, i.e. number of rows selected.
- **fraction**: The fraction of this subset relative to the whole `Data` table, i.e. `size/len(<data>)`.
- **expression**: A hint of the expression of the selection that specifies this subset. 
- **label**: The string that is used for this subset in plots (e.g. labels in legends and titles).

Note that subset groups starting with `$` are *special groups* that contain *virtual subsets*. See documentation on [special subsets](subset_use.md#special-subsets) for details.


### Clearing subsets
You may clear the subsets in a group by:
```{code-cell}
d.clear_subsets('x') # clears subset group named 'x'
d.clear_subsets() # clears ALL subsets
```
There is always a subset in the 'default' group named 'all', which includes all rows:
```{code-cell}
d.subsum()
```

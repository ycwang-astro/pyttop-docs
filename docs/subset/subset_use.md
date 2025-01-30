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

# Using Subsets

## Retrieving subsets
Any subset defined for a `Data` can be retrieved, or referred to (when calling methods that uses subsets, e.g. `data.subset_data(), data.plots()`), by its name, and the name of the group it belongs to (if necessary). 
One can use the `get_subests()` method to simply retrieve a subset with its "path" (i.e., the combination of a group name and a subset name). For example, to get a subset named "all" in "default" group, use:
```Python
s = data.get_subsets('default/all') # path is <group_name>/<subset_name>
```
For subsets in the "default" group, `default/` can be omitted:
```Python
s = data.get_subsets('all') # retrieving from the "default" group
```
You can also provide group and subset names separately. For example:
```Python
data.get_subsets(name='all', group='default') # subset 'all' in group 'default'
data.get_subsets(name='my_subset') # get subset 'my_subset' in group 'default' by default; however, if 'my_subset' is not found in group 'default', it is searched in other groups
```
If a subset is provided, this subset itself will be returned:
```Python
data.get_subsets(my_subset) # returns `my_subset` itself
```

Multiple subsets can be retrieved simultaneously by providing a list (or lists):
```Python
data.get_subsets(['default/all', 'mygroup/mysubset'])
data.get_subsets(name=['all', 'mysubset'])
data.get_subsets(name=['all', 'mysubset'], group='default')
```
Multiple subsets are returned as a list.


## Special subsets
A "special subset" is a virtual subset that is *temporarily* created when retrieving (or referring to) it. It is never considered a real subset of a `Data` (and thus cannot be seen with `data.subset_summary()`) unless explicitly adding it to a `Data` using `data.add_subsets()`.

```{tip}
"Retrieving" a special subset is actually creating a new subset as if using an existing subset of `Data`. This can be useful when [making plots](../plot/plot_subsets), as you can specify a special subset without defining it in advance, and it will be automatically created (temporarily).
```

### `$unmasked`
`$unmasked` is a special group used to obtain a subset that indicates whether an item in a column is not masked. The paths of subsets in this group follow the form `$unmasked/col_name`.

The use of `$unmasked` is demonstrated in the example below:
```{code-cell}
:tags: [remove-cell]
from pyttop.table import Data, Subset
import numpy as np
```
```{code-cell}
d = Data(name='example')
d['x'] = [1, 2, -99, 4, -99]
d.mask_missing('x', -99) # values -99 in column 'x' are masked
print(d.t)
s = d.get_subsets('$unmasked/x')
print(s)
print(s.selection) # s.selection is a boolean array indicating whether each row is included in this subset
```
As seen, the subset `'$unmasked/x'` includes the three items in column `'x'` that are not masked.

## Subset calculations
For any two subsets of the same `Data`, the intersection set, union set, and complementary set can be evaluated. This is done using operators `&`, `|`, and `~`.
```Python
# sub1, sub2 are two subsets
sub1 & sub2 # intersection: rows in both `sub1` AND `sub2`
sub1 | sub2 # union: rows in either `sub1` OR `sub2`
~sub1 # complementary: rows NOT in `sub1`
```

## Cutting data given subsets
You can obtain a data cut given a subset using the `subset_data()` method (or its alias, `subdat()`), so that it only contains rows in the specified subset. For example:
```{code-cell}
d['name'] = ['Alice', 'Bob', 'Carol', 'Dave', 'Eve']
d.add_subsets(Subset([True, False, True, False, True], name='good_guy'))
cut_d = d.subset_data('good_guy')
cut_d.t
```
The rules of referring to subsets in `subset_data()` is identical to those for [Retrieving subsets](retrieving-subsets) (`get_subsets()`).

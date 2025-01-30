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

# Dealing with masked values for subsets
## Description of the caveat
A row [subset](../subset/subset) is a selection of rows in a table. For a given subset and each row in the table, there are two (and only two) possibilities: the row **is** an element of the subset, or it **is not** an element of the subset. This means that a subset is similar to a boolean array, or a sequence of `True` or `False`.

If the columns (arrays) used to define a subset result in a *masked* boolean array, any masked values are treated as (and converted to) `False`, meaning the corresponding rows are not considered elements of the subset. This is demonstrated in the example below:
```{code-cell}
from pyttop.table import Data, Subset
data = Data(name='data')
data['col'] = [-99, -1, 1]
data.mask_missing(missval=-99)

print(data.t)

print("\nthe value of 'col > 0' is:")
print(data.eval('col > 0'))

subset = data.add_subsets(Subset('col > 0'))
print("\nthe selection of a subset evaluated by 'col > 0' is:")
print(subset.selection)
```
The selection of any subset only consist of `True` or `False` without any masks. 

As a result, the [intersection/union/complementary set](../subset/subset_use.md#subset-calculations) can differ from using the corresponding logical operators when defining subsets. For the example shown above, `~subset` is not the same as a subset defined by the expression `'~(col > 0)'` (equivalently, `'col <= 0'`):
```{code-cell}
print('subset =', subset, '\nwith selection:', subset.selection)
print('\nThe complementary set of subset:\n~subset =', ~subset, '\nwith selection:', (~subset).selection)

subset1 = data.add_subsets(Subset('~(col > 0)'))
print('\nBut for', subset1, '\nthe selection is:', subset1.selection)
print("because the value of '~(col > 0)' is:")
print(data.eval('~(col > 0)'))
```
As shown above, the evaluation of the expressions always results in a missing value (`--`) for the frist row, making the frist row not an element of the defined subsets for both `'col > 0'` and `'~(col > 0)'`.


## General suggestions
- When using subsets, keep in mind the features mentioned above, and ensure you understand the meaning of the subsets you define.
- The name of the calculated [intersection/union/complementary set](../subset/subset_use.md#subset-calculations) uses strings like 'AND', 'OR', or 'NOT' instead of the logical operators (e.g., `'NOT(col > 0)'` instead of `'~(col > 0)'` as shown above). This helps distinguish between different cases.

## Discussions
Below are some considerations and reasons for choosing this behavior. However, any feedback or suggestions for changing or improving the current behavior are welcome. 
- If masked (missing) values are not treated as `False` (where `False` means "not an element of the subset") when defining and evaluating a new subset, this results in three possibilities for a row: `True` (in the subset), `False` (not in the subset), and `--` ("I don't know"). This breaks the "law of excluded middle" and makes logical calculations of subsets tricky. For example, the intersection of the `'~(col > 0)'` and `'col > 0'` subsets is not the universal set (all rows), which can cause confusion.
- A masked value usually indicates that the measurement of a property (column) of an individual (row) is unavailable. When defining a subset based on the properties of individuals, it seems more reasonable (in most cases) to exclude those without measurements. 
- Treating `--` as `False` seems to be more intuitive in some subset calculations. For example:
  - Say `subset1 | subset2` means selecting individuals that either "have property 1" or "have property 2". Even if we cannot determine whether an individual "has property 2" due to missing measurements, if we are sure it "has property 1", it is reasonable to select it. However, if "property 1" is `True` and "property 2" is `--`, the result would be `--` (i.e., "I don't know" it if it has either "property 1" or "property 2").
  - Say `subset1 & ~subset2` means selecting individuals that "have property 1" but exclude those that "have property 2". If there is no evidence that an individual has "property 2" due to missing measurements, it is reasonable to keep it. (However, if this is not your case, you may prefer to use these logical operators directly in the expressions when defining the subsets.)
- I find handling missing values tricky in logical judgment. The current behavior is the best I can think of so far. As long as the behavior is understood, one can proceed with the analysis according to their needs.


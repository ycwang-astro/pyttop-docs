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

# Debuging and History Tracking Using Metadata of `Data`
PyTTOP stores relavant information in the metadata of tables and their columns. This can be helpful for debugging and tracking the history of tables, especially when working with a `Data` object where you have not recorded the operations performed to it.

```{warning}
Please be aware that the metadata is **only for reference**, and **its correctness is NOT guaranteed** (especially for the metadata of columns). PyTTOP can only record a very limited number of operations that you perform, and is *never* aware of any operations done directly to the underlying astropy `Table` or `Column` objects. As a result, the metadata information can be incorrect. Please avoid relying fully on the metadata. It is recommanded to stay clear about your operations and keep a record of all your data processing and analysis code. 
```

## The metadata of `Data`
For a `Data` object, the metadata primarily records how it was initialized and the operations ([merging](../match/tree_match_merge.md#merging-data-tables) and [cutting given subsets](../subset/subset_use.md#cutting-data-given-subsets), as described in corresponding pages) that have been performed on the data. This is illustrated in the examples below.

```{code-cell}
from pyttop.table import Data, Subset
from pyttop.matcher import ExactMatcher

d1 = Data(
    {
        'index': [0, 1, 2, 3, 4],
        'id': [101, 102, 104, 105, 108],
        },
    name='d1',
    )

d1.print_meta()
```
By printing the metadata withs `d1.print_meta()`, we can see that this `Data` object was initialized from a dictionary. If it was loaded from a file, the `'path'` field will indicate the file's location. 


If the data is [cut given a subset](../subset/subset_use.md#cutting-data-given-subsets), the metadata will indicate the details of the cut:
```{code-cell}
d1.add_subsets(Subset('id > 103'))
d1_cut = d1.subset_data('id > 103')

d1_cut.print_meta()
```
The information of the subset used to cut the data is recorded in the `'subset'` field, and the metadata of the original data is recorded in the `'meta'` field.

If the data is [merged](../match/tree_match_merge.md#merging-data-tables) from several tables, the metadata will record the details of the matching and merging process:
```{code-cell}
d2 = Data(name='d2')
d2['index'] = [0, 1, 2, 3, 4, 5]
d2['ID'] = [103, 104, 105, 101, 107, 106]

d1.match(d2, ExactMatcher('id', 'ID'), verbose=False)

d_merged = d1.merge(verbose=False)

d_merged.print_meta()
```

The metadata of the original `Data` objects is always included. If the original `Data` objects themselves were generated through merging or cutting, their metadata---containing the merging/cutting information---will be nested within the metadata of the new `Data` object.

## The metadata of columns
The metadata of columns records the original `Data` they originate from, as well as modifications made to them. For example, when data is merged from several tables, you can use this metadata to determine which original table a particular column came from.

The information can be seen using `data.from_which(<colunm name>)`. For example: 
```{code-cell}
for col in ['id', 'ID']:
    print(col, '-->', d_merged.from_which(col))
```
This indicates that the column named `'id'` comes from a `Data` object named `'d1'`, which was initialized from a dictionary, while the 'ID' column was added by the user (via `d2['ID'] = ...`).

For example, if we overwrite `'id'` using the `eval()` method (as introduced [here](../basics/operations.html#evaluating-expressions)), this change is also recorded:
```{code-cell}
d_merged.eval('10 * id', to_col='id')
d_merged.from_which('id')
```

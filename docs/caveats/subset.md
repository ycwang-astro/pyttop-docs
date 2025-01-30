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

# The Static Subsets 
<!-- (and Comparison with TOPCAT) -->

<!-- ```{note}

``` -->

## Description of the caveat
In PyTTOP, a row subset is *static* rather than *dynamic*. This means that once it is defined, the rows it contains never change, even if the data in the table changes. Subsets are independent of the table and do not monitor any potential changes to it. Even if a subset is defined using an expression, such as `data.add_subsets(Subset('col1 > 0'))`, the expression only indicates how the subset *was* initially created. This behaviour can be illustrated in the following example:
```{code-cell}
from pyttop.table import Data, Subset
data = Data(name='data')
data['col'] = [-1, 0, 1]

subset_positive = data.add_subsets(Subset('col > 0'))
print(subset_positive.selection) # selection is a boolean array indicating whether each row is included in this subset

data['col'] = [1, 0, 1]
print('after changing the data:', subset_positive.selection)

print('but now it should be:', data.eval('col > 0'))
```

Note that a "[special subset](../subset/subset_use.md#special-subsets)" is never considered a real subset of a `Data` unless explicitly adding it to a `Data` using `data.add_subsets()`. This means that even if a special subset is "retrieved" by its path, a new subset is created when the same path is "retrieved" again.
```{code-cell}
data['col'] = [-99, -1, 1]
subset = data.get_subsets('$unmasked/col') # this is actually creating a subset rather than retrieving an existing subset of data
print(subset.selection)

data.mask_missing(missval=-99)
subset1 = data.get_subsets('$unmasked/col')
print('the newly generated subset:', subset1.selection)
print('but the old subset is still:', subset.selection)

# by adding `subset` to `data`, it becomes a real subset that can be retrived with the normal path 'default/$unmasked(col)'
data.add_subsets(subset)
print('after adding the old subset to data, it is still:', data.get_subsets('default/$unmasked(col)').selection)
```

## General suggestions
- If you have to change the values in the table, do it *immediately after* loading the data or before converting it into a `Data` object.
- Perform data preprocessing or calculations before defining subsets. It is advisable to define subsets only when the values are finalized and ready for analysis.


## Discussions
Below are some considerations and reasons for choosing this behavior. However, any feedback or suggestions for changing or improving the current behavior are welcome. 
- A dynamic behavior may make subsets less flexible and harder to implement, and may also increase computational costs. They may need to be restricted to unevaluated expressions that are evaluated each time they are used, or continuously monitor changes in the table and re-evaluate. On the other hand, the static behavior makes subsets array-like, and their logical operations become more intuitive.
- I view PyTTOP primarily as a data analysis tool, rather than tool for *editing* existing data in a table. Thus, I do not expect the existing data to change, as they represent original data that should typically remain intact.

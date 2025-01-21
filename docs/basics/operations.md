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

# `Data` Operations
Since the data table itself is stored with `astropy.table.Table` and can be accessed with `data.t`, you may perform operations on it with the methods provided by Astropy (e.g., [Modifying a Table](https://docs.astropy.org/en/stable/table/modify_table.html) and [Table Operations](https://docs.astropy.org/en/stable/table/operations.html)). In addition, you may also make use of methods provided by PyTTOP, as detailed below.

```{caution}
It is generally discouraged to directly perform operations on the astropy `Table` object of `Data`, `data.t`, as this may cause inconsistencies to the information stored by `Data` itself. One example is the [subsets](../subset/subset). Due to the *static* nature of PyTTOP subsets (see [discussions here](../caveats/subset)), they cannot be updated in case the values in the table are changed, some rows are added, removed, or reordered, etc. These will cause unexpected errors when using subsets. 

The general suggestions are: Only use methods for `astropy.table.Table` if it is not provided by PyTTOP; However, if you have to perform such operations, it is safer to manipulate an `astropy.table.Table` object  *before* converting it to `Data`.
```

## Basics
Below shows some basic methods to use `Data`:
```Python
data.t # get the `astropy.table.Table`
data.colnames # get a list of column names
data.df() # returns a `pandas.DataFrame` (equivalent to `data.t.to_pandas()`)
data['col'] # getting the column named 'col'
data['col'] = [1, 2, 3] # setting the value of column named 'col' (creates the column if it does not exist)
```

## Masking missing values
There may be missing entries indicated by a particular value. As an example, missing entries are indicated by `-99` the following table:
```{code-cell}
from pyttop.table import Data
import numpy as np

d = Data(name='example')
d['col1'] = [1, 2, -99, 4, 5]
print(np.mean(d['col1'])) # -17.4, which is probabily not what you need 
```
When analyzing data and making plots (e.g. calculating `np.mean(d['col1'])`), the missing entries should be masked and ignored rather than ragarding them as normal values.

You may simply mask the values using the `mask_missing()` method:
```{code-cell}
d.mask_missing('col1', missval=-99)
print(d['col1'])
print(np.mean(d['col1']))
``` 
It is possible to mask multiple columns simultaneously:
```Python
d.mask_missing(['col1', 'col2'], missval=-99) # mask value `-99` in 'col1' and 'col2'
d.mask_missing(missval=-99) # mask value `-99` in all columns
```


## Evaluating expressions
It can be useful to evaluate expressions using the columns in the table. For example:
```Python
d['x'] = [1, 2, 3]
d['y'] = [6, 4, 2]

x_plus_y = d.eval('x + y') # this returns the result of adding x and y
d.eval('x + y', to_col='x_plus_y') # this also creates a column named 'x_plux_y' and set the values the result of the expression 'x + y'
```
In general, you can use any existing column name that can be regarded as valid Python variable names. Alternatively, you may also refer to column names with symbols `$(name)`, for example:
```Python
d.eval('x + $(1 random.column-name!)')
```
NumPy functions can also be used:
```Python
d.eval('(np.sin(x) > 0) & (y < 1)')
```
You may also use variables/names defined in your script. But to make it usable, you should pass the names as arguments:
```Python
myvar = 2
d.eval('x * myvar') # this raises an exception because 'myvar' is not defined within the table
d.eval('x * myvar', myvar=myvar)
```

```{tip}
Executing `d.eval('x + y')` is similar to executing `d['x'] + d['y']`, and the columns (e.g. 'x' and 'y') behave like numpy arrays. Thus, you should use array operations (e.g., 'x & y' rather than 'x and y').

`d.eval('x + y', to_col='x_plus_y')` is preferred to `d['x_plus_y'] = d.eval('x + y')`, as the former writes the expression ('x + y') into the metadata of the new column ('x_plus_y'), making it easier to [debug](meta.md) (if you were to forget how the column was calculated).
```


## Applying functions to rows
A function can be applied iteratively to each row. For example, 
```Python
d = Data(name='example')
d['x'] = [1, 2, 3]

def x_square(row):
    return row['x']**2

result = d.apply(x_square)
```
The `result` is a list of the return values of the applied function (`x_aquare` in this example). This is similar to:
```Python
result = [x_square(row) for row in d]
```

Below is a more sophisticated example using multiprocessing and additional arguments:
```Python
def linear(row, a, b):
    return a * row['x'] + b

if __name__ == '__main__': # NECESSARY if multiprocessing is enabled
    result = d.apply(linear, 
                     args=(2, 1), # additional arguments (a, b)
                     processes=8, # number of processes
                     )
```
In this example, we have passed more additional arguments to the function. We also enabled multiprocessing (which is disabled by default) by setting the number of processes (`processes=8`). More details can be seen in the API reference for `Data.apply`. 

```{caution}
The multiprocessing feature of `Data.apply` uses the Python module `multiprocessing`. With multiprocessing enabled, your code (at least the line containing `d.apply()`) MUST be under `if __name__ == '__main__':`. For why this is necessary, see "Safe importing of main module" in the [multiprocessing documentation](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming).
```


## Checking for duplicate values
There may be duplicate values in a column, e.g. repeated observation of the same object. This can be checked using the method `check_duplication()` (or using the abbreviation, `chkdup()`):
```{code-cell}
d = Data(name='example')
d['x'] = [1, 1, 3, 4]
d['y'] = [1, 2, 2, 1]
d['z'] = [1, 2, 3, 4]
d.chkdup()
```
By default, `check_duplication()` checks all columns and print the results. You can also specify columns to check, and get a dictionary containing the duplicate values. For example:
```{code-cell}
d.chkdup('x', 'y', # check columns 'x' and 'y' only
         action='detail')
```


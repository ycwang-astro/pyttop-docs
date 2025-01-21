---
jupytext:
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

# Tutorial on matching

This notebook will demostrate how to match catalogs with this package. 

```{code-cell} ipython3
:tags: [remove-cell]
import warnings
warnings.filterwarnings('ignore')
```

## Data I/O
<!-- To initialize an `pyttop.table.Data` object, input path to the data, an `astropy.table.Table` object, or anything that can be converted to an `astropy.table.Table` object.  -->
In this package, data are handled by the `pyttop.table.Data` class. Intuitively (though not accurately), a 'class' is a type of objects that can store information and perform certain operations. As we will see later, we can initialize an `pyttop.table.Data` object, which can store a data table (in the memory) and allows you to do operations including matching, merging and more.

To get started, import the `pyttop.table.Data` class:

```{code-cell} ipython3
from pyttop.table import Data
```

### Reading and writing `Table`-like data

The data table of an `pyttop.table.Data` object is stored in an `astropy.table.Table` object, so you can load anything that can be converted to an `astropy.table.Table`. For introduction to tables and documentations on the `astropy.table.Table`, see [Astropy's documentation on Data Tables](https://docs.astropy.org/en/stable/table/index.html), especially the supported formats of Astropy's [Built-In Table Readers/Writers](https://docs.astropy.org/en/stable/io/unified.html#built-in-readers-writers).

The most straightforward way to load a table is to initialize a `Data` object with the path to the data file (note that the files in `./samples/` directory are randomly generated datasets):

```{code-cell} ipython3
cat1 = Data('samples/catalog1.csv', name='cat1') # cat1 is an astropy.table.Table object
```

It is highly recommended to input a `name` keyword argument, as this name will be used to distinguish different datasets. If `Data` is initialized with a path to file and no `name` is given, it will be automatically set to the file name.

The `astropy.table.Table` object used to store the table can be accessed with `data.t`. Thus, you can do anything as you can do with an `astropy.table.Table` object. 

```{code-cell} ipython3
print("Name:", cat1.name)
cat1.t # an astropy.table.Table
```

You may also add keyword arguments to be passed to `astropy.table.Table.read()`:

```{code-cell} ipython3
cat3id = Data('samples/catalog3_id.txt', name='cat3id',
              names=['cat3ID', 'survey2_id'], format='ascii') # 'ascii' is one of the supported formats of Astropy's Built-In Table Readers/Writers.
cat3v = Data('samples/catalog3_measurement.txt', name='cat3v',
             names=['survey2_id', 'x', 'y', 'class1'], format='ascii')
```

A `Data` object can also be created with an `astropy.table.Table` object, or anything that can be converted to an `astropy.table.Table` object. 

```{code-cell} ipython3
from astropy.table import Table
cat2_table = Table.read('samples/catalog2.fits')
cat2 = Data(cat2_table, name='cat2')
cat4_dict = dict(Table.read('samples/catalog4.hdf5'))
# print(cat4_dict)
cat4 = Data(cat4_dict, name='cat4')
```

You can save the table to files with Astropy's table writers (see Astropy's documentation on [Reading and Writing Table Objects](https://docs.astropy.org/en/stable/table/io.html)):

```{code-cell} ipython3
# cat1.t.write(filename, format=supported_format)
```

### Reading and writing an `pyttop.table.Data` object
Though you can write the table with the `astropy`'s writers (as shown above), it is more recommended to use the `save()` method of the `pyttop.table.Data` object itself (note that it is different from `cat1.t.write()`). This method not only saves the data table, but also saves other key properties of the data (e.g. the user-defined row subsets) of an `pyttop.table.Data` object. 

```{code-cell} ipython3
cat1.save('samples/output/cat1', overwrite=True) # set overwrite=True to overwrite an existing file
```

The data is saved to `'samples/output/cat1.data'`.

Note that the data's matching with other data is not saved. If you want to match the data (say `cat1`) with other datasets (say `cat4`), you have to merge the datasets into one dataset (say `merged_cat`), and save the merged dataset (`merged_cat`). If you save `cat1` to a ".data" file and load it later, you are unable to recover the match between `cat1` and `cat4`.

To load a ".data" file and (mostly) recover `cat1`, the `pyttop.table.Data` object:

```{code-cell} ipython3
cat1 = Data.load('samples/output/cat1.data')
```

## Matching
In this package, catalog B is said to be *matched to* A, if each record (row) in A is assigned two values:
- Whether it can be matched to a record in catalog B;
- The index of the best match record in catalog B (if no match possible, the index can be any number but means nothing).

A is referred to as the *base data* of the match.

### Matching with a built-in matcher
To match `cat4` to `cat1` with the exact value of the `'survey1_id'` field in `cat1` and the `'survey1_id'` field in `cat4`, use an `ExactMatcher`:

```{code-cell} ipython3
from pyttop.matcher import ExactMatcher
cat1.match(cat4, ExactMatcher('survey1_id', 'survey1_id'))
```

Since there are more than one records for the same `'survey1_id'` in `cat4`, matching `cat1` to `cat4` is not equal to matching `cat4` to `cat1`:

```{code-cell} ipython3
cat4.match(cat1, ExactMatcher('survey1_id', 'survey1_id'))
```

You may use any iterable object (e.g. an array) to match the catalogs, provided that what is used to match catalogs has the same length (i.e. number of records) as the catalogs.

```{code-cell} ipython3
print('len(cat3v) =', len(cat3v))
print('len(cat3id) =', len(cat3id))
cat2.match(cat3v, ExactMatcher('survey2_id', cat3id.t['survey2_id']))
```

You can also match data with thier coordinates:

```{code-cell} ipython3
from pyttop.matcher import SkyMatcher
import astropy.units as u
cat1.match(cat2, SkyMatcher(unit=u.deg, unit1=(u.h, u.deg))) # RA for cat1 is dms; RA for cat2 is hms.
```

For more information on `SkyMatcher`, use `help(SkyMatcher)`.

To remove all matches to `cat1`, use:

```{code-cell} ipython3
# cat1.reset_match()
```

To unmatch `cat2` from `cat1`:

```{code-cell} ipython3
cat1.unmatch(cat2)
```

For `SkyMatcher`, you can also explore the distribution of minimum sky distances, i.e. the distance to the nearest object in `cat2` for each object in `cat1`:

```{code-cell} ipython3
skymatcher = SkyMatcher(unit=u.deg, unit1=(u.h, u.deg))
skymatcher.explore(cat1, cat2)
cat1.match(cat2, skymatcher)
```

### Defining custom matchers
**Note**: This part is for advanced users. If you are new to this package, you may skip this part for now.

You may also define your own matchers. A macther class should be defined like this:

```{code-cell} ipython3
class MyMatcher():
    def __init__(self, args): # 'args' means any number of arguments that you need
        # initialize it with args you need
        pass
    
    def get_values(self, data, data1, verbose=True): # data1 is matched to data
        # prepare the data that is needed to do the matching (if necessary)
        pass
    
    def match(self):
        # do the matching process and calculate:
        # idx : array of shape (len(data), ). 
        #     the index of a record in data1 that best matches the records in data
        # matched : boolean array of shape (len(data), ).
        #     whether the records in data can be matched to those in data1.
        return idx, matched
```

## Merging catalogs

+++

### Match tree
If B is matched to A, I call A as the *child data* of B, and B as the *parent data* of A.

Say B, C are matched to A, and D is matched to B. Then B, C are children of A, and D is child of B. When we try to merge everything into A<!-- (i.e. add the information of corresponding records in B, C, D into A)--> (i.e. merge the information in A's chilren, grandchildren, etc. into A), it may be useful to see all of its children/grandchildren, or what I call the *match tree*:

```{code-cell} ipython3
cat1.match_tree(detail=False)
```

From the *match tree* we may see that `cat4` and `cat2` are matched to `cat1` and `cat3v` is matched to `cat2`. Although `cat1` is also matched to `cat4`, this match is a duplication in this match tree, and will be ignored when merging everything (`cat4`, `cat2` and `cat3v`) into `cat1`.

For more information on how they are matched:

```{code-cell} ipython3
cat1.match_tree(detail=True)
```

For example, we may also use `cat4` as the base catalog:

```{code-cell} ipython3
cat4.match_tree(detail=False)
```

### Catalog merging
Now we can merge everything possible to be merged into `cat1`:

```{code-cell} ipython3
merged_cat = cat1.merge(outname='my_merged_catalog')
print("Name:", merged_cat.name)
merged_cat.t
```

Note that columns with the same names are renamed by the `name` of the `Data` objects. You may also check that the match is indeed correct.

We may now save the merged catalog for later use:

```{code-cell} ipython3
merged_cat.save('samples/output/merged_cat', overwrite=True) # you don't need overwrite=True if file 'samples/output/merged_cat.data' does not exist
# load with:
# merged_cat = Data.load('samples/output/merged_cat.data')
```

### Source of a column
Sometimes we forget the name of the data from which a column is merged. For example, we are sure that the column named `'A'` is merged from somewhere, but cannot recall the name of that dataset. We can use the `from_which()` method:

```{code-cell} ipython3
merged_cat.from_which('A')
```

The column `'A'` indeed comes from `cat1`, which is loaded from `"samples/catalog1.csv"`.

**WARNING**. The `from_which()` method has several limitations:
- In some cases, the software cannot decide the source of the column, and `from_which()` will return an empty string.
- Direct operations on the data table (`astropy.table.Table`), especially adding columns to the data table using the values of other columns, can result in incorrect results of `from_which()`. For example, instead of:

```{code-cell} ipython3
# WRONG:
merged_cat.t['A+i (wrong)'] = merged_cat.t['A'] + merged_cat.t['i'] # adding a new column
merged_cat.from_which('A+i (wrong)') # wrong result
```

use:

```{code-cell} ipython3
# CORRECT:
merged_cat['A+i (right)'] = merged_cat['A'] + merged_cat['i'] # adding a new column
merged_cat.from_which('A+i (right)') # right result: this column is added by the user
```

### Merging options

Maybe you want to keep records that cannot be matched to `cat3v` and only want to merge subsets of columns from the catalogs:

```{code-cell} ipython3
merge_columns = { # specify columns to be merged
    'cat1': ['survey1_id', 'RA', 'Dec'],
    'cat4': ['i', 'j'],
    'cat2': ['survey2_id'],
    'cat3v': ['class1'],
    }

keep_unmatched = ['cat3v'] # keep records that cannot be matched to cat3v

another_merged_cat = cat1.merge(keep_unmatched=keep_unmatched, 
                                merge_columns=merge_columns) # use default outname
print("Name:", another_merged_cat.name)
another_merged_cat.t
```

You can also specify the columns to be ignored during merging (the `ignore_columns` argument of `merge()`); see `help(Data.merge)` for details.

You may also set the depth for `match_tree` and `merge` methods. Setting depth to 0 means only keeping the base catalog itself.

```{code-cell} ipython3
cat4.match_tree(depth=2)
```

```{code-cell} ipython3
cat4.match_tree(depth=0)
```

```{code-cell} ipython3
cat4.merge(depth=1).t
```

## Things to be noted

This package does not support matching multiple records to a single record in the base data. For example, `table2` below has two records with the same `survey_id`:

```{code-cell} ipython3
table1 = Data({'survey_id': [0, 1, 2], 'value': ['A', 'B', 'C']}, name='t1')
table1.t
```

```{code-cell} ipython3
table2 = Data({'table2_id': [0, 1, 2], 'survey_id': [0, 1, 0]}, name='t2')
table2.t
```

If you match `table2` to `table1` by `survey_id` using `ExactMatcher`, the first exact match in `table2` will be used:

```{code-cell} ipython3
table1.match(table2, ExactMatcher('survey_id', 'survey_id')).merge().t
```

If you wish to keep the records with the same `sruvey_id` in `table2`, you may match `table1` to `table2` instead of matching `table2` to `table1`:

```{code-cell} ipython3
table2.match(table1, ExactMatcher('survey_id', 'survey_id')).merge().t
```

Or you may merge these records (with the same `sruvey_id`) before matching and merging the catalogs.

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

# A quickstart example
This tutorial demonstrates the basic steps and workflows for using PyTTOP in your project. It serves as a solid starting point and reference guide to help you understand its usage. For more detailed information, you can explore the documentation on specific topics.

```{code-cell}
:tags: [remove-cell]
from astropy import conf
conf.max_lines = 17
```

## Loading Data
The main class used to store tabular data in PyTTOP is `Data`. Let us first import it:
```{code-cell}
from pyttop.table import Data
```

PyTTOP includes example datasets for testing and exploring its features. Let us start by loading an example table called 'LGM.bio', which contains the biological properties of the fictional "little green men" (LGM).
```{code-cell}
from pyttop import get_example
lgm_bio = get_example('LGM.bio')
lgm_bio
```

```{note}
The "LGM" datasets are randomly generated solely for testing and demonstration purposes, with no real meaning. Please do not interpret them seriously.
```

As seen, the resulting `lgm_bio` is as `Data` object named 'LGM.bio'. In most cases, you may want to load data from a file, which can be done with:
```Python
my_data = Data('path/to/data/file', name='my_data')
```
See the documentation [here](../basics/IO.md#) for details.

Now, we can take a look at the table we have:
```{code-cell}
lgm_bio.t
```



## Data preprocessing
It appears that the table contains some missing values, such as `-99` in the 'age' column. To exclude these values from calculations, plots, and other analyses, we can mask them using the `mask_missing()` method:
```{code-cell}
lgm_bio.mask_missing(missval=-99)
```
The output shows that 625 out of 5000 values in the 'age' column are missing, while the other columns do not contain -99. However, the `'N/A'` entries in the 'sex' column also appear to represent missing values. We can mask `'N/A'` specifically in the 'sex' column as follows:
```{code-cell}
lgm_bio.mask_missing(['sex'], missval='N/A')
```
Now let us check how the above steps take effect:
```{code-cell}
lgm_bio.t
```

We can make more operations on the tables, as introduced [here](../basics/operations).

```{warning}
This step should be done before any subset is defined (in the [Defining subsets](#defining-subsets) section). This is due to the *static* nature of susbets (see [discussions here](../caveats/subset)).
```

## Matching and merging data
It is common for the information of interest to be recorded in several separate tables, such as when different properties are obtained from different surveys. Let us load more example tables about LGM:
```{code-cell}
lgm_addr = get_example('LGM.addr')
lgm_name = get_example('LGM.name')
lgm_house = get_example('LGM.house')
lgm_addr, lgm_name, lgm_house
```
`'LGM.addr'` contains the addresses, described by longitude called `'ra'` and latitude called `'dec'`:
```{code-cell}
lgm_addr.t
```
`'LGM.name'` contains the names of LGM:
```{code-cell}
lgm_name.t
```
and `'LGM.house'` contains information about the houses at each location:
```{code-cell}
lgm_house.t
```

To analyze our data, we may need to match rows that represent the same instance and merge all the table mentioned above (`lgm_bio`, `lgm_addr`, `lgm_name`, `lgm_house`). To match tables, we first need to import the appropriate "matchers":
```{code-cell}
from pyttop.matcher import ExactMatcher, SkyMatcher
```

Note that `lgm_bio`, `lgm_addr`, and `lgm_name` all contain the "id" column, so we can use the `ExactMatcher`to match rows where the "id" values are exactly the same. 
```{code-cell}
lgm_bio.match(lgm_name, ExactMatcher('id', 'id')) # `'id', 'id'` are colmun names for `lgm_bio` and `lgm_name`, respectively.
lgm_bio.match(lgm_addr, ExactMatcher('id')); # if the two column names are the same
```
As shown in the output, all 5000 rows in `"lGM.bio"` have matching rows in `"LGM.addr"`, but only 3500 of them have matches in `"LGM.name"`. In other words, 1500 "id" values in `"lGM.bio"` do not have corresponding entries in `"LGM.name"`.

To merge the `"LGM.house"` table, we can match it with `"LGM.addr"` based on the longitude and latitude coordinates `(ra, dec)`, using `SkyMatcher`, since the `"LGM.house"` table does not contain the "id" column.
```{code-cell}
lgm_addr.match(lgm_house, SkyMatcher());
```
In this case, the 'ra' and 'dec' columns are automatically detected. Since `(ra, dec)` may contain errors, entries with the closest positions and distances smaller than a threshold (defaulting to 1 arcsec) are matched. We can also explicitly specify the RA and Dec column names and the threshold `thres`:
```{code-cell}
lgm_house.match(lgm_addr, SkyMatcher(thres=1, coord='ra-dec', coord1='ra-dec'));
```

```{caution}
In astronomy, RA can be expressed in *hours*. If the RA column does not have units (e.g. `data.t['RA'].unit`), `SkyMatcher` assumes the values are in *degrees*. You may need to specify the units as described [here](../match/tree_match_basic.md#skymatcher). Failing to do so can lead to incorrect matching results, which may not be immediately apparent.
```
<!-- Otherwise, there can be problems in the matching result, which you may not even realize. -->


```{tip}
By matching using `data.match(data1, <...>)`, we try to find *one* best matching row (if any) in `data1` for each row in `data`. This implies differences between `lgm_addr.match(lgm_house, <...>)` and `lgm_house.match(lgm_addr, <...>)`when , e.g., some instances are included in one table but not in the other table. For more details, see the [matching documentation](match/match).
```

For simplicity, we will refer to `data.match(data1, <...>)` as "data1 matched to data". 

We can check how tables are matched to `"lGM.bio"`, either directly or through intermediate tables, using the `match_tree()` method:
```{code-cell}
lgm_bio.match_tree()
```
As shown, two datasets, `"LGM.name"` and `"LGM.addr"`, are matched directly to our base data, `"lGM.bio"`, using the `ExactMatcher`. `"LGM.house"` is matched to `"LGM.addr"` and can thus be merged as well. Although `"LGM.addr"` is also matched to `"LGM.house"`, since we already have `"LGM.addr"`, we do not need to count it again.

We can now merge all these tables using the `merge()` method:
```{code-cell}
lgm = lgm_bio.merge() # `lgm` is the merged dataset
lgm.t
```

The resulting table includes only 3500 rows, as only 3500 out of 5000 were matched when matching `"LGM.name"` to `"LGM.bio"`. However, we may want to retain rows without `"LGM.name"` information. Also, the resulting table appears redundant as, e.g., it includes the "id" column from three datasets: `'id_LGM.bio'`, `'id_LGM.name'`, and `'id_LGM.addr'`. We can control which columns should be ignored or merged. To achieve this, we can make some adjustments to the code:
```{code-cell}
lgm = lgm_bio.merge(
    keep_unmatched=['LGM.name'], # set it to True to keep unmatched rows for all tables
    merge_columns={
        'LGM.name': ['surname'], # only merges this column
    },
    ignore_columns={
        'LGM.addr': ['id'],
        'LGM.house': ['ra', 'dec'], # do not merge these columns
    },
)
lgm.t
```
As seen, those without match for "surname" are masked.

Now with our combined dataset, we are ready to analyze it.

## Defining subsets
It is common to be interested in subsets of a dataset. For example, we might want to know how many of the "little green men" have the surname "Smith". So let us import `Subset` and add a subset:
```{code-cell}
from pyttop.table import Subset

smith = lgm.add_subsets(
    Subset.by_value('surname', 'Smith'), # Defines a subset for rows where the 'surname' column is 'Smith'
    )
smith
```
As shown in the output, 38 out of 5000 entries have the surname "Smith".

We may also want to study potential systematic differences between sexes. To do so, let us create a "subset group" for the different sexes:
```{code-cell}
lgm.subset_group_from_values('sex')
lgm.get_subsets(name=['sex=Male', 'sex=Female'])
```

<!-- Let us define more subsets
```{code-cell}
lgm.add_subsets(
    # Subset()
    )
``` -->

Here are all subsets we have:
```{code-cell}
lgm.subset_summary()
```

There are more ways to define and use subsets. See the [subset documentation](../subset/subset) for details.

## Making plots
To quicky explore the patterns in our data, it is a good idea to create some plots.

Let us begin by plotting weight versus height for all entries:
```{code-cell}
lgm.scatter('height', 'weight', c='k', s=.5, subsets='all')
```
Is there any differences between sexes?
```{code-cell}
lgm.scatter('height', 'weight', s=.5, 
    group='sex',
    )
```
It seems there is no sexual dependance for height and weight. Then, can there be a dependance on age? We can color-code the markers by age:
```{code-cell}
lgm.scatter('height', 'weight', s=.5, c='age', cmap='jet', subsets='all')
```
Great, so their weight at a given height is determined by the age. 
By the way, we can change `lgm.scatter()` to `lgm.plots()` for more general usage (see [here](../plot/plot_single) for details):
```{code-cell}
lgm.plots(
    'scatter', # making a scatter plot
    cols=('height', 'weight'), # column 'weight' versus 'height'
    kwcols={'c': 'age'}, # color-coding: column 'age'
    s=.5, cmap='jet', # more scatter settings
    subsets='all', # plotting all rows
    )
```
It seems that the dependence on age can be better shown by the following correlation:
```{code-cell}
lgm.plots(
    'scatter', # making a scatter plot
    cols=('10*height - weight', 'age'), # 'np.log10(age)' versus'weight - 10**height' 
    eval=True, # we need to evaluate the above expression (since '10*height - weight' is not a column name)
    s=.5, # more scatter settings
    subsets='all', # plotting all rows
    )
```
This appears to be a good correlation. So let us define a new quantity $q$:
$$
q = W - 10H,
$$
where $W$ denotes weight and $H$ denotes height. We can calculate this and add it as a new column:
```{code-cell}
lgm.eval('10*height - weight', to_col='q') # evaluate '10*height - weight' and store it in the column 'q'
```

Great, let us report this in our new paper, *On the Properties of the Little Green Men*. However, we might want to set the labels more formally, rather than directly showing column names:
```{code-cell}
lgm.set_labels(
    height='$H/\mathrm{cm}$',
    weight='$W/\mathrm{g}$',
    age=r'$\tau/\mathrm{yr}$',
    )
```
Now, with more control, we can further customize the plots:
```{code-cell}
import matplotlib.pyplot as plt
fig, axes = plt.subplots(1, 2, figsize=(10.8, 4.8))

lgm.plots(
    'scatter', # making a scatter plot
    cols=('height', 'weight'), # column 'weight' versus 'height'
    kwcols={'c': 'age'}, # color-coding: column 'age'
    s=.5, cmap='jet', # more scatter settings
    subsets='all', # plotting all entries
    ax=axes[0], # plot it in the first axis
    )
lgm.plots(
    'scatter', # making a scatter plot
    cols=('10*height - weight', 'age'), # 'np.log10(age)' versus'weight - 10**height' 
    eval=True, # we need to evaluate the above expression ('10*height - weight' is not a column name)
    s=.5, c='k', # more scatter settings
    subsets='all', # plotting all entries
    ax=axes[1], # plot it in the second axis
    )

# manual adjustments
axes[0].set_title('W-H correlation')
axes[1].set_title('My correlation!')
axes[1].set_xlabel('$10H-W$')
plt.tight_layout()
```

As another example, let us study the house information of the LGMs. 
```{code-cell}
lgm.plots(
    'scatter',
    cols=('ra', 'dec'),
    s=1, c='k',
    subsets='all',
    )
```
Suppose we are interested in the locations where LGMs with the surname "Smith" live:
```{code-cell}
lgm.plots(
    'scatter',
    cols=('ra', 'dec'),
    subsets=(~smith, smith), # `smith` is a subset defined eariler; `~smith` is its complementary set (not in `smith`)
    iter_kwargs={ # for the two subsets, `~smith` and `smith`, use two different settings
        's': [.5, 5],
        'c': ['gray', 'r'],
        'marker': ['.', '*'],
        },
    )
```
There appears to be a group of LGMs with the surname "Smith" living around ra = 80, dec = 60.

There are many more features of the `plots()` method. Check the [documentation](../plot/plot) for details.


## Saving and exporting data

Now, we would like to save our merged dataset for later use. With the following code, we can save it to the `lgm.data` file, which can be loaded later by PyTTOP (see [here](../basics/IO) for details).
```{code-cell}
# lgm.save('path/to/lgm.data')
```
To publish or share our dataset, we may also need to export it using a certain format (see [here](../basics/IO.md#exporting-data-to-files) for more information):
```{code-cell}
# lgm.save('path/to/lgm.txt', format='ascii')
```

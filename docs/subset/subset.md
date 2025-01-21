# Row Subsets

A row subset is a selection of rows in the table. This feature is particularly useful for tasks such as generating plots for specific selections of rows. PyTTOP makes it simple to define and use subsets, and allows you to easily calculate intersection sets, union sets, and complementary sets.

```{tableofcontents}
```


```{caution}
Especially for TOPCAT users: The row subset in PyTTOP is defined differently from its counterpart in TOPCAT.
Due to the *static* nature of PyTTOP subsets (see [discussions here](../caveats/subset)), they are not updated in case the values in the table are changed, some rows are added or removed, etc. 
```


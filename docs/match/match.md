# Table Matching and Merging

Matching tables refers to the process of determining which row (if any) in one table corresponds to a specific row in another table. This is a common task when, for example, you need to combine different properties of the same set of objects that are stored in separate tables. 

Different algorithms for matching are possible. As of version 0.4.x, PyTTOP provides a matching framework called "tree matching", as defined [here](tree_match_basic). "Tree matching" is particularly useful for merging multiple tables simultaneously, even when different tables are matched using different criteria.

This section covers the basic matching settings and the method for merging tables based on the matchings.

```{note}
The term "tree matching" may not be a standard term elsewhere; it is a concept introduced in PyTTOP to describe the basic matching framework.

In future versions of PyTTOP, additional matching and merging methods outside of the "tree matching" framework may be introduced. These will extend the package's capabilities and allow for matching and merging tasks not supported by "tree matching."
```


```{tableofcontents}
```

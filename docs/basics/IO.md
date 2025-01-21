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

# Data Input/Output
This section covers how you can construct and save a `Data`, the class to store the table/catalogs. To use `Data`, you need to import it:
```Python
from pyttop.table import Data
```

## Reading tables from files
You can simply load the data given the path to the file and a name for the `Data`:
```Python
data = Data('path/to/file.fits', name='data')
```
And you may access the data table with the `data.t` property. `data.t` is an `astropy.table.Table` object: see [Astropy's documentation on Data Tables](https://docs.astropy.org/en/stable/table/index.html) on how to use it.


```{tip}
A name is not mandatory when constructing a `Data`, but it is suggested to specify one, as the names may be used to distinguish different `Data` instances, generating understandable names, etc. It would usually be a good idea to set the name the same as the variable name. In case `Data` is initialized with a path to a file and no name is given, it will be automatically set to the file name.
```

You may need to specify the format of the file (especially for ascii files). For example:
```Python
data = Data(filename, format='ascii.cds', name='data')
``` 
The file is loaded with the `astropy.table.Table.read()` method, so refer to [Astropy's Built-In Table Readers/Writers](https://docs.astropy.org/en/stable/io/unified.html#built-in-readers-writers) for supported formats.


<!-- ```{tip}
A data table is saved as an  within a `Data`, and one may construct a `Data` in similar ways to constructing . However, 
``` -->


## Constructing a `Data` from supported objects
A `Data` can be created with anything that may be converted to an `astropy.table.Table` object, with the similar ways of constructing `astropy.table.Table` objects. For example:
```Python
data = Data(data_object, name='data')
```
The `data_object` may be a `pandas.DataFrame`, an `astropy.table.Table`, or anything that can be converted to `astropy.table.Table` with `astropy.table.Table(data_object)`.

You may also create a `Data` from scratch:
```Python
data = Data(name='data')
data['col1'] = [1, 2, 3]
```
In general, you can construct a `Data` as if constructing an `astropy.table.Table` by simply replacing `Table` with `Data`. See [Astropy's documentation on constructing a Table](https://docs.astropy.org/en/stable/table/construct_table.html) for details.


## Saving and loading `Data`
To save a `Data` to a file, you can simply use the `data.save()` method:
```Python
data.save('path/mydata.data')
```
This will save the `Data` with PyTTOP's built-in file format. A file with such a format can be simply loaded with:
```Python
Data.load('path/mydata.data')
```
Unless you are sharing your table with people that do not use PyTTOP, it is generally recommended to save `Data` with the built-in format, as the format preserves many properties of it (e.g., the metadata, name, [row subsets](../subset/subset)), whereas most of them will be lost if you save `Data` with e.g. ascii, fits or any other formats.

```{tip}
The file extension `.data` is just a convention. When loading a file with the built-in format, the extension is not checked.
```



## Exporting `Data` to files
To write the `Data` to a file with a specified format, you can use `data.save()` the same way as using the `write()` method of an `astropy.table.Table` (see [Astropy's Built-In Table Readers/Writers](https://docs.astropy.org/en/stable/io/unified.html#built-in-readers-writers) for supported formats):
```Python
data.save('data.fits', format='fits')
```

```{warning}
PyTTOP saves relavant information in the metadata of tables and their columns. This helps with debugging and history tracking. However, the metadata may include information that you might not want to publish, such as the paths of your data in your system, operations you have performed on the data, etc. For more information on the metadata, see the documentation [here](../basics/meta). If you want to remove the metadata before exporting it for publication, you may execute: 
```Python
data.meta.clear()
for coln in data.colnames: 
    data[coln].meta.clear()
```





# Changelog
This log highlights key changes relevant to users. For more technical details, see the [commit history](https://github.com/ycwang-astro/pyttop/commits/).

## 0.4.5 (20 Sep 2026)
### Additions and new features
- Labels for simple mathematical expressions are now generated according to column labels set by `Data.set_labels()`.
- Columns can now be selected using regular expressions matching column names, e.g. `data[re.compile(pattern)]`.
- `Data.mask_missing()` now supports specifying columns using regular expressions, e.g. `data.mask_missing(cols=re.compile(pattern))`
- Added `pyttop.plot.merged_legend()`.
- `pyttop.plot.hist()` now supports logarithmically spaced bins along the x-axis using the `logx=True` argument.

### Improvements
- `Data.match_tree()` now includes the fractions of rows matched in its printed information. 
- Path-like objects (e.g. `pathlib.Path`) are now supported when saving, loading, and initializing `Data` objects.
- Improved handling of input expressions and namespaces in `Data.eval()`.
- `Data.set_labels()` now accepts a dictionary as input.
- Improved label generation for `Subset.by_range()` and `Subset.by_value()`.

### Changes
- `Data.match()` no longer returns the `Data` object itself. Code such as `data.match(data1, matcher).merge()` should be changed to `data.match(data1, matcher); data.merge()`.
- The global namespace available to `Data.eval()` is now more restricted.


## 0.4.4 (9 Jan 2026)
### Additions and Improvements
- Added a plotting function [`pyttop.plot.binned_quantiles()`](../api/plotfuncs.rst#pyttop.plot.binned_quantiles).
- The arguments `merge_columns` and `ignore_columns` in `Data.merge()` now support regular expressions.
- `Data` and `Subset` can now be imported directly from the top-level package (`from pyttop import Data, Subset`).
- Added new special subsets: `$unmasked:` and `$eval:`.

### Changes
- Renamed `config.data_name_repr_maxlen` to `config.display.data_name_maxlen`.
- `ExactMatcher()` now applies stricter checks to dtypes, etc. 

## 0.4.3
### Additions and Improvements
- `Data` can now be initialized from a file-like object (e.g., `f = open(...)`).
- The revised and fixed plotting function [`pyttop.plot.refline()`](../api/plotfuncs.rst#pyttop.plot.refline) is now available.

### Bug Fixes
- Fixed an unclear error message when initializing `Data(...)` with another `Data` instance.

### Changes
- Long data names are now abbreviated. The maximum length can be configured via `pyttop.config.data_name_repr_maxlen` (renamed to `config.display.data_name_maxlen` in a later version).

## 0.4.2
### Additions and Improvements
- API doc improved
- Added example data "P1"
- Added `Data.sort()`

### Bug Fixes
- Fixed: function missing, unexpected automatic matplotlib plot settings, and additional window popup when making plots in some cases
- Fixed: `get_subsets()` not returning a list as expected in some cases


## 0.4.1
### Bug Fixes
- Fixed an error in merging when using a later version of astropy

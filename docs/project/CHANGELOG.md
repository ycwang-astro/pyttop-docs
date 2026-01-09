# Changelog
This log highlights key changes relevant to users. For more technical details, see the [commit history](https://github.com/ycwang-astro/pyttop/commits/).

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

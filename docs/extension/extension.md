# Extending the capability of PyTTOP

By design, PyTTOP is extendable. Both matching and plotting are handled not by the `Data` class itself, but by standalone matchers (e.g., `pyttop.matcher.ExactMatcher`) and plot functions (e.g., `pyttop.plot.plot`). This allows you to define custom matchers and plot functions if the [built-in matchers](../match/tree_match_basic.md#built-in-matchers) and [built-in plot functions](../plot/plot_single.md#built-in-plot-functions) do not meet your needs. 

In other words, you have full control over how matching is implemented (as of PyTTOP 0.4.x, within the [tree matching](../match/tree_match_basic.md#basics-concepts-of-tree-matching) framework) and how plots are generated on each axis. You are also welcome to contribute to PyTTOP by creating a pull request [here](https://github.com/ycwang-astro/pyttop/pulls) to add your custom plot functions and matchers.

<!-- ```{tip}

``` -->

```{tableofcontents}
```

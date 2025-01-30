# Writing Custom Matchers for Tree Matching

A custom matcher in the tree matching framework should follow the [definition](../match/tree_match_basic.md#basics-concepts-of-tree-matching) for matching a table with another. 

To define a custom matcher, some basic knowledge of object-oriented programming in Python is required. Below is a template for creating a custom matcher: 

```Python
class MyMatcher():
    def __init__(self, <define necessary arguments here>):
        # initialize with necessary arguments
        pass
    
    def get_values(self, data, data1, verbose=True): # `data1` is "matched with" `data`
        # prepare the data necessary for matching (if needed)
        # for example: if column names are provided, prepare the column values given `data` and `data1` here
        pass
    
    def match(self):
        # perform the matching process and calculate:
        # idx : array of shape (len(data), ). 
        #     the index of a row in `data1` that best matches the rows in `data`
        # matched : boolean array of shape (len(data), ).
        #     whether the rows in `data` can be matched to those in `data1` (i.e., whether a corresponding row exists in `data1`)
        # note: `len(data)` is the number of rows in `data`
        return idx, matched

    def __repr__(self):
        return 'MyMatcher' # replace with a text representation of the matcher object
        # this will be shown when calling `data.match_tree()`
```

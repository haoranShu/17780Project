# Assigment 3: Improving the Pandas API

We propose to improve the Pandas API, which is an open source library providing high-performance, easy-to-use data structures and data analysis tools for Python. The API is widely in use by data scientists and is renowned for its power in analyzing time series like data.

The API, though powerful, could be hard to use and easy to misuse in some cases. Also, the resulting user code can sometimes be unreadable, given the number of parameters of a method and the abuse of int, string and boolean flags instead of enums. (This problem is worse with Python because it does not need to be compiled, but if you use an IDE, enums will be in different color) Last but not least, we contend that some sematic of the API is flawed and users can make mistakes and not discover them forever.

The library is too big for us to redesign, thus we will focus on the fixing the APIs related to Series, DataFrame and Index, which are the core data structures of the API. Also, we will try to improve the Groupby API, which is one of the most commonly used functionality of the library.


## Part I. Series, DataFrame and Index: the building blocks

Series and DataFrames are the core data structures of Pandas. They both are indexed by indexed objects, and DataFrames have one row index and one column index. Different from what people might usually assume about indices, in Pandas all indices could contain duplicate entries, which we will discuss in Part III. Also, multiple kinds of indices are supported, the most popular ones being DatetimeIndex, MultiIndex and general object Index. We found there is a lot of space for improving the API surrounding these Data Structures but we will be focusing on the following three aspects.

### Construction



### Indexing and Iteration

### Computation of derived statistics

### Examples of user code

## Part II. Groupby: a powerful API made hard to use

### Parameter Explosion

### Inconsistent behavior between Groupers and Groupby

### Limitations of Groupby with mappings

### Examples of user code

## Part III. Discussion of whether duplicate indices should be allowed in Series and DataFrames

### Undefined or Ill-defined Behaviors

### Examples of user code

## Pandas documentation

The documentation for Pandas could be found [here](http://pandas.pydata.org/pandas-docs/stable/).

# Assigment 3: Improving the Pandas API

We propose to improve the Pandas API, which is an open source library providing high-performance, easy-to-use data structures and data analysis tools for Python. The API is widely in use by data scientists and is especially renowned for its power in analyzing time series.

The API, though powerful, could be hard to use and easy to misuse in some cases. Also, the resulting user code can sometimes be unreadable, given the often long parameter lists of methods and constructors, and the abuse of int, string and boolean flags instead of enums. Last but not least, we contend that some sematic of the API is flawed and users can easily make mistakes and not discover them forever.

The library is too big for us to redesign, thus we will focus on the fixing the APIs related to Series, DataFrame and Index, which are the core data structures of the API. Also, we will try to improve the Groupby API, which is one of the most commonly used functionality of the library.


## Part I. Series, DataFrame and Index: the building blocks

Series and DataFrames are the core data structures of Pandas. They both are indexed by Index objects, and DataFrames have one row index and one column index. Different from what people might usually assume about indices, in Pandas all indices could contain duplicate entries, which we will discuss in Part III. Also, multiple kinds of indices are supported, the most popular ones being DatetimeIndex, MultiIndex and general object Index. We found there is a lot of space for improving the API surrounding these Data Structures but we will be focusing on the following three aspects.

### Construction

* These objects could be constructed by either constructors or factories and the factories only cover a small part of all object creation use cases, which is a wierd thing by itself.
* Also, the authors of the API tried hard to make the constructors as versatile and powerful as possible by adding a bunch of optional parameters. To make it worse, often times these parameters are boolean or string flags. For example, the [DatetimeIndex constructor](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.DatetimeIndex.html#pandas.DatetimeIndex) accepts as many as 12 parameters, 3 of them being boolean flags and 4 of them are *possibly* strings. Note that string flags are especially bad for Python because the program will only fail at run-time if there is a typo in the string. Enum flags, however, will be checked by IDE.
* Last but not least, some constructors also include functionalities that should have been decoupled. For example, the [DataFrame constructor](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.DataFrame.html#pandas.DataFrame) also is able to convert the input data source to another data type. 

**Possible solutions**
* Provide more factories to cover all use cases and prohibit the direct use of constructors
* Provide builders for objects that have too many optional parameters
* Provide enums for flags
* Decouple functionalities that are not essentially relevant to objecgt creation into separate class methods

### Indexing and Iteration

* Indexing could be done in the following ways for DataFrames:
~~~~
    df[column_name]                         # select one column from DataFrame, returns Series

    df[[column_name1, column_name2]]        # select multiple columns from DataFrame, returns DataFrame

    df[a:b]                                 # select multiple rows from DataFrame by row numbers

    df[row_name1:row_name2]                 # select multiple rows from DataFrame by row names

    df.column_name                          # select one column from DataFrame, returns Series
                                            # can only be used for column_names containing no spaces

    df.loc[row_names, column_names]         # select by row names and column names, returns DataFrame

    df.iloc[row_numbers, column_numbers]    # select by row numbers and column numbers, returns DataFrame

    # and more...
~~~~
* There are simply too many ways to index and select from the DataFrames/Series and they are messed up. Different syntaxes have no hint of what kind of indexing they do (by column or by row, by name or by number) and some syntactic sugar applies only to restricted cases.
* DataFrame.loc and DataFrame.iloc are class methods but they are used with [] instead of (), which is very counter-intuitive and easy to get wrong. Again, it is expensive to have a typo in Python, because there is no compiler to check that for us.
* People can argue that one can stick to only one or two methods of indexing and selection so he/she will eventually get used to the syntax, but it would still be hard to read other people's code if they have chosen to use a different set of indexing methods.
* Iterations could be done in the following way
~~~~
    for col_label, col in df.iteritems():               # col is a Series
        for row_label, entry in col.iteritems():        # entry is an element
            . . .

    # or

    for row_label, row in df.iterrows():                # row is a Series
        for col_label, entry in row.iteritems():        # entry is an element
            . . .
~~~~
It is clear that DataFrame.iteritems() should have been named as DataFrame.itercols().

**Possible Solutions**
* Remove redundant indexing methods
* Rename iteration methods

### Computation of descriptive statistics

It is common usecase to compute some descriptive statistics from a Series or some columns/rows of a DataFrame, for example, [mean](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.DataFrame.mean.html#pandas.DataFrame.mean). There are more than several dozens of similar methods and there are a couple of problems with these APIs.

* These computations could be applied either along rows or along columns, and this configuration is indicated by a numeric flag in the function call: 0 for index (row) and 1 for columns. Even the documentation itself is ambiguous: does "1 for columns" mean the statistic will be computed along each column or across each column? The resulting user code is just unreadable and usually requires the programmer to leave a comment.
~~~~
    # get mean of each column in a dataframe
    df.mean(axis=0)

    #get mean of each row in a dataframe
    df.mean(axis=1)
~~~~
* MultiIndex always requires special care and one parameter is dedicated to that matter (in a lot of other APIs as well).

* Parameter *numeric_only* is rarely of any use, and when it is, actually encourages poor programming practices and yields unreliable results

**Possible Solutions**
* Reflect axis in function name or change numeric flag to enums
* Remove unnecessary and rarely used parameters
* Currently cannot think of a great way to address the MultiIndex problem

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

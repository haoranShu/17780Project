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

    # get mean of each row in a dataframe
    df.mean(axis=1)
~~~~
* MultiIndex always requires special care and one parameter is dedicated to that matter (in a lot of other APIs as well).

* Parameter *numeric_only* is rarely of any use, and when it is, actually encourages poor programming practices and yields unreliable results

**Possible Solutions**
* Reflect axis in function name or change numeric flag to enums
* Remove unnecessary and rarely used parameters
* Currently cannot think of a great way to address the MultiIndex problem

## Part II. Groupby: a powerful API made hard to use

Groupby is a commonly used tool in Pandas to divide rows of a DataFrame into groups and then do some useful computation based on each group. However, several design mistakes make groupby hard to use in some common usecases. Sometimes we have to write clutter code to achieve a simple purpose.

Pandas also provides a Grouper object to feed to the groupby() method but there are some problems there as well.

### Parameter Explosion

Documentation for both [groupby()](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby) and [Grouper constructor](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.Grouper.html#pandas.Grouper) have the same old problem of long parameter lists. The optional parameters would actually only be used in a couple of combinations and the API should have been exported via different methods or at least overloadings that are documented separately.

Some options, like sort, are actually orthogonal functionalities that should have been decoupled from the Groupby component.

### Inconsistent behavior between Groupers and Groupby

The semantics of Groupers and groupby() are subtly different and may lead to unexpected results. Grouper objects define a set of groups and then bin the rows into their corresponding bins and thus there could be empty groups, auto-filled with NaN for any subsequent computations. The method groupby(), however, maps rows according to a specific mapper on the index of the DataFrame and will never result in an empty group.

This difference is not found in the documentation, but might lead to horrible results, especially if NaN set to be filled by 0 somewhere in the code later on. The following usecase illustrates why this could be disastrous.
~~~~
    # Usecase: compute the intra-day mean stock price of Google
    #
    # Note that stock price data is only available for weekdays
    #
    #    stockprice layout:
    #                            price
    #       date    
    #    2018-10-04 09:10:12     1000.0
    #    2018-10-04 11:12:13     1000.0
    #    2018-10-05 10:00:03     1020.0
    #    2018-10-05 12:23:41     1020.0
    #    2018-10-05 16:39:12     1020.0
    #    2018-10-08 15:34:54      998.0
    #

    # attempt 1
    meanprice = stockprice.groupby(Grouper('date', freq='1D')).mean()
    # result:
    #                price
    #       date    
    #    2018-10-04  1000.0
    #    2018-10-05  1020.0
    #    2018-10-06     NaN
    #    2018-10-07     NaN
    #    2018-10-08   998.0
    #
    
    # attempt 2
    def mapper(date):
        return date.year*10000 + date.month*100 + date.day

    meanprice = stockprice.groupby(mapper).mean()
    # result:
    #                price
    #       date    
    #    20181004    1000.0
    #    20181005    1020.0
    #    20181008     998.0
    #

    # Attempt 1 is prettier but the result is not what we wanted, attempt 2 yields the
    # desired result but is unnecessarily spoilerplate. We should have had the merits
    # of both.
~~~~
The empty group behavior is sometimes desirable and sometimes not. The fact that Grouper cannot take a mapper makes it hard to have an empty group when we actually want to, like in the following usecase.
~~~~
    # Usecase: Given DataFrame A indexed by Andrew ID, with columns Score, Major and Year of Study,
    #          want to get the average score for groups ‘under year 3’, ‘year 3’, and ‘year 4 or above’
    #          if no student in such group, fill NaN

    # attempt
    import numpy as np

    def mapper(year):
        if year <= 2:
            return ‘Under Year 2’
        elif year > 3:
            return ‘Year 3’
        else:
            return ‘Year 4 and Above’

    year_groups = [‘Under Year 2’, ‘Year 3’, ‘Year 4 and Above’]

    mean_by_year = A[[‘Year of Study’, ‘Score’]].set_index(‘Year of Study’).Groupby(mapper).mean()

    for year in year_groups:
        if year not in mean_by_year.index:
            mean_by_year.loc[year] = np.NaN
~~~~
This code is purely evil.

**Possible solutions**
* Decouple orthogonal functionalities from both groupby() and Grouper API
* Document the different behavior and add new parameters to control the behavior

## Pandas documentation

The documentation for Pandas could be found [here](http://pandas.pydata.org/pandas-docs/stable/).

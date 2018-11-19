# Assigment 3: Improving the Pandas API

We propose to improve the Pandas API, which is an open source library providing high-performance, easy-to-use data structures and data analysis tools for Python. The API is widely in use by data scientists and is renowned for its power in analyzing time series like data.

The API, though powerful, could be hard to use and easy to misuse in some cases. Also, the resulting user code can sometimes be unreadable, given the number of parameters of a method and the abuse of int, string and boolean flags instead of enums. (This problem is worse with Python because it does not need to be compiled, but if you use an IDE, enums will be in different color) Last but not least, we contend that some sematic of the API is flawed and users can make mistakes and not discover them forever.

The library is too big for us to redesign, thus we will focus on the fixing the APIs related to Series, DataFrame and Index, which are the core data structures of the API. Also, we will try to improve the Groupby API, which is one of the most commonly used functionality of the library.


## Part I: Series, DataFrame and Index: the building blocks

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to install the software and how to install them

```
Give examples
```

### Installing

A step by step series of examples that tell you how to get a development env running

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo

## Pandas documentation

The documentation for Pandas could be found [here](http://pandas.pydata.org/pandas-docs/stable/).

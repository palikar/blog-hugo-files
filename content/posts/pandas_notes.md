+++
title = "Notes on learning Pandas"
author = ["Stanislav Arnaudov"]
description = "My notes on pandas when I started looking into the library"
date = 2018-07-13T00:00:00+02:00
keywords = ["machine-learning", "python", "pandas"]
lastmod = 2019-10-12T11:27:00+02:00
categories = ["machine-learning"]
draft = false
weight = 100
+++

<div class="NOTES">
  <div></div>

For transparency's sake - those are my notes while learning about _pandas_ from [this](https://www.tutorialspoint.com/python%5Fpandas/index.htm%20) tutorial. At times this here is just copy-paste from it, at others, it's my own thoughts and explanations. This is meant to be a condensed version of the tutorial more or less just for me. If you found it useful though, well, good for you, I guess.

</div>


## Abstract {#abstract}


### Basic {#basic}

**pandas** is a Python package providing fast, flexible, and expressive data structures designed to make working with "relational Heterogeneous data" or "labeled" data both easy and intuitive. It aims to be the fundamental high-level building block for doing practical, **real world** data analysis in Python. Additionally, it has the broader goal of becoming the most powerful and flexible open-source data analysis/manipulation tool available in any language. It is already well on its way toward this goal. <br /> <br /> It's like database-like objects to deal with your data.

-   data could be loaded from _.csv_ files and be written again
-   some basic statistical analysis is possible


### Features {#features}

A shortlist of features as advertised on the official site of _pandas_.

-   Fast and efficient DataFrame object with the default and customized indexing.
-   Tools for loading data into in-memory data objects from different file formats.
-   Data alignment and integrated handling of missing data.
-   Reshaping and pivoting of data sets.
-   Label-based slicing, indexing, and subsetting of large data sets.
-   Columns from a data structure can be deleted or inserted.
-   Group by data for aggregation and transformations.
-   High-performance merging and joining of data.
-   Time Series functionality.


### Tl;DR {#tl-dr}

Managing data in structures and easily manipulating the data.


## Basic Structures {#basic-structures}


### DataFrame {#dataframe}

It's like a table in RDB.

-   Columns are attributes
-   Rows are the actual data
-   Indices are the labels for each row

This is the most common object when dealing with data and using _pandas_. The data in the object is heterogeneous, the shape and the size are mutable. The last part means that the models you build in one object that contains data can easily(ish) be transformed into another model(by model I mean the way your data is structured- names and count of columns(attributes), indexing of the rows, type of the data inside). <br /> <br /> The basic construction of the DataFrame object is as follows:

```python
pandas.DataFrame( data, index, columns, dtype, copy)
```

-   `data` - ndarray, series, map, lists, dict, constants..etc. The raw data that will be stored in the DataFame. This could be for example a list of lists. In this case, the 'inner' lists will become the columns and the column names will be given through **columns**. The number of the columns must be the same as the number of inner lists
-   `index` - the 'names' of the rows. Usually just an index (0,1,2,3....). The size of this must be the count of the entries in the dataset
-   `columns` - the names of the columns
-   ... - the other ones are none of our concern

Some of the constructions:

```python
import panda as pd
import numpy as np


data = [['Alex',10],['Bob',12],['Clarke',13]]
df = pd.DataFrame(
    data,
    columns=['Name','Age'])

df = pd.DataFrame(
    np.random.randn(10, 3),
    columns = ['col1','col2','col3']
)

data = {
    'Name':['Tom', 'Jack', 'Steve', 'Ricky'],
    'Age':[28,34,29,42]
}
df = pd.DataFrame(data)

data = {'Name':['Tom', 'Jack', 'Steve', 'Ricky'],'Age':[28,34,29,42]}
df = pd.DataFrame(data, index=['rank1','rank2','rank3','rank4'])


d = {'one' : pd.Series([1, 2, 3], index=['a', 'b', 'c']),
      'two' : pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])}
df = pd.DataFrame(d)
```

All of the constructions are pretty intuitive and easy to understand. The thing to remember - all of the possible way that you can construct _DataFrame_. <br /> <br /> The column selections is just as easy:

```python
df['column_name'] # return an Series containing the data in the respective column
```

Adding of columns is possible:

```python
 #Adding a new column by passing as Series
df['new']=pd.Series([10,20,30],index=['a','b','c'])
print df
```

as well as deletion:

```python
del df['important']
```

Selection of rows either by index or by lable (the thing that was in the **index** attribute in the constructor)

```python
df.loc['b']  # by lable
df.iloc[2]   # by index
df[2:4]      # splicing
```

Appending rows to an existing _DataFrame_

```python
df = pd.DataFrame([[1, 2], [3, 4]], columns = ['a','b'])
df2 = pd.DataFrame([[5, 6], [7, 8]], columns = ['a','b'])

df = df.append(df2)
```


### Series {#series}

Series is a one-dimensional labeled array capable of holding data of any type (integer, string, float, python objects, etc.). The axis labels are collectively called index. Construction:

```python
pandas.Series( data, index, dtype, copy)
```

The same thing as the _DataFrame_! Some examples:

```python
data = np.array(['a','b','c','d'])
s = pd.Series(data,index=[100,101,102,103])

data = {'a' : 0., 'b' : 1., 'c' : 2.}
s = pd.Series(data)

print s[0]

print s[1:3]

print s[-3:] # the last three elements

```


### DataPanel {#datapanel}

From what I understand, this is not widely used and I don't think I need it for my project so... nah. <span class="underline">Skip!!!</span>


## Basic usage {#basic-usage}


### DataFrame basic function {#dataframe-basic-function}

The most useful functions of the DataFrame-class

-   `T` : Transposes rows and columns.
-   `axes` : returns a list with the row axis labels and column axis labels as the only members.
-   `dtypes`: Returns the dtypes in this object.
-   `empty` : True if NDFrame is entirely empty [no items]; if any of the axes are of length 0.
-   `ndim` : umber of axes / array dimensions. This is just two
-   `shape` : Returns a tuple representing the dimensionality of the DataFrame. The first element is the number of rows, the second - the number of attributes
-   `size` : umber of elements in the NDFrame.
-   `values`: Numpy representation of NDFrame.
-   `head()`: Returns the first n rows. Could be used as `df.head(n)/df.tail(n)` to get the first/last **n** elements.
-   `tail()`: Returns last n rows.


### Basic statistics {#basic-statistics}

A bunch of simple 'statistical' functions can be applied on the columns of a DataFrame object. Those include:

-   `count()` : Number of non-null observations
-   `sum()` : Sum of values
-   `mean()` : Mean of Values
-   `median()` : Median of Values
-   `mode()` : Mode of values
-   `std()` : Standard Deviation of the Values
-   `min()` : Minimum Value
-   `max()` : Maximum Value
-   `abs()` : Absolute Value
-   `prod()` : Product of Values
-   `cumsum()` : Cumulative Sum
-   `cumprod()` : Cumulative Product

An example that demonstrates some of these:

```python
import pandas as pd
import numpy as np

#Create a Dictionary of series
d = {'Name':pd.Series(['Tom','James','Ricky','Vin','Steve','Smith','Jack',
   'Lee','David','Gasper','Betina','Andres']),
   'Age':pd.Series([25,26,25,23,30,29,23,34,40,30,51,46]),
   'Rating':pd.Series([4.23,3.24,3.98,2.56,3.20,4.6,3.8,3.78,2.98,4.80,4.10,3.65])}

#Create a DataFrame
df = pd.DataFrame(d)
print df.mean()
print df['Age'].mean()
print df['Age'].min()
print df['Age'].max()
print df['Age'].std()
```

There exists also a `describe` function that shows summarized information about the data in the _DataFrame_. This includes `mean`, `std` and **IQR** values. The function excludes the textual columns and looks only at the numeric columns. **include** is the argument which is used to pass necessary information regarding what columns need to be considered for summarizing. It can be:

-   `object` − Summarizes String columns
-   `number` − Summarizes Numeric columns
-   `all` − Summarizes all columns together

```python
import pandas as pd
import numpy as np

#Create a Dictionary of series
d = {'Name':pd.Series(['Tom','James','Ricky','Vin','Steve','Smith','Jack',
   'Lee','David','Gasper','Betina','Andres']),
   'Age':pd.Series([25,26,25,23,30,29,23,34,40,30,51,46]),
   'Rating':pd.Series([4.23,3.24,3.98,2.56,3.20,4.6,3.8,3.78,2.98,4.80,4.10,3.65])}

#Create a DataFrame
df = pd.DataFrame(d)
print df.describe()
```

This gives us:

|       | Age       | Rating    |
|-------|-----------|-----------|
| count | 12.000000 | 12.000000 |
| mean  | 31.833333 | 3.743333  |
| std   | 9.232682  | 0.661628  |
| min   | 23.000000 | 2.560000  |
| 25%   | 25.000000 | 3.230000  |
| 50%   | 29.500000 | 3.790000  |
| 75%   | 35.500000 | 4.132500  |
| max   | 51.000000 | 4.800000  |

OK, kinda. The table is from me and I am kinda showing off.


## Applying Functions on data in _DataFrame_ {#applying-functions-on-data-in-dataframe}

There are a few ways that we can transform a _DataFrame_ into another one by applying a map-like function on the data. Depending on our needs we have the following options:

-   `pipe()` - Table wise Function Application:
-   `apply()` - Row or Column Wise Function Application
-   `applymap()` - Element wise Function Application


### Piping {#piping}

From the official documentation:

> Use .pipe when chaining together functions that expect Series, _DataFrames_ or _GroupBy_ objects

From what I understand - One would use that when applying a bunch of functions on all elements of _DataFrame_ (or whatever) while still having the possibility that the applied function takes more than just one argument. For example, let's add two to every element in _DataFrame_ through an adder function that just adds its two arguments.

```python
import pandas as pd
import numpy as np

def adder(ele1,ele2):
   return ele1+ele2

df = pd.DataFrame(np.random.randn(5,3),columns=['col1','col2','col3'])
df = df.pipe(adder,2)
print(df)
```

We can also do something like this:

```python
df.pipe(foo_fun1, arg1=1).
pipe(foo_fun2, arg2=2).
pipe(foo_fun3, arg3=3)
```

This applies the three functions one after the other while the second argument of those functions is 1, 2 and 3.


### Applying {#applying}

The `apply` function of _DataFrame_ applies function on whole columns(or rows). The given function to be applied must take one argument - Series - and return again a Series. Think of it like that - they give you a whole array of numbers, you make something with it and give back a different (or not different) array of the same size. An illustrative example:

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(np.random.randn(5,3),columns=['col1','col2','col3'])
return df.apply(np.mean)
```

Which gives us:

```text
col1    0.228874
col2   -0.561032
col3   -0.321606
dtype: float64
```

Note how the **np.mean** return a single number so in the final result there is only one row - the mean of each column.


### ApplyMap {#applymap}

Not all functions can be vectorized (neither the Numpy arrays which return another array nor any value), the methods `applymap()` on DataFrame and analogously map() on Series accept any Python function taking a single value and returning a single value. This is similar to the **pipe** but it's less flexible. It just treats the whole _DataFrame_ as on a big list and performs a mapping function on it. Example:

```python
import pandas as pd
import numpy as np

# My custom function
df = pd.DataFrame(np.random.randn(5,3),columns=['col1','col2','col3'])
return df.applymap(lambda x:x*100)
```

With result:

```text
col1        col2        col3
0  124.741017  -39.997356 -197.724001
1  -83.817763   56.487720  -16.127531
2  173.797264  187.089676  -38.871016
3  -94.927338  -60.133882   15.271702
4 -167.875460   83.420648 -179.131762
```


## Iterating and sorting over data in structures {#iterating-and-sorting-over-data-in-structures}


### Iterating _DataFrame_ {#iterating-dataframe}

Using a _DataFrame_ object in a plane _for_-loop iterates over the names of the columns.

```python
import pandas as pd

df = pd.DataFrame({
    'A': [1,2,3,4,5],
    'x': [1,2,3,4,5],
    'y': [1,2,3,4,5],
    'C': [1,2,3,4,5],
    'D': [1,2,3,4,5]
    })

for col in df:
   print(col)
```

This just prints A, B, C,...,etc.

```text
A
C
D
x
y
```

<br /> <br /> Iterating over the date in the _DataFrame_ can be done in several ways:

-   `iteritems()` − to iterate over the (key,value) pairs. Key here again is the 'index'-name-thing that is configurable through the **index** in the constructor.
-   `iterrows()` − iterate over the rows as (index,series) pairs. Here the index is just a number.
-   `itertuples()` − iterate over the rows as named tuples

The most useful of the above is probably `iterrows()`.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(np.random.randn(4,3),columns = ['col1','col2','col3'])
for row_index,row in df.iterrows():
   print(str(row_index) + "\n" + str(row))
```

```text
0
col1    2.117955
col2   -0.263560
col3   -0.600124
Name: 0, dtype: float64
1
col1   -0.620081
col2   -1.355647
col3   -0.568608
Name: 1, dtype: float64
2
col1    1.792265
col2   -0.494137
col3   -1.395912
Name: 2, dtype: float64
3
col1   -1.189506
col2   -0.479746
col3    0.329728
Name: 3, dtype: float64
```

To note is that the iterated _row_-objects contain information for each column so if you want to get the second column of the row:

```text
row.col2
```


### Sorting _DataFrame_ {#sorting-dataframe}

There are two possibilities for sorting:

1.  By lable - i. e. by index
2.  By value of some column

The first one is useful when the data is saved out of order. By appropriately creating the index in the construction and then sorting by lable, you can load the data in memory in the right order. <br /> <br /> The two functions are:

-   `sort_index([ascending=True/False])`
-   `sort_values(by=col_name,[ascending=True/False])`

By default _ascending_ is set to _True_. Example:

```python
import pandas as pd
import numpy as np

unsorted_df = pd.DataFrame(
    np.random.randn(10,2),
index=[1,4,6,2,3,5,9,8,0,7],
    columns = ['col2','col1']
)

sorted_index_df = unsorted_df.sort_index()
sorted_val_df = unsorted_df.sort_values(by='col2')

print(sorted_index_df)
print("-------------")
print(sorted_val_df)
```

Output:

```text
       col2      col1
0  0.562948  0.768513
1  1.776865 -0.217141
2 -0.040029 -2.300772
3 -1.695105  0.294038
4  0.163922  0.934361
5  0.998288 -1.149822
6 -0.641102  0.539689
7  1.190690  0.027898
8  0.745714  0.916117
9  0.144558  2.581345
-------------
       col2      col1
3 -1.695105  0.294038
6 -0.641102  0.539689
2 -0.040029 -2.300772
9  0.144558  2.581345
4  0.163922  0.934361
0  0.562948  0.768513
8  0.745714  0.916117
5  0.998288 -1.149822
7  1.190690  0.027898
1  1.776865 -0.217141
```


## Slicing {#slicing}

There are several custom ways of slicing through data that are optimized and are the recommended way of slicing data when dealing with production code.

1.  **loc()** - label based indexing. Used as df.loc[</rows/>,</columns/>]. For _rows_ and _columns_ could be given pretty much everything that makes sense - single char, list of labels, slice object, boolean array.

    ```python
    df.loc[['a','b','f','h'],['A','C']]
    ```
2.  **iloc()** - index based indexing. Used the same way as **loc()** but just with integer indices.

    ```python
    df.iloc[1:5, 2:4]
    ```


## Tougher statistics {#tougher-statistics}


### Some functions {#some-functions}

There are some useful statistical functions already in _pandas_ that help with the understanding and analyzing the behavior of data

1.  **Percent\_change** -This function compares every element with its prior element and computes the change percentage.

    ```python
    import pandas as pd
    import numpy as np
    s = pd.Series([1,2,3,4,5,4])
    print(s.pct_change())
    print("-----")
    df = pd.DataFrame(np.random.randn(5, 2))
    print(df.pct_change())
    ```

Gives us:

```text
0         NaN
1    1.000000
2    0.500000
3    0.333333
4    0.250000
5   -0.200000
dtype: float64
-----
          0          1
0       NaN        NaN
1 -1.156977 -16.169034
2  0.234270  -0.647137
3 -3.203838  -1.043420
4 -1.548769  -9.686350
```

The first row is _NaN_ because there is no previous element to compare it to.

1.  **Covariance** - the _Series_ object has a method **cov** to compute covariance between two objects.

```python
import pandas as pd
import numpy as np
s1 = pd.Series(np.random.randn(10))
s2 = pd.Series(np.random.randn(10))
print(s1.cov(s2))
```

Output:

```text
-0.23077206068332465
```

_NaN_ values are ignored automatically.

1.  **Correlation** - the natural follow up of course.

```python
import pandas as pd
import numpy as np
frame = pd.DataFrame(np.random.randn(10, 5), columns=['a', 'b', 'c', 'd', 'e'])

print( frame['a'].corr(frame['b']))
print("-------")
print( frame.corr())
```

The latter table shows in each cell shows what is the correlation between the respective columns.

```text
0.584632830437208
-------
          a         b         c         d         e
a  1.000000  0.584633  0.178898 -0.199928 -0.073022
b  0.584633  1.000000  0.045907  0.111284  0.006680
c  0.178898  0.045907  1.000000 -0.455127  0.065576
d -0.199928  0.111284 -0.455127  1.000000 -0.596656
e -0.073022  0.006680  0.065576 -0.596656  1.000000
```


### Window statistics {#window-statistics}

Some of the supported operations are:

1.  Rolling - Imagine a window that rolls over the data and computes 'something' with the elements that are currently in the window. Usueall the restult is written then in the first element of the wondow. The function in use here is **rolling(windows=/window\_size/)**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(
    np.random.randn(10, 4),
    index = pd.date_range('1/1/2000', periods=10),
    columns = ['A', 'B', 'C', 'D']
)
print(df.rolling(window=3).mean())
```

```text
                   A         B         C         D
2000-01-01       NaN       NaN       NaN       NaN
2000-01-02       NaN       NaN       NaN       NaN
2000-01-03  0.448896 -0.457281 -0.320112 -0.629398
2000-01-04  0.142268 -0.523835 -0.627615  0.228978
2000-01-05 -0.200259 -1.242848 -0.683303 -0.575579
2000-01-06 -0.867429 -0.211049 -0.207560  0.048380
2000-01-07 -1.117081 -0.268690  0.053126 -0.376906
2000-01-08 -1.117306  0.142617 -0.295222 -0.505522
2000-01-09 -0.324298 -0.065450 -0.333431 -1.186115
2000-01-10 -0.186834 -0.131418 -0.530804 -0.624335
```

The window is big 3-elements and therefore the first two rows don't have the needed neighbors.

1.  Expanding - Calculate something for the first element, then for the first and second together, then for the first, second and third together and so forth. This is done with **expanding(min\_periods=n)**. _min\_periods_ shows when the computations begin (the number of rows needed in order the generated row not to be _NaN_).

    ```python
    import pandas as pd
    import numpy as np

    df = pd.DataFrame(np.random.randn(10, 4),
                      index = pd.date_range('1/1/2000', periods=10),
                      columns = ['A', 'B', 'C', 'D'])
    print(df.expanding(min_periods=3).mean())
    ```

```text
                   A         B         C         D
2000-01-01       NaN       NaN       NaN       NaN
2000-01-02       NaN       NaN       NaN       NaN
2000-01-03  1.330880 -0.758586  0.172522  0.657721
2000-01-04  1.161148 -0.147130 -0.110188  0.473314
2000-01-05  0.565535  0.120415 -0.038158  0.647995
2000-01-06  0.721102  0.268228 -0.037819  0.563833
2000-01-07  0.734176  0.232185 -0.090012  0.420967
2000-01-08  0.681839  0.240921 -0.204159  0.223472
2000-01-09  0.772570  0.121897  0.053970  0.105342
2000-01-10  0.837397  0.266325  0.109070  0.280632
```

1.  Exponentially moving weights - Not sure for the exact mathematics of this one, but...it averages the data in some weird way. It's used to get the 'general idea' for the behaiviour of the data

    ```python
    import pandas as pd
    import numpy as np

    df = pd.DataFrame(np.random.randn(10, 4),
                      index = pd.date_range('1/1/2000', periods=10),
                      columns = ['A', 'B', 'C', 'D'])
    print( df.ewm(com=0.5).mean())
    ```


### Grouping and Aggregating {#grouping-and-aggregating}

In many situations the following 'pipeline' occurs:

1.  Split and object by grouping it entries by some key
2.  Perform some operations to get a single number for each group
3.  Combine the result into a new object

The second step could have some variations. Maybe we can want to transform or filter the data but the general idea stays. Pandas offers some great functions to achieve all of this. <br /> <br /> Firstly, in order to create the groups we can use the **groupby()** function. It can take the name of single column or multiple ones. In the latter case, the appropriate combinations between the keys of the columns are generated. Each combinations is it's own group. Once the grouping object is created, the groups can be examined with `gr.groups`. The groups can also be easly iterated over. Selecting a group is also easy by specifying its key. <br /> <br /> Summarizing example:

```python
import pandas as pd
ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)
gr = df.groupby(['Team','Year'])

print(gr.groups)
print("------")
for name,group in gr:
    print(name)
    print(group)
print("-----")
print(gr.get_group(('Riders', 2014)))

```

```text
{('Devils', 2014): Int64Index([2], dtype='int64'), ('Devils', 2015): Int64Index([3], dtype='int64'), ('Kings', 2014): Int64Index([4], dtype='int64'), ('Kings', 2016): Int64Index([6], dtype='int64'), ('Kings', 2017): Int64Index([7], dtype='int64'), ('Riders', 2014): Int64Index([0], dtype='int64'), ('Riders', 2015): Int64Index([1], dtype='int64'), ('Riders', 2016): Int64Index([8], dtype='int64'), ('Riders', 2017): Int64Index([11], dtype='int64'), ('Royals', 2014): Int64Index([9], dtype='int64'), ('Royals', 2015): Int64Index([10], dtype='int64'), ('kings', 2015): Int64Index([5], dtype='int64')}
------
('Devils', 2014)
   Points  Rank    Team  Year
2     863     2  Devils  2014
('Devils', 2015)
   Points  Rank    Team  Year
3     673     3  Devils  2015
('Kings', 2014)
   Points  Rank   Team  Year
4     741     3  Kings  2014
('Kings', 2016)
   Points  Rank   Team  Year
6     756     1  Kings  2016
('Kings', 2017)
   Points  Rank   Team  Year
7     788     1  Kings  2017
('Riders', 2014)
   Points  Rank    Team  Year
0     876     1  Riders  2014
('Riders', 2015)
   Points  Rank    Team  Year
1     789     2  Riders  2015
('Riders', 2016)
   Points  Rank    Team  Year
8     694     2  Riders  2016
('Riders', 2017)
    Points  Rank    Team  Year
11     690     2  Riders  2017
('Royals', 2014)
   Points  Rank    Team  Year
9     701     4  Royals  2014
('Royals', 2015)
    Points  Rank    Team  Year
10     804     1  Royals  2015
('kings', 2015)
   Points  Rank   Team  Year
5     812     4  kings  2015
-----
   Points  Rank    Team  Year
0     876     1  Riders  2014
```

Now comes the fun part. The **agg()** function returns a single aggregated value for each group. It takes a function on its own that does the actual work. Many of the _numpy_ functions are supported. Multiple aggregations per **agg()** call are also possible. To note is that **agg()** is usually applied to single column

```python
import pandas as pd
import numpy as np

ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
                     'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
            'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
            'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
            'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

grouped = df.groupby('Year')

print(grouped['Points'].agg(np.mean))
print("----")
print(grouped['Points'].agg([np.sum, np.mean, np.std]))
```

```text
Year
2014    795.25
2015    769.50
2016    725.00
2017    739.00
Name: Points, dtype: float64
----
       sum    mean        std
Year
2014  3181  795.25  87.439026
2015  3078  769.50  65.035888
2016  1450  725.00  43.840620
2017  1478  739.00  69.296465
```

Groups can be filtered with the **filter()** function.

```python
import pandas as pd
import numpy as np
ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)
print(df.groupby('Team').filter(lambda x: len(x) >= 3))
```

```text
    Points  Rank    Team  Year
0      876     1  Riders  2014
1      789     2  Riders  2015
4      741     3   Kings  2014
6      756     1   Kings  2016
7      788     1   Kings  2017
8      694     2  Riders  2016
11     690     2  Riders  2017
```


## Concatenating {#concatenating}

When working with a little bin more complex data like in _pandas_ the concatenation can become a tricky thing. _pandas_ offers a special function:

```text
pd.concat(objs,axis=0,join='outer',join_axes=None,ignore_index=False)
```

The _axis_ parameter controls how the concatenation is done - either by columns or by rows (row default).

```python
import pandas as pd
one = pd.DataFrame({
    'Name': ['Alex', 'Amy', 'Allen', 'Alice', 'Ayoung'],
    'subject_id':['sub1','sub2','sub4','sub6','sub5'],
    'Marks_scored':[98,90,87,69,78]},
                   index=[1,2,3,4,5])
two = pd.DataFrame({
    'Name': ['Billy', 'Brian', 'Bran', 'Bryce', 'Betty'],
    'subject_id':['sub2','sub4','sub3','sub6','sub5'],
    'Marks_scored':[89,80,79,97,88]},
                   index=[1,2,3,4,5])
print(pd.concat([one,two]))
print("-------")
print(pd.concat([one,two],axis=1))
```

-   by row

    ```text
      Marks_scored    Name subject_id
    1            98    Alex       sub1
    2            90     Amy       sub2
    3            87   Allen       sub4
    4            69   Alice       sub6
    5            78  Ayoung       sub5
    1            89   Billy       sub2
    2            80   Brian       sub4
    3            79    Bran       sub3
    4            97   Bryce       sub6
    5            88   Betty       sub5

    ```

-   by column

    ```text
      Marks_scored    Name subject_id  Marks_scored   Name subject_id
    1            98    Alex       sub1            89  Billy       sub2
    2            90     Amy       sub2            80  Brian       sub4
    3            87   Allen       sub4            79   Bran       sub3
    4            69   Alice       sub6            97  Bryce       sub6
    5            78  Ayoung       sub5            88  Betty       sub5
    ```

By concatenation new labels can be assigned to the different parts:

```python
import pandas as pd
one = pd.DataFrame({
    'Name': ['Alex', 'Amy', 'Allen', 'Alice', 'Ayoung'],
    'subject_id':['sub1','sub2','sub4','sub6','sub5'],
    'Marks_scored':[98,90,87,69,78]},
                   index=[1,2,3,4,5])
two = pd.DataFrame({
    'Name': ['Billy', 'Brian', 'Bran', 'Bryce', 'Betty'],
    'subject_id':['sub2','sub4','sub3','sub6','sub5'],
    'Marks_scored':[89,80,79,97,88]},
                   index=[1,2,3,4,5])
print(pd.concat([one,two],keys=['x','y'],ignore_index=False))
```

```text
     Marks_scored    Name subject_id
x 1            98    Alex       sub1
  2            90     Amy       sub2
  3            87   Allen       sub4
  4            69   Alice       sub6
  5            78  Ayoung       sub5
y 1            89   Billy       sub2
  2            80   Brian       sub4
  3            79    Bran       sub3
  4            97   Bryce       sub6
  5            88   Betty       sub5
```

Later those new keys can be used in order to distinguish from which set the row came from. <br /> <br /> Appending is also possible and it takes the simple form:

```python
one.append(two)
```


## Categories {#categories}

A lot of times some string-fields in the data aren't just some random text but a repetitive and an element of some predefined set of possible values. Those are the categorical types of data. Something like [big, medium, small]. Categorical variables can take on only a limited, and usually fixed number of possible values. Besides the fixed length, categorical data might have an order but cannot perform the numerical operation. Categorical is a _Pandas_ data type. <br /> <br /> A simple example to create a _Series_ object that only can contain [a, b, c]

```python
import pandas as pd
s = pd.Series(["a","b","c","a"], dtype="category")
print(s)
```

This gives us:

```text
0    a
1    b
2    c
3    a
dtype: category
Categories (3, object): [a, b, c]
```

There is also a _Categorical_ object in _pandas_ specifically for dealing with categorical data. The general constructor is as follows:

```text
pandas.Categorical(values, categories, ordered)
```

If _categories_ aren't provided, they are inferred from the values. _ordered_ specifies whether or not on categories is bigger or not than other.

```python
import pandas as pd
cat = cat=pd.Categorical(
    ['a','b','c','a','b','c','d'],
    ['c', 'b', 'a'],
    ordered=True)
print(cat)
```

```text
[a, b, c, a, b, c, NaN]
Categories (3, object): [c < b < a]
```

<br /> <br /> **obj.cat.categories** command is used to get the categories of the object. <br /> <br /> Removing categories is also something that comes in handy and of course it's possible with _pandas_

```python
import pandas as pd

s = pd.Series(["a","b","c","a"], dtype="category")
print("Original object:")
print(s)

print("After removal:")
print( s.cat.remove_categories("a"))
```

```text
Original object:
0    a
1    b
2    c
3    a
dtype: category
Categories (3, object): [a, b, c]
After removal:
0    NaN
1      b
2      c
3    NaN
dtype: category
Categories (2, object): [b, c]
```


## Reading Data from _.csv_-files {#reading-data-from-dot-csv-files}

At the beginning probably each applications loads some date from the file system or link or whatever. _pandas_ provides **IO API** for reading data from _.cvs_-files. The two main functions for reading text files are **read\_csv()** and **read\_table()**. They use similar procedures to intelligently convert tabular data into a _DataFrame_ object. The general form of the functions:

```text
pandas.read_csv(filepath_or_buffer, sep=',', delimiter=None, header='infer',
names=None, index_col=None, usecols=None
---------
pandas.read_csv(filepath_or_buffer, sep='\t', delimiter=None, header='infer',
names=None, index_col=None, usecols=None
```

So say our **temp.cvs** looks like this:

```text
S.No,Name,Age,City,Salary
1,Tom,28,Toronto,20000
2,Lee,32,HongKong,3000
3,Steven,43,Bay Area,8300
4,Ram,38,Hyderabad,3900
```

and represents this:

| S.No | Name   | Age | City      | Salary |
|------|--------|-----|-----------|--------|
| 1    | Tom    | 28  | Toronto   | 20000  |
| 2    | Lee    | 32  | HongKong  | 3000   |
| 3    | Steven | 43  | Bay Area  | 8300   |
| 4    | Ram    | 38  | Hyderabad | 3900   |

We can read this as:

```python
import pandas as pd
df=pd.read_csv("temp.csv")
print(df)
```

A lot of the data in _.cvs_ files has a special column that specifies the index of the row. Pandas can take this into consideration:

```python
import pandas as pd

df=pd.read_csv("temp.csv",index_col=['S.No'])
print(df)
```

Skipping rows can be achieved through the _skiprows=n_ argument of **read\_cvs()**

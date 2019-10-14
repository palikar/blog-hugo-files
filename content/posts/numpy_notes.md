+++
title = "Notes on learning Numpy"
author = ["Stanislav Arnaudov"]
description = "My notes on numpy when I started looking into the library"
date = 2018-07-20T00:00:00+02:00
keywords = ["machine-learning", "python", "numpy"]
lastmod = 2019-10-12T10:47:34+02:00
categories = ["machine-learning"]
draft = false
weight = 100
+++

<div class="NOTES">
  <div></div>

My notes while reading [this](https://www.tutorialspoint.com/numpy/index.htm). More or less copy paste. Take with a grain of salt...or sugar.

</div>


## Abstract {#abstract}

NumPy is a Python package. It stands for 'Numerical Python'. It is a library consisting of multidimensional array classes and a collection of routines for processing of an array. <br /> From the officials:

> NumPy is the fundamental package for scientific computing with Python. It contains among other things:
>
> -   a powerful N-dimensional array object
> -   sophisticated (broadcasting) functions
> -   tools for integrating C/C++ and Fortran code
> -   useful linear algebra, Fourier transform, and random number capabilities
>
> Besides its obvious scientific uses, NumPy can also be used as an efficient multi-dimensional container of generic data. Arbitrary data-types can be defined. This allows NumPy to seamlessly and speedily integrate with a wide variety of databases.

In layman's terms (according to me) - better python-arrays with native C-code under the hood so that everything is blazingly fast. Easy array generation, generation of ranges, intervals and distributions of random numbers in some interval. Conveniently iterating over ranges and the generated arrays. _Numpy_ also provides some basic/advanced statistical functions. Linear Algebra is also something that is already built-in.

Installation:

```text
pip3 install numpy
```


## Ndarray {#ndarray}

The most important object defined in _NumPy_ is an N-dimensional array type called _ndarray_. This is the main workhorse of the package. It describes the collection of items of the same type. Items in the collection can be accessed using a zero-based index. It's much better than the standard python arrays and lists. Basic construction:

```python
numpy.array(object, dtype = None, copy = True, order = None, subok = False, ndmin = 0)
```

-   `object` - Any object exposing the array interface method returns an array, or any (nested) sequence.
-   `dtype` - the desired data type of array, optional
-   `copy` - Optional. By default (true), the object is copied
-   `order` - C (row-major) or F (column-major) or A (any) (default)
-   `subok` - By default, the returned array forced to be a base class array. If true, sub-classes passed through
-   `ndmin` - Specifies minimum dimensions of the resultant array

```python
import numpy as np
a = np.array([1, 2,3,4])
print(a)

a = np.array([[1, 2], [3, 4]])
print(a)

a = np.array([1, 2,3,4], dtype=complex)
print(a)
```

```text
[1 2 3 4]
[[1 2]
 [3 4]]
[1.+0.j 2.+0.j 3.+0.j 4.+0.j]
```


## Data Types {#data-types}

There are a lot of possible data types that are supported by _numpy_. Some of the scalar types include:

-   bool\_, int\_, intc, intp, int8, int16, int32, uint8, uint16, uint32
-   float\_, float16, float32, flaot64
-   complex, complex64, complex128

<br /> <br /> The types can be encapsulated in _Data Type Object_.

```python
numpy.dtype(object, align, copy)
```

-   **Object** − To be converted to data type object
-   **Align** − If true, adds padding to the field to make it similar to C-struct
-   **Copy** − Makes a new copy of dtype object. If false, the result is a reference to builtin data type object

<br /> <br /> The object can be later used when a given function or a constructor takes _dtype_ argument. Those objects can also create struct-like types for structured data. Think of it like defining a custom struct in C++ and then creating a vector with elements of this struct.

```python
import numpy as np

dt = np.dtype([('age',np.int8),('time', np.int32)])
a = np.array([(10,30),(20,30),(30,30)], dtype = dt)

print(a)
print(a['age'])
print(a['time'])

```

Which yields:

```text
[(10, 30) (20, 30) (30, 30)]
[10 20 30]
[30 30 30]
```

<br /> <br /> This way we can define a map-like structure that contains a bunch of different arrays, each addressable with a key. The following code defines a 'human' data type with _string_ field _name_, _int_ field age, and _float_ field social class.

```python
import numpy as np
human = np.dtype([('name', 'S20'), ('age', 'i1'), ('social_class', 'f4')])
print(human)

a = np.array([('Lenin', 45, 100),('John', 19, 3.5)], dtype = human)
print(a)
```

```text
[('name', 'S20'), ('age', 'i1'), ('social_class', '<f4')]
[(b'Lenin', 45, 100. ) (b'John', 19,   3.5)]
```

This shows another important point. Each time can be specified with a single character.

-   `'b'` − boolean
-   `'i'` − (signed) integer
-   `'u'` − unsigned integer
-   `'f'` − floating-point
-   `'c'` − complex-floating point
-   `'m'` − time delta
-   `'M'` − DateTime
-   `'O'` − (Python) objects
-   `'S'`, 'a' − (byte-)string
-   `'U'` − Unicode
-   `'V'` − raw data (void)


## Array Attributes {#array-attributes}

Attributes, as the name implies, give us some information about a given object. In the case of _numpy_, the most important thing about an array is its _shape_ and _ndim_


### Shape and dimension {#shape-and-dimension}

_Shape_ of an array is its layout in memory. Normally _numpy_ represents the ndarray as a n-dimensional array in memory. This means that that the shapes is nothing more than a tuple that consists of each dimension's size. For example, the array `[1,2,3]` has shape `(3)`, the array `[[1,2],[1,2]]` has shape `(2,2)` because it consists of two array and each of them contains two elements.

```python
import numpy as np

a = np.array([1,2,3,4])
print(a.shape)

a = np.array([[1,2,3,4],[1,2,3,4]])
print(a.shape)

```

```text
(4,)
(2, 4)
```

<br /> <br /> _Ndim_ of an _numpy_ array on the other hand is the number of dimensions. I. e. `[1,2]` has _ndim_ equal to 1, and `[[1,3],[1.3]]` has _ndim_ 2

```python
import numpy as np
import numpy as np
a = np.arange(24)
print(a.ndim)

a = np.array([[1,2,3,4],[1,2,3,4]])
print(a.ndim)
```

```text
1
2
```


### Reshaping {#reshaping}

A lot of times we want to change the basic structure of and array without changing the information in it. In such cases, the _reshape_ method comes to help.

```python
import numpy as np
a = np.arange(24)
print(a.ndim)

# now reshape it
b = a.reshape(2,4,3)
print(b)

```

The elements stay the same and occupy the same memory location but the access to them is happening in different way.\*\* 'Size' of an array `numpy.itemsize` - this array attribute returns the length of each element of array in bytes.

```python
import numpy as np
x = np.array([1,2,3,4,5], dtype = np.int8)
print(x.itemsize)
```

```text
1
```


### Num elements {#num-elements}

The attribute that everyone has been waiting for... `numpy.size`

```python
import numpy as np

a = np.arange(24)
print(a.size)
```

```text
24
```


## Slicing {#slicing}


### Basic slicing {#basic-slicing}

Basic slicing is an extension of Python's basic concept of slicing to _n_ dimensions. A Python slice object is constructed by giving start, stop, and step parameters to the built-in slice function. This slice object is passed to the array to extract a part of array.

```python
import numpy as np
a = np.arange(10)
b = a[2:7:2]
print(b)
print(a[2:])
```

```text
[2 4 6]
[2 3 4 5 6 7 8 9]
```


### Advanced {#advanced}

This mechanism helps in selecting any arbitrary item in an array based on its _Ndimensional_ index. Each integer array represents the number of indexes into that dimension. When the index consists of as many integer arrays as the dimensions of the target ndarray, it becomes straightforward.

```python
import numpy as np

x = np.array([[ 0,  1,  2],[ 3,  4,  5],[ 6,  7,  8],[ 9, 10, 11]])

print('Our array is:')
print(x)
print('\n')

rows = np.array([[0,0],[3,3]])
cols = np.array([[0,2],[0,2]])
y = x[rows,cols]

print('The corner elements of this array are:' )
print(y)
```

```text
Our array is:
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 9 10 11]]
```

The corner elements of this array are:

```text
[[ 0  2]
 [ 9 11]]
```


#### Boolean Array Indexing {#boolean-array-indexing}

This type of advanced indexing is used when the resultant object is meant to be the result of Boolean operations, such as comparison operators.

```python
import numpy as np

x = np.arange(0,20)
print(x[x>5])
```

```text
[ 6  7  8  9 10 11 12 13 14 15 16 17 18 19]
```


## Broadcasting {#broadcasting}

This refers to the way arithmetic operations are handled. _Numpy_ is intelligent enough to perform arithmetical operations on arrays element-wise as long as the shapes of the objects are 'compatible'.

```python
import numpy as np

a = np.array([1,2,3,4])
b = np.array([10,20,30,40])
c = a * b
print(c)
```

```text
[ 10  40  90 160]
```


## Iterating {#iterating}

`numpy.nditer()` is your best friend. It returns an iterable (for loop) object that can be further used to go through an array. It can also be constructed with two arrays to go through the both of them simultaneously. The array must be _broadcastable_. Constructing an iterator that can modify the values of an array happens through a flag in the constructor.

```python
import numpy as np

a = np.arange(0, 60, 5)
b = np.arange(0, 12)

for num in np.nditer(a):
    print(str(num))
print('\n')

for a_num,b_num in np.nditer([a,b]):
    print(str(a_num) + '+' + str(b_num))
print('\n')

for num in np.nditer(a, op_flags=["readwrite"]):
    num += 10
print(a)
```

```text
0
5
10
15
20
25
30
35
40
45
50
55


0+0
5+1
10+2
15+3
20+4
25+5
30+6
35+7
40+8
45+9
50+10
55+11


[10 15 20 25 30 35 40 45 50 55 60 65]
```


## Manipulating {#manipulating}


### Changing shape {#changing-shape}

| Method    | Description                          |
|-----------|--------------------------------------|
| `reshape` | Change shape                         |
| `flat`    | 1D Iterator                          |
| `flatten` | Returns a new array                  |
| `revel`   | Returns a contiguous flattened array |


### Transpose {#transpose}

| Method      | Description                           |
|-------------|---------------------------------------|
| `transpose` | Permutes the dimensions of an array   |
| `ndarray.T` | Same as self.transpose()              |
| `rollaxis`  | Rolls the specified axis backwards    |
| `swapaxes`  | Interchanges the two axes of an array |


### Joining Arrays {#joining-arrays}

| Method        | Desc                                        |
|---------------|---------------------------------------------|
| `concatenate` | Joins several arrays along an existing axes |
| `stack`       | Joins several arrays along a new axes       |
| `hstack`      | Stacks arrays in sequence horizontally      |
| `vstack`      | Stacks arrays in sequence vertically        |


### Adding and removing elements {#adding-and-removing-elements}

| Method | Description                                              |
|--------|----------------------------------------------------------|
| append | Pushes new value at the end                              |
| insert | Inserts value along a given axis before a given index    |
| delete | Return a new array with sub-arrays along an axis deleted |
| unique | Finds all unique elements in an array                    |


## I/O {#i-o}

There are two flavors of saving/loading a _ndarray_ to/from file.


### _.npy_ files {#dot-npy-files}

This _.npy_ file stores data, shape, _dtype_ and other information required to reconstruct the _ndarray_ in a disk file such that the array is correctly retrieved even if the file is on another machine with different architecture. <br /> <br /> The _IO_ is simple and is done through the functions `numpy.save()` and `numpy.load()`

```python
import numpy as np
a = np.array([1,2,3,4,5])
np.save('outfile',a)
b = np.load('outfile.npy')
```


### _.txt_ files {#dot-txt-files}

This is just a simple, cheap and dirty way to save an array to a file "symbolically". The function that come into play are `numpy.savetxt()` and `numpy.loadtxt()`

```python
import numpy as np

a = np.array([1,2,3,4,5])
np.savetxt('out.txt',a)
b = np.loadtxt('out.txt')
print(b)
```

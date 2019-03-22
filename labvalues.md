
# Pandas


```python
from nose.tools import assert_true, assert_false, \
    assert_almost_equal, assert_equal, assert_raises
```


```python
import os
import pandas as pd
import matplotlib.pyplot as plt
from collections import defaultdict, OrderedDict
import datetime
import numpy as np
from dateutil import parser
```

**Problem 1 (10 points):** Write functions`get_date` and `get_time` that each takes a single positional argument and uses the parser defined in the dateutil package to convert the argument to a `datetime.date` (`get_date`) and a `datetime.time` object (`get_time`). If the argument cannot be converted to a time object, the functions should return a Pandas NaT (not a time) object.


**Hint:** Use a try/except code block.


```python
a=parser.parse("9/13/2014")

b=parser.parse("26 October 2017")
c=parser.parse("Sept 13 1919")
```


```python
#c.date?   ask what an attribute does

```


```python
# YOUR CODE HERE
def get_date(x):
    try:
        return parser.parse(x).date()   
    except:
        return pd.NaT

def get_time(x):
    try:
        return parser.parse(x).time()
    except:
        return pd.NaT

#raise NotImplementedError()
```


```python
assert_true(pd.isnull(get_time("Hello")))
```


```python
assert_equal(get_date("5/17/2007"), datetime.date(2007, 5, 17))
```


```python
assert_true(pd.isnull(get_date("Hello")))
```

**Problem 2 (15 points):** Write a function `get_range` that takes a positional argument containing a string from which to extract a range and a keyword argument named `delimiter` that has a string indicating the delimiter between the low and high values of the range. The function should **try** to return a tuple of two floating point values (the low range value and the high range value). 

The function should return a two-tuple `np.nan` values  (`(np.nan, np.nan)`) for the following conditions:

1. The string does not contain exactly one instance of `delimiter`.
1. The attempt to extract two floating point values fails.   

If either the positional or keyword arguments are not strings, return a `TypeError`.

#### Examples
* "80:120" should return (80.0,120.0) using ":" as a delimiter.
* "0-1" Should return (0.0,1.0) using "-" as a delimiter


```python
# YOUR CODE HERE
def get_range(value,delim="-"):
    if (type(value)!=str) or (type(delim) != str):
        raise TypeError
    else:
        try: 
            array = value.split(delim)
            if len(array)==2:
                return (float(array[0]),float(array[1]))
            else:
                return (np.nan,np.nan)
        except:
            return (np.nan,np.nan)


#raise NotImplementedError()
```


```python
assert_raises(TypeError, get_range, 5.4)
```


```python
assert_raises(TypeError, get_range, 5.4, delimiter=(5,4,3))
```


```python
assert_true(np.isnan(get_range("12-24-1972")).all())
```


```python
assert_equal(get_range("15-20"), (15,20))
```

**Problem 3 (10 points):** Write a function `is_number` that takes a single positional argument and returns `True` if that argument can be converted to a `float` and `False` if it cannot. Use try/except


```python
# YOUR CODE HERE

def is_number(x):
    try:
        xfloat=float(x)
        return True
    except:
        return False


#raise NotImplementedError()

```


```python
assert_true(is_number("5.3e4"))
```


```python
assert_false(is_number(">5000"))
```

## Problem 4 (5 points)

Write a function `get_value` that takes a single positional argument (e.g. `x`) and returns `x` converted to a float. If the conversion fails, return `np.nan`.


```python
# YOUR CODE HERE
def get_value(x):
    try:
        return (float(x))
    except:
        return (np.nan)


#raise NotImplementedError()
```


```python
assert_true(np.isnan(get_value("[5.4]")))
```


```python
assert_equal(get_value("5.7"), 5.7)
```

### For the following problems we will use the `sampllLabs.txt` file
#### In this directory is a file (`samplelabs.txt`) with a set of lab values. The file includes lab values obtained on different individuals during their in-patient hospitalizations. 

## Problem 5. (35 Points):

Use Pandas to write a function named `get_labs` that does the following:

* Takes the name of the file to read as a positional argument
* Takes as a keyword argument named `converters` a dictionary with keys equal to the test component names (columns in DataFrame) and as values, functions that convert the expected column value to the appropriate type. You should use the ordered dictionary (`converters`) I define below.
* Reads the contents of the input file into a **default dictionary** keyed by **visitid**. The value for each **visitid** should be a **list** of tests. For each row, use `converters` to convert the row value from a string to the appropriate data type (eg. float, datetime.date). Each test result result should be stored as a tuple of 2-tuples with the first element being the explanatory metadata (i.e, the column name) and the second element being the converted value.

#### Hints

1. Use the DataFrame `iterrows` method to iterate over each row in the DataFrame.
1. Use the function `get_test_values` to convert the data types of each row.
1. Use slicing on the row to grab the test values (`visitid` is going to be the key for the dictionary and should not be part of the test result tuple.
1. A row is a Pandas Series and we can access the elements of the row using either an index or the column name:

```Python
print(row)
print()
print(row[2])
print(row["ctime"])

visitid    OMHioJh8XEeq7152
cdate             6/13/2007
ctime                 06:30
pqno       1181750718122403
test                  CREAT
result                  1.0
unit                  mg/dL
range               0.5-1.4
Name: 5, dtype: object

06:30
06:30
```

Your resulting output should look something like this:

```Python
defaultdict(list,
            {'+yhZLyY5Uqra5115': [[('cdate', datetime.date(2005, 4, 29)),
               ('ctime', datetime.time(6, 30)),
               ('pqno', 1114780124780442),
               ('test', 'CREAT'),
               ('result', 3.4),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))],
              [('cdate', datetime.date(2005, 4, 28)),
               ('ctime', datetime.time(6, 45)),
               ('pqno', 1114692308737109),
               ('test', 'CREAT'),
               ('result', 8.6),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))],
              [('cdate', datetime.date(2005, 5, 2)),
               ('ctime', datetime.time(4, 0)),
               ('pqno', 1115041801503288),
               ('test', 'CREAT'),
               ('result', 1.6),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))],
              [('cdate', datetime.date(2005, 4, 30)),
               ('ctime', datetime.time(4, 0)),
               ('pqno', 1114875733484671),
               ('test', 'CREAT'),
               ('result', 1.9),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))],
              [('cdate', datetime.date(2005, 4, 26)),
               ('ctime', datetime.time(16, 5)),
               ('pqno', 1114551787814371),
               ('test', 'CREAT'),
               ('result', 16.0),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))],
              [('cdate', datetime.date(2005, 5, 1)),
               ('ctime', datetime.time(4, 0)),
               ('pqno', 1114957388877371),
               ('test', 'CREAT'),
               ('result', 1.7),
               ('unit', 'mg/dL'),
               ('range', (0.5, 1.4))]],}
```


```python
from collections import OrderedDict
converters = OrderedDict((('cdate', get_date), 
                          ('ctime', get_time), 
                          ('pqno', lambda x:x), 
                          ('test', lambda x:x), 
                          ('result', get_value), 
                          ('unit', lambda x:x), 
                          ('range', get_range)))
```


```python
#s="2017-01-01"
#resultdate=converters["cdate"](s)

#print(s)
#print(type(resultdate))

#file.iloc() = .iloc() identifies the location or column of interest
```


```python
import datetime
import os
import pandas as pd
from collections import defaultdict

        
def get_test_values(row, converters):
    
#index is cdate/get_date and convert is ctime/get_time  
    result=[]
    for index, convert in converters.items():
        result.append((index,convert(row[index])))
    return tuple(result)

def get_labs(fname, converters):
    
# YOUR CODE HERE
    result=defaultdict(list)
    file=pd.read_table(fname)
    for index, row in file.iterrows():
        result[row["visitid"]].append(get_test_values(row[1:], converters))
    return result


```


```python
srtd_labs = get_labs("samplelabs.txt", converters)

assert_equal(srtd_labs['X4HZ5/mbdrls7055'][8][1][1], datetime.time(4, 4))
```


```python
assert_almost_equal(srtd_labs["3WobBYVBHdIa6106"][5][6][1],(0.5,1.4))
```


```python
assert_true('QLE6UIMptiKf5228' in srtd_labs)
```


```python
assert_equal(len(srtd_labs['n9b8R+Aynzy+5257']),  23)
```


```python
assert_equal(srtd_labs['yyutduwkJpYJ7131'][1][1], 
             ('ctime', datetime.time(6, 15)))
```


```python
assert_true(type(srtd_labs['yyutduwkJpYJ7131'][1]),tuple)
```


```python
assert_true(np.isnan(srtd_labs['deaOcpSo0N617039'][14][4][1]))
```

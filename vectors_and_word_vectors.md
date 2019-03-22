
# Vectors and Word Vectors




```python
from nose.tools import assert_almost_equal, assert_true, assert_equal, assert_raises
from numbers import Number
```


```python

import numpy as np
import locale
import os
import re
import pandas as pd
from textblob import TextBlob
```


```python
from math import sqrt
```

## **Problem 1 (20 points).** Compute the Euclidean norm for each of the following vectors
$$v_1= \begin{bmatrix}5\\-1\\1\\0\\-5\end{bmatrix}$$


```python
norm_v1 = None
v1 = np.array([[5],[-1],[1],[0],[-5]])
v1t = np.transpose(v1)
norm_v1 = np.linalg.norm(v1)


print(norm_v1)

# YOUR CODE HERE
#raise NotImplementedError()
```

    7.211102550927978


$$v_2= \begin{bmatrix}2\\ 3\\ 9\\ 4\end{bmatrix}$$


```python
norm_v2 = None
v2 = np.array([[2],[3],[9],[4]])
v2t = np.transpose(v2)
norm_v2 = np.linalg.norm(v2)

print(norm_v2)

# YOUR CODE HERE
#raise NotImplementedError()
```

    10.488088481701515


$$v_3=\begin{bmatrix}-2, -3, -7, -5\end{bmatrix}^T$$


```python
norm_v3 = None
v3 = np.array([-2,-3,-7,-5])
v3t = np.transpose(v3)
norm_v3 = np.linalg.norm(v3t)

print(norm_v3)


# YOUR CODE HERE
#raise NotImplementedError()
```

    9.327379053088816


$$v_6= \begin{bmatrix}4\\  2\\ -8\end{bmatrix}$$


```python
norm_v4 = None
v4 = np.array([4,2,-8])
v4t = np.transpose(v4)
norm_v4 = np.linalg.norm(v4)

print(norm_v4)

# YOUR CODE HERE
#raise NotImplementedError()
```

    9.16515138991168


## Problem #2 (10 points)

One of the limitations of word vectors as we have pictured them is [sparsity](https://en.wikipedia.org/wiki/Sparse_array): while our vocabulary is large (tens or hundreds of thousands of words), a typical document (e.g. radiology report) only has tens or hundreds of unique words. Write a class (`sparsev`) that inherits from a `defaultdict` to represent a "sparse" vector. The keys would be the indicies to a vector and the values would be the word counts (how many times that word occurred in the document). The class should have an attribute `self.__dim` that indicates the dimension of the vector space (e.g. the dimension of the vocabulary). The class should have a property `dim` that returns the value in `__dim` the instance of `sparsev` represents. You should define the following methods for the class:

1. `norm`: Accepts as an argument a number `p` (default value=2) and computes the [p-norm](https://en.wikipedia.org/wiki/Norm_(mathematics)#p-norm) of the vector.
    1. If `p` is not a number, raise a `ValueError`.
    1. If $p \le 0$, raise a `ValueError`.
1. `cosine_sim`: Accepts as an argument an instance of a `sparsev`.
    1. If the two `sparsev` instances do not have the same `dim` raise a ValueError
    1. If you get a `ZeroDivisionError`, return `np.nan`
\begin{equation}
\text{similiarty}(\vec{A}, \vec{B}) = \frac{\vec{A}\cdot \vec{B}}{||\vec{A}|| ||\vec{B}||}
\end{equation}
1. a `__str__` method that shows the dimension of the vector as well as the elements (key/value pairs).


```python
from collections import defaultdict
class sparsev(defaultdict):
    def __init__(self, *args, dim=2, **kwargs):
        self.__dim = dim
        super(sparsev, self).__init__(*args, **kwargs)
                      
    @property
    def dim(self):
        return self.__dim
        
    def norm(self, p=2):
        val=[]
        for i in self.values():
            val.append(abs(i))
             
        if not type(p)==int or type(p)==float:
            raise ValueError()
       
        if  p<=0:
            raise ValueError()
            
        if  p == np.inf:
            return max(val)      
        
        else:                  
            results = []
            for i in val:
                results.append(i**p)
            return sum(results)**(1/p)
            
    def inner(self, v2):    
        if self.dim !=v2.dim:
            raise ValueError()
        keys = set(self.keys()).intersection(v2.keys())
        
        innerlist = []
        for i in keys:
            innerlist.append(self[i]*v2[i])
        return sum(innerlist)

    def cosine_sim(self,v2):
        try:
            v=self.inner(v2)/(self.norm()*v2.norm())
            if np.isnan(v):
                return 0
            else:
                return (v) 
        except ZeroDivisionError:
            return np.nan

    def __str__(self):
        return "dimention = %d %s"%(self.dim,super(sparsev, self).__str__())

    
    
# YOUR CODE HERE
#raise NotImplementedError()
```


```python
tmp1 = sparsev(int, dim=5)
tmp2 = sparsev(int, dim=3)
assert_raises(ValueError, tmp1.inner, tmp2)
```


```python
tmp1 = sparsev(int, dim=10)
tmp2 = sparsev(int, dim=10)
tmp1["Brian"] = 3
tmp1["Wendy"] = 4
tmp2["Argos"] = 9
tmp2["Helios"] = 2
tmp1.inner(tmp2)
assert_almost_equal(tmp1.inner(tmp2), 0)
```


```python
tmp1 = sparsev(int, dim=10)
tmp2 = sparsev(int, dim=10)
tmp1["Brian"] = 3
tmp1["Wendy"] = 4
tmp2["Argos"] = 9
tmp2["Brian"] = 2
tmp1.inner(tmp2)
assert_almost_equal(tmp1.inner(tmp2), 6)

```


```python
tmp1 = sparsev(int, dim=5)
tmp2 = sparsev(int, dim=3)
assert_true("5" in tmp1.__str__())
```


```python
tmp1 = sparsev(int, dim=5)
tmp2 = sparsev(int, dim=3)
assert_true("3" in tmp2.__str__())
```

## Test on MIMIC2 radiology reports


```python
import pymysql
import pandas as pd
import getpass
from textblob import TextBlob
```


```python
conn = pymysql.connect(host="mysql",
                       port=3306,user="jovyan",
                       passwd="jovyan",db='mimic2')
cursor = conn.cursor()
```

### Get some documents. Limit the query to keep corpus small for debugging


```python
rad_data = \
pd.read_sql("""SELECT DISTINCT noteevents.subject_id, 
                      noteevents.hadm_id,
                      noteevents.text 
               FROM noteevents
               WHERE noteevents.category = 'RADIOLOGY_REPORT' 
               LIMIT 5000""",conn)
rad_data.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>subject_id</th>
      <th>hadm_id</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>56</td>
      <td>28766.0</td>
      <td>\n\n\n     DATE: [**2644-1-17**] 10:53 AM\n   ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>56</td>
      <td>28766.0</td>
      <td>\n\n\n     DATE: [**2644-1-17**] 10:43 AM\n   ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>56</td>
      <td>28766.0</td>
      <td>\n\n\n     DATE: [**2644-1-17**] 6:37 AM\n    ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>56</td>
      <td>28766.0</td>
      <td>\n\n\n     DATE: [**2644-1-19**] 12:09 PM\n   ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>37</td>
      <td>18052.0</td>
      <td>\n\n\n     DATE: [**3264-8-14**] 6:06 AM\n    ...</td>
    </tr>
  </tbody>
</table>
</div>



## Create a vocabulary

We are first going to replace all digits in the reports with a "d" and convert all characters to lower case. This reduces our vocabulary size by approximately half. We will then use `TextBlob` and sets to get all the unique words in our document. This is our vocabulary. The vocabulary is represented as a dictionary which we create with the `zip` function.


```python
import nltk
nltk.download('punkt')
```

    [nltk_data] Downloading package punkt to /home/u6015361/nltk_data...
    [nltk_data]   Package punkt is already up-to-date!





    True




```python
reports = re.sub("\d", "d", " ".join([r.lower() for r in rad_data["text"]]))

words = set(TextBlob(reports).words)

vocabulary = dict(zip(words,range(len(words))))
print(len(vocabulary))
```

    12368



```python
#import nltk
#nltk.download('punkt')
```


```python
list(vocabulary.items())[:10]
```




    [('amio', 0),
     ('accounted', 1),
     ('perisplenic', 2),
     ('dddd-dd-dd', 3),
     ('cnocern', 4),
     ('commonly', 5),
     ('concurrent', 6),
     ('dumbbell-shaped', 7),
     ('tracheobronchialmalacia', 8),
     ('clavian', 9)]



## Problem 3 (20 points):

Write a function `doc2vec` that takes two arguments: 1) `txt` (a text to convert to a vector) and 2) `voc` (the vocabulary). It returns a `sparsev` instance that is the representation of `txt` in the `voc` vector space. Because `txt` may contain words that are not in the vocabulary, you will need to do exception handling.


```python

def doc2vec(txt, voc):
    blob = TextBlob(txt.lower())
    vec = sparsev(int, dim = len(voc))
    for word in blob.words:
        try:
            vec[voc[word]] += 1
            #vec[a] = vec[a]+1
        except KeyError:
            pass
    return vec
   


# YOUR CODE HERE
#raise NotImplementedError()
    
```


```python
v50 = doc2vec(re.sub("\d", "d", rad_data.loc[50,"text"]), vocabulary)
v157 = doc2vec(re.sub("\d", "d", rad_data.loc[157,"text"]), vocabulary)
assert_almost_equal(v50.norm(), 66.59579566308972)
```


```python
v50 = doc2vec(re.sub("\d", "d", rad_data.loc[50,"text"]), vocabulary)
v157 = doc2vec(re.sub("\d", "d", rad_data.loc[157,"text"]), vocabulary)
assert_almost_equal(v157.norm(), 95.40440241414439)
```

### Cosine similarity of a document with itself should be 1


```python
rad_data.loc[50,:]
```




    subject_id                                                   61
    hadm_id                                                    5712
    text          \n\n\n     DATE: [**3353-1-26**] 5:37 PM\n    ...
    Name: 50, dtype: object




```python
v50 = doc2vec(re.sub("\d", "d", rad_data.loc[50,"text"]), vocabulary)
v157 = doc2vec(re.sub("\d", "d", rad_data.loc[157,"text"]), vocabulary)
assert_almost_equal(v50.cosine_sim(v50), 1.0)
```


```python
v50 = doc2vec(re.sub("\d", "d", rad_data.loc[50,"text"]), vocabulary)
v157 = doc2vec(re.sub("\d", "d", rad_data.loc[157,"text"]), vocabulary)
assert_almost_equal(v50.cosine_sim(v157), 0.7364407600058933)
```

### Create a column in `rad_data` that has values equal to the similarity between the reports and `v50`.


```python
rad_data["50sim"] = rad_data.apply(lambda r: v50.cosine_sim(doc2vec(re.sub("\d", "d", r["text"]), vocabulary)), axis=1)
```

## Problem 4 (10 points):

Create a new DataFrame `rad_data2` that is the result of sorting `rad_data` by increasing values of `50sim`.


```python
rad_data2 = None

rad_data2 = rad_data.sort_values(by=['50sim'])

# YOUR CODE HERE

rad_data2.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>subject_id</th>
      <th>hadm_id</th>
      <th>text</th>
      <th>50sim</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4892</th>
      <td>2600</td>
      <td>21170.0</td>
      <td>\n\n\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4904</th>
      <td>2592</td>
      <td>23409.0</td>
      <td>\n\n\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4939</th>
      <td>2592</td>
      <td>828.0</td>
      <td>\n\n\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4946</th>
      <td>2638</td>
      <td>2222.0</td>
      <td>\n\n\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4982</th>
      <td>2614</td>
      <td>25291.0</td>
      <td>\n\n\n</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
assert_equal(rad_data2.iloc[0]["subject_id"], 1245)
```


    ---------------------------------------------------------------------------

    AssertionError                            Traceback (most recent call last)

    <ipython-input-29-6203bb8b0bce> in <module>()
    ----> 1 assert_equal(rad_data2.iloc[0]["subject_id"], 1245)
    

    /opt/conda/lib/python3.6/unittest/case.py in assertEqual(self, first, second, msg)
        827         """
        828         assertion_func = self._getAssertEqualityFunc(first, second)
    --> 829         assertion_func(first, second, msg=msg)
        830 
        831     def assertNotEqual(self, first, second, msg=None):


    /opt/conda/lib/python3.6/unittest/case.py in _baseAssertEqual(self, first, second, msg)
        820             standardMsg = '%s != %s' % _common_shorten_repr(first, second)
        821             msg = self._formatMessage(msg, standardMsg)
    --> 822             raise self.failureException(msg)
        823 
        824     def assertEqual(self, first, second, msg=None):


    AssertionError: 1884 != 1245



```python
assert_equal(rad_data2.iloc[-2]["subject_id"], 463)
```

## What do the most similar and most dissimilar (relative to 50) reports look like?


```python
print(rad_data.iloc[50]["text"])
```


```python
rad_data2.iloc[0]["text"]
```


```python

print(rad_data2.iloc[-2]["text"])
```

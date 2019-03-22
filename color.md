
# Classes and Object Oriented Programming
## Color


```python
from nose.tools import assert_equal, assert_true, assert_false, assert_raises, assert_almost_equal
```

**Problem 1 (10 points):** Implement `validate_alpha`. The function takes an argument, alpha, and returns alpha as a floating point number. If alpha is not between 0 and 1, a `ValueError` is raised.




```python
def validate_alpha(alpha):
    #if alpha != []:

    alpha = float(alpha)
    
    if alpha >= 0.0 and alpha <= 1.0:
        return alpha
    else:
        raise ValueError()


    #if alpha >= 0 and alpha <= 1:
        #return float(alpha)
    #else:
        #raise ValueError()


```


```python
assert_equal(validate_alpha("0.15"), 0.15)
```


```python
assert_equal(validate_alpha(0.15), 0.15)
```


```python
assert_raises(TypeError, validate_alpha, [3.4])
```


```python
assert_raises(ValueError, validate_alpha, 3.4)
```

**Problem 2 (10 points):** Implement `validate_color`. The function takes an argument, color, and returns color as an integer number. If color is not between 0 and 255, a `ValueError` is raised.


```python
def validate_color(color):
    color = int(color)
    if color >= 0 and color <= 255:
        return color
    else:
        raise ValueError()
    
#doesn't account for empty
```


```python
assert_raises(TypeError, validate_color, [])
```


```python
assert_raises(ValueError, validate_color, -5)
```


```python
assert_raises(ValueError, validate_color, 500)
```


```python
assert_equal(validate_color(55), 55)
```


```python
assert_equal(validate_color(47.3), 47)
```

**Problem 3. (30 Points):** Define a named tuple ``rgbalpha`` that represents an RGB$\alpha$ color. Define an `rgba` class that inherits from the ``rgbalpha`` named tuple and adds an attribute name. 

Because we are inheriting from an immutable class, we need to do our validation of the color using a [`__new__`](https://docs.python.org/3/reference/datamodel.html#object.__new__) method, which is a static method of the class not of the instance of the class.

Define the following methods:

* `invert_rgb`
    * See docstring for method behavior to implement
* `grayscale`
    * See docstring for method behavior to implement

* ``__str__``
    * the returned string should include the integer values as zero padded integers (e.g. `028`) for each color and the floating point value with 2 decimal (e.g. `0.27`) points for alpha
* ``__repr__``
    * The returned string should include the class name and at least the information in the `__str__` string. 
* ``__add__``
    * When adding two `rgba` instances (e.g. `c1` and `c2`) to create a new `rgba` instance (e.g. `c3`), the color channels of `c3`  should be the sum of the two values mod 256. For example, $c3=(c1.r+c2.r)\mod 256$.
    * The alpha value should be the maximum of the two alpha values ($c3.\text{alpha}=\max(c1.\text{alpha},c2.\text{alpha})$)
* ``__eq__``
    * In comparing equality, ignore the alpha values
* ``__abs__``
    * The absolute value should be the root mean square of the color values ignoring the alpha value.



```python
from collections import namedtuple
import math
rgbalpha = namedtuple("rgbalpha",['r','g','b','alpha'])
class rgba(rgbalpha):
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, validate_color(args[0]),
                          validate_color(args[1]),
                          validate_color(args[2]),
                          validate_alpha(args[3]))
    def __init__(self, *args, name="null"):
        self.name = name
        #raise NotImplementedError()
        
    @property
    def name(self):
        return self.__name
    #raise NotImplementedError()
    @name.setter
    def name(self,n):
        self.__name = n.lower()
    #raise NotImplementedError()
    
    def invert_rgb(self): 
        """
        this function inverts the RGB color by subtracting 
        all color values from 255 and returns a new rgba object with the 
        new color values.
        
        alpha is not modified
        
        name
        """
        new_r = 255 - self.r
        new_g = 255 - self.g
        new_b = 255 - self.b
        new_rgba = rgba(new_r, new_g, new_b, self.alpha)
        #raise NotImplementedError()
    
    def grayscale(self): 

        """
        this function converts RGB color to grayscale by using a 
        weight average formula: 0.299Red+0.587Green+0.114Blue
        """
        grey = 0.299*self.r + 0.587*self.g + 0.114*self.b 
        return rgba(grey, grey, grey, self.alpha) #necessary to type "grey" 3 times?
        #raise NotImplementedError() 
        
    def __str__(self):
        r = "%03d"%(self.r)
        g= "%03d"%(self.g)
        b = "%03d"%(self.b)
        alpha = '%.2f'%(self.alpha)
        return r+g+b+alpha

        #aise NotImplementedError()
    def __repr__(self):
        return"%s, %d, %d, %d, %f"%(self.name, self.r, self.g, self.b, self.alpha)
        #aise NotImplementedError()
    def __add__(self,color):
        red3 = (self.r + color.r) % 256
        green3 = (self.g + color.g) % 256
        blue3 = (self.b + color.b) % 256
        alpha3 = max(self.alpha, color.alpha)
        return rgba(red3, green3, blue3, alpha3)

        #aise NotImplementedError()
    def __eq__(self,color):
        return self.r == color.r and self.g == color.g and self.b == color.b

        #aise NotImplementedError()
    def __abs__(self):
        return math.sqrt((self.r**2)+(self.g**2)+(self.b**2))
        #aise NotImplementedError()
```


```python
c=rgba(100,32,47,0.9785)
assert_true('032' in c.__str__())
```


```python
c=rgba(100,32,47,0.9785)
assert_true('0.9785' in c.__repr__())
```


```python
c=rgba(100,32,47,0.9785)
cg = c.grayscale()
assert_equal(cg.r,54)
assert_equal(cg.g,54)
assert_equal(cg.b,54)
```


```python
c=rgba(100,32,47,0.9785)
assert_true(isinstance(c,tuple))
```


```python
assert_true(isinstance(c,rgbalpha))
```


```python
c1 = rgba(100,32,47,0.9785)
c2 =rgba(255,0,0,0.1)
c3 = c1+c2
assert_equal(c3.r,99)
assert_equal(c3.g,32)
assert_equal(c3.alpha,0.9785)
```


```python
c1 = rgba(100,32,47,0.9785)
c2 =rgba(255,0,0,0.1)
assert_false(c1==c2)
```


```python
assert_true(c1==c1)

```


```python
assert_almost_equal(abs(c1),115.03477735015616)
```

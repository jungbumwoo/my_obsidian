
Return a new [partial object](https://docs.python.org/3/library/functools.html#partial-objects) which when called will behave like _func_ called with the positional arguments _args_ and keyword arguments _keywords_.

```python
def partial(func, /, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = {**keywords, **fkeywords}
        return func(*args, *fargs, **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```


```python
from functools import partialc

def my_func(a, b, c):
    print(a, b, c)

f = partial(my_func, 10)

f(20, 30)  # 10 20 30 

# or

fn = lambda b, c: my_func(10, b, c)

f(20, 30)  # 10 20 30

```

it is quite flexible with parameters:

```python
def my_func(a, b, *args, k1, k2, **kwargs):
    print(a, b, args, k1, k2, kwargs)

f = partial(my_func, 10, k1='a')

f(20, 30, 40, k2='b', k3='c') 
# 10 20 (30, 40) a b {'k3': 'c'}

# same thing using regular func

def f(b, *args, k2, **kwargs):
    return my_func(10, b, *args, k1='a', k2=k2, **kwargs)

f(20, 30, 40, k2='b', k3='c')
```

## Caveat

```python
def my_func(a, b, c):
    print(a, b, c)

a = 10
f = partial(my_func, a)

f(20, 30)  # 10 20 30
a = 100
f(20, 30)  # 10 20 30   a != 100
```

the value for **a** is fixed once the partial has been created.
In fact, the memory address of **a** is baked in to the partial, and **a** is immutable.

```python
a = [10, 20]
f = partial(my_func, a)

f(100, 200)  # [10, 20] 100 200

a.append(30)

f(100, 200) # [10, 20, 30] 100 200
```

## Use Case

We tend to use partials in situation where we need to call a function that actually requires more parameters than we can supply.

Often this is because we are working with exiting libraries or code, and we have a special case.

For example, suppose we have points (represented as tuples), and we want to sort them based on the distance of the point from some other fixed point.

```python
origin = (0, 0)

l = [(1,1), (0, 2), (-3, 2), (0,0), (10, 10)]
sorted(l) # [(-3, 2), (0, 0), (0, 2) ...]  but, I want to sort by distance

dist2 = lambda x, y: (x[0]-y[0])**2 + (x[1]-y[1])**2
dist2((1, 1), origin) # 2

sorted(l, key=?) # key is what key needs to be a function that takes in a single parameter
sorted(l, key=dists2(end, origin)) # not work. this is function call. not func. key has to be callable. func.

# how to solve?
# sorted(l, key=dist_from_origin)



```


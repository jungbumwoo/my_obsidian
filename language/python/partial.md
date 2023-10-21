
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
# 10 20 (30, 40) a b

```

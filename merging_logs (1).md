
# Merging log files with iterators

This code is based on ideas from @jima and @nfultz



First, some test data ...


```python
testa = [
    {"time": 1, "who":"neala"},
    {"time": 3, "who":"jima"},
    {"time": 4, "who":"neala"},
    {"time": 7, "who":"jima"},
    {"time": 8, "who":"neala"},
]

testb = [
    {"time": 1, "who":"nealb"},
    {"time": 2, "who":"jimb"},
    {"time": 4, "who":"nealb"},
    {"time": 8, "who":"jimb"},
    {"time": 9, "who":"jimb"},
]

filenames = ['testa', 'testb']
```

## We need iterators
It turns out you can't use next() on a list


```python
print(next(testa))
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-195-4b01cec03353> in <module>()
    ----> 1 print(next(testa))
    

    TypeError: 'list' object is not an iterator


We'll need an iterator for each list, so let's make them


```python
lists = [testa, testb]
iterators = [iter(L) for L in lists]
```

## The cache

The rest of this code is based on the idea of having a data cache for each iterator and keeping the cache updated as I work through the lists.


```python
def update_cache(iterators, old_cache):
    new_cache = [ 
        next(it,None) if old_value is None else old_value 
        for it,old_value in zip(iterators, old_cache)
    ]
    return new_cache
```

First, initialize the cache


```python
cache = [None] * len(iterators)
print(cache)
```

    [None, None]



```python
cache = update_cache(iterators, cache)
print(cache)
```

    [{'time': 1, 'who': 'neala'}, {'time': 1, 'who': 'nealb'}]



```python
while [_ for _ in cache if _ is not None]:
    cache_min = min(
        (c for c in cache if c is not None) , # ignore the None values 
        key=lambda x: (x['time'])
    )
    i = cache.index(cache_min)
    print(filenames[i], cache[i]) # change this to yield
    cache[i] = None
    cache = update_cache(iterators, cache)
```

    testa {'time': 1, 'who': 'neala'}
    testb {'time': 1, 'who': 'nealb'}
    testb {'time': 2, 'who': 'jimb'}
    testa {'time': 3, 'who': 'jima'}
    testa {'time': 4, 'who': 'neala'}
    testb {'time': 4, 'who': 'nealb'}
    testa {'time': 7, 'who': 'jima'}
    testa {'time': 8, 'who': 'neala'}
    testb {'time': 8, 'who': 'jimb'}
    testb {'time': 9, 'who': 'jimb'}


## Putting it all together in one function


```python
def merge_lists(lists, filenames):
    '''
    Merge lists of dictionaries. The result is ordered by timestamp.
    '''
    def update_cache(iterators, old_cache):
        new_cache = [ 
            next(it,None) if old_value is None else old_value 
            for it,old_value in zip(iterators, old_cache)
        ]
        return new_cache

    iterators = [iter(L) for L in lists]
    cache = [None]*len(iterators)
    cache = update_cache(iterators, cache)
    
    while [_ for _ in cache if _ is not None]:
        cache_min = min(
            (c for c in cache if c is not None) , # ignore the None values 
            key=lambda x: (x['time'])
        )
        i = cache.index(cache_min)
        yield (filenames[i], cache[i])
        cache[i] = None
        cache = update_cache(iterators, cache)

```


```python
for x in merge_lists([testa, testb], filenames):
    print(x)
```

    ('testa', {'time': 1, 'who': 'neala'})
    ('testb', {'time': 1, 'who': 'nealb'})
    ('testb', {'time': 2, 'who': 'jimb'})
    ('testa', {'time': 3, 'who': 'jima'})
    ('testa', {'time': 4, 'who': 'neala'})
    ('testb', {'time': 4, 'who': 'nealb'})
    ('testa', {'time': 7, 'who': 'jima'})
    ('testa', {'time': 8, 'who': 'neala'})
    ('testb', {'time': 8, 'who': 'jimb'})
    ('testb', {'time': 9, 'who': 'jimb'})



```python

```

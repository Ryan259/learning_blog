缓存使用前

```
import requests

cache = dict()

def get_article_from_server(url):
    print("Fetching article from server...")
    response = requests.get(url)
    return response.text

def get_article(url):
    print("Getting article...")
    if url not in cache:
        cache[url] = get_article_from_server(url)

    return cache[url]

get_article("https://realpython.com/sorting-algorithms-python/")
get_article("https://realpython.com/sorting-algorithms-python/")

```
shell

```
$ pip install requests
$ python caching.py
Getting article...
Fetching article from server...
Getting article...

```

Before using @lru_cache
```
def steps_to(stair):
    if stair == 1:
        # You can reach the first stair with only a single step
        # from the floor.
        return 1
    elif stair == 2:
        # You can reach the second stair by jumping from the
        # floor with a single two-stair hop or by jumping a single
        # stair a couple of times.
        return 2
    elif stair == 3:
        # You can reach the third stair using four possible
        # combinations:
        # 1. Jumping all the way from the floor
        # 2. Jumping two stairs, then one
        # 3. Jumping one stair, then two
        # 4. Jumping one stair three times
        return 4
    else:
        # You can reach your current stair from three different places:
        # 1. From three stairs down
        # 2. From two stairs down
        # 2. From one stair down
        #
        # If you add up the number of ways of getting to those
        # those three positions, then you should have your solution.
        return (
            steps_to(stair - 3)
            + steps_to(stair - 2)
            + steps_to(stair - 1)
        )

print(steps_to(4))
```

shell
```
$ python stairs.py
7
```

Great! The code works for 4 stairs, but how about counting how many steps to reach a higher place 
in the staircase? Change the stair number in line 33 to 30 and rerun the script:

shell
```
$ python stairs.py
53798080
```

Timing Your Code
```
from timeit import repeat
setup_code = "from __main__ import steps_to"
stmt = "steps_to(30)"
times = repeat(setup=setup_code, stmt=stmt, repeat=3, number=10)
print(f"Minimum execution time: {min(times)}")
```

shell
```
$ python stairs.py
53798080
Minimum execution time: 40.014977024000004
```

Using Memoization to Improve the Solution
```
from functools import lru_cache
from timeit import repeat

@lru_cache
def steps_to(stair):
    if stair == 1:

```

shell
```
$ python stairs.py
53798080
Minimum execution time: 7.999999999987184e-07
```

```
from functools import lru_cache
from timeit import repeat

@lru_cache(maxsize=16)
def steps_to(stair):
    if stair == 1:
        # You can reach the first stair with only a single step
        # from the floor.
        return 1
    elif stair == 2:
        # You can reach the second stair by jumping from the
        # floor with a single two-stair hop or by jumping a single
        # stair a couple of times.
        return 2
    elif stair == 3:
        # You can reach the third stair using four possible
        # combinations:
        # 1. Jumping all the way from the floor
        # 2. Jumping two stairs, then one
        # 3. Jumping one stair, then two
        # 4. Jumping one stair three times
        return 4
    else:
        # You can reach your current stair from three different places:
        # 1. From three stairs down
        # 2. From two stairs down
        # 2. From one stair down
        #
        # If you add up the number of ways of getting to those
        # those three positions, then you should have your solution.
        return (
            steps_to(stair - 3)
            + steps_to(stair - 2)
            + steps_to(stair - 1)
        )

print(steps_to(30))

print(steps_to.cache_info())

```

shell
```
$ python stairs.py
53798080
CacheInfo(hits=52, misses=30, maxsize=16, currsize=16)
```


Adding Cache Expiration
```
import feedparser
import requests
import ssl
import time

if hasattr(ssl, "_create_unverified_context"):
    ssl._create_default_https_context = ssl._create_unverified_context

def get_article_from_server(url):
    print("Fetching article from server...")
    response = requests.get(url)
    return response.text

def monitor(url):
    maxlen = 45
    while True:
        print("\nChecking feed...")
        feed = feedparser.parse(url)

        for entry in feed.entries[:5]:
            if "python" in entry.title.lower():
                truncated_title = (
                    entry.title[:maxlen] + "..."
                    if len(entry.title) > maxlen
                    else entry.title
                )
                print(
                    "Match found:",
                    truncated_title,
                    len(get_article_from_server(entry.link)),
                )

        time.sleep(5)

monitor("https://realpython.com/atom.xml")
```

shell
```
$ pip install feedparser requests
$ python monitor.py

Checking feed...
Fetching article from server...
The Real Python Podcast  Episode #28: Using ... 29520
Fetching article from server...
Python Community Interview With David Amos 54256
Fetching article from server...
Working With Linked Lists in Python 37099
Fetching article from server...
Python Practice Problems: Get Ready for Your ... 164888
Fetching article from server...
The Real Python Podcast  Episode #27: Prepar... 30784

Checking feed...
Fetching article from server...
The Real Python Podcast  Episode #28: Using ... 29520
Fetching article from server...
Python Community Interview With David Amos 54256
Fetching article from server...
Working With Linked Lists in Python 37099
Fetching article from server...
Python Practice Problems: Get Ready for Your ... 164888
Fetching article from server...
The Real Python Podcast  Episode #27: Prepar... 30784

```


Evicting Cache Entries Based on Both Time and Space
```
from functools import lru_cache, wraps
from datetime import datetime, timedelta

def timed_lru_cache(seconds: int, maxsize: int = 128):
    def wrapper_cache(func):
        func = lru_cache(maxsize=maxsize)(func)
        func.lifetime = timedelta(seconds=seconds)
        func.expiration = datetime.utcnow() + func.lifetime

        @wraps(func)
        def wrapped_func(*args, **kwargs):
            if datetime.utcnow() >= func.expiration:
                func.cache_clear()
                func.expiration = datetime.utcnow() + func.lifetime

            return func(*args, **kwargs)

        return wrapped_func

    return wrapper_cache
```

Caching Articles With the New Decorator
```
import feedparser
import requests
import ssl
import time

from functools import lru_cache, wraps
from datetime import datetime, timedelta

if hasattr(ssl, "_create_unverified_context"):
    ssl._create_default_https_context = ssl._create_unverified_context

def timed_lru_cache(seconds: int, maxsize: int = 128):
    def wrapper_cache(func):
        func = lru_cache(maxsize=maxsize)(func)
        func.lifetime = timedelta(seconds=seconds)
        func.expiration = datetime.utcnow() + func.lifetime

        @wraps(func)
        def wrapped_func(*args, **kwargs):
            if datetime.utcnow() >= func.expiration:
                func.cache_clear()
                func.expiration = datetime.utcnow() + func.lifetime

            return func(*args, **kwargs)

        return wrapped_func

    return wrapper_cache

@timed_lru_cache(10)
def get_article_from_server(url):
    print("Fetching article from server...")
    response = requests.get(url)
    return response.text

def monitor(url):
    maxlen = 45
    while True:
        print("\nChecking feed...")
        feed = feedparser.parse(url)

        for entry in feed.entries[:5]:
            if "python" in entry.title.lower():
                truncated_title = (
                    entry.title[:maxlen] + "..."
                    if len(entry.title) > maxlen
                    else entry.title
                )
                print(
                    "Match found:",
                    truncated_title,
                    len(get_article_from_server(entry.link)),
                )

        time.sleep(5)

monitor("https://realpython.com/atom.xml")

```

shell
```
$ python monitor.py

Checking feed...
Fetching article from server...
Match found: The Real Python Podcast  Episode #28: Using ... 29521
Fetching article from server...
Match found: Python Community Interview With David Amos 54254
Fetching article from server...
Match found: Working With Linked Lists in Python 37100
Fetching article from server...
Match found: Python Practice Problems: Get Ready for Your ... 164887
Fetching article from server...
Match found: The Real Python Podcast  Episode #27: Prepar... 30783

Checking feed...
Match found: The Real Python Podcast  Episode #28: Using ... 29521
Match found: Python Community Interview With David Amos 54254
Match found: Working With Linked Lists in Python 37100
Match found: Python Practice Problems: Get Ready for Your ... 164887
Match found: The Real Python Podcast  Episode #27: Prepar... 30783

Checking feed...
Match found: The Real Python Podcast  Episode #28: Using ... 29521
Match found: Python Community Interview With David Amos 54254
Match found: Working With Linked Lists in Python 37100
Match found: Python Practice Problems: Get Ready for Your ... 164887
Match found: The Real Python Podcast  Episode #27: Prepar... 30783

Checking feed...
Fetching article from server...
Match found: The Real Python Podcast  Episode #28: Using ... 29521
Fetching article from server...
Match found: Python Community Interview With David Amos 54254
Fetching article from server...
Match found: Working With Linked Lists in Python 37099
Fetching article from server...
Match found: Python Practice Problems: Get Ready for Your ... 164888
Fetching article from server...
Match found: The Real Python Podcast  Episode #27: Prepar... 30783
```


## sorting list of lists
```python
xs = {'a': 4, 'c': 2, 'b': 3, 'd': 1}
sorted(xs.items(), key=lambda x: x[1], reverse=True)
```

---

## read from a text file by a pattern
```python
import re

with open(path, "r", encoding="utf-8") as f:
        lines = f.readlines()
        pat = re.compile(r"microsec: (\d+)")
        lines = [int(i.group(1)) for i in (re.search(pat, l) for l in lines if "microsec" in l) if i]
```

---

## argsparse
```python
import argparse
from os import getenv


def get_args():
    parser = argparse.ArgumentParser()
    
    parser.add_argument("--connector", "-c",
                        help="Get connector: api, oracle",
                        default="api")
    parser.add_argument("--testsymbol", "-s",
                        help="test symbol: ada, bnb ...")
    parser.add_argument("--testmodel", "-m",
                        help="Test model: 1, 2 ...")
                        
    # adding default argument value from environment
    parser.add_argument("-l", "--log", action="store",
                        help="log file path, default is console output",
                        default=getenv("APP_LOG", None))

    # set certain type to given argument and check if one has valid type
    parser.add_argument("--entries-number", action="store",
                        help="number of parsed entries, default is 10000",
                        type=int,
                        default=int(getenv("APP_ENTNUMBER", 10000)))

    # add possible argument values: as sequence and as some range 
    parser.add_argument("--log-format", action="store",
                        help="log messages format, default is short",
                        choices=["short", "full", "json"], metavar="short | full | json",
                        default=getenv("APP_LOGFORMAT", "short"))

    parser.add_argument("--log-level", action="store",
                        help="log facility level, default is 2: ERROR",
                        type=int, choices=range(1, 6), metavar="{1-5}",
                        default=int(getenv("APP_LOGLEVEL", 2)))

    # add given argument value validation
    def _non_empty_printable(string):
        if not string or not string.isprintable():
            raise argparse.ArgumentTypeError(
                "Argument has to be printable non-empty string.")
        return string

    parser.add_argument("-s", "--session-name", action="store",
                        help="name of session, default is New session",
                        type=_non_empty_printable,
                        default=getenv("APP_SESSIONNAME", "New session"))
    
    return parser.parse_args()
```
---

## set logger

```python
import logging

def set_logger():
    handlers = []

    log_file = "signal_server.log"
    handlers.append(logging.FileHandler(log_file))
    handlers.append(logging.StreamHandler())

    lglvl = logging.INFO
    logging.basicConfig(
        level=lglvl,
        format="%(asctime)s %(levelname)s %(message)s",
        handlers=handlers
    )
```

---

## Finding the Largest or Smallest N Items
```python
import heapq
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2] 
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23] 
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]

portfolio = [
{"name": "IBM", "shares": 100, "price": 91.1}, {"name": "AAPL", "shares": 50, "price": 543.22}, {"name": "FB", "shares": 200, "price": 21.09}, {"name": "HPQ", "shares": 35, "price": 31.75}, {"name": "YHOO", "shares": 45, "price": 16.35}, {"name": "ACME", "shares": 75, "price": 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s["price"])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s["price"])
```

---
## Exceptions

```python
import signal
import sys

class StopException(Exception):
    def __init__(self, message):
        # Now for your custom code...
        self.message = message

def handler(signum, frame):
    msg = f"Signal handler called with signal{signum}"
    # custom code
    raise StopException(msg)

def excepthook(exc_type, exc_value, exc_traceback):
    logger.error(f"Exception hook has been fired: {exc_value}", exc_info=(exc_type, exc_value, exc_traceback))

signal.signal(signal.SIGINT, handler)
signal.signal(signal.SIGTERM, handler)
sys.excepthook = excepthook

```

---
## Chunking

```python
data = [1, 2, 3, 4, 5, 6, 7, 8]
n = 3
[data[i:(i + n)] for i in range(0, len(data), n)]  # --> [[1, 2, 3], [4, 5, 6], [7, 8]]
```

---
## some useful itertools examples

```python

from itertools import chain, combinations, count, groupby, islice, zip_longest


# infinite iterator which returns sequenced values with defined step, like 1, 2, 3...
counter = count()
counter10 = count(10, 10)
abc = "abcd"
[i for i in zip(abc, counter)]  # --> [("a", 0), ("b", 1), ("c", 2), ("d", 3)]
[i for i in zip(abc, counter10)]  # --> [("a", 10), ("b", 20), ("c", 30), ("d", 40)]


# iterator which combines several iterables as one
# Important note: expired iterator, can be used only ones
abc = "abcd"
efg = "efgh"
chars = ["i", "j", "k", "l"]
[i for i in chain(abc, efg, chars)]  # --> ["a", "b", "c", "d", "e", "f",
                                     #     "g", "h", "i", "j", "k", "l"]


# similar iterator which can flat included iterables
strings = ["abcd", "efgh"]
numbers = [[1, 2, 3], [4, 5, 6]]
[i for i in chain.from_iterable(chain(strings, numbers))]  # --> ["a", "b", "c", "d", "e", "f",
                                                           #     "g", "h", 1, 2, 3, 4, 5, 6]


# iterator which can group some values by defined key
# Important notes: requires sorted data
#                  expired iterator, can be used only ones
#                  is not the same SQL's GROUP BY
string = "abacbcacc"
[(i, list(j)) for i, j in groupby(sorted(string))]  # --> [("a", ["a", "a", "a"]), ("b", ["b", "b"]),
                                                    #     ("c", ["c", "c", "c", "c"])]

pairs = [("c", 300), ("c", 100), ("a", 0),
        ("a", 1), ("c", 400), ("a", 2),
        ("b", 10), ("b", 20), ("c", 200)]

grouped = groupby(sorted(pairs, key=lambda x: x[0]), key=lambda y: y[0])
[(i, list(j)) for i, j in grouped]  # --> [("a", [("a", 0), ("a", 1), ("a", 2)]),
                                    #     ("b", [("b", 10), ("b", 20)]),
                                    #     ("c", [("c", 300), ("c", 100), ("c", 400), ("c", 200)])]
# `grouped` can not be use one more time, see notes
[(i, list(j)) for i, j in grouped]  # --> []


# iterator which is the same as indexing operator, but it provides values one by one instead full range
# Important note: expired iterator, can be used only ones
data = [1, 2, 3, 4, 5, 6, 7, 8]
data[1:5]  # --> [2, 3, 4, 5] given range here is real list of values
sliced = islice(data, 1, 5)  # --> no values, only iterator that can be used for extracting values
[next(sliced) for _ in range(2)]  # --> [2, 3]
[next(sliced) for _ in range(2)]  # --> [4, 5]


# iterator which is similar to zip, but allows you filling absent values with defined pad
abc = "abcd"
numbers = [1, 2]
[i for i in zip(abc, numbers)]  # --> [("a", 1), ("b", 2)] due `numbers` is shorter than `abc`
[i for i in zip_longest(abc, numbers, fillvalue=0)]  # --> [("a", 1), ("b", 2), ("c", 0), ("d", 0)]


# combinatoric iterator which can generate all possible combinations of defined length
# Important notes: no repeated elements
#                  expired iterator, can be used only ones
abc = "abcd"
cs2 = combinations(abc, 2)
cs3 = combinations(abc, 3)
list(cs2)  # --> [("a", "b"), ("a", "c"), ("a", "d"), ("b", "c"), ("b", "d"), ("c", "d")]
list(cs3)  # --> [("a", "b", "c"), ("a", "b", "d"), ("a", "c", "d"), ("b", "c", "d")]
```

---
## some asyncio examples

```python

# this example demonstrates handling shell command in asynchronous way
# here it is fping which checks if given hosts are alive asynchronously
import asyncio
from itertools import chain


async def is_alive(hosts):
    # create subprocess for shell command execution, here it is fping tool
    shell_cmd = asyncio.create_subprocess_shell(f"fping -a -t 100 {hosts}",
                                                stdin=asyncio.subprocess.DEVNULL,
                                                stdout=asyncio.subprocess.PIPE,
                                                stderr=asyncio.subprocess.PIPE)
    result = await shell_cmd
    msg, _ = await result.communicate()

    return msg


# execute shell command with given arguments and gather output data
async def checking_alive(hosts):
    # task list from arguments, one task per provided argument
    checking_hosts = [is_alive(h) for h in hosts]
    # wait for performing all tasks
    checked_hosts = await asyncio.gather(*checking_hosts)

    return [x for x in chain.from_iterable(i.decode("utf-8", "ignore").split("\n") for i in checked_hosts if i) if x]


# handling results
async def print_result(hosts):
    # wait for data gathering
    alive_hosts = await checking_alive(hosts)

    for h in alive_hosts:
        print(h)


# create subprocess for each argument, here it is a host IP
hosts = ["192.168.88.1", "127.0.0.1", "8.8.8.8", "1.1.1.1", "8.8.4.4", "192.168.0.1"]
asyncio.run(print_result(hosts))

# making subprocess for each argument if there are many of them could be inefficient
# but if shell command can handle several of them simultaneously, we can use this feature
# create subprocess for each several arguments, here those are hosts IPs
# due fping can handle several hosts at the same time
sliced_hosts = [" ".join(hosts[i:i + 10]) for i in range(0, len(hosts), 10)]  # --> ["host1 host2 host3...",
                                                                              #     "host11 host12 host13...", 
                                                                              #     "hostN...", ...]
asyncio.run(print_result(sliced_hosts))


# this example demonstrates working with some asyncio libs
# here there is a list of image url that will be downloaded
# and saved on disk asynchronously
import asyncio
import aiofiles
import aiohttp


async def download_images(save_path, links):
    # dowload image and save one
    async def download_save(session, name, url):
        async with session.get(url) as resp:
            if resp.status != 200:
                return

            extention = url.split(".")[-1]
            # writing file incrementally and asynchronously,
            # chunk by chunk as new portion of data is received
            f = await aiofiles.open(f"{save_path}/{name}.{extention}", mode='wb')
            await f.write(await resp.read())  # write new chunk if only one is received
            await f.close()

    timeout = aiohttp.ClientTimeout(total=30)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        tasks = [download_save(session, name, url) for name, url in links]
        await asyncio.gather(*tasks)

links = [("kitty", "https://example.com/1.jpg"),
        ("doggy", "https://example.com/3.jpg"),
        ("panda", "https://example.com/117.png")]
save_path = "/images"  # here we suggest directory is reachable and open for writing

asyncio.run(download_images(save_path, links))
# as a result we will see doggy.jpg, kitty.jpg, panda.png in /images directory

```

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
    logger.error(f'Exception hook has been fired: {exc_value}', exc_info=(exc_type, exc_value, exc_traceback))

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
## Generator objects
def get_logline(file_name):
    """
    create generator from 11.log file
    pattern is like 19.05.2020 18:24:03.044269
    :return:
    """
    with open(file_name, "r") as log_file:
        for line in log_file:
            pattern = re.compile(r"\d{2}.\d{2}.\d{4} \d{2}:\d{2}:\d{2}.\d{3}")
            result = pattern.search(line)
            if result:
                yield result.group(0)


def test_gen():
    log = get_logline("11.log")
    for line in log:
        print(line)

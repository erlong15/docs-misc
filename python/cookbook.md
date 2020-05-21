## sorting list of lists
```python
xs = {'a': 4, 'c': 2, 'b': 3, 'd': 1}
sorted(xs.items(), key=lambda x: x[1], reverse=True)
```

---

## read from a text file by a pattern
```python
import re
def check_line(pat, line):
    result = pat.search(l)
    if result:
        return int(result.group(1))

with open(path, 'r') as f:
    lines = f.readlines()
    pat = re.compile(r"microsec: (\d+)")
    lines = [check_line(pat, l) for l in lines if "microsec" in l]
```

---

## argsparse
```python
import argparse 

def get_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('--connector', '-c',
                        help='Get connector: api, oracle',
                        default='api')
    parser.add_argument('--testsymbol', '-s',
                        help='test symbol: ada, bnb ...')
    parser.add_argument('--testmodel', '-m',
                        help='Test model: 1, 2 ...')
    return parser.parse_args()
```
---

## set logger

```python
import logging

def set_logger():
    handlers = []

    log_file = 'signal_server.log'
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
{'name': 'IBM', 'shares': 100, 'price': 91.1}, {'name': 'AAPL', 'shares': 50, 'price': 543.22}, {'name': 'FB', 'shares': 200, 'price': 21.09}, {'name': 'HPQ', 'shares': 35, 'price': 31.75}, {'name': 'YHOO', 'shares': 45, 'price': 16.35}, {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
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
    msg = f'Signal handler called with signal{signum}'
    # custom code
    raise StopException(msg)

def excepthook(exc_type, exc_value, exc_traceback):
    logger.error(f'Exception hook has been fired: {exc_value}', exc_info=(exc_type, exc_value, exc_traceback))

signal.signal(signal.SIGINT, handler)
signal.signal(signal.SIGTERM, handler)
sys.excepthook = excepthook

```


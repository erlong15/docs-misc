## sorting list of lists
```python
xs = {'a': 4, 'c': 2, 'b': 3, 'd': 1}
sorted(xs.items(), key=lambda x: x[1], reverse=True)
```

---

## read and sort file
```python
import re

with open(path, 'r') as f:
        lines = f.readlines()
        pat = r"microsec: (\d*)"
        lines = [int(re.search(pat, l).group(1)) for l in lines if "microsec" in l]
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
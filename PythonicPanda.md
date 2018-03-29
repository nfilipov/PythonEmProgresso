## Pythonic Pandas
### Playing with Python and Pandas to produce some nice figures
###### (probably we'll put this in a ipynb someday)

Let's begin with an import:

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
```

We're now plotting the simplest of Series:

```python
# careful about the syntax of numpy random here
ts = pd.Series(np.random.randn(1000), index=pd.date_range('1/1/2000', periods=1000))
```

This outputs the following image:
![alt text](figures\Figure_1.png)
[Figure_1]: C:\Users\Nicolas Filipovic\Documents\GitHub\PythonEmProgresso\figures\Figure_1.png "Random number generated with pd.Series and a nice axis!"



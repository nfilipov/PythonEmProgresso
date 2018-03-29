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

![Image](https://github.com/nfilipov/PythonEmProgresso/blob/master/figures/Figure_1.png?raw=true)

Turning into a DataFrame was ok. Then I did the following:

```python
# didnt work because they don't tell you periods should equal the arg in random.randn!!
ts = pd.DataFrame(np.random.randn(1000), index=pd.timedelta_range(0, periods=10, freq='H'))
# periods = 1000 worked.
```

The output of periods=1000 shows that timedelta_range is for points that will be regularly spaced and that is not what I want in the end...

![figure2](https://github.com/nfilipov/PythonEmProgresso/blob/master/figures/Figure_2.png?raw=true)
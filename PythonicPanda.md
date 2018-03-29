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
And on it we see that the x axis labeling and the number of tick marks is done automatically, printing things like "5 days 18:53:20" so this is not what I thought (I thought I had to specify the number of ticks I want).
Side note, here I am not plotting a time-series dataset yet, just random numbers that have probably some numpy data type. Yet, the axis recognizes time thanks to the date range arguments given to the function.

![figure2](https://github.com/nfilipov/PythonEmProgresso/blob/master/figures/Figure_2.png?raw=true)

Now that I think of it, I find odd the tick labeling since I gave freq='H' so I'll have to investigate on that.

Let's try to incorporate our timeseries data and give a unit of milliseconds (disrespectful to the fact that our points should in turn not be regularly distributed over time).

```python
print len(df.index) ## that was 693, btw.
ts = pd.DataFrame(df.API_IMS, index=pd.date_range(0, periods=len(df.index), freq='H'))
```

this returns a ``` ValueError: cannot reindex from a duplicate axis```. This is because my data was defined previously as:

```python
df = pd.read_csv(filepath_or_buffer=path_for_data+".csv",sep=";", index_col='time', infer_datetime_format=True) 
df.columns=['event#','ISIN@MIC','Status','MIC','Timestamp','ExchQH','QH_API','API_IMS','IMS_TA']
```

so instead of changing the DataFrame definition like removing the column index etc., I pump np.arrays out of it and parse them to my time series.

```python
ts = pd.DataFrame(df.API_IMS.values, index=pd.date_range(0, periods=len(df.index), freq='ms'))
ts.plot()
plt.show()
```

![figure3](https://github.com/nfilipov/PythonEmProgresso/blob/master/figures/Figure_3.png?raw=true)

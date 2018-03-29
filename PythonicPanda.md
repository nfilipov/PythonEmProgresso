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

Now, I'd like to remind myself of the goal: plot this data as a function of the proper timestamp points at which it was produced. We're missing:
- the timestamp data from index into time series
- the corresponding coordinates (for the moment we have fake ones)
- the proper axis marking-labeling (beauty is accessory one would say)

Let's just try plotting a (x,y) scatter plot of (time,API_IMS). Based off the definition of my dataframe ```df```, we do:

```python
df.plot.scatter(x=df.index, y='QH_API')
```

... and that returns a ```KeyError: "Index([u'01:30:01:030', {....}  u'07:57:54:194'],\n      dtype='object', name=u'time', length=693) not in index"```

From here I can either convert it to a dtype=Timestamp or numeric datetime64 object, or I can remove the index_col argument in the pandas DataFrame initial defintion. But I won't because indexes are the point of Pandas :)

There are keyword arguments to make the ```pandas.read_csv()``` method better interpret and import the data. 
https://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_csv.html

```python
## read the data
df = pd.read_csv(filepath_or_buffer=path_for_data+".csv",
				 sep=";",
				 memory_map = True, # removes the I/O overhead by accessing a memory-based object instead of the actual filepath object.
				 infer_datetime_format=True,
				 parse_dates=True,
				 names=['event#','time','ISIN@MIC','Status','MIC','Timestamp','ExchQH','QH_API','API_IMS','IMS_TA'])
df.info()
```
output:
```
sys:1: DtypeWarning: Columns (5,6,7,8,9) have mixed types. Specify dtype option on import or set low_memory=False.
RangeIndex: 454521 entries, 0 to 454520
Data columns (total 10 columns):
event#       454520 non-null float64
time         454521 non-null object
ISIN@MIC     454521 non-null object
Status       454521 non-null object
MIC          454521 non-null object
Timestamp    454521 non-null object
ExchQH       454521 non-null object
QH_API       454521 non-null object
API_IMS      454521 non-null object
IMS_TA       454521 non-null object
dtypes: float64(1), object(9)
memory usage: 34.7+ MB
```

Sometimes, the solution is simple. Python may be full of methods and libs that make you want to try crazy code, but in the end trying things in an interpreter is the only way to reach conclusions quickly (not like I did this time).
After studying this problem for a long time, here are my conclusions:

- timestamp objects should be read properly if the milliseconds are separated with a **comma** and not a colon.
- specify the dtypes when there's a header: it's recommended because if you do, pandas will becom more verbose and infer with more wisdom your data objects. 

Note the change with the following:

```python
df = pd.read_csv(filepath_or_buffer=path_for_data+".csv",
				 sep=";",
				 memory_map = True, # removes the I/O overhead by accessing a memory-based object instead of the actual filepath object.
				 infer_datetime_format=True,
				 parse_dates=['time'],
				 index_col = 'time',
				 dtype={'event#': np.float64, 'ISIN@MIC': str, 'Status' : str, 'MIC': str, #'time': np.datetime64,
						'Timestamp':np.int64,'ExchQH': np.float64,'QH_API': np.float64,'API_IMS': np.float64,'IMS_TA': np.float64}) #,
df.info()
```
output:
```
DatetimeIndex: 6043 entries, 2018-03-29 06:45:59.413000 to 2018-03-29 14:24:51.296000
Data columns (total 9 columns):
Unnamed: 0    6043 non-null int64
ISIN@MIC      6043 non-null object
Status        6043 non-null object
MIC           6043 non-null object
Timestamp     6043 non-null int64
Exch_QH       6043 non-null int64
QH_API        6043 non-null float64
API_IMS       6043 non-null float64
IMS_TA        6043 non-null float64
dtypes: float64(3), int64(3), object(3)
```

which is exactly what I wanted

## Shit getting real: we plot hard

Let's try to plot 1D figures for the four timedeltas: Exch_QH, QH_API, API_IMS, IMS_TA.
Oddly picking stuff here and there I tried the following:

###### the style wasn't great, so I changed one setting after another to do this. here's what I settle with for today:

```python
fig, ax = plt.subplots(4, sharex=True)
df.Exch_QH.resample('30S').mean().plot(ax=ax[0], style='-o', alpha=0.3)
df.QH_API.resample('30S').mean().plot(ax=ax[1], style='-o', alpha=0.3) #.plot(style='-o')
df.API_IMS.resample('30S').mean().plot(ax=ax[2], style='-o', alpha=0.3) #.plot(style='-o')
df.IMS_TA.resample('30S').mean().plot(ax=ax[3], style='-o', alpha=0.3) #.plot(style='-o')
plt.show()
```

output:


![figure4](https://github.com/nfilipov/PythonEmProgresso/blob/master/figures/Figure_4.png?raw=true)


## Final output : file:///Z:/Nicolas/Performances/index_20180327.html

Good job, bro :) 



# Pandas Stock

```python
from pandas_datareader import data as pdr
import yfinance as yf
yf.pdr_override()

sec = pdr.get_data_yahoo('005930.KS', start='2018-05-04')
msft = pdr.get_data_yahoo('MSFT', start='2018-05-04')
# [*********************100%***********************]  1 of 1 completed
# [*********************100%***********************]  1 of 1 completed

print(sec.head(10))
#                Open     High      Low    Close     Adj Close    Volume
# Date
# 2018-05-04  53000.0  53900.0  51800.0  51900.0  46191.074219  39565391
# 2018-05-08  52600.0  53200.0  51900.0  52600.0  46814.070312  23104720
# 2018-05-09  52600.0  52800.0  50900.0  50900.0  45301.074219  16128305
# 2018-05-10  51700.0  51700.0  50600.0  51600.0  45924.066406  13905263
# 2018-05-11  52000.0  52200.0  51200.0  51300.0  45657.078125  10314997
# 2018-05-14  51000.0  51100.0  49900.0  50100.0  44589.066406  14909272
# 2018-05-15  50200.0  50400.0  49100.0  49200.0  43788.066406  18709146
# 2018-05-16  49200.0  50200.0  49150.0  49850.0  44366.574219  15918683
# 2018-05-17  50300.0  50500.0  49400.0  49400.0  43966.070312  10365440
# 2018-05-18  49900.0  49900.0  49350.0  49500.0  44055.070312   6706570

tmp_msft = msft.drop(columns='Volume')
print(tmp_msft.tail())
#                   Open        High         Low       Close   Adj Close
# Date
# 2022-02-14  293.769989  296.760010  291.350006  295.000000  294.391296
# 2022-02-15  300.010010  300.799988  297.019989  300.470001  299.850006
# 2022-02-16  298.369995  300.869995  293.679993  299.500000  299.500000
# 2022-02-17  296.359985  296.799988  290.000000  290.730011  290.730011
# 2022-02-18  293.049988  293.859985  286.309998  287.929993  287.929993

print(sec.index)
# DatetimeIndex(['2018-05-04', '2018-05-08', '2018-05-09', '2018-05-10',
#                '2018-05-11', '2018-05-14', '2018-05-15', '2018-05-16',
#                '2018-05-17', '2018-05-18',
#                ...
#                '2022-02-09', '2022-02-10', '2022-02-11', '2022-02-14',
#                '2022-02-15', '2022-02-16', '2022-02-17', '2022-02-18',
#                '2022-02-21', '2022-02-22'],
#               dtype='datetime64[ns]', name='Date', length=935, freq=None)

print(sec.columns)
# Index(['Open', 'High', 'Low', 'Close', 'Adj Close', 'Volume'], dtype='object')
```

## 시각화
```python
from pandas_datareader import data as pdr
import yfinance as yf

yf.pdr_override()

sec = pdr.get_data_yahoo('005930.KS', start='2018-05-04')
msft = pdr.get_data_yahoo('MSFT', start='2018-05-04')

import matplotlib.pyplot as plt

plt.plot(sec.index, sec.Close, 'b', label='Samsung Elactronics')
plt.plot(msft.index, msft.Close, 'r--', label='Microsoft')
plt.legend(loc='best')
plt.show()
```
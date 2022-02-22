# Pandas DataFrame

```python
import pandas as pd

# Dictionary를 이용한 DataFrame 생성
df = pd.DataFrame({'KOSPI': [1915, 1961, 2026, 2467, 2041],
                   'KOSDAQ': [542, 682, 631, 798, 675]})
print(df)
#    KOSPI  KOSDAQ
# 0   1915     542
# 1   1961     682
# 2   2026     631
# 3   2467     798
# 4   2041     675

df = pd.DataFrame({'KOSPI': [1915, 1961, 2026, 2467, 2041],
                   'KOSDAQ': [542, 682, 631, 798, 675]},
                  index=[2014, 2015, 2016, 2017, 2018])
print(df)
#       KOSPI  KOSDAQ
# 2014   1915     542
# 2015   1961     682
# 2016   2026     631
# 2017   2467     798
# 2018   2041     675

print(df.describe())
#              KOSPI      KOSDAQ
# count     5.000000    5.000000
# mean   2082.000000  665.600000
# std     221.117616   92.683871
# min    1915.000000  542.000000
# 25%    1961.000000  631.000000
# 50%    2026.000000  675.000000
# 75%    2041.000000  682.000000
# max    2467.000000  798.000000

print(df.info())
# <class 'pandas.core.frame.DataFrame'>
# Int64Index: 5 entries, 2014 to 2018
# Data columns (total 2 columns):
#  #   Column  Non-Null Count  Dtype
# ---  ------  --------------  -----
#  0   KOSPI   5 non-null      int64
#  1   KOSDAQ  5 non-null      int64
# dtypes: int64(2)
# memory usage: 120.0 bytes
# None

# Series를 이용한 DataFrame 생성
kospi = pd.Series([1915, 1961, 2026, 2467, 2041],
                  index=[2014, 2015, 2016, 2017, 2018],
                  name='KOSPI')
print(kospi)
# 2014    1915
# 2015    1961
# 2016    2026
# 2017    2467
# 2018    2041
# Name: KOSPI, dtype: int64

kosdaq = pd.Series([542, 682, 631, 798, 675],
                  index=[2014, 2015, 2016, 2017, 2018],
                  name='KOSDAQ')
print(kosdaq)
# 2014    542
# 2015    682
# 2016    631
# 2017    798
# 2018    675
# Name: KOSDAQ, dtype: int64

df = pd.DataFrame({kospi.name: kospi, kosdaq.name: kosdaq})
print(df)
#       KOSPI  KOSDAQ
# 2014   1915     542
# 2015   1961     682
# 2016   2026     631
# 2017   2467     798
# 2018   2041     675

# List를 이용한 DataFrame 생성
colums = ['KOSPI', 'KOSDAQ']
index = [2014, 2015, 2016, 2017, 2018]
rows = []
rows.append([1915, 542])
rows.append([1961, 682])
rows.append([2026, 631])
rows.append([2467, 798])
rows.append([2041, 675])
print(df)
#       KOSPI  KOSDAQ
# 2014   1915     542
# 2015   1961     682
# 2016   2026     631
# 2017   2467     798
# 2018   2041     675

# DataFrame Loop
for i in df.index:
    print(i, df['KOSPI'][i], df['KOSDAQ'][i])
# 2014 1915 542
# 2015 1961 682
# 2016 2026 631
# 2017 2467 798
# 2018 2041 675

for row in df.itertuples(name='KRX'):
    print(row)
# KRX(Index=2014, KOSPI=1915, KOSDAQ=542)
# KRX(Index=2015, KOSPI=1961, KOSDAQ=682)
# KRX(Index=2016, KOSPI=2026, KOSDAQ=631)
# KRX(Index=2017, KOSPI=2467, KOSDAQ=798)
# KRX(Index=2018, KOSPI=2041, KOSDAQ=675)

# itertuples()메소드가 iterrows()보다 더 빠르다.
for row in df.itertuples():
    print(row[0], row[1], row[2])
# 2014 1915 542
# 2015 1961 682
# 2016 2026 631
# 2017 2467 798
# 2018 2041 675

for idx, row in df.iterrows():
    print(idx, row[0], row[1])
# 2014 1915 542
# 2015 1961 682
# 2016 2026 631
# 2017 2467 798
# 2018 2041 675
```

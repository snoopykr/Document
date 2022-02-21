# Pandas

```python
import pandas as pd

s = pd.Series([0.0, 3.6, 2.0, 5.8, 4.2, 8.0])

print(s)
# 0    0.0
# 1    3.6
# 2    2.0
# 3    5.8
# 4    4.2
# 5    8.0
# dtype: float64

# 인덱스 변경
s.index = pd.Series([0.0, 1.2, 1.8, 3.0, 3.6, 4.8])
s.index.name = 'MY_IDX'
print(s)
# MY_IDX
# 0.0    0.0
# 1.2    3.6
# 1.8    2.0
# 3.0    5.8
# 3.6    4.2
# 4.8    8.0
# dtype: float64

s.name = 'MY_SERIES'
print(s)
# MY_IDX
# 0.0    0.0
# 1.2    3.6
# 1.8    2.0
# 3.0    5.8
# 3.6    4.2
# 4.8    8.0
# Name: MY_SERIES, dtype: float64

# 데이터 추가
s[5.9] = 5.5
print(s)
# MY_IDX
# 0.0    0.0
# 1.2    3.6
# 1.8    2.0
# 3.0    5.8
# 3.6    4.2
# 4.8    8.0
# 5.9    5.5
# Name: MY_SERIES, dtype: float64

ser = pd.Series([6.7, 4.2], index=[6.8, 8.0])
s = s.append(ser)
print(s)
# 0.0    0.0
# 1.2    3.6
# 1.8    2.0
# 3.0    5.8
# 3.6    4.2
# 4.8    8.0
# 5.9    5.5
# 6.8    6.7
# 8.0    4.2
# dtype: float64
```
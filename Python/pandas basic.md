# Pandas 기본

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

# 마지막 인덱스
print(s.index[-1])
# 8.0

# 마지막 벨류
print(s.values[-1])
# 4.2

# 로케이션 인덱스
print(s.loc[8.0])
# 4.2

# 정수 인덱스
print(s.iloc[-1])
# 4.2

# values는 배열 반환
print(s.values[:])
# [0.  3.6 2.  5.8 4.2 8.  5.5 6.7 4.2]

# iloc은 시리즈 반환
print(s.iloc[:])
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

# 데이터 삭제
print(s.drop(8.0))
# 0.0    0.0
# 1.2    3.6
# 1.8    2.0
# 3.0    5.8
# 3.6    4.2
# 4.8    8.0
# 5.9    5.5
# 6.8    6.7
# dtype: float64

print(s.describe())
# count    9.000000     원소 개수
# mean     4.444444     평균
# std      2.430078     표준편차
# min      0.000000     최솟값
# 25%      3.600000     제1 사분위수
# 50%      4.200000     제2 사분위수
# 75%      5.800000     제3 사분위수
# max      8.000000     최댓값
# dtype: float64
```
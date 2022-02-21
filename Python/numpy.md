# Numpy

```python
import numpy as np

# 생성
A = np.array([[1, 2], [3, 4]])
print(A)
# [[1 2]
#  [3 4]]

# 정보
print(type(A))
# <class 'numpy.ndarray'>

print(A.ndim)
# 2

print(A.shape)
# (2, 2)

print(A.dtype)
# int32

# 최대, 평균, 최소, 합계
print(A.max(), A.mean(), A.min(), A.sum())
# 4 2.5 1 10

# 접근
print(A[0], A[1])
# [1 2] [3 4]

print(A[0][0], A[0][1], A[1][0], A[1][1])
# 1 2 3 4

print(A[0, 0], A[0, 1], A[1, 0], A[1, 1])
# 1 2 3 4

# 형태 변환
print(A.T)
# [[1 3]
#  [2 4]]

print(A.flatten())
# [1 2 3 4]

# 연산
print(A + A)
# [[2 4]
#  [6 8]]

print(A - A)
# [[0 0]
#  [0 0]]

print(A * A)
# [[ 1  4]
#  [ 9 16]]

print(A / A)
# [[1. 1.]
#  [1. 1.]]

# 브로드 캐스팅 [10, 100] ==> [[10, 100], [10, 100]]
B = np.array([10, 100])

print(A * B)
# [[ 10 200]
#  [ 30 400]]

# 내적
print(A.dot(B))
# [210 430]

print(B.dot(B))
# 10100
```

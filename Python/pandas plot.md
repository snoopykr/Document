# Pandas 시리즈 출력

```python
import pandas as pd
import matplotlib.pyplot as plt

s = pd.Series([0.0, 3.6, 2.0, 5.8, 4.2, 8.0, 5.5, 6.7, 4.2])
s.index = pd.Index([0.0, 1.2, 1.8, 3.0, 3.6, 4.8, 5.9, 6.8, 8.0])

s.index.name = 'MY_IDX'
s.name = 'MY_SERIES'

plt.title('ELLIOTT_WAVE')
plt.plot(s, 'bs--')
plt.xticks(s.index)
plt.yticks(s.values)
plt.grid(True)
plt.show()
```
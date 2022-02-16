# Python 이것 저것...

## while과 for에 else를 사용
```python
i = 10
while i > 0:
    i -= 1
    if i % 2 == 0:
        continue
    # if i > 5:
    #     break
    print(i)
else:
    print('Condition is False')
```
반복을 종료하고 else블록을 실행, Python만의 특징...

## sort()와 sorted()
```python
>>> li = [2, 4, 3, 1, 4]
>>> li.sort()
>>> li
[1, 2, 3, 4, 4]
```
sort()는 리스트를 직접 정렬

```python
>>> li = [2, 4, 3, 1, 4]
>>> sorted(li)
[1, 2, 3, 4, 4]
>>> li
[2, 4, 3, 1, 4]
```
sorted()는 string, tuple, dictionary에서 사용 가능 (리스트를 복사해서 반환하기 때문에 다소 느림)

## append()와 extend()
```python
>>> L = [1, 2]
>>> L.append([3, 4])
>>> L
[1, 2, [3, 4]]
```
append()는 파라미터의 타입에 상관 없이 그대로 추가

```python
>>> L = [1, 2]
>>> L.extend([3,4])
>>> L
[1, 2, 3, 4]
```
extend()는 파라미터가 반복 자료인 경우 원소단위로 추가

## split(), join(), replace()
```python
>>> '-'.join('2012/01/04'.split('/'))
'2012-01-04'
>>> '2012/01/04'.replace('/', '-')
'2012-01-04'
>>> ''.join('1,234,567,890'.split(','))
'1234567890'
>>> '1,234,567,890'.replace(',', '')
'1234567890'
>>> format(1234567890, ',')
'1,234,567,890'
```

## 리스트 복사
```python
>>> myList = ['Thoughts', 'become', 'things']
>>> newList = myList[:]
>>> newList
['Thoughts', 'become', 'things']

>>> newList[-1] = 'actions'
>>> newList
['Thoughts', 'become', 'actions']
>>> myList
['Thoughts', 'become', 'things']
```

## Comprehension
```python
>>> nums = [1, 2, 3, 4, 5]
>>> squares = []
>>> for x in nums:
...    squares.append(x ** 2)
...    
>>> squares
[1, 4, 9, 16, 25]
```

```python
>>> nums = [1, 2, 3, 4, 5]
>>> squares = [x ** 2 for x in nums]
>>> squares
[1, 4, 9, 16, 25]
```

```python
>>> nums = [1, 2, 3, 4, 5]
>>> squares = [x ** 2 for x in nums if x % 2 == 0]
>>> squares
[4, 16]
```

## 문자열 포맷
```python
>>> crispr = {'EDIT': 'Editas Medicine', 'NTLA': "Intellia Therapeutics"}
>>> crispr['CRSP'] = 'CRISPR Therapeutics'
>>> len(crispr)
3
>>> for x in crispr:
...    print('%s : %s' % (x, crispr[x]))
...    
EDIT : Editas Medicine
NTLA : Intellia Therapeutics
CRSP : CRISPR Therapeutics
>>> for x in crispr:
...    print('{} : {}'.format(x, crispr[x]))
...    
EDIT : Editas Medicine
NTLA : Intellia Therapeutics
CRSP : CRISPR Therapeutics
>>> for x in crispr:
...    print(f'{x} : {crispr[x]}')
...    
EDIT : Editas Medicine
NTLA : Intellia Therapeutics
CRSP : CRISPR Therapeutics
```

## set
```python
>>> s = {'A', 'P', 'P', 'L', 'E'}
>>> s
{'P', 'E', 'A', 'L'}
>>> if 'A' in s:
...    print("'A' exists in", s)
...    
'A' exists in {'P', 'E', 'A', 'L'}
>>> setA = {1, 2, 3, 4, 5}
>>> setB = {3, 4, 5, 6, 7}
>>> setA & setB
{3, 4, 5}
>>> setA | setB
{1, 2, 3, 4, 5, 6, 7}
>>> setA - setB
{1, 2}
>>> setB - setA
{6, 7}
```

```python
>>> ls = []
>>> d = {}
>>> t = ()
>>> s = set()
>>> ls = [1, 3, 5, 2, 2, 3, 4, 2, 1, 1, 1, 5]
>>> list(set(ls))
[1, 2, 3, 4, 5]
```
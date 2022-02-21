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

## timeit
```python
>>> import timeit

>>> iteration_test = """
...for i in itr:
...    pass
..."""
>>> timeit.timeit(iteration_test, setup='itr = list(range(100000))', number=10000)
6.676608499999986
>>> timeit.timeit(iteration_test, setup='itr = tuple(range(100000))', number=10000)
6.654467300000022
>>> timeit.timeit(iteration_test, setup='itr = set(range(100000))', number=10000)
15.978206999999998

>>> search_test = """
...import random
...x = random.randint(0, len(itr)-1)
...if x in itr:
...  pass
..."""
>>> timeit.timeit(search_test, setup='itr = list(range(100000))', number=10000)
5.642429400000083
>>> timeit.timeit(search_test, setup='itr = tuple(range(100000))', number=10000)
5.339672700000051
>>> timeit.timeit(search_test, setup='itr = set(range(100000))', number=10000)
0.021988299999975425
```

## python의 int
```python
>>> googol = 10 ** 100
>>> type(googol)
<class 'int'>
>>> googol
10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```
python에서 정수는 제한이 없다.

## 연평균 성장률 (CAGR)
```python
def getCAGR(first, last, years):
    return (last / first) ** (1 / years) - 1


cagr = getCAGR(65300, 2669000, 20)

print('SEC CAGR : {:.2%}'.format(cagr))
```
삼성전자 연평균 성장률을 계산 했다.

## None
```python
def func1():
    pass


def func2():
    return


def func3():
    return None


print(func1())
print(func2())
print(func3())

print(type(None))
print(func1() is None)
```

## function
```python
def myFunc():
    var1 = 'a'
    var2 = [1, 2, 3]
    var3 = max
    return var1, var2, var3


print(myFunc())

s, l, f = myFunc()

print(s)
print(l)
print(f)
```

## lambda
```python
>>> InsertComma = lambda x : format(x, ',')
>>> InsertComma(1234567890)
'1,234,567,890'
```

```
[참고]
def lambda (인수):
    return 표현식
```

## help()
```python
>>> help('modules')
Please wait a moment while I gather a list of all available modules...
PIL                 _warnings           idna                queue
PyPDF2              _weakref            imaplib             quopri
PyQt5               _weakrefset         imghdr              random
__future__          _winapi             imp                 re
_abc                abc                 importlib           reprlib

~~~ 이하 생략 ~~~

>>> help('modules time')
Here is a list of modules whose name or summary contains 'time'.
If there are any, enter a module name to get more help.
_datetime - Fast implementation of the datetime type.
time - This module provides various functions to manipulate time values.
_strptime - Strptime-related classes and functions.

~~~ 이하 생략 ~~~

>>> help('datetime')
Help on module datetime:
NAME
    datetime - Fast implementation of the datetime type.
MODULE REFERENCE
    https://docs.python.org/3.7/library/datetime

~~~ 이하 생략 ~~~
```

## import
```python
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
>>> keyword.__file__
'D:\\Application\\anaconda3\\envs\\Tools\\lib\\keyword.py'

>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

## package
[ myPageckage/__init__.py ]
```python
__all__ = ['moduleA', 'moduleB']
```

[ myPageckage/moduleA.py ]
```python
def functionA():
    print("FUNCTION_A")


print('MODULE_A : ', __name__)
```

[ myPageckage/moduleB.py ]
```python
def functionB():
    print("FUNCTION_B")


print('MODULE_B : ', __name__)
```

[ PackageTest.py ]
```python
from myPackage import *

moduleA.functionA()
```

## class
```python
>>> class MyFirstClass:
...    clsVar = 'The best way to predict the future is to invent it.'
...
...    def clsMethod(self):
...        print(MyFirstClass.clsVar + '\t- Alan Curties Kay -')
...        
>>> mfc = MyFirstClass()
>>> mfc.clsVar
'The best way to predict the future is to invent it.'
>>> mfc.clsMethod()
The best way to predict the future is to invent it.	- Alan Curties Kay -
```

```python
class A:
    def methodA(self):
        print("Calling A's methodA")

    def method(self):
        print("Calling A's method")


class B:
    def methodB(self):
        print("Calling B's methodB")


class C(A, B):
    def methodC(self):
        print("Calling C's methodC")

    def method(self):
        print("Calling C's overridden method")
        super().method()


c = C()
c.methodA()
c.methodB()
c.methodC()
c.method()
```

```python
__init__() : 생성자
__del__() : 소멸자
__doc__ : 도크멘트
```

## Image 처리
```python
import requests
from PIL import Image
import hashlib
import matplotlib.pyplot as plt
import matplotlib.image as mpimg

# 이미지 파일 가져오기
url = 'http://bit.ly/2JnsHnT'
r = requests.get(url, stream=True).raw

# 이미지 보여주기
img = Image.open(r)
img.show()
img.save('src.png')

# 이미지 정보
print(img.get_format_mimetype)

# 파일 복사
BUF_SIZE = 1024
with open('src.png', 'rb') as sf, open('dst.png', 'wb') as df:
    while True:
        data = sf.read(BUF_SIZE)
        if not data:
            break
        df.write(data)

# 파일 복사 검증
sha_src = hashlib.sha256()
sha_dst = hashlib.sha256()

with open('src.png', 'rb') as sf, open('dst.png', 'rb') as df:
    sha_src.update(sf.read())
    sha_dst.update(df.read())

print("src.png's hash : {}".format(sha_src.hexdigest()))
print("dst.png's hash : {}".format(sha_dst.hexdigest()))

# matplotlib로 읽기
dst_img = mpimg.imread('dst.png')
print(dst_img)

pseudo_img = dst_img[:, :, 0]
print(pseudo_img)

# matplotlib로 이미지 가공
plt.suptitle('Image Processing', fontsize=18)
plt.subplot(1, 2, 1)
plt.title("Original Image")
plt.imshow(mpimg.imread('src.png'))

plt.subplot(1, 2, 2)
plt.title('Pseudocolor Image')
dst_img = mpimg.imread('dst.png')
pseudo_img = dst_img[:, :, 0]
plt.imshow(pseudo_img)
plt.show()
```
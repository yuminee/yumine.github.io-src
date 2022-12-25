---
title: [Python bisect_left, bisect_right] 
date: 2022-12-24 19:44:01
tags: [python3, bisect_left, bisect_right, bisect]
category: Python
mathjax: true
---

## python bisect_left, bisect_right의 기본 사용법 
```
from bisect import bisect_right, bisect_left

nums = [10, 12, 15, 16, 19]
print(bisect_left(nums, 15) #return 2
print(bisect_right(nums, 15)) #return 3
```

- bisect_left(a, x, lo=0, hi=len(a))
	- 정렬된 a에 x를 삽입할 위치를 리턴해준다.
	- x가 a에 이미 있는 값이라면 기존 값 왼쪽 위치를 리턴 한다.
	
- bisect_right(a, x, lo=0, hi=len(a))
	- 정렬된 a에 x를 삽입할 위치를 리턴해준다.
	- x가 a에 이미 있는 값이라면 기존 값 오른쪽 위치를 리턴 한다.

</br>

## python 3.11에서 추가된 내용
- bisect_left(a, x, lo=0, hi=len(a), *, key = None)
	- 정렬된 a에 x를 삽입할 위치를 리턴해준다.
	- x가 a에 이미 있는 값이라면 기존 값 왼쪽 위치를 리턴 한다.
	
- bisect_right(a, x, lo=0, hi=len(a), *, key = None)
	- 정렬된 a에 x를 삽입할 위치를 리턴해준다.
	- x가 a에 이미 있는 값이라면 기존 값 오른쪽 위치를 리턴 한다.

</br>

### [참고한 사이트]
- [bisect — Array bisection algorithm — Python 3.11.0 documentation](https://docs.python.org/3/library/bisect.html)

- [bisect — 배열 이진 분할 알고리즘 — 파이썬 설명서 주석판](https://python.flowdas.com/library/bisect.html)

- [Bisect Algorithm Functions in Python - GeeksforGeeks](https://www.geeksforgeeks.org/bisect-algorithm-functions-in-python/)
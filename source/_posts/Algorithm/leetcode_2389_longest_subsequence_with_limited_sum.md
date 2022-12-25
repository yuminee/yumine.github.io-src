---
title: [LeetCode 2389. Longest Subsequence With Limited Sum] 
date: 2022-12-25 19:44:01
tags: [LeetCode, Algorithm, bisect]
category: Algorithm
mathjax: true
---

## 문제
You are given an integer array nums of length n, and an integer array queries of length m.

Return an array answer of length m where answer[i] is the maximum size of a subsequence that you can take from nums such that the sum of its elements is less than or equal to queries[i].

A subsequence is an array that can be derived from another array by deleting some or no elements without changing the order of the remaining elements.

</br>

## 접근
1.  처음에는 python의 [itertools.combinations](https://docs.python.org/3/library/itertools.html#itertools.combinations)을 이용하여 subsequence를 구한뒤, 각각의 값을 sum을 하여 queries와 비교하려고 하였다. 
    ```
    subsequence = []
    for n in range(len(nums)+1):
        subsequence = subsequence+list(combinations(nums, n))
    ```
2. 코드를 작성하다보니, list를 sort하여 아래와 같이 nums를 만들게 되면 부분 집합 리스트의 길이는 각 nums의 index와 동일하고, 그 값은 최소값이 나오게 된다. 구해야하는것은 부분집합을 더해서 그 값이 queries보다 작거나 같을때, 가장 긴 부분집합의 길이를 구하는것이므로, 각 길이마다 나오는 최소값을 더하게 되면 문제의 조건에 부합하는 가장 긴 부분집합길이를 구할 수 있다.
    ```
    nums.sort()
    for i in range(1, len(nums)):
        nums[i] += nums[i - 1]
    ```
3. 실제로 값은 bisect.bisect_right을 통하여 구하였는데 이때의 리스트 nums의 값들은 부분 집합 길이가 n일때 그 부분집합들의 합의 최소값이므로 queries 원소인 query를 bisect_right로 넣었을때 나오는 index가 결국 정답이 된다. 
    ```
    answer = []

    for query in queries:
        index = bisect.bisect_right(nums, query)
        answer.append(index)
    ```
    
</br>

## 코드
```
from typing import List
import bisect

class Solution:
    def answerQueries(self, nums: List[int], queries: List[int]) -> List[int]:
        nums.sort()
        for i in range(1, len(nums)):
            nums[i] += nums[i - 1]

        answer = []

        for query in queries:
            index = bisect.bisect_right(nums, query)
            answer.append(index)
            
        return answer
```

</br>

### LINK
[Github: 2389. Longest Subsequence With Limited Sum](https://github.com/yuminee/algorithm/blob/master/LeetCode/2389_longest_subsequence_with_limited_sum.py)

[LeetCode: 2389. Longest Subsequence With Limited Sum](https://leetcode.com/problems/longest-subsequence-with-limited-sum/description/)

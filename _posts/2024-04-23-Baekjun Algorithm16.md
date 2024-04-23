---
title: "[백준] DP<4: LCS>"
date: 2024-04-23 19:40:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, dp, lcs]
image: /assets/baekjun.png
use_math: true
---
## 1. 최장 공통 부분 수열(LCS)

LCS(Longest Common Subsequence)는 두 수열이 주어졌을 때, 모두의 부분 수열이 되는 수열 중 가장 긴 것을 찾는 문제이다.

예를 들어, **ACA**Y**K**P 와 C**A**P**CAK**의 LCS는 ACAK가 된다.

## 2. DP를 활용한 LCS 알고리즘 풀이.

LCS 문제도 DP로 풀이 가능하다. 2차원 dp table을 준비해보자.

|  | 0 | A | C | A | Y | K | P |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| C | 0 |  |  |  |  |  |  |
| A | 0 |  |  |  |  |  |  |
| P | 0 |  |  |  |  |  |  |
| C | 0 |  |  |  |  |  |  |
| A | 0 |  |  |  |  |  |  |
| K | 0 |  |  |  |  |  |  |

계산을 위해 열과 행에 0으로 이루어진 margin을 한 줄씩 넣어준다.

이때 table의 각 숫자가 의미하는 값은 해당 인덱스까지의 문자열끼리 비교했을때, 가장 긴 LCS의 길이를 뜻한다.

예를 들어 table[3][3] 의 값은 “ACA” 인 문자열과 “CAP” 인 문자열의 LCS인 2가 들어와야 할 것이다. (”CA”가 LCS가 된다.)

그렇다면 점화식은 어떻게 구성해주어야 할까?

이 때는 2가지 경우를 나누어 준다.

1. 비교하는 인덱스의 문자가 다를 경우.
    
    비교하는 인덱스의 문자가 다르다면 해당 문자열 이전까지의 LCS를 그대로 이어주면 된다.
    
    `dp[i][j] = max(dp[i-1][j], dp[i][j-1])` 
    
2. 비교하는 인덱스의 문자가 같을 경우.
    
    비교하는 인덱스의 문자가 같다면 같은 문자열을 제외한 바로 앞까지의 문자들의 LCS에 +1을 해주면 된다.
    
    `dp[i][j] = dp[i-1][j-1] + 1`
    

이와 같은 점화식으로 table을 채우면 다음과 같아진다.

|  | 0 | A | C | A | Y | K | P |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| C | 0 | 0 | 1 | 1 | 1 | 1 | 1 |
| A | 0 | 1 | 1 | 2 | 2 | 2 | 2 |
| P | 0 | 1 | 1 | 2 | 2 | 2 | 3 |
| C | 0 | 1 | 2 | 2 | 2 | 2 | 3 |
| A | 0 | 1 | 2 | 3 | 3 | 3 | 3 |
| K | 0 | 1 | 2 | 3 | 3 | 4 | 4 |

이때 table의 가장 마지막 요소를 검사한다면 LCS의 길이를 알 수 있다.

### 2.1 LCS가 어떤 문자로 이루어져 있는지 찾는 방법.

완성된 dp table을 재귀를 통해 재탐색하면 LCS가 어떤 문자로 이루어진지 확인 가능하다.

LCS의 길이를 찾을때 사용한 점화식을 다시 생각해보자.

두 문자가 다르다면 왼쪽 혹은 위쪽과 값이 같다.

두 문자가 같다면 왼쪽 혹은 위쪽과 값이 같을 수가 없다.

즉 dp table의 가장 마지막 요소부터 시작해 왼쪽, 위쪽과 값이 모두 다르다면 해당 문자를 저장하면 된다.

이때 뒤에서부터 탐색하므로 출력하기 전에 뒤집어주면 된다.

## 3. 코드

[9251번 : LCS](https://www.acmicpc.net/problem/9251)

```python
import sys
read = sys.stdin.readline

string_1 = list(read().strip())
string_2 = list(read().strip())

dp = [[0 for _ in range(len(string_1)+1)] for _ in range(len(string_2)+1)]

for i in range(1, len(string_2)+1):
    for j in range(1, len(string_1)+1):
        if string_1[j-1] != string_2[i-1]:
            dp[i][j] = max(dp[i][j-1], dp[i-1][j])
        else: dp[i][j] = dp[i-1][j-1] + 1
        
print(dp[-1][-1])
```

[9252번 : LCS2](https://www.acmicpc.net/problem/9252)

```python
import sys
read = sys.stdin.readline
sys.setrecursionlimit(10**5)

string1 = [0] + list(read().strip())
string2 = [0] + list(read().strip())

dp = [[0] * len(string1) for _ in range(len(string2))]

lcs_string = []

def get_lcs(str1, str2):
    for i in range(1, len(str2)):
        for j in range(1, len(str1)):
            if str2[i] == str1[j]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

start_i, start_j = len(string2)-1, len(string1)-1

def find_string(dp, i, j):
    check1 = dp[i-1][j]
    check2 = dp[i][j-1]

    if dp[i][j] == 0: # 종료조건
        return

    if dp[i][j] == check1:
        find_string(dp, i-1, j)
    
    elif dp[i][j] == check2:
        find_string(dp, i, j-1)
    
    else:
        lcs_string.append(string2[i])
        find_string(dp, i-1, j-1)

get_lcs(string1, string2)
find_string(dp, start_i, start_j)
lcs_string.reverse()

length = dp[-1][-1]
print(length)
if length != 0:
    print(''.join(lcs_string))
```

앞의 코드와 알고리즘은 같지만, 문자를 역추적하는 과정이 포함됐다.
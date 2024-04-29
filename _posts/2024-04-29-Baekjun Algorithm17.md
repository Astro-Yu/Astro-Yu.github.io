---
title: "[백준] 분할정복"
date: 2024-04-29 19:21:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, divide&conquer]
image: /assets/baekjun.png
use_math: true
---
## 1. 분할 정복 알고리즘 소개

분할 정복이란 같은 모양의 거대한 문제를 잘게 잘라서 푸는 문제이다.

- 분할: 문제를 더 이상 나눌 수 없을 때까지 동일한 유형으로 나눈다.
- 정복: 가장 작은 단위의 문제를 해결한다.
- 조합: 해결한 문제들을 원래 문제로 합친다.

즉 해당 알고리즘은 프랙탈처럼 전체의 꼴이 부분의 꼴과 같을 시 유용하다.

## 2. 코드

### 2.1 색종이 만들기

[백준 2630번: 색종이 만들기](https://www.acmicpc.net/problem/2630)

분할 정복을 이해할 수 있는 대표적인 문제이다.

한 변의 길이가 2의 제곱수로 주어지고(2, 4, 8, …) 파란색과 흰색이 섞인 종이가 주어진다. 종이를 일정한 규칙에 따라 자르고, 종이의 모든 칸이 한 가지 색일 경우 더 이상 나누지 않는다. 그렇게 나누었을 때 파란색 종이의 숫자와 흰색 종이의 숫자를 센다. 즉 해당 알고리즘을 나누면 다음과 같다.

1. 종이를 확인해 한 가지 색으로 이루어져 있는지 확인한다.
2. 한 가지 색으로 이루어져 있으면 해당 색의 count를 늘린다.
3. 한 가지 색이 아니라면 종이를 4등분한다.
4. 4등분한 각각의 종이에 따라 1 ~ 3의 과정을 반복한다.

동일한 과정을 반복하므로 재귀함수를 쓰는 것이 적절하다.

```python
import sys
read = sys.stdin.readline

paperSize = int(read())

paper = [list(map(int,read().split())) for _ in range(paperSize)]

blueCount = 0
whiteCount = 0

# 종이가 전부 파란색으로 이루어졌다면 1,
# 종이가 전부 흰색으로 이루어져있다면 0을 리턴
def checkColor(paper,paperLength):
    total_sum = 0
    for idx in range(paperLength):
        total_sum += sum(paper[idx])
    
    return total_sum / paperLength**2
    
def divideAndConquer(paper, paperSize):
    global blueCount, whiteCount

    color = checkColor(paper, paperSize) # 파란색이면 1, 흰색이면 0

    if paperSize == 1: # 종료조건
        if color == 1:
            blueCount += 1
        elif color == 0:
            whiteCount += 1
        return

    if color == 1:
        blueCount += 1
    elif color == 0:
        whiteCount += 1
    else:
        paperSize //= 2
        # 종이를 4등분한다.
        top_left = [row[:paperSize] for row in paper[:paperSize]]
        top_right = [row[paperSize:] for row in paper[:paperSize]]
        bottom_left = [row[:paperSize] for row in paper[paperSize:]]
        bottom_right = [row[paperSize:] for row in paper[paperSize:]]

        divideAndConquer(top_left, paperSize)
        divideAndConquer(top_right, paperSize)
        divideAndConquer(bottom_left, paperSize)
        divideAndConquer(bottom_right, paperSize)

divideAndConquer(paper, paperSize)
print(whiteCount)
print(blueCount)
```

### 2.2 곱셈

[백준 1629번: 곱셈](https://www.acmicpc.net/problem/1629)

자연수 A를 B번 거듭제곱한 숫자를 C로 나눈 나머지를 출력하는 문제이다.

단순히 반복문으로 A를 N회 곱하는 코드를 만들었다면 $O(N)$의 시간복잡도가 걸리지만 지수법칙을 활용한 분할정복을 사용한다면 $O(logN)$에 풀이 가능하다.

예를 들어서 $A^8$를 구한다고 해보자. 

지수법칙을 활용하면 $A^8 = A^4 * A^4$ 가 된다. 

$A^4 = A^2 * A^2$ 가 된다.

$A^2 = A * A$ 가 된다.

지수가 홀수라면 어떻게 계산할까?

$A^7 = A^3 * A^3 * A$ 가 된다.

$A^9 = A^4 * A^4 * A$ 가 된다.

즉 거듭제곱 식을 점화식으로 바꿔 표현한다면 다음과 같다.

A^B = A^(B/2) * (B/2) (B가 짝수인 경우)

A^B = A^(B//2) * (B//2) * A (B가 홀수인 경우)

```python
import sys
read = sys.stdin.readline

A, B, C = map(int,read().split())

def fpow(c,n):
    if n == 1:
        return c % C
    else:
        if n % 2 == 0:
            x = fpow(c, n // 2)
            return x * x % C
        else:
            x = fpow(c, n // 2)
            return x * x * c % C
        
print(fpow(A,B))
```

계산 도중에도 충분히 숫자가 커질 수 있으므로 결과마다 C로 나눠주면 된다.
---
title: "[백준] 그리디 알고리즘 <2-1> "
date: 2024-06-14 18:42:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, heap, sort, greedy]
image: /assets/baekjun.png
use_math: true
---
풀었던 그리디 문제 중 개인적으로 어려웠던 문제들을 정리해보았다.

## 1. 책 나눠주기

[백준 9578번: 책 나눠주기](https://www.acmicpc.net/problem/9576)

N개의 책에 번호를 매긴 후, M명의 사람들에게 두 정수 a ~ b 인 신청서를 적어 내라고 지시한다. 이후 해당 신청서의 범위 내의 학생들에게 책을 분배한다. 이때 나누어 줄 수 있는 최대의 학생을 구한다.

### 1.1 발상

처음엔 **가장 범위가 작은 학생부터** 고른 후 나누어 줄 수 있는 책들 중 가장 나중의 책을 나누어주면 된다고 생각했다.

해당 발상은 다음과 같은 반례에 막혔다.

 

```
1
3 3
3 3
1 2
2 3
```

1 ~ 3 권의 책 중 3~3, 1~2, 2~3 의 신청서를 받는다면 3명이 모두 책을 받아갈 수 있다. 하지만 내 발상으로는 2명밖에 가져갈 수 없었다.

결국에는 책의 범위보다는 한 가지 기준으로 정렬한 후 나누어 주어야 한다.

1. 신청서 중 끝나는 번호가 작은 번호를 기준으로 정렬한다.
2. 해당 신청서로 나누어 줄 수 있는 책 중 가장 앞에 있는 책을 배정한다.

### 1.2 코드

```python
import sys
import heapq
read = sys.stdin.readline

tests = int(read())

for _ in range(tests):
    N, M = map(int,read().split())
    books = [-1]*(N+1) # 0번 책은 없는 책
    book_range = []
    count = 0
    for _ in range(M):
        a_i, b_i = map(int,read().split())
        heapq.heappush(book_range,(b_i, a_i))

    while book_range:
        b_i, a_i = heapq.heappop(book_range)

        for idx in range(a_i, b_i+1):
            if books[idx] == -1:
                books[idx] = 1
                count += 1
                break

    print(count)
```

정렬을 요구하므로 빠른 실행을 위해 heapq를 사용한다.

## 2. 컵라면

[백준 1781번: 컵라면](https://www.acmicpc.net/problem/1781)

문제가 N개 있고, 해당 문제마다 데드라인과 배정된 컵라면의 갯수가 있다. 데드라인을 넘기지 않고 얻을 수 있는 가장 많은 컵라면을 출력하는 문제이다.

### 2.1 발상

이전에 풀었던 [과제](https://astro-yu.github.io/posts/Baekjun-Algorithm20/#23-%EA%B3%BC%EC%A0%9C) 문제와 유사한 문제라고 생각했다.

기존 코드

```python
import sys
import heapq
read = sys.stdin.readline

N = int(read())

schedules = [True]*(N+1)

homeworks = []

for _ in range(N):
    due, cups = map(int,read().split())

    heapq.heappush(homeworks, (-cups, due))

total_cups = 0
while homeworks:
    minus_cups, due = heapq.heappop(homeworks)
    if schedules[due]: # 해당 날짜가 비어있는 경우
        schedules[due] = False
        total_cups -= minus_cups
    else: # 비어있지 않은 경우
        while due > 1:
            due -= 1 # 하나씩 줄여가며 배정
            if schedules[due]: # 해당 날짜가 비어있는 경우
                schedules[due] = False
                total_cups -= minus_cups
                break

print(total_cups)
```

컵라면이 가장 많이 배정된 문제를 가장 먼저 배치하는 식으로 문제를 풀었다.

이 코드는 시간초과를 받았다.

해당 코드의 최악의 경우를 생각해보자.

heapq 부분은 별 다른 연산이 없으니 $O(NlogN)$ 이 될 것이다.

`while` 문에서 모든 문제들이 due = 1 까지 돌아간다면 최악의 경우가 될 것이다.

즉 **이미 1일 ~해당 due 까지 문제가 배정되어 있어 배정이 불가능한 문제더라도 반드시 due = 1 까지 확인해보아야 하는게** 이 코드의 문제이다.

최악의 경우 $O(N^2)$ 까지 될 것 같다.

해결한 코드는 다음과 같다.

### 2.1 코드

```python
import sys
import heapq
read = sys.stdin.readline

N = int(read())

schedules = []
homeworks = []

for _ in range(N):
    due, cups = map(int,read().split())
    heapq.heappush(homeworks, (due, cups))

while homeworks:
    due, cups = heapq.heappop(homeworks)
    heapq.heappush(schedules, cups)

    if due < len(schedules):
        heapq.heappop(schedules) # 무조건 작은걸 뺌

print(sum(schedules))
```

1. 문제들을 컵라면이 아닌 일정 기준으로 정렬한 후 가장 빠른 일정부터 뽑는다.
2. 뽑은 문제를 새로운 heap인 `scheduels`에 컵라면 갯수를 넣는다.
3. 만약 일정보다 해당 큐가 길다면 (즉 데드라인을 벗어났다면) 가장 작은 컵라면을 제거한다.

해당 코드에선 while문 없이 heapq 만으로 구현했기 때문에 최악의 경우더라도 $O(NlogN)$을 보장한다.
---
title: "[백준] 우선순위 큐(Priority Queue)"
date: 2024-05-13 20:30:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, heap]
image: /assets/baekjun.png
use_math: true
---
## 1. 우선순위 큐

큐(Queue) 란 가장 먼저 들어온 데이터가 제일 먼저 빠지는 **FIFO**(First In First Out) 형식의 자료구조이다.

큐에서는 데이터를 pop 할때 가장 먼저 들어온 순서 기준으로 빠지게 되지만, **우선순위 큐**에서는 들어온 순서가 아닌, 가장 우선순위가 높은 데이터부터 빠지게 된다. 표로 정리하면 다음과 같다.

| 자료구조 | 빠지는 순서 |
| --- | --- |
| Stack | 가장 나중에 들어온 데이터 |
| Queue | 가장 먼저 들어온 데이터 |
| Priority Queue | 가장 우선순위가 높은 데이터 |

## 2. Python에서의 구현

파이썬에서는 `heapq` 모듈에 의해 최소 힙을 기본으로 제공하고 있다.

최소 힙에서는 크기가 가장 작은 원소부터 반환한다.

```python
import heapq

# heappush: 힙의 형태를 유지하면서 리스트에 원소를 추가한다.
heapq.heappush(list, element)

# heappop: 힙의 형태를 유지하면서 가장 작은 원소를 pop하고 반환한다.
heapq.heappop(list)

# heapify: 이미 존재하는 list를 heap으로 변환한다.
heaq.heapify(list)
```

### 2.1 최대 힙

약간의 트릭을 사용한다면 가장 큰 원소부터 정렬이 가능하다.

```python
import heapq

list = [1,2,3,4,5,6]
pq = []

# 기존 최소 힙
for num in list:
	heapq.heappush(pq, i)
	
# 트릭을 사용한 최대 힙
for num in list:
	heapq.heappush(pq, (-i, i))

# 큰 원소부터 출력
for _ in range(len(list)):
	current = heapq.heappop(pq)
	
	print(current[1])
```

튜플이 heapq에 들어오게 되면 튜플의 0번 인덱스를 기준으로 정렬하게 된다.

이 성질을 이용해 가장 -(음수)를 붙여 정렬시키면 가장 앞에 오는 원소가 가장 작은 값을 가지게 되고, 이때 이 값을 pop 한 후 1번 인덱스를 사용하면 된다.

## 3. 문제

### 3.1 순회강연

[백준 2109번: 순회강연](https://www.acmicpc.net/problem/2109)

그리디 알고리즘과 우선순위 큐를 결합한 문제이다.

1. 입력받은 페이(p)와 날짜(d)를 튜플로 변환한 후 최대힙으로 만들어준다.
2. 가장 페이가 높은 강연부터 pop한 후, 최대한 늦게 강연할 수 있는 날짜에 배정한다.
3. 만약 강연하려는 날에 이미 다른 강연이 예정되어있다면, 한 날짜씩 줄여 최대한 늦은 날짜로 배정한다.
4. 2 ~ 3의 과정을 반복한다.
5. 최대 힙에 원소가 모두 없어졌다면 반복을 중단한다.

```python
import sys
import heapq
read = sys.stdin.readline

N = int(read())
max_pq = []
schedules = [0 for _ in range(10001)]

# 최대힙
for _ in range(N):
    p, d = map(int,read().split())  
    heapq.heappush(max_pq,(-p ,d))

while max_pq:
    pay, due = heapq.heappop(max_pq)
    
    if schedules[due] == 0: # 비어있다면
        schedules[due] = -pay
    else: # 비어있지 않다면
        while True:
            due -= 1
            if due <= 0:
                break
            if schedules[due] == 0:
                schedules[due] = -pay
                break

    
print(sum(schedules))
```

[백준 13904번: 과제](https://www.acmicpc.net/problem/13904)

이 문제도 같은 알고리즘으로 풀이 가능하다.

---

### 3.2 가운데를 말해요

[백준 13904번: 가운데를 말해요](https://www.acmicpc.net/problem/1655)

숫자를 순서대로 입력받으면서 가운데 있는 숫자를 출력하는 문제이다.

입력받은 숫자가 홀수갯수일 때는 정 가운데, 짝수 갯수일 때는 가운데 두 가지 숫자 중 작은 숫자를 출력한다.

처음 해결 방법을 고안하기 쉽지 않은 문제였다. 최대힙과 최소힙 두 가지를 동시에 사용해야 한다.

중간값보다 작은 값들은 **최대힙**에, 중간값보다 큰 값들은 **최소힙**에 저장한다. 이렇게 번갈아가면서 저장하면 최대힙의 0번 인덱스는 중간값이 된다.

1. 최대힙, 최소힙을 준비한다.
2. 두 힙의 길이가 똑같을 경우 최대힙에 원소를 추가한다
3. 두 힙의 길이가 다를 경우 최소힙에 원소를 추가한다.
4. 2번과 3번 과정을 마친 후 최대힙의 0번 인덱스와 최소힙의 0번 인덱스를 비교한다.
5. 비교했을때 최대힙의 0번 인덱스가 더 작다면 두 원소를 각각의 힙에서 서로 교환한다.

```python
import sys
import heapq
read = sys.stdin.readline

N = int(read())
min_pq = []
max_pq = []

for i in range(N):
    num = int(read())
    if len(min_pq) == len(max_pq):
        heapq.heappush(max_pq, -num)
    else:
        heapq.heappush(min_pq, num)

    if min_pq and min_pq[0] < -max_pq[0]:
        max_value = heapq.heappop(max_pq)
        min_value = heapq.heappop(min_pq)

        heapq.heappush(max_pq, -min_value)
        heapq.heappush(min_pq, -max_value)
    
    print(-max_pq[0])
```
---
### 3.3 강의실

[백준 1374번: 강의실](https://www.acmicpc.net/problem/1374)

강의들의 진행 시간이 주어지고, 한 강의실에서 한 강의만 이루어진다고 했을 때, 최소로 필요한 강의실의 숫자를 구하는 문제이다.

즉 강의들을 나열했을때, 겹치는 시간대가 제일 많은 강의의 숫자를 찾는 문제가 된다.

이 문제는 먼저 주어진 데이터를 정렬한 후, 우선수위 큐에 넣고 빼는 과정이 수행되어야 한다.

1. 강의들을 시작하는 시간이 빠른 순서대로 정렬한다.
2. 가장 시작하는 시간이 빠른 강의의 종료시간을 우선순위 큐에 추가한다.
3. 반복문을 한번 돌면서 해당 가장 빠른 강의의 종료시간보다 현재 강의가 시작시간이 빠르다면 큐에 추가한다.
4. 가장 빠른 강의의 종료시간보다 현재 강의의 시작시간이 느리다면 큐에서 가장 빠른 종료시간을 빼준 후, 현재 강의의 종료시간을 추가한다.
5. 반복이 끝난 후 우선순위 큐의 길이를 측정한다.

```python
import sys
import heapq
read = sys.stdin.readline

N = int(read())

schedules = []
for _ in range(N):
    num, start, end = map(int,read().split())
    schedules.append((start, end))

schedules.sort()
pq = [schedules[0][1]] # 가장 먼저 시작하는 강의의 종료시간

for i in range(1, N):    
    if pq and pq[0] > schedules[i][0]:
        heapq.heappush(pq, schedules[i][1])
    else:
        heapq.heappop(pq)
        heapq.heappush(pq, schedules[i][1])

print(len(pq))
```

알고리즘 진행을 처음 봤을때 잘 이해가 가지 않았던 부분은 `else` 이하 부분이다.

`pq`의 길이의 의미가 겹치는 강의들의 갯수라고는 쉽게 예측할 수는 있다. 하지만 이 반복문이 진행될 때의 `pq`의 길이의 의미는 그 순서까지 진행됐을 때의 최대로 겹치는 강의의 갯수다.

즉 현재 강의가 가장 빠른 종료시간과 겹치지 않는다 하더라도 다음 강의와의 비교를 위해 현재까지의 가장 빠른 종료시간을 pop 해준 후, 현재 강의의 종료시간을 넣어주어야 한다. 실제로 어떤 강의가 겹치는지는 별 상관이 없다.(애초에 종료시간만 큐에 넣고 관리하기 때문에 알 수도 없다.)
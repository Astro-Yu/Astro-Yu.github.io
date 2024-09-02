---
title: "[백준] 위상정렬(Topological Sort)"
date: 2024-09-02 18:00:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, dp, topological sort, graph]
image: /assets/baekjun.png  
use_math: true
---  

## 1. 위상정렬에 대해

위상정렬은 방향 그래프에서 사용되는 알고리즘 중 하나이다.

주로 모든 노드의 방향이 한쪽으로 정렬된 경우에 사용 가능하고(순서가 존재해야 함), 그래프에 사이클이 존재한다면 사용 불가능하다.(방향이 없어지기 때문.)

실제 알고리즘에서는 A를 수행하기 위한 선행조건 B가 있는 경우에 주로 사용된다. 알고리즘 문제에 등장하는 몇 가지 예를 소개하면

1. 과목 A를 듣기 위해 필요한 선수과목 B,C… 가 있을 경우
2. 건물 A를 짓기 위해 필요한 선행건물 B,C… 가 있을 경우
3. 장난감 A를 완성하기 위한 부품 B,C… 가 있을 경우

등이 있다. 이때 간선에 weight가 존재해, 최종 목표에 도달하기 위해 걸리는 최소시간 등을 구하는데 사용된다.

위상정렬 알고리즘을 구현하기 위해 가장 대표적으로 사용되는 방법은 진입차수(In degree)를 사용하는 방식이다.

### 1.1 진입차수란?

방향 그래프에서 진입 차수의 의미는, 방향에 따라 **해당 노드로 들어오는 간선의 숫자**를 의미한다.

이때, 들어오는 간선의 의미는 **해당 노드에 들어가기 위해 선행되어야할 이전 노드들의 갯수**이다. 예를 들어보자.

ex) 과목 A를 듣기 위해 필요한 선수과목 B, C, D 가 있다면 과목 A의 진입차수는 3이다.

아래 코드 알고리즘을 보는 편이 이해가 빠를 것이다.

## 2. 알고리즘

이하는 위상정렬 개념 설명을 위한 알고리즘 중 일부이다.

```python
in_degree = [0]*(N+1) # 진입 차수 초기화
techs = [[] for _ in range(N+1)] # techs[i] = [j, k, ...] -> i 작업을 수행한 후, 이후에 진행 가능한 작업들 j,k,...

for _ in range(M):
    # X 과목을 듣기 위해 필요한 선수과목 Y
    X, Y = map(int, read().split())
    techs[Y].append(X)
    in_degree[X] += 1 # 선수과목이 추가될 때 마다 진입차수 1씩 증가
 
queue = deque() # 선수과목이 없는 기본과목을 위한 스택

for i in range(1, N+1):
    if in_degree[i] == 0: # 진입차수가 0 이라면 = 선수과목이 없다 = 기본과목이다.
        queue.append(i) # 스택에 추가

while queue:
    current = queue.popleft() # 하나씩 뽑아 사용하는 것으로 순서를 지킬 수 있음.
    
    for next in techs[current] # 다음 진행 가능성이 있는 과목들을 불러옴
        # 원하는 작업들 수행 ex) 건설에 걸리는 시간 계산, 수업들 듣기 위한 학기 계산 등
        in_degree[next] -= 1 # 한번 선수과목이 수행될 때 마다 다음과목의 진입차수를 1씩 제거
    
    if in_degree[next] == 0: # 진입차수가 0이 됐다면 = 선수과목을 모두 수행했다면
        queue.append(next) # 큐에 추가한 후 반복
    
```

핵심 알고리즘에 대해 주석을 달아 설명해두었다.

주고 이 코드에 dp를 활용하는 것이 일반적인듯 하다.

## 3. 문제

위상정렬 문제 중 풀었던 몇 가지 문제들을 모았다.

### 3.1 ACM Craft

[백준 1005번: ACM Craft](https://www.acmicpc.net/problem/1005)

```python
import sys
from collections import deque
read = sys.stdin.readline

# 목표: 모든 건물 건설에 걸리는 최소시간 구하기
# 선행 테크트리가 존재함
# 주어진 마지막 건물을 지으면 완성

T = int(read())

for _ in range(T):
    N, K = map(int, read().split())
    nexts = [[] for _ in range(N + 1)]  # 어떤 건물을 지었을 때, 어떤 다음 건물이 건설 가능한지
    in_degree = [0] * (N + 1)  # 진입 차수
    dp = [0] * (N + 1)
    costs = [-1] + list(map(int, read().split()))  # 건설 시간

    for _ in range(K):
        first, last = map(int, read().split())
        nexts[first].append(last)
        in_degree[last] += 1

    target = int(read())

    queue = deque()

    # 진입 차수가 0인 노드(건물)를 큐에 추가
    for i in range(1, N + 1):
        if in_degree[i] == 0:
            queue.append(i)
            dp[i] = costs[i]  # 초기 비용 설정

    # 위상 정렬
    while queue:
        current = queue.popleft()

        for next_building in nexts[current]:
            in_degree[next_building] -= 1  # 진입 차수 감소
            dp[next_building] = max(dp[next_building], dp[current] + costs[next_building])
            
            if in_degree[next_building] == 0:  # 진입 차수가 0이 되면 큐에 추가
                queue.append(next_building)

    print(dp[target])
```

기본적인 위상정렬 알고리즘에 dp를 활용해 건물의 완공 시간을 계산한다.

선행 건물 중 가장 마지막에 지어진 건물의 완공시간이 다음 건물에 영향을 주기 때문에 그 부분을 잘 고려해야 한다.

### 3.2 장난감 조립

[백준 2637번: 장난감 조립](https://www.acmicpc.net/problem/2637)

```python
import sys
from collections import deque
read = sys.stdin.readline

# 목표: 장난감의 완제품을 조립하기 위해 필요한 기본 부품의 종류별 갯수
# 기본 부품으로 중간 부품을 만들 수 있고, 중간 부품으로 완제품을 만들 수 있음.
# 선행 부품이 존재함 -> 위상정렬

N = int(read())
M = int(read())

in_degree = [0]*(N+1)
techs = [[] for _ in range(N+1)]
needed = [[0] * (N+1) for _ in range(N+1)] # needed[i][j] -> i 부품을 만들기 위해 필요한 j 부품의 양

for _ in range(M):
    # X, Y, K = 완제품이나 중부품 X를 만드는데 Y부품 K개가 필요함.
    X, Y, K = map(int, read().split())
    techs[Y].append((X, K))
    in_degree[X] += 1

queue = deque()
fundas = []
for i in range(1, N+1):
    if in_degree[i] == 0:
        queue.append(i)
        fundas.append(i)

while queue:
    current = queue.popleft()
    
    for next, next_count in techs[current]:
        if current in fundas: # current가 기본 부품인 경우
            needed[next][current] += next_count # 그대로 추가
        
        else:
            for i in range(1, N+1):
                needed[next][i] += needed[current][i] * next_count # 다음 부품에 추가
        
        in_degree[next] -= 1
        if in_degree[next] == 0:
            queue.append(next)

for i in fundas:
    print(i, needed[N][i])
```

위상정렬 알고리즘을 이용해 기본 부품의 사용 갯수를 구하는 문제이다.

2차원 리스트를 이용해 필요한 물건의 갯수를 한꺼번에 구하는 부분이 까다로웠다.
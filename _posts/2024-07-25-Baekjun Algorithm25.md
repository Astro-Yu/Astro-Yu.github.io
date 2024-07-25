---
title: "[백준] 다익스트라 <2-1> "
date: 2024-07-25 18:00:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, dijkstra]
image: /assets/baekjun.png  
use_math: true
---
풀었던 다익스트라 문제 중 개인적으로 어려웠던 문제들을 정리해보았다.   

## 1. 집 구하기

[백준 13911번: 집 구하기](https://www.acmicpc.net/problem/13911)

난이도 골드2 의 다익스트라 문제이다.

### 1.1 발상

처음엔 반복문을 통해 모든 도시에서 맥날까지의 거리, 스타벅스까지의 거리를 구해 두 개의 합 중 가장 작은 값을 구하려 했다.

하지만 이 방식대로면 시간초과가 나온다. 다익스트라 함수를 도시의 숫자만큼 시도하게 되고, 그 이후 결과 리스트에 대해서도 반복문을 시도하기 때문이다.

### 1.2 풀이

문제를 풀 수 있는 결정적 발상은 바로 가상 노드를 연결하는 것이다.

1. 모든 스타벅스가 있는 도시와 양방향 거리가 0인 가상 노드 1을 연결한다.
2. 모든 맥도날드가 있는 도시와 양방향 거리가 0인 가상 노드 2를 연결한다.

이렇게 거리 0인 가상노드를 연결해두면, 해당 위치들을 시작지점으로 삼을 수 있기에 다익스트라 2번이면 해결된다. 

이때 주의할 점은 가상노드와 연결된 길은 가짜 길이기 때문에 다익스트라로 최단거리를 구할 때 사용하면 안된다.

처음 보고 굉장히 놀라운 발상이라고 생각했다. 문제를 많이 풀어보면서 이런 발상에 대한 감을 익혀야겠다.

### 1.3 코드

```python
import sys
import heapq
import math
read = sys.stdin.readline

V, E = map(int, read().split())
graph = [[] for _ in range(V+3)]

for _ in range(E):
    u, v, w = map(int, read().split())
    graph[u].append((v, w))
    graph[v].append((u, w))

# M = 맥날의 수
# x = 맥세권 조건
M, x = map(int, read().split())
Mcs = list(map(int, read().split())) # 맥날 정점들
S, y = map(int, read().split())
Sbs = list(map(int, read().split())) # 스벅 정점들

# 맥날과 스벅이 있는 자리는 집이 있지 못함.
# 한 정점에 맥날과 스벅이 같이 있지 못함.
# 집이 있는 정점이 반드시 하나 존재함.

# 목표
# 맥날과 거리가 x 이하, 스벅과 거리가 y 이하인 집이면서, 두 가게와의 거리의 합이 최소인 집

# 모든 맥날과 이어져있는 더미노드 맥날더미 생성
# 모든 스벅과 이어져있는 더미노드 스벅더미 생성
# 이후 각 집까지의 거리를 구함. 이때 더미를 만들때 생성한 노드는 사용하면 안됨
mc_dummy = V+1
sb_dummy = V+2

for mc in Mcs: # 더미노드와 이어줌
    graph[mc_dummy].append((mc, 0))
    graph[mc].append((mc_dummy, 0))

for sb in Sbs: # 더미노드와 이어줌
    graph[sb_dummy].append((sb, 0))
    graph[sb].append((sb_dummy, 0))

def dijkstra(start: int):
    dists = [math.inf for _ in range(V+3)]
    dists[start] = 0
    pq = [(0, start)]

    while pq:
        current_weight, current_node = heapq.heappop(pq)

        if dists[current_node] < current_weight:
            continue

        for next in graph[current_node]:
            next_node, next_weight = next

            if next_node == V+1 or next_node == V+2: # 다음 노드가 가상노드로 이어져 있다면 패스
                continue

            new_weight = current_weight + next_weight

            if dists[next_node] > new_weight:
                dists[next_node] = new_weight
                heapq.heappush(pq,(new_weight, next_node))

    return dists

mc_dists = dijkstra(V+1)
sb_dists = dijkstra(V+2)

min_dist = math.inf

for town in range(1, V+1):
    if town in Sbs+Mcs: # 마을에 맥이나 스벅이 있으면 패스
        continue

    if mc_dists[town] <= x and sb_dists[town] <= y:
        min_dist = min(min_dist, mc_dists[town]+sb_dists[town])

if min_dist > 1e9:
    print(-1)
else:
    print(min_dist)
```

## 2. 인터넷 설치

[백준 1800번: 인터넷 설치](https://www.acmicpc.net/problem/1800)

난이도 골드1의 다익스트라 문제이다.

### 2.1 발상

까다로운 조건이 많아서 어려운 문제였다.

인터넷 선을 연결해야 하는데 N번 컴퓨터가 인터넷에 연결되야 하는게 목적이다.

이때 K개의 선은 무료로 연결해주고, 남은 선들의 가격 중에 가장 비싼 선에 대해서만 가격을 받는다.

이때 최소의 가격으로 N번 컴퓨터와 연결하는 경우를 찾아야 한다.

솔직하게 말하면 문제를 시도하지도 못했다. 어떻게 K+1 번째가 가장 싼 선인 경우를 걸러내야 할 지 몰랐기 때문이다.

### 2.2 풀이

해답은 이분탐색을 통해 값을 탐색해나가며 해당 값보다 비싼 선의 갯수를 세야했다.

limit 값이 다익스트라 함수에 주어지고, 탐색에 활용된 간선의 값이 limit보다 크다면 1을 추가한다.

탐색에 활용된 간선의 값이 limit 보다 작다면 추가하지 않는다.

이렇게 계산해나가면서 마지막에 dists[N] 값이 K보다 크다면 limit 값 이하로 경로를 구성하기 불가능하다는 것이므로(비싼 선이 K개보다 많다는 뜻) False를 리턴해 이분탐색에서 오른쪽을 탐색하게 한다.

그 이외라면 True를 내보내 이분탐색에서 왼쪽을 탐색하게 한다.

### 2.3 코드

```python
import sys
import heapq
import math
read = sys.stdin.readline

# 목표: N번 컴퓨터를 연결하는데까지 소모되는 최소 비용
# K개의 선은 공짜
# 나머지 인터넷 선 중 가장 비싼 선만 가격을 받음
# 마지막 N에 limit인 가격 초과의 선들이 몇개 있는지 확인

N, P, K = map(int,read().split())
graph = [[] for _ in range(N+1)]

for _ in range(P):
    a, b, cost = map(int,read().split())
    graph[a].append((b, cost))
    graph[b].append((a, cost))

def dijkstra(start, limit):
    pq = [(0, start)]
    dists = [math.inf] * (N+1)
    dists[start] = 0 # 길이

    while pq:
        cost, idx = heapq.heappop(pq)
        if dists[idx] < cost:
            continue

        for next in graph[idx]:
            next_idx, next_cost = next
            if next_cost > limit: # 다음번 선을 잇는 가격이 제한 가격보다 크다면
                if cost + 1 < dists[next_idx]:
                    dists[next_idx] = cost + 1
                    heapq.heappush(pq, (cost + 1, next_idx))
            else:
                if cost < dists[next_idx]:
                    dists[next_idx] = cost
                    heapq.heappush(pq, (cost, next_idx))

    if dists[N] > K: # 타겟에 limit 가격 이상의 N개 초과하는 선이 있을 경우
        return False
    else:
        return True

left, right = 0, 1000001
answer = math.inf

while left <= right:
    mid = (right + left) // 2
    is_exist = dijkstra(1, mid)

    if is_exist:
        right = mid - 1
        answer = mid
    else:
        left = mid + 1

if answer > 1e10:
    print(-1)
else:
    print(answer)
```

*~~(솔직히 이 문제는 골드1 보다 어려운듯)~~*
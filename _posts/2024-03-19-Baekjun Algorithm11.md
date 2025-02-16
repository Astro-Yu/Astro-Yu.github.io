---
title: "[백준] 최단거리 알고리즘 <3 : 플로이드 - 워셜>"
date: 2024-03-19 20:00:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, floydwarshall, graph]
image: /assets/baekjun.png
use_math: true
---
## 0. 이전 글

[최단거리 알고리즘 <1 : 다익스트라 >](https://astro-yu.github.io/posts/Baekjun-Algorithm6/)

[최단거리 알고리즘 <2 : 벨만 - 포드 >](https://astro-yu.github.io/posts/Baekjun-Algorithm10/)

같은 최단거리를 찾는 알고리즘들을 미리 보고온다면 차이점 비교가 쉽다

## 1. 플로이드 - 워셜 알고리즘의 소개

플로이드 - 워셜(**Floyd-Warshall Algorithm**) 은 최단거리를 찾는 알고리즘 중 하나이다.

플로이드 - 워셜 알고리즘은 2차원 최단거리 배열을 만들어 문제를 해결하려고 시도한다. 각 노드에서 갈 수 있는 노드까지 거리를 배열에 입력한 후, 각 노드들을 **경유**하는 방식으로 최단 거리를 찾아낸다.

3중 반복문을 사용하기 때문에 같은 구조라면 다익스트라 알고리즘보다 느리다.

하지만 **음수의 간선**이 있어도 사용 가능하다는 장점이 있다.

## 2. 플로이드 - 워셜 알고리즘의 매커니즘

경유한다는 말이 무엇일까? 

예를 들어 5개의 노드 중 1에서 출발해 5에 도착하는 최단 거리를 찾는다고 해보자.

노드 1과 노드 5 사이의 최단 거리는 반드시 노드 1과 노드 x 의 최단거리, 노드 x와 노드 5의 최단 거리의 합이 될 것이라는게 이 알고리즘의 핵심 발상이다.

출발 노드와 도착 노드는 이미 알고 있을것이기 때문에 중간에 **경유하는 노드 x를** 전체 노드 중에서 찾는 알고리즘이다.

### 2.1 알고리즘 진행 방식

백준의 11404 번 문제를 예로 들어 설명해보자.(링크는 아래 코드에 있다.)

1. 2차원 배열 `graph` 를  준비해 `inf` 값으로 초기화한다. 열과 행 에는 노드들이 들어가고, 각 요소들은 해당 열과 행을 노드로 갖는 가중치를 입력한다.
    
    아래 그래프에선 행이 출발 도시, 열이 도착 도시이다. 최초 그래프는 다음과 같다.
    
    |  | 1 | 2 | 3 | 4 | 5 |
    | --- | --- | --- | --- | --- | --- |
    | 1 | 0 | 2 | 3 | 1 | 10 |
    | 2 | inf | 0 | inf | 2 | inf |
    | 3 | 8 | inf | 0 | 1 | 1 |
    | 4 | inf | inf | inf | 0 | 3 |
    | 5 | 7 | 4 | inf | inf | 0 |
2. 3중 반복문을 통해 각 노드들이 경유지(via)일 경우, 출발 도착에 대해 최소값이 갱신될 경우 `graph`를 갱신해준다.
    
    즉 `graph[start][end]` > `graph[start][via]` + `graph[via][end]`가 된다면 갱신한다.
    
    하나의 예를 들어보자. 경유지가 1인 경우 5에서 출발해 4로 간다고 해보자. 
    
    graph[5][4]의 값은 현재 `inf` 이다. 가는 길이 없기 때문이다. 하지만 `graph[5][1] = 7`, `graph[1][4] = 1` 이다. 즉 이 합인 `8`보다 `inf`이 크므로 해당 칸을 갱신 가능하다.
    

1. 해당 과정을 모든 노드에 대해 해준다. 이 경우의 경우 1 ~ 5 번 노드를 경유지로 하여 진행한다.

1. 최종 결과에서 중간에 거쳐갈 수 있는 길이 없다면 `inf`로 남고, 있다면 해당 값을 출력해주면 된다.

매커니즘에서 3중 반복문을 사용하기 때문에 시간 복잡도는 $O(N^3)$ 이다.

앞서 소개한 다익스트라나 벨만 포드보다 시간 복잡도는 크지만, 모든 길에 대해 확실하게 찾아낼 수 있고, 코드가 간결하다는 장점이 있다.

## 3. 코드

[11404: 플로이드](https://www.acmicpc.net/problem/11404)

```python
import sys
import math

read = sys.stdin.readline
node_count = int(read().strip())
edge_count = int(read().strip())

graph = [[math.inf] * (node_count) for _ in range(node_count)]

for i in range(node_count):
    for j in range(node_count):
        if i == j:
            graph[i][j] = 0

for _ in range(edge_count):
    start, end, weight = map(int,read().split())
    if graph[start-1][end-1] > weight:
        graph[start-1][end-1] = weight
    

# 중간에 m을 경유하는 방법 고려
for via in range(node_count): # 경유
    for start in range(node_count): # 출발
        for end in range(node_count): # 도착
            via_value = graph[start][via] + graph[via][end]
            if graph[start][end] > via_value: # 기존 결과보다 경유해 도착한 결과가 작다면 갱신
                graph[start][end] = via_value # 갱신

for i in range(node_count):
    for j in range(node_count):
        if graph[i][j] > 1e9: # 갈 수 없는 경로가 있는 경우
            graph[i][j] = 0
    
for idx in range(node_count):
    print(*graph[idx])
```
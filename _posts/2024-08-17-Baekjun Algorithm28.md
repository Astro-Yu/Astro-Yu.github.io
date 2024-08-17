---
title: "[백준] 최소 신장 트리2"
date: 2024-08-17 20:11:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, kruskal, prim, greedy]
image: /assets/baekjun.png  
use_math: true
---  

풀어본 최소 스패닝 트리 문제 중 어려웠던 문제를 정리했다.    
MST에 대한 개념은 아래 링크에서 정리했다.   
[최소 신장 트리](https://astro-yu.github.io/posts/Baekjun-Algorithm27/)

## 1. 세부

[백준 13905번: 세부](https://www.acmicpc.net/problem/13905)

난이도 골드 4의 최소 스패닝 트리 문제이다. 특정 발상을 요구했기에 체감은 골드 4보다 어려웠다.

### 1.1 발상

출발하는 섬과 도착하는 섬이 지정되어 있고, 두 섬을 잇는 경로들 가운데 경로의 최소값이 가장 큰 값을 알아내는 문제이다. 예를 들어보자.

출발과 도착까지의 경로의 다리 무게 제한이 다음과 같다고 해보자.

1. 2 → 5 → 2
2. 4 → 4 → 3
3. 4 → 5 → 2
4. 4 → 1

1번 경로의 최소값은 2이다. 2번 경로의 최소값은 3이다. 3번 경로의 최소값은 2이다. 4번 경로의 최소값은 1이다. 이때 각각의 경로들의 최소값 중 가장 큰 값은 3 이므로 2번 경로의 3이 정답이다.

처음엔 **각각의 경로를 알아내야 하기 때문에 DFS**를 생각했다. 하나의 경로를 DFS로 탐색 후 가장 작은 값들을 모아 그 중 큰 값을 구하려고 했지만, 섬의 수와 다리의 수가 너무 많았기 때문에 시간복잡도 측면을 고려해 포기했다.

### 1.2 풀이

그렇다면 수많은 경우의 수를 줄이기 위해 선행되는 과정이 필요할 것이다. 그것이 최소 스패닝 트리였다. 풀이과정은 다음과 같다.

1. 최소 스패닝 트리를 거꾸로 시도해 다리들 중 가장 큰 값들만 남긴다.
2. 루트에 사이클이나, 하중이 작은 경로가 포함되면 안되므로 위 논리가 가능하다.
3. 남은 다리들 중에서 BFS를 통해 루트들 중 이전에 지났던 다리보다 작은 값들만 이동 가능하므로, 작은 다리들을 선택해 이동한다.
4. MST를 통해 사이클의 가능성이 없어졌으므로, 마지막에 도달한다면 루트의 최소값을 반환한다.

### 1.3 코드

문제에 설명은 나와있지 않지만, 마지막 섬에 도달할 수 없는 경우 0을 출력한다.

```python
import sys
import math
sys.setrecursionlimit(10**5)
from collections import deque
read = sys.stdin.readline

# 숭이 -> 혜빈
# 출발 -> 도착 의 루트 중 가장 최소값이 큰 루트의 값을 알아내야함.
# 최소(최대) 스패닝 트리로 도시들을 최대값으로 연결
# 이후 연결된 다리 중에서 시작과 끝을 잇는 루트를 찾아 최소값을 찾음.

N, M = map(int, read().split())
s, e = map(int, read().split())
uf = [i for i in range(N+1)]
routes = [[] for _ in range(N+1)]
visited = [False] * (N+1)
bridges = []

def find(x):
    if x != uf[x]:
        uf[x] = find(uf[x])

    return uf[x]

def union(a, b):
    a = find(a)
    b = find(b)

    if a < b:
        uf[b] = a
    
    else:
        uf[a] = b

for _ in range(M):
    a, b, weight = map(int, read().split())
    bridges.append((weight, a, b))

bridges.sort(reverse=True)

for i in range(M):
    weight, node1, node2 = bridges[i]
    if find(node1) != find(node2):
        union(node1, node2)
        routes[node1].append((node2, weight))
        routes[node2].append((node1, weight))

stack = deque([(s, math.inf)])
visited[s] = True

while stack:
    current, min_value = stack.popleft()
    if current == e:
        print(min_value)
        exit()
        
    for next, next_weight in routes[current]:
        if not visited[next]:
            min_value = min(min_value, next_weight)
            stack.append((next, min_value))
            visited[next] = True

print(0)
```

## 2. 복제로봇

[백준 1944번: 복제로봇](https://www.acmicpc.net/problem/1944)

난이도 골드1의 최소 스패닝 트리 문제이다. 문제를 단순화 하는 과정이 필요하다. 개인적으로 골드1에 알맞은 문제라고 느꼈다. 문제의 핵심을 보고 단순화하는 머리를 가져야겠다.

### 2.1 발상

주어진 미로에서 출발점과 각각의 목표들(열쇠)가 주어져 있고, 모든 열쇠가 있는 장소를 탐색해야 하는 문제이다. 탐색하면서 모든 로봇이 움직인 거리를 계산한다. 이때 로봇이 최소로 움직이는 횟수를 구한다.

문제의 설명에서 “복제” 라는 키워드 때문에 혼동이 올 수 있는데, 어차피 모든 로봇의 움직임의 합을 구해야 하기 때문에 큰 의미는 없다고 판단했다.

최초엔 로봇의 객체를 열쇠와 시작 지점에 두고, 해당 지점에서 남은 열쇠들 중 가장 가까운 열쇠들을 BFS로 탐색하려 했다. 하지만 객체를 반복문으로 탐색하고 다시 큐에 집어넣는 과정이 부적절해 다시 생각해보았다.

### 2.2 풀이

문제를 단순하게 풀었어야 했다. 

1. 그래프를 단순화해 출발지점과 모든 열쇠의 지점들 간의 거리 그래프를 만든다.
2. MST를 통해 최소가 되는 간선들을 더한다.

위에서 말했듯이, 어느 로봇이 어디에서 복제되는것이 중요한게 아니라, 실제 움직이는 거리가 중요하기 때문에 각 열쇠들간의 거리를 구해 MST로 푸는 것이 핵심이다.

추가로 크루스칼 알고리즘을 사용하기 때문에 각각 열쇠와 출발지점에 특정 id를 부여해 딕셔너리로 관리했다.

### 2.3 코드

```python
import sys
from collections import deque
read = sys.stdin.readline

# 목표: 로봇의 움직임을 최소로 하면서 미로에서 모든 열쇠 찾기
# 복제된 로봇의 움직임도 움직인것으로 간주
# 하나의 칸에 복수의 로봇 존재 가능.
# 복제장치: 출발장소, 열쇠가 있는 장소에 존재

# 1: 미로의 벽, 0: 지날 수 있는 길, S: 출발, K: 열쇠

# 모든 S와 K를 찾고, BFS로 거리잰 후 커넥션화 -> 이후 최소스패닝

N, M = map(int, read().split())

keys = []
key_dicts = {}
graph = []
dist_infos = []
key_addr = 0

for i in range(N):
    line = list(read().strip())
    graph.append(line)
    for j in range(N):
        if line[j] == "S" or line[j] == "K":
            keys.append([i,j])
            key_dicts[f"{i,j}"] = key_addr
            key_addr += 1

def get_next_posi(x, y):
    up = [x-1, y]
    down = [x+1, y]
    left = [x, y-1]
    right = [x, y+1]

    return [up, down, left, right]

def is_wall(x, y):
    return graph[x][y] == "1"

def bfs(start, others):
    queue = deque([start])
    s_x, s_y = start
    visited = [[False] * N for _ in range(N)]
    dists = [[0]* N for _ in range(N)]
    visited[s_x][s_y] = True
    while queue:
        current = queue.popleft()
        c_x, c_y = current

        for next_posi in get_next_posi(c_x, c_y):
            n_x, n_y = next_posi
            
            if is_wall(n_x, n_y):
                continue
            if not visited[n_x][n_y]:
                queue.append(next_posi)
                dists[n_x][n_y] = dists[c_x][c_y] + 1
                visited[n_x][n_y] = True
                if next_posi in others:
                    start_key = key_dicts[f"{s_x,s_y}"]
                    end_key = key_dicts[f"{n_x,n_y}"]
                    dist = dists[n_x][n_y]
                    dist_infos.append((dist, start_key, end_key))

for i in range(M+1):
    start = keys[i]
    others = keys[i+1:]
    bfs(start, others)

uf = [i for i in range(M+1)]

def find(x):
    if x != uf[x]:
        uf[x] = find(uf[x])
    return uf[x]

def union(a, b):
    a = find(a)
    b = find(b)

    if a < b:
        uf[b] = a
    else:
        uf[a] = b

dist_infos.sort()
total_dist = 0
count = 0
for dist_info in dist_infos:
    dist, start, end = dist_info
    
    if find(start) != find(end):
        union(start, end)
        count += 1
        total_dist += dist

if count == M:
    print(total_dist)
else:
    print(-1)
```

## 3. 전기가 부족해

[백준 10423번: 전기가 부족해](https://www.acmicpc.net/problem/10423)

난이도 골드3의 최소 스패닝 문제이다. 간단한 응용이 필요했다. 아직 이런 응용에 익숙하지 않아서 많은 MST 문제를 접해봐야겠다고 느꼈다.

### 3.1 발상

문제의 핵심은 모든 도시를 연결하되, 하나의 도시가 두 개 이상의 발전소와 연결되면 안 되는 것이었다.

처음엔 이 문제를 해결하기 위해 모든 도시를 MST를 사용해 연결 후, 연결된 케이블 중 가장 큰 값들 (발전소의 수 -1) 개를 빼려고 했다. 러프한 생각으로 이렇게 하면 자연스레 연결 될 줄 알았다.

하지만 반례가 있어서 다시 생각해보았다.

### 3.2 풀이

생각해보면 도시가 어느 발전소에서 전력을 공급받는지는 중요하지 않았다. 그저 단 한개의 발전소에서만 전력을 공급받아야 했다.

그렇기에 문제를 단순화해 모든 발전소를 같은 부모(0) 값으로 만들어 0에 연결된 도시, 연결되지 않은 도시들로 단순화 할 수 있었다.

이때 주의점은 union-find 테이블에서 발전소의 위치를 인덱스에서 가장 작은 0으로 해야 나머지 노드들이 최상위 부모로 합쳐진다.

### 3.3 코드

```python
import sys
import heapq
read = sys.stdin.readline

# 목표: 케이블 건설비용을 최소한으로 하기
# 이미 특정 도시에 발전소가 건설되어 있음.
# 주의점: 하나의 도시는 반드시 하나의 발전소에서만 전기를 받아야 낭비가 안됨.
# 모든 도시가 연결될 필요는 없음 
# 발전소의 위치를 0으로 하고, 이미 연결된 발전소, 아닌 발전소로 나누어서 진행.

N, M, K = map(int, read().split())
uf = [i for i in range(N+1)]
cables = []
powers = list(map(int, read().split()))

for power in powers:
    uf[power] = 0 # 발전소를 반드시 uf에서 부모로 만들기 위해 0번 처리

for _ in range(M):
    u, v, w = map(int, read().split())
    cables.append((w, u, v))

cables.sort()

def find(x):
    if x != uf[x]:
        uf[x] = find(uf[x])
    return uf[x]

def union(a, b):
    a = find(a)
    b = find(b)

    if a < b:
        uf[b] = a
    else:
        uf[a] = b

total_cost = 0

for i in range(M):
    cost, city1, city2 = cables[i]
    if find(city1) != find(city2):
        union(city1, city2)
        total_cost += cost

print(total_cost)
```
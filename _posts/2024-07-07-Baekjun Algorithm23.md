---
title: "[백준] DFS <2-1> "
date: 2024-07-07 17:40:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, dfs, bfs, dp, memoization]
image: /assets/baekjun.png
use_math: true
---
풀었던 DFS 문제 중 개인적으로 어려웠던 문제들을 정리해보았다.   

## 1. 문자판

[백준 2186번: 문자판](https://www.acmicpc.net/problem/2186)

난이도 골드3의 문제이다.

한 칸에 알파벳이 하나씩 쓰여진 보드가 주어졌을 때, 주어진 단어를 완성시킬 수 있는 경로의 수를 출력하는 문제이다.

### 1.1 발상

탐색이 가능한 범위 K가 주어져 있으므로, 해당 범위 내에 원하는 알파벳이 존재하는지 확인하면서 진행해야 한다.

정답으로 주어진 단어를 리스트화 해서, 한번 나아갈 때 마다 idx를 하나씩 증가시켜 원하는 알파벳을 찾았다.

최초에는 한번씩 나아간다는 발상 때문에 bfs로 풀이했지만 시간 초과를 받았다. (이마저도 틀린 코드였다.)

해법은 메모이제이션과 3차원 dp에 있었다.

3차원 dp는 다음과 같이 구성된다.

```python
dp = list(list(list(-1 for _ in range(M)) for _ in range(N)) for _ in range(len(word)))

```

즉 한 평면은 보드와 같으며, 깊이는 주어진 단어의 길이이다.

dp의 각 요소들은 해당 깊이(idx)에서 해당 문자(x,y)를 지나는 정답의 갯수를 뜻한다.

문제의 예시를 들어 설명해보자.

|0|0|0|0
| --- | --- | --- | --- |
| K | A | K | T |
| X | E | A | S |
| Y | R | W | U |
| Z | B | Q | P |

가 주어졌고, 우리가 찾을 단어는 BREAK 이다.

BREAK 이기 때문에 우리는 B에서 부터 시작해야 한다.

하나의 루트를 선정해보면 B(3,1) → R(2,1) → E(1,1) → A(0,1) → K(0,2)가 있다.

이때 dfs를 활용해 마지막 깊이에서 마지막에 해당하는 문자를 만났다면 해당 칸을 1로 만들어준다. 즉

dp[3][0][2] = 1 이 된다.

dfs를 활용했으므로 백트래킹으로 그 전까지의 루트는 모두 1이 된다.

이후 A(0,1) → K(0,0) 도 가능하다. 그러므로 dp[3][0][0] = 1 이 된다.

이때 이미 A(0,1) → dp[2][0][1] = 1 이므로 해당 경우에서 하나의 루트가 더 생겼기 때문에 2가 된다.

이런 식으로 dfs를 활용한 백트래킹을 사용하면 시작점 B에 모든 경우의 수가 모이게 된다.

주의점: 

- 같은 경로를 여러번 방문 가능하다.
- 시작 지점이 여러 곳일 수 있다.

### 1.2 코드

```python
import sys
sys.setrecursionlimit(10**5)
read = sys.stdin.readline

N, M, K = map(int,read().split()) # K = 상하좌우 움직일 수 있는 칸의 갯수
board = list(list(read().strip()) for _ in range(N))
word = list(read().strip())
start_str = word[0] # 시작 알파벳

dp = list(list(list(-1 for _ in range(M)) for _ in range(N)) for _ in range(len(word)))

def out_of_box(x, y):
    if x < 0 or x >= N or y < 0 or y >= M:
        return True
    return False

def get_next_posi(x, y, K):
    candi = []
    for dx in range(-K, K+1): # -2, -1, 1 ,2
        new_x = x + dx
        if dx == 0:
            continue
        candi.append([new_x, y])

    for dy in range(-K, K+1): # -2, -1, 1 ,2
        new_y = y + dy
        if dy == 0:
            continue
        candi.append([x, new_y])

    return candi

def get_str(x, y):
    return board[x][y]

def dfs(current: list, idx: int):
    c_x, c_y = current

    if dp[idx][c_x][c_y] != -1: # 메모이제이션을 통해 계산의 수를 줄인다.
        return dp[idx][c_x][c_y]

    if idx == len(word)-1:
        return 1
    
    cnt = 0
    for next in get_next_posi(c_x, c_y, K):
        n_x, n_y = next
        if out_of_box(n_x, n_y):
            continue

        if get_str(n_x, n_y) == word[idx+1]:
            cnt += dfs(next, idx+1)
    dp[idx][c_x][c_y] = cnt # postorder를 통해 경우의 수를 기록한다.

    return cnt

total_count = 0
for i in range(N):
    for j in range(M):
        if board[i][j] == start_str: # 시작 알파벳이 여러개일 수 있기 때문에 반복문을 통해 계산한다.
            total_count += dfs([i,j], 0)

print(total_count)
```

## 2.  서울 지하철 2호선

[서울 지하철 2호선](https://www.acmicpc.net/problem/16947)

난이도 골드3의 문제이다.

정수(1 ~ N)으로 되어있는 지하철 역들이 주어지고, 해당 역들 간의 노선이 주어진다. 

이때 주어진 역과 순환선 사이의 거리를 구하는 문제이다.

만약 주어진 역이 순환선 고리 안에 포함되어 있다면 거리는 0 이다. 순환선으로부터 1역당 1의 거리를 가지고 있다. 즉 지선의 첫 번째 역의 거리는 1이다.

### 2.1 발상

1. dfs를 통해 어떤 역이 순환선 안에 있는지 파악한다.
    1. 이때 순환선 고리가 되려면 최소 3개의 역이 있어야 한다. 이것에 주의
    2. dfs를 돌면서 시작지점과 다음 역이 같고, 최소 3개의 역을 지났다면 순환선에 포함
2. bfs를 통해 역 간의 거리를 계산한다.
    1. 만약 순환선 내에 있다면 큐에 넣고 거리를 0으로 계산.
    2. 큐를 진행하면서 다음이 -1 이라면(즉 순환선 내에 없고 방문하지 않았다면) 다음을 큐에 넣고, 거리를 현재거리 + 1 해줌

### 2.2 코드

```python
import sys
from collections import deque
sys.setrecursionlimit(10**5)
read = sys.stdin.readline
N = int(read())
graph = [[] for _ in range(N+1)]

for _ in range(N):
    a, b = map(int,read().split())
    graph[a].append(b)
    graph[b].append(a)

def dfs(start: int, node: int, count: int):
    global is_cycle
    if start == node and count >= 2:
        is_cycle = True
        return
    visited[node] = True

    for next_node in graph[node]:
        if not visited[next_node]:
            dfs(start, next_node, count+1)
        elif next_node == start and count >= 2:
            dfs(start, next_node, count)

def bfs():
    dists = [-1]*(N+1)
    queue = deque()
    for i in range(1, N+1):
        if cycle_nodes[i]:
            dists[i] = 0
            queue.append(i)
    
    while queue:
        current = queue.popleft()
        for next in graph[current]:
            if dists[next] == -1:
                queue.append(next)
                dists[next] = dists[current] + 1
    dists.pop(0)
    return dists

cycle_nodes = [False] * (N+1)
for i in range(1, N+1):
    is_cycle = False
    visited = [False] * (N+1)

    dfs(i, i, 0)

    if is_cycle:
        cycle_nodes[i] = True

answer = bfs()

print(*answer)
```

순환선을 계산하기 위해 여러 방법을 생각해봤지만, 가장 간단하게 생각난 방법으로 구현했다. 이후 순환선으로부터 지선의 거리를 구하는 발상에 BFS를 사용하는데 시간이 오래 걸렸다.
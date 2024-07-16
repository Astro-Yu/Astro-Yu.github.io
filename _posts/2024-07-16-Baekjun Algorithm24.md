---
title: "[백준] DFS <2-2> "
date: 2024-07-16 19:50:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, dfs, bfs, dp, memoization]
image: /assets/baekjun.png  
use_math: true
---
풀었던 DFS 문제 중 개인적으로 어려웠던 문제들을 정리해보았다.   

## 3. 피리 부는 사나이

[백준 16724번: 피리 부는 사나이](https://www.acmicpc.net/problem/16724)

난이도 골드 3의 문제이다.

문제의 설명은 꽤나 복잡하지만 하나하나 뜯어보면 결국 맵에 존재하는 사이클의 숫자를 세는 문제이다. (하나의 사이클 안에 존재하는 장판은 모두 한 곳으로 수렴하기 때문.)

### 3.1 발상

맵에 존재하는 총 사이클의 갯수를 구하기 위해 union-find를 사용했다. 발상은 다음과 같다.

1. 2차원 좌표에 고유한 숫자를 부여(x * M + y 값을 부여했다. M은 가로길이)
2. 모든 좌표를 반복문으로 확인하며, union-find 진행. (합쳐진다면 처음 시작좌표가 parent가 됨)
3. 만약 해당 좌표가 이미 사이클에 포함됐다면 패스, 포함되지 않았다면 uf 진행.

주의할 점은 미방문지에서 사이클 찾기를 시작한 후, 진행하다가 다른 사이클에 편입되는 경우가 있다. 이 경우엔 바로 부모를 이어준 다음 종료해준다.

dfs와 uf를 적절하게 섞어서 활용하는 방법에 꽤나 고전했다.

### 3.2 코드

```python
import sys
read = sys.stdin.readline
# dfs 로 사이클 찾기
# 사이클의 수 = safe존의 수

N, M = map(int,read().split())
graph = list(list(read().strip()) for _ in range(N))
visited = list(list(False for _ in range(M)) for _ in range(N))
uf = [i for i in range(N*M)]

def two_d_to_one_d(node: list):
    n_x, n_y = node
    return n_x * M + n_y

def find(x):
    if uf[x] != x:
        uf[x] = find(uf[x])

    return uf[x]

def union(a, b):
    a = find(a)
    b = find(b)

    if a < b:
        uf[b] = a

    else:
        uf[a] = b

def get_next_position(node: list, input: str):
    n_x, n_y = node

    if input == "U":
        return [[n_x-1, n_y]]
    
    elif input == "D":
        return [[n_x+1, n_y]]
    
    elif input == "R":
        return [[n_x, n_y+1]]
    
    elif input == "L":
        return [[n_x, n_y-1]]
    
    
def find_cycle(node: list):
    c_x, c_y = node
    current_node = two_d_to_one_d(node)
    visited[c_x][c_y] = True
    for n_x, n_y in get_next_position(node, graph[c_x][c_y]):
        next_node = two_d_to_one_d([n_x, n_y])
   
        if not visited[n_x][n_y]: # 다음 방문지가 아직 미방문일 경우
            union(current_node, next_node) # 부모를 합쳐줌
            find_cycle([n_x, n_y])
        
        if visited[n_x][n_y]: # 다음이 방문지일 경우
            union(current_node, next_node) # 부모를 합쳐줌 
            visited[n_x][n_y] = True
            break # 바로 정지

for i in range(N):
    for j in range(M):
        if not visited[i][j]:
            find_cycle([i,j])

print(len(set(uf)))
```

## 4. 게임

[백준 1103번: 게임](https://www.acmicpc.net/problem/1103)

난이도 골드2의 문제이다.

2차원 좌표에서 좌표에 쓰여진 숫자만큼 상 하 좌 우 로 움직일 수 있으며, 중간에 보드에 존재하는 구멍에 빠지거나, 보드를 벗어난다면 게임이 종료된다.

이때 가장 많이 움직일 수 있는 횟수를 구하는 문제이다.

### 3.1 발상

단순히 dfs를 이용해서 가장 많이 움직인 횟수를 구하는 문제라고 생각했지만, 그렇게 푼다면 높은 확률로 시간 초과를 받는다.

DP를 이용한 메모이제이션을 적극적으로 사용해 풀어야 한다. 

이유는 특정 칸에서 움직일 수 있는 다음 칸은 제한적이고(바닥에 쓰여진 숫자만큼만 움직여야 하므로 루트가 제한적이다.), 이때 적은 숫자로 해당 칸에 도착했으면 더 앞으로 진행할 필요가 없기 때문이다.

또한 사이클이 존재한다면 -1를 출력해야 하기 때문에 이 부분도 생각해야한다.

### 3.2 코드

```python
import sys
sys.setrecursionlimit(10**5)
read = sys.stdin.readline
N, M = map(int,read().split())
board = list(list(read().strip()) for _ in range(N))
visited = list(list(False for _ in range(M)) for _ in range(N))
dp = list(list(0 for _ in range(M)) for _ in range(N))
# 구멍에 빠지거나 박스 바깥을 벗어나지 않는 경우의 수 중 최대 이동 횟수 

def out_of_box(x, y):
    if x < 0 or x >= N or y < 0 or y >= M:
        return True
    
    return False

def is_whole(x, y): # 구멍이면 True
    if board[x][y] == "H":
        return True
    
    return False

def get_next_position(current: list):
    c_x, c_y = current
    d = int(board[c_x][c_y])

    up = [c_x - d, c_y]
    down = [c_x + d, c_y]
    left = [c_x, c_y - d]
    right = [c_x, c_y + d]

    return [up, down, left, right]

max_count = 0
is_cycle = False
def dfs(node: list, count):
    global max_count
    global is_cycle
    c_x, c_y = node
    dp[c_x][c_y] = count
    max_count = max(max_count, count)

    for nexts in get_next_position(node):
        n_x, n_y = nexts

        if out_of_box(n_x, n_y) or is_whole(n_x, n_y): # 박스 바깥으로 나가거나 구멍을 만나면
            continue
        
        elif dp[n_x][n_y] > count: # 이동횟수가 현재 칸의 이동횟수보다 작다면 지나감
            continue

        elif visited[n_x][n_y]: # 사이클이 있음. 거기서 종료
            is_cycle = True
            break
        
        elif not visited[n_x][n_y]: # 다음이 이동하지 않은 곳이라면
            visited[n_x][n_y] = True
            dfs([n_x, n_y], count+1)
            visited[n_x][n_y] = False # 초기화

dfs([0,0], 1)

if is_cycle:
    print(-1)
else:
    print(max_count)
```
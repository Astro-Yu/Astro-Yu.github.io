---
title: "[백준] 최단거리"
date: 2024-08-01 17:51:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, dijkstra, floyd-warshall]
image: /assets/baekjun.png  
use_math: true
---
풀었던 최단거리 문제(다익스트라, 플로이드 워셜 등) 중 개인적으로 어려웠던 문제들을 정리해보았다.   

## 1. 궁금한 민호

[백준 1507번: 궁금한 민호](https://www.acmicpc.net/problem/1507)

난이도 골드 2의 플로이드 - 워셜 문제이다.

### 1.1 발상

플로이드 - 워셜 그래프가 미리 주어졌을때, 역으로 도시에 존재해야할 최소 갯수의 다리와, 모든 도로의 시간의 합을 구하는 문제이다.

문제 설명이 조금 어렵게 되어있어서(혹은 내가 문제를 제대로 이해하지 못한 것일수도 있다.) 문제를 해결하는데 시간이 걸렸다.

문제에서 요구하는 것은 

1. 다리가 최소 갯수가 되게 유지할 것
2. 모든 도시가 연결되어 있을 것

이므로, 문제에서 주어진 “최단시간 그래프”를 잘 활용하는게 핵심이다.

### 1.2 풀이

문제 해결을 역으로 생각해보자. 예를 들어 도시가 3개 있고, A → B, B → C 로 다리가 연결되어 있다. 시간은 아래와 같다.

A → B : 5초

B → C : 3초

이렇다면 문제에서 주어질 최단시간 그래프는 다음과 같다.

|  | A | B | C |
| --- | --- | --- | --- |
| A | 0 | 5 | 8 |
| B | 5 | 0 | 3 |
| C | 8 | 3 | 0 |

다시 생각해서 위 정보가 없고 그래프만 주어졌을 때 우리는 어느 도시가 다리로 연결되어 있는지 풀어야한다.

이 그래프를 해석하자면

1. A → C의 8초 소요되는 직통 도로를 포함해 3개의 도로가 있다.
2. A → B , B → C의 다리 2개만 있다.

1이 성립하지 않는 이유는, A를 출발, B를 경유, C에 도착하는 시간과 경유하지 않는 시간이 **같기 때문에** 다리 갯수를 최소로 하기 위해선 직통도로가 필요가 없어지기 때문이다. 즉 풀이는 다음과 같다.

1. 먼저 그래프에 있는 모든 경로가 존재한다고 생각한다.
2. 경유경로와 직통경로를 비교한 후 **경유 경로 시간 == 직통 경로 시간** 이라면 직통경로를 삭제한다.
3. 만약 직통 경로가 우회 경로보다 시간이 오래 걸린다면 불가능한 경우이므로 -1을 출력한다.

### 1.3 코드

```python
import sys
import math
read = sys.stdin.readline

N = int(read())
time_table = list(list(map(int,read().split())) for _ in range(N))
roads = list([True for _ in range(N)] for _ in range(N))

result = 0
is_possible = True
for via in range(N):
    for start in range(N):
        for end in range(N):
            via_value = time_table[start][via] + time_table[via][end]
            if start != end and end != via and start != via:
                if time_table[start][end] == via_value: # 우회로랑 직통이랑 길이가 같음 -> 직통경로는 필요없음
                    roads[start][end] = False
                
                elif time_table[start][end] > via_value: # 직통 경로가 우회로보다 긴 경우 -> 불가능한 경우
                    is_possible = False

if is_possible:
    result = 0
    for i in range(N):
        for j in range(i, N):
            if roads[i][j]:
                result += time_table[i][j]
    
    print(result)

else:
    print(-1)
```

## 2. 소가 길을 건너간 이유 7

[백준 14461번: 소가 길을 건너간 이유](https://www.acmicpc.net/problem/14461)

난이도 골드 2의 다익스트라 문제이다.

### 2.1 발상

시작과 끝이 정해져 있고, 한 칸 이동하는데 T초 소요된다. 그리고 3칸씩 이동할 때 마다 해당 칸에 적힌 숫자만큼 시간이 소요된다. 가장 최소로 걸리는 시간을 찾는다.

최초엔 단순한 다익스트라 문제라고 생각해 한 칸씩 이동하며 3칸을 이동했을 경우를 카운팅해 해당 칸의 숫자를 더해줬다. 하지만 이 문제는 단순한 다익스트라 접근 방식으로는 풀지 못한다. 이유는

1. 한 칸씩 나아가는 방식에서 다익스트라는 해당 칸이 최소의 값이라는것을 보장함(Greedy한 방식)
2. 하지만 이 문제에선 한 칸씩 나아가는 방식으로는 해당 칸이 값에 도달하기까지의 최소 값이라는걸 보장하지 못함. 이유는 3번째 나아갔을때, 해당 칸을 더해주는 케이스가 있기 때문.

### 2.2 풀이

어떠한 방법으로 마지막칸에 도달하든 간에, 마지막칸의 숫자는 **더해주지 않아야** 최소 값이 된다. 그렇다면 마지막 칸으로부터 2칸 이내의 최소 값을 구해야 한다. (3칸 나아가서 마지막에 도달하면 해당 값을 더해줘야 하기 때문.)

추가로 해당 칸(마지막 칸으로부터 2칸 이내의 범위 내의 칸)에서 풀을 섭취한 상태여야 한다. 그렇다면 풀을 먹은 상태일때만 칸을 갱신해주면 된다.

### 2.3 코드

```python
import sys
import math
import heapq
read = sys.stdin.readline

N, T = map(int, read().split())
graph = list(list(map(int, read().split())) for _ in range(N))

# 상하좌우 이동 가능
# 3칸을 이동하면 그 자리의 풀 섭취
# 이동하는데 T초 소요
# 도착지와 출발지는 섭취하지 않아도 됨.

start = [0, 0]
end = [N-1, N-1]

def out_of_box(x, y):
    if x < 0 or x >= N or y < 0 or y >= N:
        return True
    return False

dxs = [0, 0, 1, -1, 0, 1, 2, 3, 2, 1, 0, -1, -2, -3, -2, -1]
dys = [1, -1, 0, 0, 3, 2, 1, 0, -1, -2, -3, -2, -1, 0, 1, 2]

def dijkstra(start, end):
    s_x, s_y = start
    time_table = [[math.inf for _ in range(N)] for _ in range(N)]
    time_table[s_x][s_y] = 0
    pq = [(0, start)] # (총 이동시간, 위치, 이동한 갯수)
    answer = math.inf

    while pq:
        current_time, current = heapq.heappop(pq)
        c_x, c_y = current
        if current_time > time_table[c_x][c_y]:
            continue

        dist_to_end = abs(N-1 - c_x) + abs(N-1 - c_y)

        if dist_to_end <= 2: # 도착지의 풀을 먹지 않아도 되는 경우
            answer = min(answer, current_time + dist_to_end * T)

        for dx, dy in zip(dxs, dys):
            n_x, n_y = c_x + dx, c_y + dy
            if out_of_box(n_x, n_y):
                continue
            
            eat_cost = graph[n_x][n_y]
            cross_cost = T

            next = [n_x, n_y]
            new_cost = 3 * cross_cost + eat_cost + current_time # 풀을 먹을때 갱신

            if time_table[n_x][n_y] > new_cost:
                time_table[n_x][n_y] = new_cost
                heapq.heappush(pq, (new_cost, next))

    return answer

result = dijkstra(start, end)

print(result)
```

## 3. 좀비

[백준 11952번: 좀비](https://www.acmicpc.net/problem/11952)

난이도 골드 2의 다익스트라 문제이다

### 3.1 발상

출발지와 도착지가 정해져 있고, 좀비 점령지와 S칸 초과로 떨어져 있으면 안전한 숙박지, S칸 이하로 떨어져 있으면 위험한 숙박지이다. **참고로 좀비 점령지는 숙박하지 못한다.**

즉 문제를 풀기 위해 필요한 것들은 다음과 같다.

1. 지나갈 도시가 안전한 숙박지인지, 위험한 숙박지인지?
2. 도시의 숙박 가격을 알았다면 최소로 지나갈 수 있는 가격은 어떻게 되는지?

2번 조건은 다익스트라를 통해 손쉽게 구할 수 있다.

### 3.2 풀이

그렇다면 1번 조건을 구하는 것이 문제의 핵심이 된다. 노드와 노드 간의 최소 거리를 구하는 방식엔 여러가지가 있다.

1. BFS를 통한 최단거리 구하기
2. 다익스트라를 통한 최단거리 구하기
3. 플로이드 워셜을 통한 최단거리 구하기

음수인 거리는 없으므로 벨판 - 포드는 고려하지 않는다.

2번 방법을 쓰면 도시간의 거리를 구할때마다 다익스트라 함수를 불러와야 하기 때문에 비효율적이다. 그래서 처음엔 3번 방법을 시도했다.

안타깝게도 N의 범위가 꽤나 커서 이 방법은 시간초과를 받았다. **애초에 도시와 도시 간의 거리가 1로 고정되어 있어서 BFS를 사용해도 문제가 없었다.**

또한 시작 도시와 도착 도시에선 숙박비를 내지 않아도 되므로 이 점에 유의한다.

### 3.3 코드

```python
import sys
import math
import heapq
from collections import deque
read = sys.stdin.readline

# N: 도시의 수, M: 길의 수, K: 점령당한 도시의 수, S: 위험한 범위의 도시
# 위험한 도시와 안전한 도시는 지불하는 비용이 다름
# 점령당한 도시는 머물 수 없음
N, M, K, S = map(int, read().split())

# p: 안전한 비용, q: 위험한 비용
p, q = map(int, read().split())

# 도시들간의 거리를 구했으면 다익스트라로 비용 구하기
# 1번 도시와 N번 도시에서는 비용을 지불하지 않아도 됨
dij_graph = [[] for _ in range(N+1)]
zombies = []
visited = [-1 for _ in range(N+1)]

for _ in range(K):
    zombie = int(read())
    zombies.append(zombie)
    visited[zombie] = 0

for _ in range(M):
    a, b = map(int, read().split())

    dij_graph[a].append(b)
    dij_graph[b].append(a)

def get_danger_area(zombies: list):
    danger_area = [False] * (N+1)
    queue = deque(zombies)

    while queue:
        current = queue.popleft()

        for neighbor in dij_graph[current]:
            if visited[neighbor] == -1: # 아직 방문하지 않았다면
                queue.append(neighbor)
                visited[neighbor] = visited[current] + 1
                if visited[neighbor] <= S:
                    danger_area[neighbor] = True
                    queue.append(neighbor)

    return danger_area

danger_area = get_danger_area(zombies)

def get_cost(next):
    if next == N:
        return 0
    
    if danger_area[next]:
        return q
    
    else:
        return p
    
def dijkstra(s: int):
    costs = [math.inf for _ in range(N+1)]
    costs[s] = 0
    pq = [(0, s)]

    while pq:
        current_cost, current = heapq.heappop(pq)

        if costs[current] < current_cost:
            continue

        for next in dij_graph[current]:
            if next in zombies:
                continue

            next_cost = get_cost(next)
            new_cost = next_cost + current_cost

            if costs[next] > new_cost:
                costs[next] = new_cost
                heapq.heappush(pq, (new_cost, next))
    
    return costs[N]

answer = dijkstra(1)
print(answer)
```
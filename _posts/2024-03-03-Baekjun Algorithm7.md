---
title: "[백준] DFS"
date: 2024-03-03 19:01:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, dfs, bfs, graph]
image: /assets/baekjun.png
use_math: true
---
## 0. 목표

- DFS 작동방식 알아보기
- DFS 구현방법 알아보기
- 코드

## 1. DFS란?

DFS는 Depth First Search의 약자로, **그래프** 탐색 알고리즘의 방식 중 하나이다.

여기서 그래프란, 정점(Vertex) 와 변(Edge) 로 구성된 자료구조로, 정점과 정점 사이는 변으로 이어져 있고, 이때 변에는 가중치, 혹은 방향이 존재할 수도 있다.

DFS는 이 그래프를 **순회**하여 탐색하는데 사용된다. **한 방향의 루트를 끝까지 탐색한 이후**, 다음 루트를 탐색한다. 모든 조합을 탐색할 수 있고, 특정 조합은 거를 수 있기 때문에 반복문보다 더 효율적인 방식으로 순서쌍 찾기가 가능하다.

반대로 모든 노드를 탐색하기 때문에 최단거리의 길 찾기를 할 때에는 불리한 점이 있다. 이때는 ***BFS*** 알고리즘을 쓰면 좋다.

## 2. DFS의 작동방식

DFS는 한 가지 루트를 전부 탐색한 이후, 더 탐색할 루트가 없다면 이전으로 돌아와 탐색 루트를 찾는다. 그림으로 설명하자면…

![](/assets/dfs1.jpeg)

1에서 탐색을 시작한다면 이웃한 2와 3으로 이동 가능하다. 이때 왼쪽을 먼저 탐색한다고 정하자. 2로 이동한 이후 인접한 4와 5로 이동이 가능하다. 

4로 이동 후엔 한 가지 길 밖에 없으므로 차례로 8 과 14를 탐색해준다.

14를 다 탐색한 이후엔 뒤로 되돌아가 방문하지 않은 루트를 찾아준다. 8 → 4 → 2까지 되돌아간 이후, 방문하지 않은 루트인 5로 이동한다. 

그 이후 인접한 9를 탐색 후 다시 5로 되돌아가 10을 탐색한다.

해당 그래프의 탐색 순서를 표시했다. 백트래킹으로 돌아가는 부분은 보라색으로 표시했다.

![](/assets/dfs2.jpeg)

## 3. DFS의 구현방법

앞서서 DFS의 작동방식을 어떻게 구현할까 생각해보면 가장 고민인 부분은

1. 그래프의 표현
2. 방문한 정점의 표시
3. 백트래킹의 구현

를 고민하게 될 것이다. 예시의 그래프를 하나 그려 설명해보자. 이동 방향은 없는 그래프이다.

![](/assets/dfs3.jpeg)

### 3.1 그래프의 표현

먼저 그래프를 배열로 표현해준다. 이때 각 정점의 숫자를 인덱스로 가지고 그 정점에 이웃한 정점을 배열로 표시한다. 즉 이 그림의 경우엔

`graph[1] = [2, 4]`

`graph[2] = [1, 3]`

`graph[3] = [2, 4, 5]`

…

이고 전체 그래프는

`graph = [[], [2, 4], [1, 3], [2, 4, 5], [1, 3, 5], [3 ,4]]`    
로 표현할 수 있다.

이때 0번 인덱스는 인덱스 번호와 정점 번호를 맞추기 위한 더미 데이터이다.

### 3.2 방문한 정점의 표시

DFS에선 그래프가 정점을 순회할 때 방문하지 않은 그래프로 이동해야한다. 그렇지 않다면 역방향으로 이동할 수도 있다.

그렇기에 `visited` 라는 배열을 만들고, DFS가 순회할 때 방문한 노드의 인덱스를 `True`로 표현할 수도 있고, -1 로 초기화해 만든 경우 방문 순서를 표시해 -1이 아닌 경우에만 방문했다고 표현할 수도 있다.

### 3.3 백트래킹 구현

우리는 이미 그래프에 대한 데이터를 모두 갖고있기 때문에 실제로 거슬러 올라가는 형태로 구현할 필요는 없다. graph에 도착 지점이 모두 표시되어 있기 때문에 해당 표시지점 중 방문하지 않은 노드를 골라 방문한다면 백트래킹과 같은 방식으로 구현 가능하다.

## 4. 코드

### 4.1 24479. **알고리즘 수업 - 깊이 우선 탐색 1**

[24479번: 알고리즘 수업 - 깊이 우선 탐색 1](https://www.acmicpc.net/problem/24479)

기본적인 DFS 알고리즘 맛보기 문제이다.

```python
import sys
sys.setrecursionlimit(10 ** 6)
read = sys.stdin.readline

N,M,start = map(int,read().split()) # 입력

info = [[] for _ in range(N+1)]
visited = [0] * (N+1) # 0으로 초기화하여 dfs 순회 시 방문 순서를 기록한다.

for _ in range(M): # 그래프에 출발 지점 인덱스에 도착 정점 번호를 기록한다.
    node, line = map(int,read().split())
    info[node].append(line)
    info[line].append(node)
		# 양방향 이동가 가능하기 때문에 node 와 line에 대해 각각 기록해야한다.

for i in range(N+1):# 문제에서 오름차순으로 이동할 것을 지시하기 때문에 정렬해준다.
    info[i].sort()

count = 1 # 방문 순서
def dfs(graph,start,visited):
    global count
    visited[start] = count # 시작 정점의 방문순서는 1번

    for i in graph[start]: # i에 도착 정점이 들어간다.
        if visited[i] == 0: # 방문한 적이 없는 정점의 경우
            count += 1 # 방문 순서 기록
            dfs(graph,i,visited) # 도착 정점이 시작 정점이 되게 재귀한다.

dfs(info,start,visited)

for i in range(1,N+1): # 각 인덱스의 정점의 방문 순서를 출력
    print(visited[i])
```

이때 재귀함수를 사용해 `for i in graph[start]` 문 이하에서 재귀함수로 더 이상 나아갈 수 없을때 자연스럽게 다음 반복문이 실행되어 백트래킹을 구현한다.

visited를 -1로 초기화해 만든 후, 방문한 순서들을 기록해주는 문제이다.

### 4.2 2606. **바이러스**

https://www.acmicpc.net/problem/2606

바이러스에 걸리게 되는 컴퓨터의 갯수를 구하는 문제이다. 전체 pc를 모두 조회해보아야 하기 때문에 DFS로 풀기 적합하다.

```python
import sys
read = sys.stdin.readline

pc = int(read().strip())
lines = int(read().strip())

graph = [[] for _ in range(pc+1)]
visited = [True] * (pc+1)

for _ in range(lines):
    node1, node2 = map(int,read().split())
    graph[node1].append(node2)
    graph[node2].append(node1)

count = 0
def virus(graph,visited,start):
    global count
    visited[start] = False

    for line in graph[start]:
        if visited[line]:
            count += 1
            virus(graph, visited, line)    

virus(graph,visited,1)

print(count)
```

바이러스에 걸리는 pc를 False로 표현해, 시작pc와 이어지면서 True 인 pc를 찾아 count를 하나씩 늘려주고 재귀하면 된다. 

그렇다면 한번 바이러스에 걸린 pc는 방문하지 않게 된다.


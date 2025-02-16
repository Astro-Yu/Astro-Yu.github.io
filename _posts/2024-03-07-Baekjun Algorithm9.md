---
title: "[백준] BFS"
date: 2024-03-07 20:49:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, bfs, graph, binary tree]
image: /assets/baekjun.png
use_math: true
---
## 0. 목표

- BFS 작동방식 알아보기
- BFS 구현방법 알아보기
- 코드

관련글 [DFS](https://astro-yu.github.io/posts/Baekjun-Algorithm7/링크)

*DFS와 그래프의 정의를 보고 싶다면 해당 링크를 클릭.*

## 1. BFS란?

BFS는 **B**readth **F**irst **S**earch 의 약자로, 그래프 탐색 알고리즘의 방식 중 하나이다. 앞서 알아본 DFS의 경우 한 가지 leaf로 도달하는 경로를 한번에 탐색했다. BFS는 DFS와 달리 하나의 **부모 노드에 연결된 모든 자식 노드들을 한번에 탐색한다.** Tree 그래프에서 생각해본다면 DFS는 0 → 1 → 2→ 3 … 으로 나아간다면, BFS는 0 → 1 → 1 → 1 → 2 …. 처럼 한 가지 level을 모두 탐색한다.

BFS의 탐색방식을 머릿속으로 잘 그려본다면 알 수 있겠지만, 가중치가 없는 그래프에서 가장 짧은 거리를 찾는데 유용하게 사용된다.

## 2. BFS의 작동방식

BFS는 부모 노드에 연결된 모든 자식노드를 탐색한다. 그림으로 설명하자면…

![](/assets/bfs2.jpeg)

0에서 탐색을 시작한다면 자식노드인 1, 2, 3을 탐색한다.

이때 0에 연결된 자식노드가 더 없다면 1, 2, 3을 부모노드로 하여 그 자식노드들을 탐색한다. 그렇다면 그 이후엔 4, 5, 7 ,8 을 탐색한다.

이 과정을 반복하여 방문하지 않은 노드가 없을때까지 찾아준다.

## 3. BFS의 구현방식.

BFS 알고리즘 구현의 핵심은 

1. 다음 탐색 목적지인 자식 노드를 저장해놓는것.
2. DFS와 같이 이미 방문한 노드의 경우 방문하지 않는 것.

이다.

2의 경우엔 DFS와 마찬가지로 방문 여부를 기억해놓는 배열을 작성하면 된다.

1의 경우엔 **Queue** 를 사용한다.

queue에 다음 자식 노드들을 저장해둔 다음 하나씩 탐색 과정에서 queue에 들어있는 노드들을 하나씩 popleft 한 후, pop 한 자식 노드들이 조건에 맞다면(= 이전에 탐색하지 않았다면) queue에 append 해준다.

### 3.1 설명

queue의 **first in, first out**의 특성을 이용한 구현 방식이다. 그림으로 그려보자.

![](/assets/bfs3.jpeg)

처음 시작하는 부모 노드는 수동으로 queue에 넣어준다.

`queue = [0]`

그 이후 0이 popleft로 빠지게 되고 에 연결된 1, 2, 3이 queue에 차례대로 들어온다.

`queue = [1, 2, 3]` 

다음 탐색 노드는 1의 자식 노드이지만 존재하지 않기 때문에 queue에 더해지는 노드는 없다.

`queue = [2, 3]`

다음엔 popleft로 2를 꺼낸 후, 2의 자식 노드 7을 넣어준다.

`queue = [3, 7]`

다음엔 3을 꺼내, 그 자식 노드인 8을 넣어준다.

`queue = [7, 8]`

7을 꺼내 탐색 후, 자식 노드가 없으므로 추가하지 않고, 8 역시 마찬가지로 추가하지 않는다.

`queue = []`

queue가 비었으므로 반복문을 종료한다.

## 4. 코드

[24444번: 알고리즘 수업 - 너비 우선 탐색 1](https://www.acmicpc.net/problem/24444)

기본적인 BFS의 구현 방식을 물어보는 문제이다.

```python
import sys
from collections import deque
read = sys.stdin.readline

N,M,start = map(int,read().split())

info = [[] for _ in range(N+1)]
visited = [0] * (N+1)

for _ in range(M):
    node, line = map(int,read().split())
    info[node].append(line)
    info[line].append(node)

for i in range(N+1):
    info[i].sort()

count = 1
def bfs(graph,start,visited):
    global count
    queue = deque([start])
    visited[start] = 1

    while queue:
        current = queue.popleft()
        for neighbor in graph[current]:
            if visited[neighbor] == 0:
                count += 1
                queue.append(neighbor)
                visited[neighbor] = count

bfs(info,start,visited)

for num in visited[1:]:
    print(num)
```
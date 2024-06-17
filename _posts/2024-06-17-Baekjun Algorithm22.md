---
title: "[백준] 그리디 알고리즘 <2-2> "
date: 2024-06-17 20:01:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, heap, sort, greedy]
image: /assets/baekjun.png
use_math: true
---
풀었던 그리디 문제 중 개인적으로 어려웠던 문제들을 정리해보았다.   

이전 글    
[그리디<2-1>](https://astro-yu.github.io/posts/Baekjun-Algorithm21/)

## 3. 공항

[백준 10775번: 공항](https://www.acmicpc.net/problem/1092)

난이도 골드2의 문제이다.

문제가 약간 설명이 이해하기 어렵게 되어있는듯 하다. 

공항에 도킹할 수 있는 게이트의 숫자가 주어지고, 도킹을 시도하려는 비행기가 순서대로 주어진다.

이때 비행기는 1번부터 주어진 숫자 g_i 까지 도킹을 시도하며, **단 하나의 비행기라도 도킹이 불가하다면 이후부터는 도킹을 시도하지 않는다.**

즉 중간에 도킹이 불가하다면 이후에 들어오는 비행기가 조건을 만족해 도킹이 가능하더라도 하지 않는다는 뜻이었다.

### 3.1 발상

처음에는 단순한 그리디식 자리 찾기 문제인 줄 알고 코드를 작성했지만 시간 초과를 받았다.

***시간초과 코드***

```python
import sys
read = sys.stdin.readline

G = int(read())
P = int(read())

g_is = [False] * (G+1)

def find_place(p_i: int):
    if not g_is[p_i]: # 해당 자리가 비어있다면 들어감
        g_is[p_i] = True
        return True
    
    while p_i != 0: # 비어있지 않다면 한 자리씩 내려가면서 들어감
        p_i -= 1
        if not g_is[p_i] and p_i != 0:
            g_is[p_i] = True
            return True
    
    return False
   
count = 0
for _ in range(P):
    p_i = int(read())

    if find_place(p_i):
        count += 1
    else:
        break

print(count)
```

이전 글에서 설명했듯 이 코드는 최악의 경우 $O(N^2)$의 시간복잡도를 갖는다.

더 간결한 코드로 바꿔야 했다.

### 3.2 해결

해당 문제는 union-find로 해결 가능했다.

해당 문제의 2번 예제로 설명해보면 다음과 같다.

1. 먼저 각각의 인덱스를 값으로 가지는 airports 테이블을 준비한다.
    
    
    | 0 | 1 | 2 | 3 | 4 |
    | --- | --- | --- | --- | --- |
    | 0 | 1 | 2 | 3 | 4 |
2. 해당 인덱스에 해당하는 비행기가 들어온다면 그 비행기의 부모를 찾아준다. 가장 첫번째 비행기는 g_i가 2 비행기이니 parent = 2.
3. **해당 부모의 -1 된 값을 부모로 설정한다.**
4. parent = 2 였으니 2번 비행기의 부모는 1번 port 가 된다.
    
    
    | 0 | 1 | 2 | 3 | 4 |
    | --- | --- | --- | --- | --- |
    | 0 | 1 | 1 | 3 | 4 |
5. 다음 비행기도 g_i = 2 이다. 이때 find 과정을 통해 해당 비행기의 parent = 1 이 된다.
6. 역시 -1의 parent값으로 설정해준다. 이때는 0이 된다.
    
    
    | 0 | 1 | 2 | 3 | 4 |
    | --- | --- | --- | --- | --- |
    | 0 | 0 | 1 | 3 | 4 |
7. g_i = 3 인 3번 비행기는 2번 port에 배치된다.
    
    
    | 0 | 1 | 2 | 3 | 4 |
    | --- | --- | --- | --- | --- |
    | 0 | 0 | 1 | 2 | 4 |
8. 이때 다음으로 g_i = 3 인 4번 비행기가 들어왔다. 해당 비행기의 parent = 0 이 된다.
9. parent = 0 인 비행기가 나타나면 그 곳에서 코드를 종료한다.

코드의 개념 자체는 처음 시도한 끝에서부터 한 자리씩 찾는 코드와 유사하다. 볼드 처리한 3번 과정이 해당 번호부터 배치할 수 있는 가장 나중의 게이트를 찾는 과정이기 때문이다.

하지만 union-find로 반복문 없이 빠르게 찾을 수 있기 때문에 전체적인 시간 복잡도는 $O(G + P)$ 로 실행된다.

## 4. 전구와 스위치

[백준 2138번: 전구와 스위치](https://www.acmicpc.net/problem/2138)

난이도 골드4의 문제이다. 특정한 발상을 요구하고 있어서 골드4 문제 치고는 꽤나 어려웠다.

### 4.1 발상

처음 해당 문제를 시도했을땐 모든 방법을 시도해보아야 하는 브루트포스 문제라고 생각했다. 하지만 브루트포스라고 하기엔 경우의 수가 너무 많고 주어진 숫자도 커서 결국 포기했다.

### 4.2 해결

해당 문제의 재미있는 점은 

1. 전구의 누르는 순서는 관계 없다는 것이었다.
2. 2가지 방법으로 나누게 되면 특정 전구에 영향을 주는 스위치를 제한할 수 있다.

는 것이었다.

1번은 직접 시도해보면 쉽게 알 수 있고, 2번은 어떤 뜻일까?

해당 문제에서는 0번 스위치를 **누르는 경우**, **누르지 않는 경우** 2가지로 나눌 수 있다.

우리가 주어진 스위치들을 순차적으로 누르기로 결정했다면, 이미 누른 스위치는 절대 다시 누르지 않을 것이지 때문에 N번 전구에 영향을 주는 스위치는 N+1의 스위치가 된다.

하지만 0번 전구는 0번 스위치, 1번 스위치 모두에게 영향을 받게 된다. 

즉 이런 경우의 수를 나누어서 

1. 0번 스위치를 미리 누른 경우
2. 0번 스위치를 누르지 않고 시작하는 경우

2 가지로 나누어서 스위치를 적게 누른 값을 출력하면 된다. 또한 이 방법으로 불가능한 경우의 수 까지 걸러 낼 수 있으니 안성맞춤이다.

```python
import sys
from copy import deepcopy
read = sys.stdin.readline

N = int(read())

init_state = list(map(int,read().strip()))
goal_state = list(map(int,read().strip()))

def change_state(state: int):
    if state == 1:
        return 0
    
    return 1

def get_next_state(switch: int, current_state: list):
    for i in range(switch-1, switch+2):
        if i < 0:
            continue

        if i >= N:
            continue
        current_state[i] = change_state(current_state[i])

    return current_state

def press_0():
    copy_init_state = deepcopy(init_state)
    copy_init_state = get_next_state(0, copy_init_state) # 0번 누름
    count = 1
    for i in range(N-1):
        if goal_state[i] != copy_init_state[i]:
            copy_init_state = get_next_state(i+1, copy_init_state)
            count += 1
    
    if goal_state == copy_init_state:
        return count
    else:
        return -1

def dont_press_0():
    copy_init_state = deepcopy(init_state)

    count = 0
    for i in range(N-1):
        if goal_state[i] != copy_init_state[i]:
            copy_init_state = get_next_state(i+1, copy_init_state)
            count += 1
    
    if goal_state == copy_init_state:
        return count
    else:
        return -1

answer_a = press_0()
answer_b = dont_press_0()

if answer_a >= 0 and answer_b >= 0:
    print(min(answer_a, answer_b))
elif answer_a < 0 or answer_b < 0:
    print(max(answer_a, answer_b))
else:
    print(-1)
```

최초 발상이 해당 문제를 풀 수 있는지 없는지 가르는 문제였다.
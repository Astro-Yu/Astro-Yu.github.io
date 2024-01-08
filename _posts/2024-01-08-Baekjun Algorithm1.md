---
title: "[백준] 재귀, 백트래킹 <1>"
date: 2024-01-08 19:23:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, backtracking, recursion, baekjoon]
image: /assets/baekjun.png
---
## #15649 순열

```python
import sys

def permutation(n, r, p = []): # n개중 r개
    if (r == 0):
        print(*p) # 하나의 경우 출력
    
    elif (r != 0):
        for i in range(1, 1+n):
            if (p.count(i) == 0): ## i가 리스트에 없는 경우 추가
                p.append(i)
                permutation(n, r - 1, p) ## r을 하나 줄여서 재귀
                p.remove(i)

N, M = map(int,sys.stdin.readline().strip().split()) # 입력
permutation(N,M)
```

순열(permutation)이란 <span style="color:red"> **순서를 고려하여** </span> n개중 r개를 뽑는 경우의 수를 말한다.

예를 들어 [1, 2, 3, 4] 중 2개를 뽑는 경우의 수라면 [1, 2] 와 [2, 1]은 서로 다른 경우라는 뜻이다.

수학적인 식은 굳이 여기선 적지 않고 코드를 통해 어떻게 순열의 경우의 수를 조합하는지 알아보자.

`permutation(n, r, p = [])` 은 n 개중 r개를 뽑고, 경우의 수를 p = []에 저장한다.

`if (r == 0)` 의 경우 r의 갯수를 재귀로 하나씩 줄여나가 하나의 step을 통과했을시 r이 1씩 줄어들기 때문에 r == 0 이란 뜻은 r개를 모두 뽑은 경우를 말한다. 이때는 우리가 원한 하나의 경우를 뽑았기 때문에 출력해준다.

그 아래의 elif 문에선 아직 다 조합하지 못한 경우를 처리하며, `p.count` 에선 해당 숫자가 중복되지 않을 경우에만 아래 코드를 실행하도록 한다. (즉 1 1 과 같은 경우를 거르기 위함이다.)

**핵심은 이곳이다. ( 맨 처음 진입한 함수를 0. 이라 할 때…)** 

1.  하나의 숫자를 추가하면 r을 하나 줄여 재귀한다. r을 줄이는 이유는 하나의 숫자를 추가했기 때문에 나머지 하나만 뽑으면 되기 때문이다. 
2. 하나를 뽑은 이후 재귀해 들어간 함수에선 앞서서 뽑은 숫자 이외의 숫자를 추가한다. 이후 다시 재귀하면 r을 하나 줄여 0이 되기 때문에 하나의 경우를 출력한다. 

0. 이후 처음 들어온 재귀 함수를 탈출한 후 맨 처음 추가한 숫자를 `p.remove` 로 지워 다음 경우의 수를 탐색한다.

숫자가 반복문에 의해 1부터 차례로 증가하기 때문에 문제에서 제시한 오름차순은 자연스럽게 만족하게 된다.

## #15650 조합

조합(combination) 이란 <span style="color:red"> **순서를 고려하지 않고** </span> n 개중 r개를 뽑는 경우의 수를 말한다.

예를 들어 [1, 2] 와 [2, 1]은 같은 경우이기에 2번 세지 않는다.

이 경우에는 permutation의 함수를 조금 변형하면 된다.

```python
import sys

def combination(n, r, start = 1, p = []):
    if (r == 0):
        print(*p)

    elif (r != 0):
        for i in range(start, n+1):
            if (p.count(i) == 0): 
                p.append(i)
                combination(n, r-1, i + 1, p) # i 다음부터 시작하도록 재귀
                p.pop()

N, M = map(int, sys.stdin.readline().strip().split())
combination(N, M)
```

달라진 부분은 함수에 start를 지정해주었다.

이유는 재귀하여 들어갈때 순열과 달리 중복을 방지하기 위함이다.

예를 들어서 직접 세어보자

> [1, 2, 3, 4] 중 2개를 뽑는 경우
> 
> 
> [1, 2] [1, 3] [1, 4]
> 
> [2, 3] [2, 4]
> 
> [3, 4]
> 

첫번째 요소가 2인 경우, 순열처럼 [2, 1] 같이 1부터 세게되면 앞서 카운팅한 경우와 중복된다.

두번째 요소는 첫번째 요소보다 1이 큰 숫자부터 시작해 세야 한다. 그렇기 때문에 start를 추가했다.

즉 재귀를 돌면서 첫번째 요소가 i 이고, 시작하는 수는 `start = i + 1` 이어야 한다는 것.
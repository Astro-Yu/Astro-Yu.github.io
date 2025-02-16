---
title: "[백준] 주사위 굴리기"
date: 2024-11-20 19:39:00 +09:00 # 시간
categories: [Algorithm]
published: true
tags: [python, algorithm, queue, simulation]
image: /assets/baekjun.png  
use_math: true
---  

사실 100% 구현 문제라 특별한 알고리즘은 없긴 한데, 주사위 굴리기 구현 발상이 나름대로 괜찮은 것 같아서 적어본다.

[백준 14499번: 주사위 굴리기](https://www.acmicpc.net/problem/14499)

아래로는 전체 문제 내용 정리는 없고, 주사위 알고리즘에 대해서만 설명한다.

## 1. 초기 주사위 상태

보드 위에서 주사위가 굴러갈 방향에 따라(동, 서, 남, 북) 어떻게 주사위 면면이 달라지는지 알아내는게 핵심이다.

주사위의 전개도는 아래와 같다.

```
  2
4 1 3
  5
  6
```

이때 주사위는 1의 면이 위쪽을 바라보고, 3이 동쪽(오른쪽)을 바라보게 놓여있다.

![사진자리1](/assets/dice1.jpeg)

대략 이런 느낌으로 놓여 있다.

이때 주사위가 동쪽으로 굴러간다면 위쪽이 4, 아래쪽이 3이 된다.

서쪽으로 굴러간다면 위쪽이 3, 아래쪽이 4가 된다.

굴러갈때 마다 방향이 달라지기 때문에 단순한 구현으로는 문제가 있었다.

## 2. 주사위 굴리기 알고리즘.

나는 이 알고리즘을 다음과 같이 정리했다. 설명 기준은 초기 주사위 상태에서 한다.

- 탑 → 동 방향의 주사위 면을 deque로 정리 [1, 3, 6, 4]
- 탑 → 북 방향의 주사위 면을 deque로 정리 [1, 2, 6, 5]
- 이때 탑 방향은 무조건 0번 인덱스고, 바텀 방향은 무조건 2번 인덱스가 된다.

### 주사위 굴리기

예를 들어서 동쪽으로 한번 굴러간다고 해보자.

1. deque를 1의 값으로 rotate 시킨다. → [4, 1, 3, 6]
2. 동쪽으로 굴러갔기 때문에 탑 → 북 리스트의 1, 3번 인덱스는 변화가 없다.
3. 탑 → 북 리스트의 0번 인덱스(탑)와 2번 인덱스(바텀)을 1의 과정에서 구한 탑과 바텀으로 수정해준다.
4. 탑 → 동 리스트 = [4, 1, 3, 6], 탑 → 북 리스트 [4, 2, 3, 5]
    
    ![사진자리2](/assets/dice2.jpeg)
    
    실제로 똑같이 움직인다.
    

위 방식과 똑같게 서쪽으로 움직인다면 -1, 북쪽으로 움직인다면 탑 → 북 리스트를 1만큼 회전시키면 된다.

문제 풀이시 사용했던 주사위 객체 코드이다.
```python
class Dice:
    def __init__(self, x, y):
        self.dice = [0,0,0,0,0,0,0] # 1번 인덱스가 1번칸
        self.x = x
        self.y = y
        self.top_to_east = deque([1,3,6,4])
        self.top_to_north = deque([1,2,6,5])
    
    def get_top_value(self):
        return self.dice[self.top_to_east[0]]
    
    def get_bottom_value(self):
        return self.dice[self.top_to_east[2]]
    
    def copy_value_from_map(self, value):
        self.dice[self.top_to_east[2]] = value

    def get_next_position(self, direction):
        d = [[], [0,1], [0,-1], [-1,0], [1,0]]
        nx, ny = self.x + d[direction][0], self.y + d[direction][1]

        return [nx, ny]
    
    
    def move(self, direction):
        nx, ny = self.get_next_position(direction)
        
        if direction == 1: # 동쪽
            self.top_to_east.rotate(1)
            self.top_to_north[0] = self.top_to_east[0]
            self.top_to_north[2] = self.top_to_east[2]
        
        if direction == 2: # 서쪽
            self.top_to_east.rotate(-1)
            self.top_to_north[0] = self.top_to_east[0]
            self.top_to_north[2] = self.top_to_east[2]
        
        if direction == 3: # 북쪽
            self.top_to_north.rotate(1)
            self.top_to_east[0] = self.top_to_north[0]
            self.top_to_east[2] = self.top_to_north[2]

        if direction == 4:
            self.top_to_north.rotate(-1)
            self.top_to_east[0] = self.top_to_north[0]
            self.top_to_east[2] = self.top_to_north[2]

        self.x = nx
        self.y = ny
```
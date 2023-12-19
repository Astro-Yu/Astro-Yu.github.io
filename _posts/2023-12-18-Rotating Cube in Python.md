---
title: Rotating Cube in Python <1>
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [PYTHON]
published: true
tags: [python, ascii, cube, mac]
---
# Rotating Cube in Python <1>.

### 0. 발단  
시작은 유튜브에서 본 한 영상

Link: [**회전하는 큐브**](https://www.youtube.com/watch?v=p09i_hoFdd0&t=36s)
    
꽤나 멋있다...

코드는 깃헙에 있고, 영상 중간에 나오는 수학만 이해하면 충분히 해볼만 하겠는데?

---

### 1. 코드 뜯어보기
물론 나는 C를 할 줄 모르기 때문에 Python으로 작성해볼 것이다.

먼저 코드를 살펴보기 위해 [**원본 깃허브**](https://github.com/servetgulnaroglu/cube.c)로 이동

대충 중요한 부분만 보자면

```
float calculateX(int i, int j, int k) {
    return j * sin(A) * sin(B) * cos(C) - k * cos(A) * sin(B) * cos(C) +
    j * cos(A) * sin(C) + k * sin(A) * sin(C) + i * cos(B) * cos(C);
}

float calculateY(int i, int j, int k) {
    return j * cos(A) * cos(C) + k * sin(A) * cos(C) -
    j * sin(A) * sin(B) * sin(C) + k * cos(A) * sin(B) * sin(C) -
    i * cos(B) * sin(C);
}  

float calculateZ(int i, int j, int k) {
    return k * cos(A) * cos(B) - j * sin(A) * cos(B) + i * sin(B);
}

```
터미널에 표현될 각각의 픽셀들의 3차원 위치를 계산해주는 코드다.   
여기서 A,B,C는 회전하는 각도이고 이 값들을 일정하게 변화시키면 회전하는 모양을 볼 수 있다.

```
void calculateForSurface(float cubeX, float cubeY, float cubeZ, int ch) {
x = calculateX(cubeX, cubeY, cubeZ);
y = calculateY(cubeX, cubeY, cubeZ);
z = calculateZ(cubeX, cubeY, cubeZ) + distanceFromCam;

ooz = 1 / z;

xp = (int)(width / 2 + horizontalOffset + K1 * ooz * x * 2);
yp = (int)(height / 2 + K1 * ooz * y);

```
함수명을 보면 대충 할 역할이 보인다. `calculateForSurface`   
즉 큐브의 표면을 계산하는 녀석이다. 아마 앞으로 큐브의 6면을 계산할 것으로 예상된다.

못 보던 변수가 있다. `distanceFromCam`    
간단하게 설명하자면 터미널 창으로부터 큐브까지의 거리다. 원본 코드를 보면 전역에서 설정하고 들어온다.

이 값을 z 값에 더해주는걸로 보아 z 값이 흔하게 알려진 높이가 아니라, 깊이의 역할을 할 것 같다.

`ooz`   
단순히 표현되기로는 z값의 역수. 관측자로부터 안쪽에 있는 픽셀일수록 작은 값을 갖게된다.


`xp`,`yp`   
우리는 3차원 큐브를 실제로 2차원 평면에 출력해야 하기 때문에 3차원 좌표를 2차원으로 투영(projection) 해야한다.

```
idx = xp + yp * width;
if (idx >= 0 && idx < width * height) {
    if (ooz > zBuffer[idx]) {
    zBuffer[idx] = ooz;
    buffer[idx] = ch;
        }
    }
}
```
이곳에서 `ooz`의 사용이 나온다.   
`idx`로 원하는 위치의 캔버스에 선언 후, 내가 원하는 캔버스 범위 (첫 번째 if문) 에 있으면서, 앞에 가리는 픽셀이 없을 경우(두 번째 if문) `buffer[idx]`, `zBuffer[idx]`에 추가시킨다.   

즉    
`buffer[idx]` = 실제 터미널에 표현될 위치.   
`zBuffer[idx]` = 그 위치 픽셀의 깊이 정보.

라고 이해하면 될 듯 하다. (`ooz`가 클 수록 앞에 있는 픽셀이란 뜻.)

이 것들을 반복문으로 하나씩 출력해주면 끝.

```
int main() {
  printf("\x1b[2J");
  while (1) {
    memset(buffer, backgroundASCIICode, width * height);
    memset(zBuffer, 0, width * height * 4);
    cubeWidth = 20;
    horizontalOffset = -2 * cubeWidth;
    // first cube
    for (float cubeX = -cubeWidth; cubeX < cubeWidth; cubeX += incrementSpeed) {
      for (float cubeY = -cubeWidth; cubeY < cubeWidth;
           cubeY += incrementSpeed) {
        calculateForSurface(cubeX, cubeY, -cubeWidth, '@');
        calculateForSurface(cubeWidth, cubeY, cubeX, '$');
        calculateForSurface(-cubeWidth, cubeY, -cubeX, '~');
        calculateForSurface(-cubeX, cubeY, cubeWidth, '#');
        calculateForSurface(cubeX, -cubeWidth, -cubeY, ';');
        calculateForSurface(cubeX, cubeWidth, cubeY, '+');
      }
    }
```
원본엔 한 화면에 3개의 큐브가 돌아가도록 코드가 짜여져 있다. 그중 하나를 보면...

`printf("\x1b[2J");`   
검색해보니 터미널을 clear 하고 커서를 맨 왼쪽 위로 올리는 역할을 하는 듯 하다.

그 아래로 큐브의 크기, Offset(터미널 위에서의 위치)를 설정하고   
`calculateForSurface`를 이용해 6면을 하나하나 계산한다. 그 후 각 면마다 다른 문자열을 할당해준다.

```
printf("\x1b[H");
    for (int k = 0; k < width * height; k++) {
      putchar(k % width ? buffer[k] : 10);
    }

    A += 0.05;
    B += 0.05;
    C += 0.01;
    usleep(8000 * 2);
  }
```
각도를 회전시키면서 그림 정보가 담겨있는 buffer 전체를 출력한다면 끗.


`usleep(8000 * 2);`   
이건 아마 프레임 조절용으로 넣어둔듯 하다.

다음은 이 코드를 python으로 변환시키고, 추가로 각 면마다 컬러도 넣어서 다채롭게 만들어 볼 계획이다.

---

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fastro-yu.github.io%2Fposts%2FRotating-Cube-in-Python%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)


    





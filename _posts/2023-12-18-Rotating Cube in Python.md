---
title: Rotating Cube in Python
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [PYTHON]
tags: [python, ascii, cube, mac]
---
# Rotating Cube in Python.

### 0. 발단  
    발단은 유튜브에서 본 한 영상

    Link: [회전하는 큐브][yutubelink]
    
    꽤나 멋있다...

    코드는 깃헙에 주어져있고, 영상 중간에 나오는 수학만 이해하면 충분히 이해할만 하겠는데?

### 1. 코드 뜯어보기
    물론 나는 C를 할 줄 모르기 때문에 Python으로 작성해볼 것이다.
    
    먼저 코드를 살펴보기 위해 [원본 깃허브][original_github_link]로 이동

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
    수학에 관련된 코드이기 때문에 

    ```
    void calculateForSurface(float cubeX, float cubeY, float cubeZ, int ch) {
    x = calculateX(cubeX, cubeY, cubeZ);
    y = calculateY(cubeX, cubeY, cubeZ);
    z = calculateZ(cubeX, cubeY, cubeZ) + distanceFromCam;

    ooz = 1 / z;

    xp = (int)(width / 2 + horizontalOffset + K1 * ooz * x * 2);
    yp = (int)(height / 2 + K1 * ooz * y);

    idx = xp + yp * width;
    if (idx >= 0 && idx < width * height) {
        if (ooz > zBuffer[idx]) {
        zBuffer[idx] = ooz;
        buffer[idx] = ch;
            }
        }
    }
    
    ```




---
[yutubelink] = https://www.youtube.com/watch?v=p09i_hoFdd0&t=36s   
[original_github_link] = https://github.com/servetgulnaroglu/cube.c
    





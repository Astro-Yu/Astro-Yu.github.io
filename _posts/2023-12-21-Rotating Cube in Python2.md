---
title: Rotating Cube in Python <2>
date: 2023-12-21 21:35:00 +09:00
categories: [PYTHON]
published: false
tags: [python, ascii, cube, mac]
image: /assets/spinningcube.png
---

# 2. Python으로 변경하기 전에...

이제 코드에 대한 이해는 끝났다. C 코드를 Python으로 옮겨적는 일만 남았다.   

같아야 할 부분과 달라질 부분을 정리해보면...

***같아야 할 부분***

1. 각종 수학 함수들   
2. 전역변수


**_달라져야 할 부분_**

1. buffer(buffer, zBuffer)들의 선언방식   
2. 컬러를 넣기 위한 부분
  
이 점에 유의하며 옮겨보았다.
    
# 3. Python으로 변경
  Link : [**여기에서 코드 전문 확인**](https://github.com/Astro-Yu/spinningCube)

  이걸 옮기면서 크게 바뀌는 부분은 없었기에 딱히 어렵진 않았다.   

  굳이 어려웠던 부분을 꼽자면 `buffer`와 `zBuffer`가 무엇을 뜻하는 것인지 헤맨 부분인데, [아무래도 이걸](https://en.wikipedia.org/wiki/Framebuffer) 말하는 것 같았다.

  그럼에도 달라진 부분을 소개하자면 원본 코드의 main 부분인데
  ```python
    while(True):
    
    buffer = [backgroundASCIICode] * (width * height)
    zBuffer = [0] * (width * height)

    cubeWidth = 20
    horizontalOffset = 0.5 * cubeWidth
    for cubeX in np.arange(-cubeWidth,cubeWidth,incrementSpeed):
        for cubeY in np.arange(-cubeWidth,cubeWidth,incrementSpeed):
            calculateOnSurface(cubeX,cubeY,-cubeWidth,'\033[95m' + '@' + '\033[0m')
            calculateOnSurface(cubeWidth,cubeY,cubeX,'\033[34m'+ '$' + '\033[0m')
            calculateOnSurface(-cubeWidth,cubeY,-cubeX,'\033[32m' + '~' + '\033[0m')
            calculateOnSurface(-cubeX,cubeY,cubeWidth,'\033[38;5;208m' + '#' + '\033[0m')
            calculateOnSurface(cubeX,-cubeWidth,-cubeY,'\033[38;5;159m' + ';' + '\033[0m')
            calculateOnSurface(cubeX,cubeWidth,cubeY,"+" + '\033[0m')

  ## 이하 생략
  ```

  굳이 정신사납게 큐브가 3개나 돌아가도록 하진 않고 하나만 돌아가는 대신 큐브의 면 마다 다른 색을 배정했다.


  ```python
  '\033[95m' + '@' + '\033[0m'
  ```

  이처럼 표현하고자 하는 Ascii 문자를 설정하고, 앞 뒤로 원하는 색에 대한 정보를 입력해주면 된다.   

  나는 [여기서](https://sosomemo.tistory.com/59) 출력에 색 입히는 법을 보고 했다. 문자 뿐만 아니라 배경에도 색을 넣을 수 있다고 하니 꽤나 유용할지도?







    





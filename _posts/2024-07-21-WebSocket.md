---
title: "[Web] 웹소켓 <1> "
date: 2024-07-21 01:10:00 +09:00 # 시간
categories: [WEB]
published: true
tags: [web, cs, websocket]
image: /assets/websocket1.png  
use_math: true
---
프론트 서버에서 실시간으로 API 정보를 전달받고 싶어서 **웹소켓(Web Socket)** 기술에 대해 알아보았다.

## 1. 웹소켓이란?

웹소켓(Web Socket)은 HTML5 사양의 일부로, 클라이언트와 서버 간의 **양방향 통신**을 가능하게 하는 프로토콜이다.

단순히 클라이언트와 서버 간의 통신만 한다면 평소에 사용하던 HTTP 통신과 다를 바 없다. 굳이 WebSocket을 사용하는 이유는 무엇일까?

## 2. 특징

웹 소켓이 차별화 될 수 있는 특이점은 다음과 같다.

### 2.1 웹소켓의 특징

1. **양 방향 통신(Full Duplex)**
    
    내가 웹 소켓을 사용하려는 이유 중 가장 큰 비중을 차지하는 특징이다. 기존의 HTTP 방식에선 반드시 클라이언트가 GET 요청을 날려야 서버에서 Response가 돌아왔다. (**단방향 통신**)
    
    웹 소켓을 사용하면 클라이언트 뿐만 아니라, **서버가 먼저 Response**를 보낼 수 있다.
    
2. **단일 연결(Single Connection)**
    
    한번 GET 요청을 통해 웹소켓 연결이 성립되면, 동일한 TCP 연결을 통해 데이터를 주고받아 기존 HTTP와 달리 새로운 연결을 매 요청마다 실시할 필요가 없다.
    
3. **낮은 지연시간**
    
    웹소켓은 헤더 정보의 크기를 최소화하고, 연결과 해제에 드는 시간을 줄여 실시간 통신에서 높은 성능을 보여준다.
    

### 2.2 웹소켓 동작 방식
![](/assets/websocket1.png)
[사진출처](https://johnie.dev/computer-science/network/web_socket-VS-server_sent_events/)

1. **핸드 쉐이크**
    
    웹소켓 통신은 반드시 GET 요청을 통해 시작된다. 클라이언트는 서버에 HTTP 요청을 보내고, 이 요청에 특정 헤더를 포함하고 있으면 서버는 **HTTP 101 Switching Protocols** 응답을 보낸다.
    
    **HTTP 요청 (클라이언트 to 서버)**
    
    ```
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== //(클라이언트가 서버에 보내는 보안을 위한 임의의 고유 키)
    Sec-WebSocket-Protocol: chat, superchat // (클라이언트와 서버 간의 서브프로토콜 정의)
    Sec-WebSocket-Version: 13
    
    ```
    
    Upgrade: websocket를 통해 웹소켓 요청 통신이라는 것을 알린다.
    
    **HTTP 101 Switching Protocols 응답 (서버 to 클라이언트)**
    
    ```
    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
    Sec-WebSocket-Protocol: chat
    ```
    
2. **WS frame 통신**
    
    1번의 핸드쉐이크 과정이 끝나면 프로토콜이 ws로 변경된다.
    
    이후 클라이언트와 서버는 **frame** 이라는 단위로 통신하게 된다.
    
    **Frame의 구조**
    
    Frame은 웹소켓 커뮤니케이션에서 사용되는 가장 작은 단위이다. Frame은 서버와 클라이언트의 메시지를 작은 조각으로 나눈 것이다. 이 Frame이 모여 전체 메시지를 이루게 된다.
    
    Frame은 작은 헤더와 내용인 Payload로 나누어져 있다.
    
    **Frame Header의 구조**
    
    ![](/assets/websocket2.jpeg)
    [사진출처](https://alnova2.tistory.com/915)
    
    - FIN : 현재 프레임이 메시지의 마지막인지 아닌지 나타낸다
    - Opcode : 현재 프레임의 종류를 나타낸다.
        - 0x0 : 전체 메세지의 일부임
        - 0x1 : 포함된 데이터가 UFT-8 텍스트임
        - 0x2 : 포함된 데이터가 바이너리 데이터임
        - 0x3 ~ 7 : 현재 사용되지 않는 값. 이 값이 존재한다면 잘못된 값이다.
        - 0x8 : 연결 종료
        - 0x9 : 핑 프레임. 현재 연결이 유효한지 확인용
        - 0xA : 퐁 프레임. 핑 프레임의 대한 응답으로 사용
        - 0xB ~ F : 현재 사용되지 않는 값. 이 값이 사용된다면 잘못된 값이다.
    - Payload Length : 페이로드의 길이를 나타낸다. 7비트에서 초과될 경우, 추가 비트를 사용해 확장한다.
    
    **Payload Data**
    
    데이터 유형별로 UTF-8 혹은 바이너리 데이터가 주어진다.
    

### 2.3 포트

포트는 ws://(80) , wss://(433)을 사용한다.
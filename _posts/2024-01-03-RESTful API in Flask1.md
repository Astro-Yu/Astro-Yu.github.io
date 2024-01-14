---
title: "[Flask] RESTful API 구현 <1>"
date: 2024-01-03 05:40:00 +09:00 # 시간
categories: [Flask]
published: true
tags: [python, flask, web, backend, server]
image: /assets/flask.png
---
## 0. 목표

- Python Flask를 활용해 작동하는 RESTful API를 제작한다.
- 웹 기술에 대해 알아보고 이해한다.

---

## 1. 왜 Flask인가?

친구랑 간단하게 웹 프로젝트를 진행하고 있는데 웹 백엔드로 무엇을 써야할지 고민했다.

최근에 열심히 했다곤 하지만 아무래도 익숙하지 않은 Java(Spring) 보단 Python(Django, Flask, FastAPI)로 구현해보는게 낫다고 판단했다.

그렇다면 많은 기능을 지원하지만 본격적이고 무거운(사실 무겁다고만 들은) Django 보단 가볍게 공부해보고 실행할 Flask가 적합하다고 판단했다. 대충 검색해서 찾아본 Flask의 장점은…

1. 가볍다.
2. 코드가 짧고 쉽다!! <span style="color:red"> ***(중요)*** </span>
3. 마이크로 서비스에 적합하다. (잘 모름)

정도인데 뭐 간단하게 하려면 적당해 보였다. 애초에 Django나 Spring처럼 너무 본격적이면 다루기 부담스럽다.

---

## 2. RESTful API가 뭔데

그래서 RESTful 한 API가 뭔데? REST와 API를 나누어서 알아보자.

### 2.1 REST

***REST*** 는 Representational State Transfer의 약자로 API의 작동방식에 특정 조건을 부여하여 자원을 표현하는 방식이라고 한다.

여기서 말하는 특정 조건이란? 바로 ***HTTP URI*** 다. URI란 Uniform Resource Identifier의 약자다. 이름에서 알 수 있듯이 균일할 것을 요구하고 있다. 웹 사이트의 모든 것들 (텍스트, 이미지 데이터 등)에게 URI를 부여하여 관리한다.

그래서 구체적으로 뭘 요구하냐? 해당 자원들에 대해 CRUD를 적용해 자원에 대한 행위를 4가지로 표현하게 한다. 여기서 CRUD란

> **C** = **C**reate. 리소스를 생성하는 행위를 한다.
> 
> **R** = **R**ead. 리소스를 조회하고, 정보를 가져온다.
> 
> **U** = **U**pdate. 리소스를 수정한다.
> 
> **D** = **D**elete. 리소스를 삭제한다.
> 

이고 이것들을 Flask에선 각각 POST, GET, PUT, DELETE로 제어한다.

### 2.2 API

API란 Application Programming Interfaced의 약자로 응용 프로그램에서 사용 가능하도록 운영체제나 프로그래밍 언어로 제어하게 만든 인터페이스를 말한다.

즉 유저에게서 적당한 input을 받아 이것을 API에서 처리한 후 서버로 보낸다. 이후 서버에서 보내온 응답을 다시 API에서 처리해 다시 유저에게 돌려준다. 이것을 매개하는 것이 API라고 이해했다.

이때 데이터들을 모두 json의 형태로 주고받는듯 하다. 특별한 이유가 있는지?

몇몇 홈페이지에서 카카오 지도나 네이버 지도를 사용하는 것을 볼 수 있는데 이것은 카카오나 네이버에서 제공하는 지도 API를 사용한 것이다.

그림으로 표현해보자면 다음과 같다.

![](/assets/RESTAPI.jpeg)

위에서 설명한 REST한 규칙을 API에 적용시켜 확장성을 추구한것이 RESTful API라고 볼 수 있다. 애초에 요즘 API들은 다 REST 규칙을 따르게 만들어진다고 하는듯.

---

## 3. 구체적으로 할 일

당장 생각나는건 적당한 input을 받고, 모델에서 input을 처리 후 반환되는 값을 다시 표시하는 정도를 목표로 했다. 더 생각나는 점이 있으면 추가하면 되고…

즉 필요한 것은 다음과 같다.

- 테스트용 적절한 Model
- 사용자 입력 인터페이스
- 출력화면
- Flask 서버

코드 작성과 실행은 다음 글부터 해볼 예정이다.
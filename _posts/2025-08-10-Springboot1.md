---
title: "[SpringBoot] 실시간 알림을 위한 SSE, 트러블 슈팅"
date: 2025-08-10 16:50:00 +09:00 # 시간
categories: [SrpingBoot]
published: true
tags: [woowa, java, springboot, troubleshooting, sse, web]
image: /assets/spring.webp
use_math: true
---  

## 서론

우아한 테크코스 Level3 과정을 진행하면서 처음으로 팀 프로젝트에 참여하고 있다.

프로젝트 도중 우리 사용자에게 실시간 알림을 보내야 할 필요성이 생겼고, 내가 이 알림 기능을 맡게 됐다.

약 3~4일 동안 구현하면서 정리한 지식과 트러블 슈팅을 기록하고자 한다.

## HTTP 통신이 실시간 알림에 부적합한 이유

먼저 생각한 것은 **`기존 HTTP 통신으로 실시간 알림을 보낼 수는 없는가?`** 였다. 할 수만 있다면 익숙한 기술로 빠르게 구현하는 것이 베스트라고 생각했다. 먼저 실시간 알림의 흐름을 정리해보자.

1. 사용자 A가 글을 작성, DB에 저장
2. 댓글 작성자 B가 글에 댓글을 작성, DB에 저장. (`트리거`)
3. 댓글이 BD에 저장되는 순간, A에게 알림 전송.

HTTP 프로토콜의 가장 큰 특징은, 클라이언트의 요청 없이 서버에서 응답을 보낼 수 없다는 것이다. 하지만 실시간 응답의 트리거는 서버 쪽에 있다. HTTP 통신으로는 요청이 없이 A에게 응답을 보낼 수는 없다.

즉 다른 방식의 통신 기술이 필요했다.

## 실시간 알림을 위한 기술들 + SSE를 선택한 이유.

먼저 요청 없이 응답을 보내기 위한 기술들을 찾아봤다. `Polling`, `WebSocket`, `SSE`를 후보에 올려두었다. 

### Polling

앞에서 HTTP 통신으로는 실시간 알림 구현이 불가능하다고 했지만, 사실 방법이 있다. 바로 짧은 주기마다 새로고침을 해서 Get 요청을 보내는 방법이다. 이것을 기술로 구현한 것이 `Polling` 이다.

Polling의 장단점은 명확하다. 구현하기가 쉽지만, 주기적으로 클라이언트의 요청을 받아야 해서 서버에 무리가 갈 수 있다. (수 많은 api 요청…) 여러 사용자가 동시에 사용하게 되면 HTTP 커넥션을 모두 소모할 수도 있다는 단점이 있다.

### WebSocket

아무래도 실시간 통신이라면 `WebSocket`이 가장 유명할 것이다. 나 또한 예전에 Python으로 구현해 본 경험이 있다.

자세한 설명은 블로그 링크로 대체하고 여기서는 장단점만 다뤄보겠다. [웹소켓](https://astro-yu.github.io/posts/WebSocket/)

`WebSocket`의 가장 큰 특징은 양방향성에 있다. 덕분에 많은 분야에서 유연하게 사용 가능하다. 특히 채팅 구현에는 필수적이다.

하지만 학습 곡선이 크고, 단순히 서버에서 트리거되는 알림 기능에는 오버 엔지니어링이라 생각해 도입하지 않았다.

### SSE

사실 HTTP 통신으로 실시간 통신을 하는 방법이 하나 더 있다. 바로 `SSE`다.

`SSE`는 `Server Sent Event`의 약자이다. 말 그대로 서버에서 보내는 이벤트이다. 

클라이언트에서 최초 한 번 Get 요청을 통해 SSE 연결을 시작하면, 서버 쪽에서 연결이 끊기기 전까지는 계속 데이터를 전송할 수 있게 된다. 즉

`Polling` 처럼 주기적인 요청 - 응답을 하지 않아 서버와 네트워크에 부담도 가지 않으며,

`WebSocket` 처럼 양방향도 아니면서 러닝커브가 낮게 구현 가능하다. 그래서 채택하게 됐다.

SSE 통신은 다음과 같은 흐름으로 이루어진다.

1. 클라이언트 쪽에서 최초에 서버로 SSE 통신을 시작하고자 알린다. 이때 요청 헤더에 `Accept: text/event-stream` 를 포함한다. 일반적인 웹 페이지나 json이 아닌, 주기적인 text를 받기 원한다는 의미이다.
2. 서버에서는 해당 요청을 받고 헤더에 `Content-Type: text/event-stream` 를 포함하여 응답한다. 중요한 점은 응답 후에 HTTP 연결을 끊지 않는다는 것이다.
3. 이제 서버는 클라이언트 쪽으로 요청 없이 응답을 보낼 수 있다.

## SpringBoot에서의 구현

Java에서는 `SseEmitter API`를 제공해 손쉽게 `SSE`를 구현할 수 있다.

### Subscribe

먼저 클라이언트가 SSE 통신을 요청하는 컨트롤러 메서드를 만든다. 이것을 `Subscribe`라고 한다.

```java
@GetMapping(value = "/subscribe", produces = "text/event-stream")
    public SseEmitter subscribe(@AuthenticationPrincipal Authentication authentication, HttpServletResponse response) {
        response.setHeader("X-Accel-Buffering", "no");
        return sseNotificationService.subscribe(authentication.id());
    }
```

`SseEmitter`는 라디오같은 역할을 한다. 클라이언트에게 준 후, 라디오를 통해 우리가 원하는 데이터를 계속 전송할 수 있다.

헤더에 
`response.setHeader("X-Accel-Buffering", "no");`

같은 코드가 추가됐는데, 이는 추후에 설명하도록 한다.

### sseNotificationService.subscribe()

서비스에서는 구독시, sseEmitter를 만든 후 여러 처리를 해서 내보낸다.

```java
		private static final long VALID_TIME = 10 * 60 * 1000L;
		
		// 동시성을 고려하여 Thread-safe한 map 자료구조인 ConcurrentHashMap를 사용한다.
		private final Map<Long, SseEmitter> emitters = new ConcurrentHashMap<>();

    public SseEmitter subscribe(Long userId) {
		    
		    // SseEmitter를 유효시간을 설정한 후 생성한다.
        SseEmitter emitter = new SseEmitter(VALID_TIME);
				
        emitters.put(userId, emitter);
				
        // 서버 쪽에서 클라이언트 쪽으로 지정한 VALID_TIME 동안 아무 데이터도 전송하지 않으면 발동한다.
        // 만약 한번이라도 데이터를 보내게 되면, 타이머는 초기화된다.
        // 아무도 사용하지 않게 된 유령 emitter를 정리하는 역할
        emitter.onTimeout(() -> emitters.remove(userId));
	       
	      // 연결이 완료되는 모든 경우. onTimeoout을 포함하고 있음.
	      // 정상적인 완료, 클라이언트측의 종료, 네트워크 오류, 타임아웃 등등 시 발동한다.
	      // 메모리 누수 방지를 위한 확실한 뒷정리
        emitter.onCompletion(() -> emitters.remove(userId));
				
				// 연결 직후 아무 데이터나 보내준다.
        sendToClient(userId, "connect", "Connection success");

        return emitter;
    }
    
    // 클라이언트에게 실제 데이터를 전송한다.
    public void sendToClient(Long userId, String eventName, Object data) {
		    // id로 emitter를 찾아온다.
        SseEmitter emitter = emitters.get(userId);
        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event()
                        .name(eventName) // event의 이름을 설정한다.
                        .data(data)); // data를 전송한다.
            } catch (IOException e) {
                emitters.remove(userId); // 어떠한 에러가 난다면 emitter를 삭제한다.
            }
        }
    }
```

이후 코드 내에서 `sendToClient`  메서드를 사용해 원하는 data를 보내줄 수 있다.

### Thread-Safe한 자료구조를 사용해야 하는 이유.

이유는 하나의 `emitter`에 동시에 여러 스레드가 접근할 수 있기 때문이다. 예를 들어서 설명하자면.

1. A가 게시글을 작성한다.
2. B와 C가 게시글에 거의 동시에 댓글 작성을 한다.
3. `sendToClient()`  내부의 get() 메서드가 동시에 작동한다…? 

위 예시는 읽기를 예로 들어서 큰 문제는 없을 수 있지만, 만약 어떤 에러가 나서 emitter가 삭제된다면 다른 스레드에서 사용하는 emitter에서는 nullException이 나올 것이다.

## 트러블 슈팅

### DB 커넥션 풀 고갈 문제 (feat. OSIV)

***문제 상황:***

구현을 완료하고 프론트엔트와 연결 테스트를 하던 도중 기능이 안되면서 계속 이런 에러가 발생했다.

```bash
Caused by: java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30001ms (total=10, active=10, idle=0, waiting=6)
```

 읽어보니 DB 커넥션 풀을 모두 소모해 DB 작업을 할 수 없게 됐다. 

***해결 방법:***

- 1차 시도: 트랜잭션 내에서 SSE 통신 제외하기 (실패)

처음엔 `@Transactional`을 달아둔 메서드 내에서 SSE 통신을 하고 있는 부분이 있어 문제라고 생각해, 해당 부분을 주석 처리하고 다시 시도했다. 실패했다.

- 2차 시도: OSIV false로 설정하기 (성공)

따로 설정을 해주지 않는다면, 스프링의 기본 설정은 open session in view: true 이다. 이 설정이 true로 되어있다면, db작업을 하지 않아도 view 레벨까지 db 커넥션이 주어진다.

우리는 어차피 DTO로 응답을 내려주고 있기 때문에, view 레벨까지 영속성 컨텍스트가 존재할 이유는 없다. 오히려 있다면 10명의 클라이언트가 SSE 통신 요청 시, 모든 커넥션 풀을 사용해 더 이상 DB 작업을 할 수 없게 된다. 해당 설정을 꺼주니 잘 동작하게 됐다.

### nginx 설정 문제: Proxy buffering, Http/1.0 → Http/1.1

nginx에는 여러 기능이 있지만, 우리는 ssl 인증서를 발급받아 https 통신을 사용하기 위해 도입했다.

문제 상황:

구독 시에 서버 로그에서는 제대로 200 응답을 내보내고 있었지만, 클라이언트 쪽에서는 500, 혹은 pending이 계속되다가 취소되었다. 덕분에 스프링 앱이나 클라이언트의 문제가 아닌, 그 중간 네트워크 단계의 문제라고 인식했다.

***해결 방법:***

1. **Proxy Buffering**
    
    우리의 프로젝트에서 nginx는 프록시(대리자) 서버 역할을 한다. 클라이언트에서 오는 요청이 바로 스프링 서버로 들어가는 것이 아닌, nginx를 거쳐서 온다. 반대도 똑같다.
    
    문제는 이 nginx의 기본 작동 방식이 버퍼링이라는 것이다.
    
    기본적으로 nginx는 요청이나 응답을 받으면, 그것이 끝나기까지 기다린다. 데이터를 모두 받은 다음, 그것을 다음 단계로 전달한다. 서버의 부하를 줄일 수 있는 현명한 방법이지만, 응답이 끊기질 않는 SSE에는 맞지 않는다.
    
    지정한 시간동안 응답이 끊기질 않는다면, nginx는 해당 연결을 잘못된 응답이라고 판단해 끊어버린다. 따라서 클라이언트 측에선 이유를 알 수 없는 에러 응답만 받게 된다.
    
    nginx의 Accel-X 기능을 사용해, 응답 헤더에 
    
    `response.setHeader("X-Accel-Buffering", "no");`
    
    를 달아주는 것으로 버퍼링 문제를 해결할 수 있다.
    
2. **Http 1.0 문제**
    
    Proxy 서버는 기복적으로 Http/1.0 프로토콜을 사용한다. 주요한 특징은 하나의 요청의 하나의 응답을 보내고 끊는 것이다. 따라서 SSE와 같은 스트리밍 기능에서는 사용할 수 없다.
    
    SSE를 사용하려면 Http/1.1 를 설정해 주어야 한다.
    
    `nginx.conf` 파일이나, stie-availables 폴더 내의 `default` 를 내부를 수정해준다.
    
    ```yaml
    proxy_set_header Connection '';
    proxy_http_version 1.1;
    ```
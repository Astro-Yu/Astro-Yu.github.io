---
title: "[Refactoring] 메서드 하나에 너무 많은 책임이 있다면? Facade와 Event Listener로 리팩토링하기"
date: 2025-09-28 16:30:00 +09:00 # 시간
categories: [Refactoring]
published: true
tags: [woowa, java, springboot, refactor, facade, eventlistener]
image: /assets/refactor.png
use_math: true
---  

이 글에서는 점점 비대해지는 메서드를 어떻게 분리하고, 유지보수하기 좋은 코드로 만들 수 있는지에 대한 리팩토링 과정을 공유하려 한다.

특정 기능 하나를 실행했을 뿐인데, 그 안에서 너무 많은 일이 벌어지고 있다. **Facade 패턴**과 **Spring Event Listener**를 활용해 해결한 과정을 공유한다.

**대상 독자**
* 하나의 메서드가 너무 많은 일을 해서 분리하고 싶은 개발자
* 느슨한 결합(Loosely Coupled)에 대해 고민하는 개발자
* Spring Event Listener를 도입하고 싶은 개발자
* Facade 패턴으로 서비스 로직을 명확하게 분리하고 싶은 개발자

---

## 1. 문제 상황: 댓글 하나 달았을 뿐인데...

우리 서비스에는 특정 도메인 이벤트가 발생했을 때, 연쇄적으로 처리해야 하는 부가 기능들이 있다.

* **댓글 작성 시**: ① 댓글 저장 ② 리워드 적립 ③ 알림 저장 및 전송

여기서 '리워드 적립', '알림 전송' 등은 댓글이 성공적으로 저장된 후에 일어나야 하는 **부가적인 책임**이다. 하지만 현재 코드는 이 모든 책임을 `addComment`라는 메서드 하나가 전부 짊어지고 있다.

**[BEFORE] 문제의 `addComment` 메서드**
```java
@Transactional
public CommentCreateResponse addComment(CommentCreateRequest request, Long commenterId) {
    // 1. 댓글 생성 및 저장 (핵심 책임)
    User commenter = userQueryService.getUserById(commenterId);
    Moment moment = momentQueryService.getMomentById(request.momentId());
    if (commentQueryService.existsByMomentAndCommenter(moment, commenter)) {
        throw new MomentException(ErrorCode.COMMENT_CONFLICT);
    }
    Comment commentWithoutId = request.toComment(commenter, moment);
    Comment savedComment = commentRepository.save(commentWithoutId);
    Optional<CommentImage> commentImage = commentImageService.create(request, savedComment);
	
    // 2. 알림 전송 (부가 책임 1)
    notificationFacade.sendSseNotificationAndNotification(...);
		
    // 3. 리워드 적립 (부가 책임 2)
    rewardService.rewardForComment(...);

    return CommentCreateResponse.of(savedComment, commentImage);
}
```
### 무엇이 문제인가?

이 코드는 여러 가지 문제를 안고 있다.

* **낮은 응집도와 높은 결합도**: `addComment` 메서드가 댓글, 알림, 리워드라는 서로 다른 컨텍스트의 책임을 모두 가지고 있다. 이로 인해 알림이나 리워드 정책이 변경될 때마다 댓글 로직을 수정해야 한다. **(SRP 위반)**
* **유지보수성 저하**: 새로운 부가 기능(e.g., 활동 로그 남기기)이 추가되면 이 메서드를 또 수정해야 한다. 기능 추가에 열려있고, 수정에는 닫혀 있어야 한다는 **OCP 원칙**을 지키기 어렵다.
* **잘못된 트랜잭션 범위**: **알림 전송에 실패하면, 성공했어야 할 댓글 저장까지 모두 롤백**된다. 알림은 부가 기능일 뿐인데, 핵심 기능의 성공 여부에 영향을 미치고 있다.
* **성능 저하**: 알림 전송 로직(외부 시스템 연동 등)이 오래 걸리면, 댓글 작성 API의 전체 응답 시간이 길어진다.

---

## 2. 해결 전략: 책임과 트랜잭션 분리하기

문제를 해결하기 위한 핵심 전략은 **"책임의 성격에 따라 분리하고, 올바른 도구를 사용한다"** 이다.

1.  **동기적이고 강한 일관성이 필요한 책임**: `댓글 저장`과 `리워드 적립`. 댓글이 저장되면 리워드는 **반드시** 함께 저장되거나, 실패 시 함께 롤백되어야 한다. → **Facade 패턴**으로 묶어 하나의 트랜잭션으로 관리한다.
2.  **비동기적이고 최종 일관성으로 충분한 책임**: `알림 전송`. 댓글 저장이 성공한 후에 비동기적으로 처리되어도 괜찮다. 실패하더라도 댓글 저장에 영향을 주면 안 된다. → **Spring Event Listener**로 분리하여 별도의 트랜잭션으로 관리한다.

### 2.1. 도구 소개: Facade 패턴
Facade 패턴은 복잡한 서브시스템들을 더 쉽게 사용하기 위한 상위 레벨의 인터페이스를 제공하는 패턴이다. 이 패턴을 **"비즈니스 유스케이스를 표현하는 서비스 계층"**으로 활용할 것이다.

* **CommentService**: 순수하게 댓글 CRUD만 책임진다.
* **RewardService**: 순수하게 리워드 로직만 책임진다.
* **CommentFacade**: `댓글 작성`이라는 유스케이스를 위해 `CommentService`와 `RewardService`를 조율한다. 컨트롤러는 Facade만 호출하면 된다.

### 2.2. 도구 소개: Spring Event Listener
이벤트 기반 프로그래밍을 통해 시스템의 결합도를 낮추는 강력한 도구다.

* **Publisher (발행자)**: "나 댓글 다 만들었어!" 라고 이벤트(사실)를 외친다.
* **Listener (구독자)**: 그 소리를 듣고 "아, 댓글이 만들어졌구나. 그럼 난 알림을 보내야지" 라며 각자 할 일을 한다.

발행자는 구독자가 누구인지, 무엇을 하는지 전혀 알 필요가 없다. 덕분에 결합도가 극적으로 낮아진다.

> ** 잠깐! `@EventListener` vs `@TransactionalEventListener`**
>
> 그냥 `@EventListener`는 트랜잭션의 성공 여부와 관계없이 이벤트 발행 즉시 실행된다. 하지만 **"댓글 저장이 성공적으로 DB에 Commit된 후에만"** 알림을 보내야 한다.
>
> 이럴 때 `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)`을 사용하면, 트랜잭션이 성공적으로 커밋될 때만 리스너가 동작하도록 보장할 수 있다. 데이터 정합성을 지키기 위한 필수 옵션이다.

---

## 3. 최종 리팩토링 결과

위 전략에 따라 완성된 최종 코드다.

### Step 1: CommentFacade - 동기 작업 조율
컨트롤러가 호출할 진입점이다. 댓글 생성과 리워드 적립을 하나의 트랜잭션으로 묶고, 마지막에 비동기 처리를 위한 이벤트를 발행한다.

```java
@Service
@RequiredArgsConstructor
public class CommentFacade {

    private final CommentService commentService;
    private final RewardService rewardService;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public CommentCreateResponse addComment(CommentCreateRequest request, Long commenterId) {
        // 1. 순수 댓글 서비스 호출하여 댓글 생성
        Comment savedComment = commentService.createComment(request, commenterId);

        // 2. 리워드 서비스 호출 (같은 트랜잭션)
        rewardService.rewardForComment(savedComment.getCommenter(), Reason.COMMENT_CREATION, savedComment.getId());

        // 3. "댓글 생성 완료" 이벤트 발행 (트랜잭션 커밋 후 리스너가 처리)
        eventPublisher.publishEvent(CommentCreatedEvent.of(savedComment));

        // 이미지 처리 로직이 있다면 여기에...
        return CommentCreateResponse.from(savedComment);
    }
}
```

### Step 2: CommentCreatedEvent - 발행할 이벤트 정의
리스너에게 필요한 최소한의 데이터(ID)만 담은 불변 객체(Record)로 이벤트를 정의한다.
``` java
public record CommentCreatedEvent(
    Long momentId,
    Long momenterId,
    Long commenterId,
    Long commentId
) {
    public static CommentCreatedEvent of(Comment comment) {
        return new CommentCreatedEvent(
            comment.getMoment().getId(),
            comment.getMoment().getMomenter().getId(),
            comment.getCommenter().getId(),
            comment.getId()
        );
    }
}
```
### Step 3: NotificationEventHandler - 비동기 작업 처리
이벤트를 구독하여 실제 알림 로직을 처리한다. @Async와 @TransactionalEventListener를 통해 완벽히 분리된 환경에서 실행된다.

``` java
@Component // @Service, @Component 등 스프링 빈으로 등록
@RequiredArgsConstructor
public class NotificationEventHandler {

    private final NotificationFacade notificationFacade;
    private final UserQueryService userQueryService;

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleCommentCreatedEvent(CommentCreatedEvent event) {
        // ... 이벤트에 담긴 ID로 필요한 엔티티를 다시 조회 ...
        User momenter = userQueryService.getUserById(event.momenterId());

        // 알림 전송 로직 수행 (별도 트랜잭션, 별도 스레드)
        notificationFacade.sendSseNotificationAndNotification(...);
    }
}
```

## 4. 결론: 무엇을 얻었나?
이번 리팩토링을 통해 코드는 다음과 같은 이점을 얻게 되었다.
| 항목 | BEFORE | AFTER |
| :--- | :--- | :--- |
| **응집도/결합도** | 낮음 / 높음 | **높음 / 낮음** |
| **트랜잭션 관리**| 단일 트랜잭션 (위험) | **책임에 따라 분리 (안전)** |
| **유연성 (OCP)** | 기능 추가 시 직접 수정 | **새로운 리스너 추가로 확장** |
| **성능**| 동기 처리로 응답 지연 | **비동기 처리로 응답 시간 단축** |

물론 이벤트 리스너 방식은 IDE에서 호출 관계를 바로 추적하기 어려워 코드의 흐름을 파악하기 힘들다는 단점이 있다. 하지만 EventHandler 같은 네이밍 컨벤션을 지키고, 이벤트 클래스를 잘 관리한다면 충분히 극복할 수 있다.

무엇보다 책임과 관심사에 따라 코드를 분리함으로써 얻는 유지보수성과 확장성의 이점이 훨씬 크다고 생각한다.
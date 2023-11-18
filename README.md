# [인프런] 실습으로 배우는 선착순 이벤트 시스템

## 강의 소개
선착순 이벤트 쿠폰을 100장을 생성하여 발급할 때 발생할 수있는
문제점과 해결 방법

## 발생할 수 있는 문제점
- 쿠폰이 100개보다 많이 발급될 수 있다.
- 이벤트 페이지 접속이 안될 수있다.
- 이벤트와 전혀 상관 없는 페이지도 느려질 수 있다.

## 문제 해결
- 트래픽이 몰렸을 때 대처할 수 있는 방법을 배운다.
- redis를 활용하여 쿠폰 발급개수를 보장한다.
- kafka를 활용하여 다른 페이지들에 대한 영향도를 줄인다.

## 다루지 않는 것
- coupon 도메인
------------

# 정리

### 요구사항 정의
```agsl
선착순 100명에게 할인쿠폰을 제공하는 이벤트를 제공한다.

이 이벤트는 아래와 같은 조건을 만족하여야 한다.
- 선착순 100명에게만 지급되어야 한다.
- 101개 이상이 지급되면 안된다.
- 순간적으로 몰리는 트래픽을 버틸 수 있어야 한다.
```
### 100개의 쿠폰을 발급해주는 로직의 문제

```java
    @Test
    void 여려명_응모() throws InterruptedException {
        int threadCount = 1000;

        ExecutorService executorService = Executors.newFixedThreadPool(32); 
        CountDownLatch latch = new CountDownLatch(threadCount);

        for (int i = 0; i < threadCount; i++) {
            long userId = i;
            executorService.submit(() -> {
                try {
                    applyService.apply(userId);
                } finally {
                    latch.countDown();
                }
            });
        }

        latch.await();

        long count = couponRepository.count();

        assertThat(count).isEqualTo(100); // false 쿠폰이 110개가 생김
    }
```
멀티 스레드로 1000개의 요청을 하였을 때 테스트 케이스가 실패하는 것을 확인할 수 있다.
쿠폰이 100개 이상이 발급되버리는 문제가 발생한다.

### 레이스 컨디션

두 개 이상의 스레드가 공유 자원을 서로 사용하려고 경합하는 현상을 의미한다.

### 레디스를 사용하여 해결해보자
























# dirty checking이란?

JPA의 영속성 컨텍스트가 엔티티가 처음 들어올때(조회, 저장할 때) 그 상태를 있었던 스냅샷를 찍어놓고, flush() 발생시 이전에 기록한 스냅샷을 기준으로 현재 엔티티의 상태와 비교해서 update query를 만들어서 데이터베이스에 반영하는것이다.

## 동작 흐름

1. 엔티티 조회 -> 1차 캐시에 엔티티 + 스냅샷 저장
2. setter/도메인 메서드로 값 변경 -> 메모리(1차 캐시)의 값만 바뀜
3. flush 발생 -> 스냅샷과 비교, 변경된 엔티티의 UPDATE SQL 생성
4. 쓰기 지연 SQL 저장소의 SQL을 DB로 전송
5. 커밋 후, 확정


## flush란?

영속성 컨텍스트의 변경 내용을 SQL로 DB에 전송하는 것.

- 발생 시점: 트랜잭션 커밋 / JPQL 실행 직전 / `em.flush()` 직접 호출
- flush != commit: DB에 실행은 됐지만 커밋 전이므로 다른 트랜잭션에서는 보이지 않고, 롤백도 가능하다.
- flush를 해도 1차 캐시는 비워지지 않는다. (비우는 건 `em.clear()`)

### 그럼 직접 flush가 필요한 경우는 어떤 경우냐. 

 JPA 조회는 1차 캐시나 자동 flush로 변경이 보이므로 대부분 불필요하다. 다만 DB 제약 조건 위반을 트랜잭션 안에서 바로 잡아야 할 때(saveAndFlush + try-catch), JdbcTemplate 등 영속성 컨텍스트를 거치지 않는 접근이 같은 트랜잭션에 섞일 때, 트리거나 기본값처럼 DB가 만든 값을 읽어야 할 때, 대량 처리에서 flush + clear로 메모리를 관리할 때 사용한다.


 ## SaveAndFlush()를 쓰는 경우

Spring Data JPA의 `saveAndFlush()` = `save()` + `flush()`. 저장 후 바로 SQL을 DB로 보낸다.

**쓰는 경우**
- DB 제약 조건 위반(유니크 키 등)을 그 자리에서 잡아 커스텀 예외로 바꿔야 할 때
- 저장 직후 같은 트랜잭션에서 JdbcTemplate 등으로 DB 상태를 읽어야 할 때

```java
try {
    memberRepository.saveAndFlush(member);
} catch (DataIntegrityViolationException e) {
    throw new BusinessException(ErrorCode.CUSTOME_ERROR);
}
```

**주의**

- 이미 영속 상태인 엔티티는 save()가 필요 없다. 변경 감지로 반영된다.
- 예외를 잡아도 트랜잭션은 rollback-only로 표시되어 커밋할 수 없다. 잡아서 커스텀 예외로 다시 던지는 용도이지, 잡고 나서 정상 흐름을 이어가는 용도가 아니다.
- 동시성 문제의 해결책이 아니다. flush는 SQL 전송 시점만 앞당길 뿐 커밋이 아니고, 판단은 이미 조회 시점에 끝났기 때문이다. -> 조건부 UPDATE, 락으로 해결


### 부연설명을 좀 하자면 

1. findById()로 꺼낸 엔티티(= 이미 영속 상태)는 이미 JPA가 지켜보고 있는 상태여서 값을 바꾸기만 하면 커밋할 때 JPA가 알아서 UPDATE를 보내니까 save()를 또 부를 필요가 없다!


```java
Member m = repo.findById(1L).get(); // JPA가 지켜보기 시작
m.changeName("길동이");
// repo.save(m);   불필요!
```

&emsp;

"예외를 잡아도 커밋할 수 없다"

로직 중간에 DB 에러가 나면 트랜잭션 전체가 실패로 표시된다. try catch로 잡아서 코드를 계속 진행하더라도, 메서드가 끝나 커밋을 시도할 때 앞에서 성공한 작업까지 모두 롤백되고 `UnexpectedRollbackException` 라는 에러가 발생하게 된다. 트랜잭션의 전부 성공하거나 전부 실패해야 한다는 ACID의 원자성(Atomicity) 때문이다. 그래서 catch는 에러를 커스텀 예외로 바꿔 다시 던지는 데만 쓴다.

&emsp;

"동시성 문제의 해결책이 아니다"

saveAndFlush()는 SQL을 DB에서 실행하는 시점만 앞당길 뿐, 커밋(확정)은 트랜잭션이 끝날 때 일어난다. 커밋 전의 변경은 다른 트랜잭션에서 보이지 않기 때문에 동시 요청 문제는 막지 못한다.
saveAndFlush()는 SQL을 DB로 보내는 시점만 앞당길 뿐이지 커밋이 아니기 때문에. 그래서 동시 요청 문제는 막지 못한다.

예를 들어 보상을 한 번만 지급해야 하는 로직에서 요청 A와 B가 동시에 들어오면,

1. A와 B가 동시에 "지급받았나?"를 조회하고, 둘 다 false를 읽는다.
2. 둘 다 "지급해야지"라고 판단한다.
3. 이후 saveAndFlush로 빨리 저장해도, 이미 둘 다 지급하기로 결정한 뒤라 중복 지급이 발생한다.

문제는 저장 속도가 아니라 "조회 후 판단 후 저장"이 한 번에 처리되지 않는다는 점이다. 

이 문제를 나는 확인과 저장을 하나의 쿼리로 합친 조건부 UPDATE로 해결했었다.

```sql
UPDATE reward SET rewarded = true WHERE id = n AND rewarded = false
```

DB가 행 락을 잡은 상태에서 조건 확인과 변경을 한 번에 처리하므로, 먼저 처리된 요청만 변경된 행 수 1을 받고 나머지는 0을 받는다.
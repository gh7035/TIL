# hexagonal 이란?
기존 layerd 아키텍처에서 각 계층간의 결합도를 낮추기 위해 나온 아키텍처


## 특징

- 패키지를 application, infrastructure 로 나누어서, 외부와 내부로 나눈다.
- 내부인 application 에서는 순수한 java 가 사용된다.
- 외부와의 통신으로는 포트(인터페이스)를 구현한 어댑터(구현체)로 통신한다
- 통신에는 외부에서 내부로 요청을 보내는 Inbound Adapter와, 내부에서 외부로 요청을 보내는 Outbound Adapter가 있다.(예: Persistence Layer) 

> 핵사고날 아키텍처의 가장 큰 장점은 계층간의 결합도가 매우 낮아서 <br>
> 기술스택을 쉽게 변경할수 있다는 것이다!


<br>

### layerd 아키텍처를 대체했을때 장단점

<br>

> 장점:  헥사고날 아키텍처의 핵심은 모든 의존성이 중심(Domain/Application)을 향한다는 점이다.  

비즈니스 로직이 특정 기술(JPA, 특정 외부 API, 메시지 큐 등)에 의존하지 않아서 높은 교체 가능성이 있다.


기존 layerd 아키텍처에서는 Service가 Repository에 의존했지만, 헥사고날에서는 Service가 Port
(Interface)에 의존하고, Infrastructure에 있는 구현체(Adapter)가 이 포트를 구현한다.
이로 인해 내부 application 비즈니스 로직은 DB가 MySQL에서 MongoDB로 바뀌어도 코드를 한 줄도 수정할 필요가 없다.

<br>
<br>

> 단점 : 인터페이스와 어댑터를 매번 정의해야 하므로 클래스 파일이 많아지고 구조가 복잡해진다.

포트와 어댑터를 각각 정의해야 하므로, 단순 CRUD 하나를 만들 때도 layerd 보다 훨씬 많은 파일을 만들어야 한다.
layred 에서는 DB 엔티티를 컨트롤러까지 끌고 올라가기도 하지만, 헥사고날은 엄격히 분리하기 때문에 Persistence Entity를 Domain Model로, 다시 Response DTO로 변환하는 코드를 계속 작성해야 하기 때문에 힘들다.
# elasticsearch란?

>Apache Lucene 기반의 Java 오픈소스, 검색 엔진이다.\
Elasticsearch를 통해 방대한 양의 데이터를 신속하게, 거의 실시간으로 저장, 검색, 분석할 수 있다.


## 어떤식으로 작동되는가?
기존에 존재하는 데이터베이스를 Elastic Search라는 저장소에 주기적으로 데이터를 넣는 방식
<br>
<br>

### 어떻게 동기화 할까?

다음 코드가 있다고 가정하면..
```Java
@Entity
@Table(name = "articles")
@Getter
public class ArticleEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @Column(columnDefinition = "TEXT")
    private String content;

    private LocalDateTime updatedAt;
}
```

위 Entity를 기반으로 Elastic Search에 쓸 Document를 만들어야 한다.

```java
@Document(indexName = "articles")
@Getter
public class ArticleDocument {

    @Id
    private Long id;

    @Field(type = FieldType.Text, analyzer = "nori")
    private String title;

    @Field(type = FieldType.Text, analyzer = "nori")
    private String content;

    @Field(type = FieldType.Date)
    private LocalDateTime updatedAt;

    public static ArticleDocument from(ArticleEntity entity) {
        ArticleDocument doc = new ArticleDocument();
        doc.id = entity.getId();
        doc.title = entity.getTitle();
        doc.content = entity.getContent();
        doc.updatedAt = entity.getUpdatedAt();
        return doc;
    }
}
```

`@Document`에는 많은 옵션들이 있는데 그중에서 중요한것만 추려보자면
- indexName : 네이밍을 자유롭게 커스텀 가능하다
  - (대문자, 언더바로 시작, 공백 제외 커스텀 가능)
- shards : 샤드. 데이터를 여러 조각으로 나눠서 분산 저장하는 단위.
  - (보통 수백만건에서는 1~2, 수천만건은 3~5, 수억건은 5개 이상으로 설정한다.)
- replicas : 샤드가 만약 다운된다고 하면 대신 샤드의 역할을 하는 USB(복제본)이다.
  - (보통 장애대비용으로 1개 정도 사용한다.)
- refreshInterval : ES는 데이터를 색인한 후 바로 검색이 되는게 아니라, 일정 주기마다 검색 가능한 상태로 갱신되는데. 그 갱신 주기를 설정하는 옵션이다. 
  - (기본값은 1s, 성능을 위해 30s로 하는 경우도 있고, -1 로 하면 자동갱신 off인데, 수동으로 갱신한다.)
- createIndex : RDB의 PK같은 개념. 인덱스 생성 여부를 true/false로 결정
  - (인덱스를 직접 세밀하게 다루고 싶다면 false로 하고 세부 설정을 하면 된다. ex)한국어 ES nori 등..)

  

`ArticleEntity`(RDB) → `ArticleDocument`(ES) 로 변환하는 `from()` 메서드가 핵심이다.

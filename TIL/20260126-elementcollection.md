# @ElementCollection 컬렉션 매핑과 유효성 검사
```java
package org.example.crudboard.domain.board.entity;

import com.fasterxml.jackson.annotation.JsonBackReference;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import org.example.crudboard.domain.comment.entity.Comment;
import org.hibernate.annotations.CreationTimestamp;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
public class Board {

    @Id
    @Column(name = "board_id", nullable = false, unique = true)
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @Column(length = 255)
    private String description;

    @ElementCollection // 또 다른 테이블로 이 친구를 관리하겠다(근데 이제 @Id 없고, Board에 종속적인 테이블)
    @CollectionTable(
            name = "board_images", // 이미지들을 저장할 별도 테이블 이름
            joinColumns = @JoinColumn(name = "board_id") // board_images.board_id ← Board.id 로 FK 설정
    )
    @Column(length = 255, name="image_url") // 실제 이미지 문자열이 저장될 컬럼 이름
    private List<String> images; // Board 1 : N(image) 구조를 ElementCollection으로 매핑

    @Column
    @CreationTimestamp
    private LocalDateTime createdAt;

    private String password;

    @OneToMany(mappedBy = "board", cascade = CascadeType.ALL, orphanRemoval = true) //board 테이블과
    @JsonManagedReference
    private List<Comment> comments = new ArrayList<>();
}
```
@ElementCollection -> 이 친구를 또 다른 테이블로 관리하겠다
@CollectionTable -> 그 테이블의 세부정보
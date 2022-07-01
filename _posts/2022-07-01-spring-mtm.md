---
layout: post
title: "Spiring: ManyToMany 관계 Entity에서 컬럼 추가"
categories: [Dev, Spring]
tags: [spring, jpa, entity]

---

> *본 글에 포함된 엔티티 정의는 이해를돕기위한 예시입니다.*
>

# Simple ManyToMany Relation

![simple](/assets/img/220701_1_1.png)

위 Relation을 가지는 Entity를 정의할때, 아래 예시처럼 Recommend `Entity`에서 `ManyToMany` 관계를 작성하여 설정이 가능하다.

```kotlin
class Recommend(
    @ManyToMany(
        fetch = FetchType.LAZY,
        cascade = [CascadeType.ALL]
    )
    @JoinTable(
        name = "song_recommend",
        joinColumns = [JoinColumn(name = "recommend_id")],
        inverseJoinColumns = [JoinColumn(name = "song_id")]
    )
    var songs: MutableSet<Song>? = null,

    ...
    @JoinTable(
        name = "movie_recommend",
        joinColumns = [JoinColumn(name = "recommend_id")],
        inverseJoinColumns = [JoinColumn(name = "movie_id")]
    )
    var movies: MutableSet<Movie>? = null,
)
```

# 추가 컬럼이 필요할 때

위 예시에서 추천 대상 엔티티의 컬럼(song, enitty)외에 추천관련 정보가 추가로 필요한 경우 JoinTable만 사용해서는 구현이 불가하다.

### 해결방안

여러 방법이 있겠지만 본 글에서는 ***JoinTable Entity를 직접 작성하여*** 원하는 정보를 추가하는 방식으로 진행.

요구하는 관계 다이어그램은 아래와 같다.

![extra](/assets/img/220701_1_2.png)

### Entity

```kotlin
@Embeddable
data class RecommendSongId(
    @Column(name = "recommend_id")
    var recommendId: Long,
    @Column(name = "song_id")
    var songId: Long
) : Serializable

@Entity
class RecommendSong(
    @EmbeddedId
    var id: RecommendSongId,
    @ManyToOne
    @MapsId("recommend_id")
    var recommend: Recommend? = null,
    @ManyToOne
    @MapsId("song_id")
    var song: Song? = null,
    var extra: Int = 0,
    ...
)

@Entity
class Recommend(
    ...
    @OneToMany(mappedBy = "recommend", cascade = [CascadeType.ALL], orphanRemoval = true)
    var songs: MutableSet<RecommendSong> = mutableSetOf()
    ...
)
```

예제에는 `composite key(RecommendSongId)`를 사용했지만 `RecommendSong`에 PK, FK(recommend, song) 을 작성하는 방식으로도 구현가능하다

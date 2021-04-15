# TIL

Today I Learned

> 오늘 배운 것들, 오늘 안 것들, 생각했던 것들.

## TIL: 1

 자꾸 알고 넘어가는게 너무 많아져서 간단하게라도 정리하고, 나중에 나를 피드백 하기 위해 TIL작성을 고민하게됨.
꼭 뭔가 배우는 것만 적는게 아니라 전체적으로 적어두는게 필요했음.

### Formula

 Formula는 SubQuery를 Entity에서 콜 할수 있게 해준다.

 주의해야할 점은 H2에서는 SubQuery를 소괄호로 감싸지 않아도 되지만, MariaDB등에서는 반드시 소괄호로 감싸야 한다는 것이다.

```kotlin
class Post(
    @Id
    @GeneratedValue
    val id: Int,
    /** ... */
    @Formual("((SELECT COUNT(*) FROM comment WHERE comment.post_id = id))")
    val commentCount: Int,
)
```

 Row Query를 써야한다는 점과 Record가 반드시 하나만 나와야 한다는(Collection 안됨) 아쉬운 점이 있지만 `comments.size` 같이 모든 comments를 로드하는 방식보다는 보다 나은 방식이라고 생각한다.

### Non-Primary Key 가 Foreign Key 인 경우 OneToMany, ManyToOne 관계에 발생하는 오류

 레거시 서비스를 SpringBoot+JPA로 포팅하는 과정에서, 기존의 Table을 유지한 채로 포팅하려다 보니 `@Subselect` 같은 기능을 사용할 일이 생겼다.

 그러던 중 `Member` -> `Coin(Subselect)` 의 관계 연결 중 ManyToOne을 하였을 때 `{Entity} cannot be cast to class java.io.Serializable` 에러가 발생하였다. 이는 `Serializable` 를 `Member`에 implement 시키면 해결되는 것을 확인하였다.

 기존의 동작에선 생기지 않았던 케이스라 생각하여 테스트를 해보니 Primary Key 가 아닌 컬럼으로 관계를 연결하였을 때만 발생하는 오류임을 알게되었다. 사실 처음에는 `@Subselect` 에서만 발생하는 문제인 줄 알고 오래 해멨었다. 하지만 `Coin`의 owner(Member)를 제거하고 Join을 하였을때 로그에 찍히는 쿼리가 무조건 Primary Key 가 파라매터로 들어가는 것을 보고 Foreign Key를 의심하게 되었다.

 결론적으로 Non-Primary Key 에 의한 관계연결에선, OneToMany쪽에 `Serializable` 인터페이스를 implements 해주어야 한다.

 >If an entity instance is to be passed by value as a detached object (e.g., through a remote interface), the entity class must implement the Serializable interface.
 > "JSR 220: Enterprise JavaBeansTM,Version 3.0 Java Persistence API Version 3.0, Final Release May 2, 2006"
 
 JSR 을 보았을 때, Member 가 Serializable 을 구현할 필요는 없다고 생각하나, 아마 최적화를 위해 PersitanceManager 쪽에서 임시적으로라도 Serializable 을 사용하나 정도 까지만 유추해보고 있다. 이에 대해선 추후 좀더 연구하여 문서로 정리하면 좋을 것 같다.

## Jpa Repository

Spring 계열에서 sql 쿼리문 대신에 사용 가능한 ORM 라이브러리 이다.

```java
@Entity
@Data
public class Message {
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false)
    private Long userId;
    @Column(nullable = true,length = 40,name ="uuid")
    private String uuId;
    protected Message(){
    }
}
```
이런 엔티티가 있다고 하면 사용하기 위해선 MessageRepository.class 정의

```java
@Repository
public interface MessageRepository extends CrudRepository<Message, Long> {
    List<Message> findByUserlIdAndUuId(Long userId, String uuId);
    void deleteAllByUserIdAndUuId(Long userId,String uuId);
    @Transactional
    @Modifying
    @Query("delete from Message m where m.userId=?1 AND m.uuId=?2")
    void deleteAllByUserIdAndUuIdInQuery(Long userId,String uuId);    
}
```

작성 해주면 사용가능하다<br>
정의 해준 함수 말고도 기본 제공 함수들이 존제한다.
```java
@Autowired
MessageRepository messageRepository

void foo(){
  Message message = Message.builder().build();
  messageRepository.save(message); //저장
  
  messageRepository.deleteAllByUserIdAndUuId(userId,uuId)
  messageRepository.deleteAllByUserIdAndUuIdInQuery(userId,uuId)
}
```
#### 중요한점

여기서 deleteAll 함수를 두개 정의했다.<br>
한 개는 Jpa 내장 함수를 재정의 , 한개는 쿼리문 재정의 하였다.<br>
이유는 Jpa 자체에서 deleteAll 같은 작업 수행시 selectAll 한 후에 매칭되는 것을 delete 시킨다. 따라서 데이터가 많아지면 속도가 매우 느려진다. <br>
쿼리문으로 작성시에는 DB 단에서 바로 delete 로직이 수행되기 때문에 훨씬 빠르다

* 쿼리문을 잘 모르고 다른 로직에 집중 할 수 있어서 ORM 을 쓰는 것을 선호하지만 이런 성능 저하에 대한 고민은 하면서 해야한다.







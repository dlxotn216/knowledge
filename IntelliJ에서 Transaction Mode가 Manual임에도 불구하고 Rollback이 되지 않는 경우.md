# IntelliJ에서 Transaction Mode가 Manual임에도 불구하고 Rollback이 되지 않는 경우

<img src="https://raw.githubusercontent.com/dlxotn216/image/master/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-01-13%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%208.17.44.png" />
위와 같이 DataGrip에서 Transaction Mode가 Manual로 되어있음에도 불구하고 커밋 하지 않은 변경사항이 반영되는 문제가 생겼다.  
커넥션을 끊었다 다시 맺어보기도 했고 인텔리제이 재시작도 해보았으나 증상은 여전했다.  

한참을 찾다가 콘솔의 로그를 보니 롤백 시 아래와 같은 로그가 떠있다.  
<img src="https://github.com/dlxotn216/image/blob/master/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-01-13%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%208.21.15.png?raw=true" />

첫 번째로 의심 된 부분은 트랜잭션 격리레벨이 Read Uncommited이지 않을까 였다. 하지만 Maria의 기본 설정인 Repeatable Read였다.  
```sql
SELECT @@GLOBAL.tx_isolation, @@tx_isolation;
```

몇 가지 더 테스트를 거쳐 몇몇 테이블에서만 자동 커밋이 되고 있었던 부분을 확인했고  
```sql
show table status;
```  
위 쿼리를 통해서 문제가 발생한 Table의 Engine이 MyISAM인 것을 확인했다.  

아래와 같은 쿼리로 엔진은 변경할 수 있으나  
```sql
alter table TARGET_DB.engine = innodb;
```  
아래와 같이 모든 row에 대해 적용하기에 운영중엔 하지 말아야 한다.  
>  test> ALTER TABLE AAA ENGINE = innodb  
>  [2021-01-12 17:55:05] 36407 rows affected in 238 ms  
>  test> ALTER TABLE BBB ENGINE = innodb  
>  [2021-01-12 17:55:06] 37046 rows affected in 298 ms   

(또는 덤프를 떠서 새로운 테이블로 옮기고 drop 후에 이름을 다시 바꾸는 것도 가능하다)  

MyISAM, InnoDB는 스토리지 엔진의 종류로 각각의 장단점이 극명하다.  

MyISAM은 SELECT에 대한 동작이 최적화 되어있다. 테이블에 row count를 가지고 있어 select count(*)시에 빠른 속도를 보이고  
풀텍스트 인덱스를 지원한다. 반면 트랜잭션을 지원하지 않고 row level lock이 지원되지 않아 Table에 lock을 걸어야 한다.  
또한 외래키를 지원하지 않는 등 데이터 무결성에 대한 보장이 없다. 이러한 이유들로 보통 Read Only 전용으로 사용한다.  

InnoDB는 데이터 무결성을 보장하고 Row level lock, 트랜잭션, 외래키, 백업과 덤프 등을 지원한다.  

각 테이블의 특성에 따라 엔진을 적절히 선택해주면 좋을 것 같다. 단 문서화는 필수이다.

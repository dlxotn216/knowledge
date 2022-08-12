# MariaDB 버전 업그레이드에 따른 Access Plan 변경 점

MariaDB 10.2에서는 아래와 같이 LEFT JOIN 안에서 많은 데이터를 기반으로 인라인 뷰를 만들어야 할 경우 바깥의 테이블 참조가 불가능하기에 별도로 IN 절을 다시 넣어줘서 쿼리를 작성 해주어야 했다.  
그렇지 않으면 CW 테이블이 full scan으로 access plan이 잡힌다.  
```sql
SELECT KEY, WARNING
  ....
FROM (
    SELECT KEY , WARNING
    FROM (
        SELECT CA.KEY
             , .....
             , CW.COUNT WARNING
        FROM (
            SELECT CC.KEY, .....
              FROM CC
              WHERE CC.CASE_KEY IN (...)
         ) CA
        LEFT JOIN (
            SELECT CW.KEY, COUNT(CW.KEY) COUNT
            FROM CW
            WHERE CW.KEY IN (...)
            GROUP BY KEY
        ) CW ON CW.KEY = CA.KEY
        JOIN .....
    ) TB
) TB
ORDER BY KEY DESC;
```

10.3 업그레이드 이후 split_materialized 옵티마이저가 나오면서 위와 같은 쿼리의 결과가 아예 달라지는 현상이 발견 됐다.   

access plan이 아래와 같이 LATERIAL DERIVED로 바뀌면서 group by 결과가 맨 마지막 행에 몰렷다.  
| id | select_type | table | type | possible_keys | key | ref | rows | filtered | extra | 
| -- | ---------   | ----- | ----- | ----------- | ---- | ---- | --- | -------- | ----------- |
| 1  | PRIMARY | CC | range | CC_PK | CC_PK | NULL | 100 | Using where; Using index |
| 1  | PRIMARY | <derived5> | ref | key0 | key0 | KEY | 100 | |
| 6  | LATERIAL DERIVED | CW | range | CW_PK | CW_PK | NULL | 0.01 | Using where |
  
result
| KEY | WARNING|
| - | - |
| 1 | 0 |
| 2 | 0 |
| 3 | 0 |
| 4 | 0 |
| 5 | 9281 |
  
안쪽 IN 절을 빼주면 아래와 같이 plan이 잡힌다. LATERIAL DERIVED로 접근하는 CW의 ref가 CC의 PK로 잡힌다.  
| id | select_type | table | type | possible_keys | key | ref | rows | filtered | extra | 
| -- | ---------   | ----- | ----- | ----------- | ---- | ---- | --- | -------- | ----------- |
| 1  | PRIMARY | CC | range | CC_PK | CC_PK | NULL | 100 | Using where; Using index |
| 1  | PRIMARY | <derived5> | ref | key0 | key0 | KEY | 100 | |
| 6  | LATERIAL DERIVED | CW | range | CW_PK | CW_PK | **CC_PK** | 0.01 | Using where |

result
| KEY | WARNING|
| - | - |
| 1 | 1123 |
| 2 | 3125 |
| 3 | 612 |
| 4 | ... |
| 5 | 81 |

 
IN 절을 뺀 쿼리로 10.3에서 나온 옵티마이저를 비활성화 후 플랜을 보면 아주 난리가 난다.   
set session optimizer_switch='split_materialized=off';  

| id | select_type | table | type | possible_keys | key | ref | rows | filtered | extra | 
| -- | ---------   | ----- | ----- | ----------- | ---- | ---- | --- | -------- | ----------- |
| 1  | PRIMARY | CC | range | CC_PK | CC_PK | NULL | 100 | Using where; Using index |
| 1  | PRIMARY | derived5| ref | key0 | key0 | KEY | 100 | |
| 6  | DEPENDENT SUBQUERY | CW | range | NULL | NULL | NULL | 91028301 | |
  
IN 절을 넣은 쿼리로 10.3에서 나온 옵티마이저를 비활성화 후 플랜을 보면 정상적이고 결과도 잘 나온다.     
set session optimizer_switch='split_materialized=off';  

| id | select_type | table | type | possible_keys | key | ref | rows | filtered | extra | 
| -- | ---------   | ----- | ----- | ----------- | ---- | ---- | --- | -------- | ----------- |
| 1  | PRIMARY | CC | range | CC_PK | CC_PK | NULL | 100 | Using where; Using index |
| 1  | PRIMARY | derived5 | ref | key0 | key0 | KEY | 100 | |
| 6  | DEPENDENT SUBQUERY | CW | range | CW_PK | CW_PK | NULL | 0.01 | Using where |
  
  
## 참고

### 새로 추가 된 옵티마이저 관련
https://mariadb.com/kb/en/lateral-derived-optimization/

### on/off 설정
https://mariadb.com/kb/en/optimizer-switch/

### MySQL 기준 LATERAL DERIVED Join 기능
https://hoing.io/archives/6200

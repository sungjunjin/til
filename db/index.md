# Index
데이터베이스에서 인덱스는 조건에 맞는 튜플(들)을 빠르게 조회하거나, 정렬 혹은 그룹핑하기 위해 사용된다.

아래 예시를 확인해보자

| id | name | back_number | team_id |
| --- | --- | --- | --- |

위와 같이 이름, 등번호, 소속팀 id를 가지는 선수(player) 테이블이 있다.

만일 이 테이블의 로우 개수가 대략 백만개를 넘어갈때 다음과 같은 쿼리를 실행하면 어떻게 될까?
```sql
select *
from player
where name = "Son"
```

아마도 모든 로우를 방문하면서 name 값을 비교하여 위 조건에 맞는 값을 찾을 것 이다. 이 때 시간복잡도는 O(n)이 될 것이고 이를 **Full Scan**이라고 한다.

하지만 Index라는 것을 적용하게 되면 위 쿼리의 시간 복잡도를 O(logn)으로 줄일 수 있다. (B-tree 기반 인덱스 기준)


## Index 사용법
MySQL 기준으로 위 player 테이블에 대한 인덱스를 생성하는 방법은 다음과 같다.

```sql
-- 테이블 생성과 동시에 인덱스를 생성할 경우
create table player(
    id bigint primary key,
    name varchar(20) not null,
    team_id bigint
    back_number int,
    index player_name_idx (name),
    unique index team_id_backnumber_uidx (team_id, back_number)
);

-- 이미 테이블이 생성된 경우
create index player_email_idx on player (email);

-- Unique Index를 사용하고 싶은 경우
create unique index player_team_id_backnumber_uidx on player (team_id, back_number);
```

대부분의 RDBMS의 경우에는 Primary Key에 대한 인덱스는 기본적으로 생성되어 있다. 

위 Unique Index 예시와 같이 두개 이상의 컬럼을 인덱스로 사용하는 경우, 해당 인덱스를 **다중컬럼 인덱스 (Multicolumn Index)** 혹은 **복합 인덱스(Composite Index)** 라고 한다.

player 테이블에 적용되어 있는 인덱스 목록을 조회하고 싶으면 다음과 같은 명령문을 사용할 수 있다.

```sql
show index from player;
```
### Index 사용 시 주의점

인덱스는 조회 성능을 개선할 수 있는 확실한 장점이 있지만 이에 따라 발생하는 트레이드 오프 또한 고려해야 한다. 

인덱스를 사용할 경우 발생할 수 있는 단점들은 다음과 같다.

- 인덱스를 생성할 추가적인 공간 필요
- 읽기를 제외한 나머지 작업을 진행 시 인덱스 또한 수정 & 다시 정렬해야 함 

### Covering Index
조회하는 속성을 본 테이블에 볼 필요도 없이 인덱스 수준에서 가져올 수 있을때 해당 인덱스를 Covering Index라고 한다

예를 들어 다음과 같은 인덱스가 있다고 가정했을 때
```sql
create unique index player_team_id_backnumber_uidx on player (team_id, back_number);
```

아래와 같은 조회 쿼리를 실행하면
```sql
select team_id, back_number from player where team_id = 1;
```

team_id, back_number의 속성은 모두 위 인덱스에 포함되어 있고 where 절에 검색조건 또한 위 인덱스를 사용할 수 있으므로 본 테이블에 접근하지 않고 바로 조회 쿼리에 대한 처리가 가능하다.


### Hash Index
B-Tree 인덱스 외에 사용할 수 있는 인덱스의 종류는 Hash Index가 있다. 내부적으로 Hash Table을 사용하여 인덱스를 구현한다. 따라서 검색에 대한 시간 복잡도는 O(1)으로 B-Tree 인덱스보다 빠르지만, 범위 검색이 불가능하다는 점과 데이터가 늘어났을 경우 rehashing에 사용되는 추가 연산 비용이 필요하다는 단점이 있다.

또한 복합 인덱스를 Hash Index에 적용할 경우, 인덱스에 구성되는 모든 속성으로 조회를 해야만 인덱스가 적용된다는 특성이 있다.

### Full Scan이 더 좋은 경우
- 테이블에 데이터가 소량으로 있을 때 (몇 십 ~ 몇 백건)
- 조회하려는 데이터가 테이블의 상당 부분을 차지할 때 
    - ex) 성별 컬럼의 남자, 여자, 통신사 컬럼의 SKT, KT, LG


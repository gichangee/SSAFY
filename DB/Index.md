# DB

```jsx
select no,subject,
		case date_format(register_time,'%y%m%d')
						when date_format(now(),'%y%m%d') 
            then date_format(register_time,'%H:%i:%s') // 만약 오늘 날짜와 같다면 해당 형식으로 출력
            else date_format(register_time, '%y. %m. %d') // 만약 오늘 날짜와 다르다면 해당 형식으로 출력
		end register_time
from board
order by no desc;
```

```html
//날짜
create table board(
	register_time timestamp default now()
);

insert into board(register_time)
values(date_sub(now(),interval 3 week)); // 3주 전

insert into board(register_time)
values(date_sub(now(),interval 2 day)); // 2일 전

insert into board(register_time)
values(date_sub(now(),interval 12 hour); //12 시간 전

insert into board(register_time)
values(now());
```

```html
create table test(
	a char(5),
	b varchar(5),
	c text(5)
);

insert into test
values('123','123','123456');

char에는 5byte가 잡히고 123만 저장됨 즉 2byte가 남음
varchar는 3byte만 잡힙 -> 가변길이
text는 길이제한 주는 것이 의미가 없다

```

datetime vs timestamp

mysql에는 time-zone이라는 것이 있는데

time-zone이 미국으로 바뀌어도 datetime은 영향을 받지 않지만

timestamp는 영향을 받는다

- index

테이블의 데이터 조회(select) 시 동작속도를 높여주는 자료구조

데이터 위치를 빠르게 찾아주는 역할

컬럼의 값과 레코드가 저장된 주소를 키와 값의 쌍으로 인덱스를 만들어둠

MYI파일에 인덱스 저장

- index 문제점

예시) 책의 모든 페이지에 나오는 단어를 찾아보기에 표기하게 되면 찾아보기 분량이 엄청나게 많아져 오히려 본문보다 두꺼워지는 상황 발생

필요 없는 인덱스를 만들면 데이터베이스가 차지하는 공간만 늘어나고

index를 이용하여 데이터를 찾는 것이 전체 테이블을 찾는 것보다 느려짐

데이터베이스의 공간을 차지하므로 추가적인 공간 필요

처음 index를 생성하는데 많은 시간이 소요

데이터의 변경 작업이 자주 일어나느 경우 오히려 성능 저하가 일어날 수있음

- 인덱스의 종류

클러스터형 인덱스

특정 나열된 데이터들을 일정 기준으로 정렬해 주는 인덱스

데이터 자체가 정렬하는 기준이 된다

클러스트형 인덱스 생성 시 데이터 페이지 전체가 다시 정렬

테이블당 하나만 생성 가능

보조인덱스보다 검색은 빠르나 입력수정삭제가 느림

mysql는 pk 가 있다면 클러스형 인덱스로 pk가 없다면

unique하면서 not null이 컬럼 그것도 없으면 임의로 보이지 않는 컬럼을 만들어 클러스터형 인덱스로 지정

보조 인덱스(secondary index)

개념적으로 후보키에만 부여가능한 인덱스

보조 인덱스 생성시 데이터 페이지는 그냥 둔상태에서 별도의 페이지에 인덱스를 구성, 자동정렬되지 않음

데이터가 위치하는 주소값을 저장

검색은 느리지만 입력수정삭제시 성능 부하가 적음

- 제약조건에 따른 index 결정
    
    특정 테이블에 pk가 존재하면서 uk가 존재할 경우
    
    pk로 지정된 컬럼은 클러스터
    
    uk로 지정된 컬럼은 보조
    
    uk로 지정된 컬럼이 null을 허용하던 허용하지않던 모두 보조인덱스가 됨
    
    특정테이블에 pk가 없고 unique + not null 제약 조건이 특정 컬럼에 있을 경우
    
    해당 컬럼이 pk
    
    not null 조건이 없다면 보조가 됨
    

- index 생성 전략

인덱스는 열 단위에 생성

where 절에서 사용되는 열에 생성

where 절에 사용되는 열이라도 자주 사용해야 가치가 있음

데이터 중복도가 높은 열에는 인덱스를 만들어도 효과가 없음 → 중복도가 낮은 열에 생성

외래키를 설정한 열에는 자동으로 외래키 인덱스가 생성됨

조인에 자주 사용되는 열에는 인덱스를 생성하는 것이 좋음

데이터 변경이 얼마나 자주 일어나는지 고려해야함

클러스터형 인덱스는 테이블당 하나만 생성가능

사용하지 않는 인덱스는 제거

- 자동으로 생성되는 index
    - 클러스터형 인덱스
    
- index 생성
    
    create index 문으로는 보조인덱스만 생성가능
    
    클러스터형 인덱스를 만들려면 alter table 사용해야함
    
- 인덱스 사용이유

```html
select * from ssafy
where birth_year=1994; //전체 row를 찾음

select * from ssafy
where user_id='KCS'; //하나의 row만 찾음
```

- 인덱스 삭제
    
    인덱스를 삭제할 때 보조인덱스 부터 삭제

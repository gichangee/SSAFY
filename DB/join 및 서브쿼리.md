- join

둘 이상의 테이블에서 데이터가 필요한 경우 사용

- join의 종류

INNER JOIN → 교집합 → n개의 테이블 조인 시 n-1개의 조인 조건이 필요

OUTER JOIN

LEFT OUTER JOIN

RIGHT OUTER JOIN

- join 조건의 명시에 따른 구분

NATURAL JOIN
CROSS JOIN(FULL JOIN, CARTESIAN JOIN)

natural join vs inner join

natural join은 두 개의 테이블에서 공통된 컬럼을 하나의 컬럼으로 만들어서 보여줌

inner join도 마찬가지인데 더 안정적이다

inner join은 on을 통해 조건 제한 가능하기 때문에

- Join 주의

조인의 처리는 어느 테이블을 먼저 읽을지를 결정하는 것이 중요

inner join는 어느 테이블을 먼저 읽어도 결과가 달라지지 않아 mysql 옵티마이저가 조인의 순서를 조절해줌

outer join 반드시 outer가 되는 테이블을 먼저 읽어야 하므로 옵티마이저가 조인 순서를 선택 할 수없다

- inner join

교집합 = equi join이라고도 한다

```html
select e.employ id, e.name,e.salary, d.departmentname
from employees e, department d
where e.department_id = d.department_id
and e.eployee_id =100

inner join 사용하기

select e.employ id, e.name,e.salary, d.departmentname
from employees e join department d
on e.department_id = d.department_id
where e.eployee_id =100

select e.employ id, e.name,e.salary, d.departmentname, l.street_addresss
from employees e join department d join location l
on e.department_id = d.department_id
and d.location_id = location_id
where e.eployee_id =100

//using 사용 -> as 쓰기 귀찮을 때
select e.employ id, e.name,e.salary, d.departmentname, l.street_addresss
from employees e join department
using (department_id) // join 된 2개의 테이블에서 departmend_id를 이용해 join 하겠다는 뜻
department d join location l
using (location_id)
where e.eployee_id =100

//natural join -> using 도 귀찮을 때
주의할 점 -> 2개의 테이블에 같은 컬럼 모든 것을 비교한다.
즉 2개의 컬럼의 이름이 같다면 2개의 컬럼을 비교한다.
```

- left outer join == left join

왼쪽 테이블 기준으로 join 조건에 일치 하지 않는 데이터까지 출력

```html
from employees e left join departments d
on emdeaprtmet_id = d.department_ld;
//조건에 만족하지 않더라도 왼쪽 테이블에 있는 데이터는 모두 출력해줘
```

- mysql 에는 full outer join이 없다

```html
left join 한 결과와 right join 한 결과를
union 하면 된다
```

- selft join

```html
자기 자신 테이블과 join
```

- 서브쿼리

서브쿼리란 다른 쿼리 내부에 포함되어 있는 select 문을 의미한다

서브쿼리를 포함하고 있는 쿼리를 외부 쿼리 또는 메인 쿼리

서브쿼리는 내부쿼리라고도 부른다

스칼라 서브 쿼리 → select 문에 작성하는 서브 쿼리

인라인 뷰 → from 문에 작성하는 서브쿼리

중첩 서브 쿼리(nested 서브쿼리) → where 문에 작성하는 서브쿼리

- 서브쿼리 주의사항

반드시 ()로 감싸야 한다

서브쿼리는 단일 행 또는 다중 행 비교 연산자와 함께 사용

단일 행 비교연산자 → = 

다중 행 비교연산자 → in

- 서브쿼리 사용 가능한 곳

select

from

where 

having

order by

values

set

nested 서브쿼리 - 단일행

where department_id = ()

where department_id >  ()

nested 서브쿼리 - 다중행

서브쿼리의 결과가 다중행을 리턴 

in 일 때 

서브쿼리 결과로 나온 것만 사용

any 일 때

서브쿼리 결과로 나온것 중 하나라도 만족하면 됨

where salary > any ();

all 일때

서브쿼리 결과로 나온걸 모두 만족해야함

where salary > all ();

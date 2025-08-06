# 자바 ORM 표준 JPA 프로그래밍
## 01. JPA 소개
### 서론
- JPA를 사용하지 않을 때는 JdbcTemplate, JdbcClient와 같은 SQL Mapper를 통해 JDBC API를 사용해야함
- CRUD용 SQL의 반복 작성은 보일러플레이트 코드에 해당함
- 객체 모델링 대신 데이터 모델링을 쓰는 사람들의 이유는?
  - 객체 모델링 방식을 채택했을 때, 고도화될 수록 객체를 데이터베이스에 저장/조회하는데 어려워지고, 객체와 RDB 간의 차이를 메우기 위해 더 많은 SQL 작성이 필요
  - 이 때문에, 객체 모델이 점점 데이터 중심의 모델로 변해감
- JPA는 객체와 RDB간의 차이를 중간에서 해결해주는 ORM(Object-Relational Mapping) 프레임워크의 자바 진영에서의 기술 표준임
  - 스프링 진영에서도 스프링 데이터 JPA를 통해 JPA를 지원
  - 전자정부 표준 프레임워크의 ORM 기술도 JPA를 사용
- JPA는 반복적 CRUD SQL 처리 뿐 아니라, 객체 모델링과 RDB 사이의 차이점도 해결
### 1.1 SQL을 직접 다룰 때 발생하는 문제점
#### 1.1.1 반복, 반복, 그리고 반복
- 일반적인 Read 구문은 다음과 같이 작성됨 (Member 객체 조회 예시)
  1. 회원 조회용 SQL 작성 `SELECT * FROM MEMBER WHERE MEMBER_ID = ?`
  2. JDBC API 사용하여 SQL 실행 `ResultSet rs = stmt.executeQuery(sql);`
  3. 조회 결과를 Member 객체로 매핑 `new Member(rs.getString("MEMBER_ID"), rs.getString("NAME"));`
- Create 구문도 비슷한 흐름으로 작성될 수 있음
- 객체를 직접 저장하는 방식이 아닌, 객체의 개별 요소를 저장하는 형식임
  - 이 때문에, `list.add(member);`같은 구문을 지원하지 않음
  - 즉, 이 구문을 실행한다 해도 member 객체가 데이터베이스에 저장되지 않음
#### 1.1.2 SQL에 의존적인 개발
- 엔티티(객체)의 컬럼(멤버변수)이 추가될 때마다, 수정해야하는 SQL문과 그에 따른 코드가 매우 많아짐
  - CRU 구문 모두에서 수정이 필요함
- 특히 엔티티(객체)와 다른 연관객체 (FK 등 연관관계)가 추가되면, Join문 사용/비사용 코드의 혼재로 인해 코드가 꼬임
  - 예를 들어, 기존에 `public Member find(String memberId){...}` 메소드를 통해 멤버 조회 상정
  - 연관객체 `Team`이 추가되면, find에서 `*(와일드카드)`를 사용하여 조회하지 않고있었다면 TEAM_ID가 조회되지 않음
  - TEAM_NAME도 조회하고자 한다면 `JOIN TEAM T ON ...` 문을 필수적으로 추가해야함
- 이는 곧, 연관객체 Team의 사용가능여부는 전적으로 SQL문에 달려있음(JOIN문 사용여부)
- 지금처럼 SQL에 모든것을 의존하는 상황에서는, 개발자들이 엔티티를 신뢰하고 사용 불가
  - 이는 곧 '진정한 의미의 계층 분할의 어려움', '엔티티의 불신뢰성', 'SQL 의존적 개발 회피 어려움'의 문제점을 이야기함
#### 1.1.3 JPA와 문제 해결
- JPA를 사용하면 CRU 구문을 작성하지 않아도 됨
  - 저장(C): `jpa.persist(member);`
    - 객체와 매핑정보를 보고 적절한 Insert SQL을 전달
  - 조회(R): `Member member = jpa.find(Member.class, memberId);`
  - 수정(U): `Member member = jpa.find(Member.class, memberId); member.setName("...");`
    - 수정 메소드를 별도로 제공하지 않고, 트랜잭션 커밋 과정에서 적절히 Update SQL을 전달
  - 연관객체 조회: `Member member = jpa.find(Member.class, memberId); Team team = member.getTeam();`

(TBD)

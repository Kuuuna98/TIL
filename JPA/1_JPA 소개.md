# JPA 소개
지금 시대는 객체를 관계형 데이터베이스에 관리

**SQL 중심적인 개발의 문제점**

- 무한 반복, 지루한 코드
- 객체를 생성하고 CRUD를 만든 후에 필드를 추가해야하는 경우, 쿼리를 하나씩 확인하며 수정해야한다
- 관계형 데이터베이스를 쓰는 상황에서는 SQL에 의존적인 개발을 피하기 어렵다
- 패러다임의 불일치 (객체 vs. 관계형 데이터베이스)

> 💡 **객체와 관계형 데이터베이스의 차이**
> 
> 1. 상속
> 2. 연관관계 
> 3. 데이터 타입
> 4. 데이터 식별 방법
  

상속 관계를 사용하는 경우 관계형 데이터베이스에 객체를 저장하고 조회하는 데 복잡해진다.

자바 컬레션에 저장, 조회하는 것은 단순하다.

```java
list.add(album);
list.get(albumId);
```

객체 모델링도 자바 컬렉션에서 관리할 때가 더 간단하다.

### 엔티티 신뢰문제


```java
class MemberService{
	...
	public void process(){
		Member member = memberDAO.find(memberId);
		member.getTeam(); //???
		member.getOrder().getDelivery(); //???
	}
}
```

### 비교하기

```java
String memberId="100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2; //다르다.

class MemberDAO{
	public Member gerMember(String memberId){
		String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
		...
		//JDBC API, SQL 실행
		return nmew Menber(...);
	}
}
```

### 비교하기-자바 컬렉션에서 조회

```java
String memberId="100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2; //같다.
```

### 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까? → JPA

---

### JPA란?

- Java Persistence API
- 자바 진영의 **ORM** 기술 표준

### ORM이란?

- 객체 관계 매핑 (Object-relational mapping)
- 객체는 객체대로 설계 / 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- ORM 프레임 워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

---

JPA는 애플리케이션과 JDBC 사이에서 동작

JPA는 표준 명세

- JPA는 인터페이스의 모음
- JPA 2.1 표준 명세를 구현한 3가지 구현체: Hibernate, EclipseLink, DataNucleus

### JPA를 왜 사용해야 하는가?

- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성
- 유지보수
- 패러다임의 불일치 해결
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준

### 생산성 - JPA와 CRUD

- 저장: **jpa.persist**(member)
- 조회: Member member = **jpa.find**(memberId)
- 수정: **member.setName**(”변경할 이름")
- 삭제: **jap.remove**(member)

### 유지보수

- 기존: 필드 변경시 모든 SQL 수정
- JPA: 필드만 추가하면 됨, SQL은 JPA가 처리


> 💡 **JPA와 패러다임의 불일치 해결**
> 
> 1. JPA와 상속
> 2. JPA와 연관관계
> 3. JPA와 객체 그래프 탐색
> 4. JPA와 비교하기


### 신뢰할 수 있는 엔티티, 계층

```java
class MemberService{
	...
	public void process(){
		Member member = memberDAO.find(memberId);
		member.getTeam(); //자유로운 객체 그래프 탐색
		member.getOrder().getDelivery();
	}
}
```

### 비교하기

```java
String memberId="100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; //같다.
```

동일한 트랜잭션에서 조회한 엔티티는 같음을 보장한다

---

> 💡 **JPA의 성능 최적화 기능**
> 
> 1. 1차 캐시와 동일성(identity) 보장
> 2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
> 3. 지연 로딩(Lazy Loading)

### 1차 캐시와 동일성 보장

1. 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
2. DB Isolation Level이 Read Commit이어도 애플리케이션에 Repeatable Read 보장

```java
String memberId="100";
Member member1 = jpa.find(Member.class, memberId); //SQL
Member member2 = jpa.find(Member.class, memberId); //캐시 - 시간이 굉장히 짧다

System.out.print(member1 == member2); // true

//SQL 한번만 실행
```

### 트랜잭션을 지원하는 쓰기 지연

- Insert
    1. 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
    2. JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송
    
    ```sql
    transaction.begin(); //[트랜잭션] 시작
    
    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);
    //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
    
    //커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
    transaction.commit(); //[트랜잭션] 커밋
    ```
    

### 지연 로딩과 즉시 로딩

- 지연 로딩: 객체가 실제 사용될 때 로딩
- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회
- JPA는 옵션 설정을 통해 두 가지 로딩 중 하나를 사용할 수 있다
- 지연 로딩은 Team 객체를 잘 사용하지 않을 때 사용
- 즉시 로딩은 Member 객체와 Team 객체를 대부분 함께 사용할 때 사용

```java
/* 지연 로딩 */
Meber member = memberDAO.find(memberId); -> SELECT * FROM MEMBER
Team team = member.getTeam(); -> SELECT * FROM TEAM
String teamName = team.getName(); 
```

```java
/* 즉시 로딩 */
Meber member = memberDAO.find(memberId); -> SELECT M.*, T.* FROM MEMBER JOIN TEAM ...
Team team = member.getTeam();
String teamName = team.getName();
```

***ORM은 객체와 RDB 두 기둥 위에 있는 기술***

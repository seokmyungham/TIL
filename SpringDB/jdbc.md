# 1. JDBC 이해

## JDBC가 등장한 이유

애플리케이션을 개발할 때 중요한 데이터는 대부분 데이터베이스에 보관한다.  
  
클라이언트가 애플리케이션 서버를 통해 데이터를 저장하거나 조회하면, 애플리케이션 서버는 일련의 과정을 통해서 데이터베이스를 사용한다.  
- 1\. 커넥션 연결: 주로 TCP/IP를 사용해서 커넥션을 연결한다.
- 2\. SQL 전달: 연결된 커넥션을 통해 SQL을 DB에 전달한다.
- 3\. DB는 전달받은 SQL을 수행하여 그 결과를 서버에 응답한다.

#

문제는 각각의 데이터베이스마다 커넥션을 연결하는 방법, SQL을 전달하는 방법, 결과를 응답받는 방법이 모두 다르기 때문에,
  
개발자는 데이터베이스가 변경될 때마다, 애플리케이션 서버에 개발된  
데이터베이스 사용코드를 함께 변경해야하고, 그 방법들을 계속해서 학습해야한다.

---

## JDBC 표준 인터페이스

이런 문제를 해결하기 위해 JDBC (Java DataBase Connectivity)라는 자바 표준이 등장하게 된다.  
JDBC는 다음 3가지 기능을 표준 인터페이스로 정의해서 제공한다.

- java.sql.Connection - 연결
- java.sql.Statement - SQL을 담은 내용
- java.sql.ResultSet - SQL 요청 응답

```java
public Member findById(String memberId) throws SQLException {
    String sql = "select * from member where member_id = ?";

    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null; // executeQuery()의 결과를 반환

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, memberId);

        rs = pstmt.executeQuery(); // 데이터를 조회할 때는 executeQuery()를 사용한다.

        if (rs.next()) {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        } else {
            throw new NoSuchElementException("member not found memberId=" + memberId);
        }

    } catch (SQLException e) {
        log.error("db error", e);
        throw e;
    } finally {
        close(con, pstmt, rs);
    }
}
```

JDBC의 등장으로 다음 2가지 문제가 해결되었다.  
  
- 애플리케이션 로직은 이제 JDBC 표준 인터페이스에만 의존하기 때문에,  
- 데이터베이스를 다른 종류의 데이터베이스로 변경하고 싶으면 JDBC 구현 라이브러리만 변경하면 된다.
- 따라서 다른 종류의 데이터베이스로 변경해도 사용 코드를 그대로 유지할 수 있다.
- 
- 개발자는 JDBC 표준 인터페이스 사용법만 학습하면 된다.
- 한번 배워두면 수십개의 데이터베이스에 모두 동일하게 적용할 수 있다.

---

## JDBC와 최신 데이터 접근 기술

JDBC는 1997년에 출시될 정도로 오래된 기술이고, 사용하는 방법도 복잡하다.  
그래서 최근에는 JDBC를 직접 사용하기 보다는, JDBC를 편리하게 사용하는 다양한 기술이 존재한다.

### SQL Mapper
- 장점
    - SQL응답 결과를 객체로 편리하게 변환해준다.
    - JDBC의 반복 코드를 제거해준다.
- 단점
    - 개발자가 SQL을 직접 작성해야 한다.
- 대표 기술: 스프링 JdbcTemplate, MyBatis

### ORM 기술
- ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술이다.
- 이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신 SQL을 동적으로 만들어 실행해준다.
- 추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다.
-
- 대표 기술: JPA, 하이버네이트, 이클립스링크

---

## Reference
- [스프링 DB 1편 - 데이터 접근 핵심 원리](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard)





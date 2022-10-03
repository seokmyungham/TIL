# 스프링 JdbcTemplate

SQL을 직접 사용할 경우, 스프링이 제공하는 JdbcTemplate는 좋은 선택지가 될 수 있다.  
JdbcTemplate는 JDBC를 매우 편리하게 사용할 수 있도록 도와준다.  

### 장점
- 설정의 편리함
    - JdbcTemplate는 스프링으로 JDBC를 사용할 때 기본으로 사용되는 라이브러리인 spring-jdbc에 포함되어 있다.
    - 별도의 설정없이 바로 사용할 수 있다.
- 반복 문제 해결
    - JdbcTemplate는 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.
    - 개발자는 SQL을 작성하고, 전달할 파라미터를 정의하고, 응답 값을 매핑하기만 하면 된다.
    - EX) 커넥션 획득, statement 준비 및 실행, 커넥션 종료-statement 종료-resultset 종료, 커넥션 동기화, 스프링 예외 변환기 실행

### 단점
- 동적 SQL을 해결하기 어렵다.


---

## JdbcTemplate 설정

### build.gradle
```gradle

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'

	//JdbcTemplate 추가
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'
	//H2 데이터베이스 추가
	runtimeOnly 'com.h2database:h2'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	//테스트에서 lombok 사용
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
}
```

'org.springframwork.boot:spring-boot-starter-jdbc' 한 줄만 추가하면 JdbcTemplate를 사용할 수 있는 spring-jdbc가 라이브러리에 포함된다.

---

## JdbcTemplate의 주요 기능 정리

- JdbcTemplate
    - 순서 기반 파라미터 바인딩 지원
- NamedParameterJdbcTemplate
    - 이름 기반 파라미터 바인딩 지원 *
- SimpleJdbcInsert
    - insert sql을 편리하게 사용할 수 있다.
- SimpleJdbcCall
    - 스토어드 프로시저를 편리하게 호출할 수 있다.

---

## JdbcTemplate

### ItemRepository 인터페이스
```java
package hello.itemservice.repository;

import hello.itemservice.domain.Item;

import java.util.List;
import java.util.Optional;

public interface ItemRepository {

    Item save(Item item);

    void update(Long itemId, ItemUpdateDto updateParam);

    Optional<Item> findById(Long id);

    List<Item> findAll(ItemSearchCond cond);

}
```

#

### JdbcTemplateItemRepositoryV1
```java

package hello.itemservice.repository.jdbctemplate;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;
import java.sql.PreparedStatement;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * JdbcTemplate
 */

@Slf4j
@Repository
public class JdbcTemplateItemRepositoryV1 implements ItemRepository {
   
    /**
     * 스프링에서는 JdbcTemplate를 사용할 때 관례상
     * datasource를 의존 관계 주입 받고 생성자 내부에서 JdbcTemplate를 생성하는 방법을 많이 사용한다
     */
    
    private final JdbcTemplate template;
    
    public JdbcTemplateItemRepositoryV1(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource); //JdbcTemplate는 데이터소스를 필요로 한다.
    }

    @Override
    public Item save(Item item) {
        String sql = "insert into item(item_name, price, quantity) values (?, ?, ?)";
        
        /**
         * 데이터를 저장할 때 PK 생성에 identity(auto increment) 방식을 사용하기 때문에,
         * PK인 ID 값을 비워두고 저장한다. 그러면 데이터베이스가 PK인 ID를 대신 생성해준다.
         * 
         * PK ID값은 데이터베이스가 생성하기 때문에, 데이터베이스에 INSERT가 완료 되어야 생성된 PK ID 값을 확인할 수 있다.
         */
        
        KeyHolder keyHolder = new GeneratedKeyHolder();
        
        //template.update()의 반환값은 영향 받은 로우 수를 반환한다.
        template.update(connection -> {
            //자동 증가 키
            PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
            ps.setString(1, item.getItemName());
            ps.setInt(2, item.getPrice());
            ps.setInt(3, item.getQuantity());
            return ps;
        }, keyHolder);

        //KeyHolder와 prepareStatement를 통해 id를 지정해주면 INSERT 쿼리 실행 이후 데이터베이스에서 생성된 ID 값을 조회할 수 있다.
        long key = keyHolder.getKey().longValue();
        item.setId(key);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "update item set item_name=?, price=?, quantity=? where id=?";
        
        //sql 쿼리 ?에 바인딩할 파라미터를 순서대로 전달한다.
        template.update(sql,
                updateParam.getItemName(),
                updateParam.getPrice(),
                updateParam.getQuantity(),
                itemId);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "select id, item_name, price, quantity from item where id=?";

        /**
         * queryForObject 는 결과 로우가 하나일 때 사용한다.
         * RowMapper 는 데이터베이스의 반환 결과인 ResultSet을 객체로 변환한다.
         * 결과가 없으면 EmptyResultDataAccessException 예외가 발생한다.
         * 결과가 둘 이상이면 IncorrectResultSizeDataAccessException 예외가 발생한다.
         */

        try {
            Item item = template.queryForObject(sql, itemRowMapper(), id);
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
        
            //ItemRepository 인터페이스의 findById 는 결과가 없을 때 Optional을 반환해야 한다. 
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        String sql = "select id, item_name, price, quantity from item";

        //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        List<Object> param = new ArrayList<>();
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',?,'%')";
            param.add(itemName);
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= ?";
            param.add(maxPrice);
        }
        log.info("sql={}", sql);
        
        /**
         * template.query()
         * 결과가 하나 이상일 때 사용한다.
         * RowMapper는 데이터베이스의 반환 결과인 ResultSet을 객체로 변환한다.
         * 결과가 없으면 빈 컬렉션을 반환한다. 
         */
        return template.query(sql, itemRowMapper(), param.toArray());

    }

    private RowMapper<Item> itemRowMapper() {
        return (rs, rowNum) -> {
            Item item = new Item();
            item.setId(rs.getLong("id"));
            item.setItemName(rs.getString("item_name"));
            item.setPrice(rs.getInt("price"));
            item.setQuantity(rs.getInt("quantity"));
            return item;
        };
    }

}
```

---

## NamedParameterJdbcTemplate

### JdbcTemplateItemRepositoryV2
```java
package hello.itemservice.repository.jdbctemplate;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.Optional;

/**
 * NamedParameterJdbcTemplate
 * SqlParameterSource
 * - BeanPropertySqlParameterSource
 * - MapSqlParameterSource
 * Map
 *
 * BeanPropertyRowMapper
 *
 */
@Slf4j
@Repository
public class JdbcTemplateItemRepositoryV2 implements ItemRepository {
    
    private final NamedParameterJdbcTemplate template;

    public JdbcTemplateItemRepositoryV2(DataSource dataSource) {
    
        // NamedParameterJdbcTemplate도 내부에 dataSource가 필요하다.
        this.template = new NamedParameterJdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "insert into item(item_name, price, quantity) " +
                "values (:itemName, :price, :quantity)";

        SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        KeyHolder keyHolder = new GeneratedKeyHolder();
        template.update(sql, param, keyHolder);

        Long key = keyHolder.getKey().longValue();
        item.setId(key);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "update item " +
                "set item_name=:itemName, price=:price, quantity=:quantity " +
                "where id=:id";


        SqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("id", itemId);

        template.update(sql, param);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "select id, item_name, price, quantity from item where id=:id";

        try {
            Map<String, Object> param = Map.of("id", id);
            Item item = template.queryForObject(sql, param, itemRowMapper());
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {

        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();

        SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

        String sql = "select id, item_name, price, quantity from item";

        //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= :maxPrice";
        }
        log.info("sql={}", sql);
        return template.query(sql, param, itemRowMapper());

    }

    private RowMapper<Item> itemRowMapper() {
        return BeanPropertyRowMapper.newInstance(Item.class);
    }

}
```

### 이름 지정 파라미터

파라미터를 전달하려면 Map처럼 key(:파라미터이름), value(파라미터의 값) 데이터 구조를 만들어서 전달해야 한다.  
```java
template.update(sql, param, keyHolder);
```  
  
이름 지정 바인딩에서 자주 사용하는 파라미터의 종류는 3가지이다.
- Map
- SqlParameterSource
    - MapSqlParameterSource
    - BeanPropertySqlParameterSource

### 1. Map
```java
String sql = "select id, item_name, price, quantity from item where id=:id";

Map<String, Object> param = Map.of("id", id);
Item item = template.queryForObject(sql, param, itemRowMapper());
```

### 2. MapSqlParameterSource

Map과 유사하지만 SQL 타입을 지정할수 있는 등 SQL에 좀 더 특화된 기능을 제공한다.  
```java
String sql = "update item " +
                "set item_name=:itemName, price=:price, quantity=:quantity " +
                "where id=:id";

// 파라미터로 넘어오는 ItemUpdateDto 에는 itemId가 없으므로 이럴 땐 MapSqlParameterSource를 이용한다.
SqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("id", itemId);
template.update(sql, param);
```

### 3. BeanPropertySqlParameterSource

자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성해준다. (getItemName() -> itemNaem)  
```java
String sql = "insert into item(item_name, price, quantity) " +
                "values (:itemName, :price, :quantity)";
                
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(sql, param, keyHolder);
```

#

### BeanPropertyRowMapper
```java
private RowMapper<Item> itemRowMapper() {
    return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
}
```

BeanPropertyRowMapper는 ResultSet의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.  

#

### 별칭

자바빈 프로퍼티 규약을 활용해야 할 때 ```select item_name```의 경우 setItem_name() 이라는 메서드를 쓰지는 않는데,  
이럴 경우엔 개발자가 조회 SQL을 ```select item_name as itemName``` 으로 고쳐서 쓰면 된다.
  
특히 데이터베이스 컬럼 이름과 객체 이름이 완전히 다를 때 문제를 해결하는데 도움이 된다.  
  
예를 들어 데이터베이스에는 member_name이라고 되어있는데, 객체에는 username이라고 되어 있다면  
```select member_name as username``` 으로 별칭을 사용해서 해결할 수 있다.  
JdbcTemplate은 물론이고, MyBatis같은 기술에서도 자주 사용된다.
  
### 관례의 불일치

자바 객체는 카멜(camelCase) 표기법을 사용한다.  
반면 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 snake_case 표기법을 사용한다.  
  
서로 관례를 많이 사용하다 보니 BeanPropertyRowMapper는 언더스코어 표기법을 카멜로 자동 변환해준다.  
따라서 ```select item_name``` 으로 조회해도 setItemName()에 문제 없이 값이 들어간다.

---

## SimpleJdbcInsert

```java
/**
 * SimpleJdbcInsert
 */
@Slf4j
@Repository
public class JdbcTemplateItemRepositoryV3 implements ItemRepository {

    private final NamedParameterJdbcTemplate template;
    private final SimpleJdbcInsert jdbcInsert;

    public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
		
        this.jdbcInsert = new SimpleJdbcInsert(dataSource)
                .withTableName("item") //데이터를 저장할 테이블 이름
                .usingGeneratedKeyColumns("id"); //key를 생성하는 PK 컬럼 명을 지정
//                .usingColumns("item_name", "price", "quantity") //생략 가능
    }

    @Override
    public Item save(Item item) {
        SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        Number key = jdbcInsert.executeAndReturnKey(param); // INSERT SQL을 실행하고, 생성 키 값도 편리하게 조회할 수 있다.
        item.setId(key.longValue());
        return item;
    }
```

SimpleJdbcInsert는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다.  
따라서 usingColumns을 생략할 수 있다. 특정 컬럼만 지정해서 저장하고 싶으면 usingColumns을 사용하면 된다.

---

## Reference
- [스프링 DB 2편 - 데이터 접근 활용 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2/dashboard)
- [스프링 JdbcTemplate 사용 방법 공식 메뉴얼](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-JdbcTemplate)

Jdbc

<hr></hr>

<small>Basic</small> <spring_practice_repository>

```java
public class JdbcMemberRepository implements MemberRepository {
    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    //db에 붙으려면 datasource가 필요합니다.

    //application.properties에 설정을 해놓으면 스프링이 객체를 만들어서
    //datasource라고 칭합니다. 그리고 나중에 주입받습니다.

    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            //여기서 connection을 얻어오면 진짜 데이터베이스와 연결될 수 있는 소켓을 얻어옵니다.
            //까보면 트렌젝션이 안나도록 데이터베이스를 유지하는 용도.


            pstmt = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
            //conn.prepareStatement(sql)
            //Statement.RETURN_GENERATED_KEYS id를 얻습니다.

            pstmt.setString(1, member.getName());
            //1은 변수 번호와 getname으로 값을 얻어옵니다.
            /*
            SQL문에서 변수가 들어갈 자리는 ' ? ' 로 표시한다. , 실행시에 ?에 대응되는 값을 지정할때
            setString(int parameterIndex, String X)이나 setInt(int parameterIndex, int x)와 같이
            setXXX메소드를 통해 설정한다.
            그리고  PreparedStatement 는 SQL문에서 Like키워드를 사용할경우 사용할수없다.
             */


            pstmt.executeUpdate();
            //쿼리를 날리는 executeUpdate



            rs = pstmt.getGeneratedKeys();
            //Statement.RETURN_GENERATED_KEYS 이 상수랑 매칭

            if (rs.next()) {
                member.setId(rs.getLong(1));
                //	getLong(int columnIndex)
                //Retrieves the value of the designated column
                // in the current row of this ResultSet object as a long
                // in the Java programming language.
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);

            //리소스 반환을 꼭해줘야함.
        }
    }
```

<small> 순수 jdbc 예제 <전체 소스코드는 spring practice 저장소></small>





#### jdbcTemlate

<hr></hr>

###### query

```java

@Override
public Optional<Member> findById(Long id) {
    List<Member> result = jdbcTemplate.query("select * from member where id=?", memberRowMapper());
    //select 문의 결과를 rowMapper클래스의 mapRow로 넘겨져서 처리됨.
    //그리고 결과를 list로 리턴해줍니다.
    return result.stream().findAny();
}
```

###### 매퍼클래스 오버라이딩

<small>select 문의 결과를 rowMapper클래스의 mapRow로 넘겨져서 처리됨.
그리고 결과를 list로 리턴해줍니다.</small>

```java
private RowMapper<Member> memberRowMapper(){
    return (rs, rowNum) -> {
        Member member=new Member();
        member.setId(rs.getLong("id"));
        member.setName(rs.getString("name"));

        return member;
    };
}
/*
private RowMapper<Member> memberRowMapper(){
    return new RowMapper<Member>() {
        @Override
        public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
            Member member=new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));

            return member;
        }
    }
}
 */
```

<hr></hr>

`Long getLong("table name")`

`String getString("table name")`

```resultSet.getLong, String ...```

Retrieves the value of the designated column in the current row of this `ResultSet` object as a `long` in the Java programming language.



##### SimpleJdbcInsert, class 'MapSqlParameterSource'

<hr></hr>

```java
 SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

Map<String, Object> parameters = new HashMap<>();
parameters.put("name", member.getName());




Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));




member.setId(key.longValue());
return member;
```

- usingGeneratedKeyColumns(String... columnNames) : Specify the names of any columns that have auto generated keys.
   -> 자동으로 만들어지는 키값을 가진 어느 컬럼의 구체적인 이름

- withTableName(String tableName) : Specify the table name to be used for the insert.
  -> 구체적인 테이블 이름 삽입하기 위해 쓰여지는

``` java
 Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));   
```

- executeAndReturnKey -> 키값을 설정함에 따른 값 매칭. -> string 등 map<v k>등이 들어갈 수 있음.
- <img src="IMG\5.PNG" alt="image-20210115024745077" style="zoom:80%;" />

#### * 부록

```java
@Override
public Optional<Member> findByName(String name) {
    List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
    return result.stream().findAny();
}

@Override
public List<Member> findAll() {
    return jdbcTemplate.query("select * from member", memberRowMapper());
}
```
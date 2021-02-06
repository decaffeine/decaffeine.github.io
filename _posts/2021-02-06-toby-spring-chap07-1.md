---
layout: post
title: '7장 스프링 핵심 기술의 응용 1'
date: 2021-02-06 22:30:00
category: [toby-spring]
draft: false
comments: true
---

# 7. 스프링 핵심 기술의 응용

-   스프링의 3대 핵심 기술 : IoC/DI, 서비스 추상화, AOP
    -   결국 자바 언어가 기반을 두고 있는 객체지향 기술
    -   개발자도 스프링이 제공하는 3가지 기술을 필요에 따라 스스로 응용할 수 있어야 함
-   지금까지 살펴봤던 세 가지 기술을 애플리케이션 개발에 활용해 보자

## 7.1. SQL과 DAO의 분리

-   SQL을 적절히 분리해 DAO 코드와 다른 파일이나 위치에 두고 관리하자

### 7.1.1 XML 설정을 이용한 분리

-   개별 SQL 프로퍼티 방식 - XML 설정파일에 프로퍼티로 분리
    -   매번 새로운 SQL이 필요할 때마다 프로퍼티를 추가하고 DI를 위한 변수, setter도 만들어 주어야 함

```java
        public class UserDaoJdbc {
            private String sqlAdd;
            public void setSqlAdd(String sqlAdd) {
                this.sqlAdd = sqlAdd;
            }
        }
```

```xml
        <bean id="userDao" class="springbook.user.dao.UserDaoJdbc" >
          <property name="dataSource" ref="dataSource" />
          <property name="sqlAdd" value="insert into users.... (?,?,?)..." />
```

-   SQL 맵 프로퍼티 방식

    -   여러 개의 SQL을 하나의 컬렉션에 담아 사용하도록 수정
        -   다만 Map이므로 property 태그로는 사용이 불가
        -   스프링이 제공하는 map 태그 사용

```java
    public class UserDaoJdbc {
        private Map<String, String> sqlMap;
    }
```

```xml
     <bean id="userDao" class="springbook.user.dao.UserDaoJdbc" >
        <property name="dataSource" ref="dataSource" />
        <property name="sqlMap">
            <map>
                <entry key="add" value="insert into users...." />
                <entry key="get" value="select * from users...">
                ...
            </map>
        </property>
```

### 7.1.2 SQL 제공 서비스

-   SQL과 DI 설정정보가 섞여 있으면 보기에도 지저분하고 관리에도 좋지 않음

    -   꼭 빈 설정 방법을 이용해 XML에 담아둘 이유도 없음
        -   프로퍼티 파일, 엑셀 파일, 임의 포맷 파일, DB, 외부 저장소 등등..
    -   SQL 제공 기능을 독립시킬 필요가 있음

-   SQL 서비스 인터페이스

    -   클라이언트인 DAO를 SQL 서비스의 구현에서 독립적으로 만들도록 인터페이스를 사용
    -   DAO가 사용할 SQL 서비스의 기능은 간단
        -   키를 주면 그에 해당하는 SQL을 돌려줌
        -   무슨 기술을 써서 SQL을 가져오는지 .. 등등은 DAO의 관심 사항이 아님 ! (객체지향)

-   스프링 설정을 사용하는 단순 SQL 서비스

```java
public class UserDaoJdbc implements UserDao {
    private SqlService sqlService;

    public void setSqlService(SqlService sqlService) {}
}

public interface sqlService {
    String getSql(String key) throws SqlException;
}

public class SimpleSqlService implements SqlService {
    private Map<String, String> sqlMap;

    public void setSqlMap(Map<String, String> sqlMap) {}

    public String getSql(String key) throws SqlRetrievalFailtureException {
        String sql = sqlMap.get(key);
        if (sql == null) throw new SqlRetrievalFailtureException(..);
        else return sql;
    }
}

```

```xml
<bean id="sqlService" class="...SimpleSqlService">
    <property name="sqlMap">
        <map>
            <entry key="userAdd" value="insert into..." />
            <entry key="userGet" value="select * from users...">
        </map>
    </property>
```

-   겉으로 보기엔 앞에서 쓴 방법과 큰 차이가 없어보이지만
    -   이제 UserDao를 포함한 모든 DAO는 SQL을 어디에 저장해두고 가져오는지에 대해서는 전혀 신경쓰지 않아도 됨

## 7.2. 인터페이스의 분리와 자기참조 빈

-   인터페이스가 하나 있으니 기계적으로 구현 클래스 하나만 만들면 될 거라고 생각하면 오산!
-   어떤 인터페이스는 그 뒤에 숨어 있는 방대한 서브시스템의 관문에 불과할 수도 있음

### 7.2.1 XML 파일 매핑

-   JAXB
    -   XML에 담긴 정보를 파일에서 읽어오는 방법은 다양
        -   가장 간단하게 사용할 수 있는 방법 중 하나인 JAXB java architecture for XML binding
        -   java.xml.bind 패키지 안에서 JAXB의 구현 클래스를 찾을 수 있음
    -   스키마를 이용해 매핑할 오브젝트의 클래스까지 자동으로 만들어주는 컴파일러도 제공

```xml
...
<element type="sqlmap">
    <complexType>
        <sequence>
            <element name="sql" maxOccurs="unbounded" type="tns:sqlType" />
        </sequence>
    </complexType>
</element>

<complexType name="sqlType">
    <simpleContent>
        <extension base="string">
            <attribute name="key" use="required" type="string">
        </extension>
</complexType>

</schema>
```

-   이 파일을 JAXB 컴파일러로 컴파일하면..

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "", propOrder = {
    "sql"
})
@XmlRootElement(name = "sqlmap")
public class Sqlmap {

    @XmlElement(required = true)
    protected List<SqlType> sql;

    public List<SqlType> getSql() {
        if (sql == null) {
            sql = new ArrayList<SqlType>();
        }
        return this.sql;
    }

}

@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "sqlType", propOrder = {
    "value"
})
public class SqlType {

    @XmlValue
    protected String value;
    @XmlAttribute(required = true)
    protected String key;


    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String value) {
        this.key = value;
    }

}

```

-   언마샬링
    -   XML 문서를 읽어서 자바의 오브젝트로 변환하는 것이 언마샬링 (unmarshalling)
        -   반대로 바인딩 오브젝트 -> XML은 마샬링

```java
public class JaxbTest {
	@Test
	public void readSqlmap() throws JAXBException, IOException {

		String contextPath = Sqlmap.class.getPackage().getName();
		JAXBContext context = JAXBContext.newInstance(contextPath);
		Unmarshaller unmarshaller = context.createUnmarshaller();
		Sqlmap sqlmap = (Sqlmap) unmarshaller.unmarshal(
				getClass().getResourceAsStream("sqlmap.xml"));

		List<SqlType> sqlList = sqlmap.getSql();

		assertThat(sqlList.size(), is(3));
		assertThat(sqlList.get(0).getKey(), is("add"));
		assertThat(sqlList.get(0).getValue(), is("insert"));
		assertThat(sqlList.get(1).getKey(), is("get"));
		assertThat(sqlList.get(1).getValue(), is("select"));
		assertThat(sqlList.get(2).getKey(), is("delete"));
		assertThat(sqlList.get(2).getValue(), is("delete"));
	}
}
```

### 7.2.2 XML 파일을 이용하는 SQL 서비스

-   언제 JAXB를 사용해 XML 문서를 가져올 것인가?
    -   특별한 이유가 없는 한 XML 파일은 한 번만 읽도록 해야 한다
        -   생성자 (XmlSqlService() 에 초기화 로직)

```java
public class XmlSqlService implements SqlService {
    ...
    public XmlSqlService() {
        String contextPath = Sqlmap.class.getPackage().getName();
        try {
        JAXBContext context = JAXBContext.newInstance(contextPath);
		Unmarshaller unmarshaller = context.createUnmarshaller();
        ...
        sqlMap.put(sql.getKey(), sql.getValue());
        }
    }
}
```

### 7.2.3 빈의 초기화 작업

-   생성자에서 예외가 발생할 수도 있는 복잡한 초기화 작업을 다루는 것은 좋지 않음
    -   일단 초기 상태를 가진 오브젝트를 만들어 놓고 별도의 초기화 메소드를 사용하는 방법이 바람직
    -   위 생성자 안에 있던 로직을 loadSql()로 이동.
-   읽어들일 팡리의 위치와 이름이 코드에 고정
-   XmlSqlService 오브젝트는 빈이므로 초기화는 스프링이 한다
    -   스프링의 빈 후처리기 (컨테이너가 빈을 생성한 뒤에 부가적인 작업을 수행할 수 있도록 함)
    -   어노테이션을 이용 (`<context:annotation-config />`)

```java
public class XmlSqlService implements SqlService {

    @PostCunstruct
    public void loadSql() { ... }
}
```

### 7.2.4 변화를 위한 준비: 인터페이스 분리

-   아직 확장할 영역이 많이 남아 있다

    -   현재의 구조 : 특정 포맷 XML에서 SQL 데이터 가져오고 / 이를 HashMap 타입의 맵 오브젝트에 저장
    -   SQL을 가져오는 것과 보관해두고 사용하는 것은 충분히 독자적인 이유로 변경 가능한 독립적인 전략
        -   서로 변하는 시기와 성질이 다른 것, 변하는 것과 변하지 않는 것을 함께 두는 건 바람직한 설계구조가 아님

-   책임에 따른 인터페이스 정의
    1. SQL 정보를 외부의 리소스로부터 읽어 옴 (SqlReader)
    2. 읽어온 SQL을 보관해두고 있다가 필요할 때 제공 (SqlRegistry)
-   SqlService가 SqlReader에게 SqlRegistry 전략을 제공해주며, 이를 이용해 SQL 정보를 SqlRegistry에 저장하라고 요청하는 구조.

-   자바의 오브젝트는 데이터를 가질 수 있고, 자신이 가진 데이터를 이용해 어떻게 작업해야 할지도 가장 잘 알고 있음
    -   쓸데없이 외부로 자신의 데이터를 노출시킬 필요는 없다 (객체지향!!)

```java
public interface SqlReader {
	void read(SqlRegistry sqlRegistry);
}

public interface SqlRegistry {
	void registerSql(String key, String sql);

	String findSql(String key) throws SqlNotFoundException;
}
```

### 7.2.5 자기참조 빈으로 시작하기

-   SqlReader의 구현 클래스, SqlService의 구현 클래스, SqlRegistry의 구현 클래스 총 3개를 만들어야 하는데
    -   일단 이 3개의 인터페이스를 하나의 클래스가 모두 구현한다면 어떨까?
    -   다만 다른 인터페이스의 변수에 직접 접근하거나 하지만 않으면 된다.

```java
public class XmlSqlService implements SqlService, SqlRegistry, SqlReader {
	// --------- SqlProvider ------------
	private SqlReader sqlReader;
	private SqlRegistry sqlRegistry;

	public void setSqlReader(SqlReader sqlReader) {
		this.sqlReader = sqlReader;
	}

	public void setSqlRegistry(SqlRegistry sqlRegistry) {
		this.sqlRegistry = sqlRegistry;
	}

	@PostConstruct
	public void loadSql() {
		this.sqlReader.read(this.sqlRegistry);
	}

	public String getSql(String key) throws SqlRetrievalFailureException {
		try {
			return this.sqlRegistry.findSql(key);
		}
		catch(SqlNotFoundException e) {
			throw new SqlRetrievalFailureException(e);
		}
	}

	// --------- SqlRegistry 구현 ------------
	private Map<String, String> sqlMap = new HashMap<String, String>();

	public String findSql(String key) throws SqlNotFoundException {
		String sql = sqlMap.get(key);
		if (sql == null)
			throw new SqlRetrievalFailureException(key + "를 이용해서 SQL을 찾을 수 없습니다");
		else
			return sql;

	}

	public void registerSql(String key, String sql) {
		sqlMap.put(key, sql);
	}

	// --------- SqlReader 구현 ------------
	private String sqlmapFile;

	public void setSqlmapFile(String sqlmapFile) {
		this.sqlmapFile = sqlmapFile;
	}

	public void read(SqlRegistry sqlRegistry) {
		String contextPath = Sqlmap.class.getPackage().getName();
		try {
			JAXBContext context = JAXBContext.newInstance(contextPath);
			Unmarshaller unmarshaller = context.createUnmarshaller();
			InputStream is = UserDao.class.getResourceAsStream(sqlmapFile);
			Sqlmap sqlmap = (Sqlmap)unmarshaller.unmarshal(is);
			for(SqlType sql : sqlmap.getSql()) {
				sqlRegistry.registerSql(sql.getKey(), sql.getValue());
			}
		} catch (JAXBException e) {
			throw new RuntimeException(e);
		}
	}
}
```

-   자기참조 빈 설정
    -   흔히 쓰이는 방법은 아니지만, 책임과 관심사가 복잡하게 얽혀 있어서 확장이 힘들고 변경에 취약한 구조의 클래스를 유연한 구조로 만들려고 할 때 처음 시도해볼 수 있는 방법이다

```xml
<bean id="sqlService" class="...">
    <property name="sqlReader" ref="sqlService" />
    <property name="sqlRegistry" ref="sqlService" />
    <property name="sqlmapFile" value="sqlmap.xml" />
</bean>
```

### 7.2.6 디폴트 의존관계

-   이제 개별 클래스로 분리하고
    -   클래스가 너무 많아지는 거 같아서 귀찮겠지만
    -   확장을 고려해서 기능을 분리하고, 인터페이스와 전략 패턴을 도입하고, DI를 적용한다면 늘어난 클래스 갯수와 의존관계 설정에 대한 부담은 감수해야 함 (그렇구나..)
-   만약을 위해 기본값 세팅만 해 주면 된다.

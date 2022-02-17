## Chapter 10. 클래스

### 클래스 체계

- 변수 목록
  - 정적 공개 상수
  - 정적 비공개 변수
  - 비공개 인스턴스 변수
- 공개 함수
- 비공개 함수

=> 추상화 단계가 순차적으로 내려간다.



[캡슐화]

- 같은 패키지 안에서 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 함수나 변수를 protected로 선언하거나 패키지 전체로 공개

- 캡슐화를 풀어주는 결정은 언제나 최후의 수단



### 클래스는 작아야 한다!

> 작게! 더 작게!

- 간결한 클래스 이름이 떠오르지 않는다면?
  - 필경 클래스 크기가 너무 크기 때문에
- 클래스 이름이 모호하다면?
  - 필경 클래스 책임이 너무 많기 때문에



[단일 책임 원칙]

- 클래스나 모듈을 변경할 이유가 하나, 단 하나뿐이여야 한다는 원칙
- 소프트웨어를 돌아가게 만드는 활동과 소프트웨어를 깨끗하게 만드는 활동은 완전히 별개다
  - 우리들은 깨끗하고 체계적인 소프트웨어 보다 돌아가는 소프트웨어에 초점을 맞춘다.
- 큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템에 더 바랍직하다.
- 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.



[응집도]

- 클래스는 인스턴스 변수 수가 작아야 한다.
- 응집도가 높다 = 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다.
- 변수와 메서드를 분리해 새로운 클래스 두세 개로 쪼갠다면 응집도를 높일 수 있다.



[응집도를 유지하면 작은 클래스 여럿이 나온다]

- 큰 함수를 작은 함수 여럿으로 나눈다면 몇몇 함수만 사용하는 인스턴스 변수가 늘어나기 때문에 클래스는 응집력을 잃는다.
- 큰 함수를 작은 함수 여럿으로 쪼개다보면 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.



[10-5 PrintPrimes.java]

- 들여쓰기가 심하고, 이상한 변수가 많고, 구조가 빡빡하게 결합되었다.

  - 여러 함수로 나눠야 마땅하다.

  

[10-6 리팩터링한 코드]

- 리팩터링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다.
- 리팩터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다.
- 가독성ㅇ르 높이고자 공백을 추가하고 형식을 맞췄다.

### 변경하기 쉬운 클래스

- 대다수 시스템은 지속적인 변경이 가해진다. 그리고 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다.

- 깨끗한 시스템은 클래스를 체계적으로 정리해 변경해 수반하는 위험을 낮춘다.

```java
public class Sql {
    public Sql(String table, Column[] colums)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns)
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

- 주어진 메타 자료로 적절한 SQL 문자열을 만드는 sql 클래스이다.
- 언젠가 update 문을 지원할 시점이 오면 클래스를 고쳐야한다.
- 문제는 **코드에 손대면 위험이 생긴다**는 사실이다.
- 새로운 SQL 문을 지원하려면 반드시 sql 클래스는 손대야 한다.

- 기존 SQL문 하나를 수정할 때도 반드시 Sql 클래스를 고쳐야한다
- 위와 같이 변경할 이유가 두 가지이므로 Sql 클래스는 SRP를 위반한다.



```JAVA
abstract public class Sql {
    public Sql(String table, Column[] columns)
    abstract public String generate()
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns)
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns)
    @Override public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields)
    @Override public String generate()
    private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extneds Sql {
    public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria)
    @Override public String generate()
}

public class SelectWithMatchSql extends Sql {
     public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern)
    @Override public String generate()
}

public class FindByKeySql extends Sql {
    public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue)
    @Override public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns)
    @Override public String generate()
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria)
    public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns)
    public String generate()
}
```

- 공개 인터페이스를 각각 Sql 클래스에서 파생하는 클래스로 만들었다.
- valueList와 같은 비공개 메서드는 해당하는 파생 클래스로 옮겼다.
- 모든 파생 클래스가 공통으로 사용하는 비공개 메서드는 Where과 ColumnList라는 두 유틸리티 클래스에 넣었다.
- 이제 함수 하나를 수정했다고 다른 함수가 망가질 위험도 사라졌다.
- 클래스가 서로 분리되었기 때문에 테스트 관점에서 모든 논리를 증명하기도 쉬워졌다.
- 위 클래스는 SRP를 지원할 뿐더러 OCP도 지원한다.
  - OCP : 클래스는 확장에 개방적이고 수정에 폐쇄적이어야 한다.

- 이상적인 시스템이라면 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드는 변경하지 않는다.



[변경으로부터 격리]

> 요구사항은 변하기 마련이다. 코드도 변하기 마련이다.

- 구체적인 클래스와 추상 클래스가 있다.
- 구체적인 클래스는 상세한 구현을 포함하며 추상 클래스는 개념만 포함한다.
- 상세한 구현에 의존하는 코드는 테스트가 어렵다.



Portfolio 클래스는 만든다고 가정하자. Portfolio 클래스는 외부 TokyoStockExchangeAPI를 사용해 포트폴리오 값을 계산한다. 따라서 우리 테스트 코드는 시세 변화에 영향을 받는다. 매번 값이 달라지는 API로 테스트 코드를 짜기란 쉽지 않다.

```JAVA
public interface StockExchange {
    Money currentPrice(String symbol);
}
```

- Portfolio 클래스에서 TokyoStockExchange API를 직접 호출하는 대신 StockExchange 인터페이스 생성 후 메서드 선언



```java
public Portfolio {
    private StockExchange exchange;
    public Protfolio(StockExchange exchange){
        this.exchange = exchange;
    }
    // ...
}
```

- TokyoStockExchange 클래스 구현



```java
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub(); 
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value()); 
    }
}
```

- 테스트용 클래스는 StockExchange 인터페이스를 구현하며 고정된 주가를 반환한다.
- 위와 같은 테스트가 가능한 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.
  - 결합도가 낮다? = 각 시스템 요소가 다른 요소로부터, 변경으로부터 잘 격리되어 있다는 의미

- 결합도를 최소로 줄이면 자연스럽게 DIP를 따르는 클래스가 나온다.
  - DIP : 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙
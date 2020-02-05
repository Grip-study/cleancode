# ch10. 클래스

> **도구 상자를 어떻게 관리하고 싶은가**
>
> **작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가**
>
> **아니면 큰 서랍 몇 개를 두고 그 안에 모두 던져 넣고 싶은가**

<br>

### 클래스 체계

1. 정적 공개 static public 상수 
2. 정적 비공개 static private  변수
3. 비공개 인스턴스 private instance 변수
4. 공개 함수 public method
5. 비공개 함수 private method (자신을 호출하는 공개 함수 바로 뒤에 넣음)

<br>

### 클래스는 작아야 한다 (단일 책임 원칙 SRP)

**Bad: 공개 메서드가 약 70개가 되는 만능 클래스**

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public String getCustomizerLanguagePath()
    public void setSystemConfigPath(String systemConfigPath) 
    public String getSystemConfigDocument()
    public void setSystemConfigDocument(String systemConfigDocument) 
    public boolean getGuruState()
    public boolean getNoviceState()
    public boolean getOpenSourceState()
    public void showObject(MetaObject object) 
    public void showProgress(String s)
    public boolean isMetadataDirty()
    public void setIsMetadataDirty(boolean isMetadataDirty)
    public Component getLastFocusedComponent()
    ... 약 70개의 메소드
}
```

```
1. 클래스 설명이 "만일, 그리고, 하며, 하지만" 을 사용하지 않고서 25자 내외로 가능하지 않다.
2. 클래스 이름이 간결한 이름이 떠오르지않는다.
3. 두 개 이상의 너무 많은 책임을 지고있다.
4. 클래스를 변경해야 할 이유가 하나가 아니다.

=> 클래스를 분리해야 한다는 신호
```

**Good: 단일 책임 클래스**

(SuperDashboard에서 버전정보를 다루는 메서드 3개를 빼내 Version이라는 독자적 클래스를 생성)

```java
public class Version {
	public int getMajorVersionNumber()
	public int getMinorVersionNumber()
	public int getBuildNumber()
}

=> 이와 같이 하나의 기능(책임)을 하는 하나의 클래스가 되도록 각각 분리시켜야한다
```

<br>

### 응집도

클래스는 인스턴스 변수 수가 작아야 하고, <u>클래스의 각 메서드는 클래스 인스턴스 변수를 하나 이상 사용</u>해야한다.

(이처럼 모든 인스턴스 변수를 메서드마다 사용하는 클래스 => 응집도가 높음)

응집도가 높다는 말은 클래스에 속한 **메서드와 변수가 서로 의존하며 논리적인 단위로 묶여있다**는 걸 의미한다.

<br>

'함수를 작게, 매개변수 목록을 짧게' 전략을 따르다보면 메서드가 쪼개짐 

=> 몇몇 메서드만 사용하는 인스턴스 변수가 많아짐 

=> 새로운 클래스로 쪼개야한다.

**=> 큰 함수를 작은 함수로 쪼개다보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생긴다.** 

**(이렇게 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.)**

<br>

### 변경하기 쉬운 클래스

대다수 시스템은 지속적인 변경이 필요 => 변경할 때마다 의도하지 않은 위험이 따름 

=> 클래스가 체계적으로 정리된 깨끗한 시스템은 변경에 동반되는 위험을 낮춘다.

**Bad: 여러 책임에 따라 변경이 필요해 손대야 하는 클래스**

(Ex. update문을 지원해야함 => 코드 추가,  select문을 수정해야함 => 코드 수정 etc...)

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) 
	  private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

**Good: 닫힌 클래스들의 집합**

Sql클래스에서 파생하는 클래스들을 각각 생성

valueList와 같은 비공개 함수는 해당하는 클래스로 이동

모든 파생 클래스가 공통으로 사용하는 비공개 함수는 Where와 ColumnList 두 유틸리티 클래스로 분리

```java
abstract public class Sql {
	public Sql(String table, Column[] columns) 
	abstract public String generate();
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

public class SelectWithCriteriaSql extends Sql { 
	public SelectWithCriteriaSql(
	String table, Column[] columns, Criteria criteria) 
	@Override public String generate()
}

public class SelectWithMatchSql extends Sql { 
	public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
	@Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
	String table, Column[] columns, String keyColumn, String keyValue) 
	@Override public String generate()
}

public class PreparedInsertSql extends Sql {
	public PreparedInsertSql(String table, Column[] columns) 
	@Override public String generate() {
	private String placeholderList(Column[] columns)
}

public class Where {
	public Where(String criteria) public String generate()
	public String generate() {
}

public class ColumnList {
	public ColumnList(Column[] columns) public String generate()
	public String generate() {
}
```

**결과**

1. 테스트 관점에서 모든 논리를 하나하나 증명하기 쉬워짐

2. update문을 추가하려면 UpdateSql이라는 클래스만 새로 만들어주면됨 (기존 코드 수정 불필요)

   (단일 책임 원칙의 SRP를 만족)

3. 객체 지향 설계의 또 다른 핵심 원칙인 OCP : Open-Closed Principle 도 만족

   (OCP: 확장에 개방적이고 수정에 폐쇄적이어야한다는 원칙

   => 새 기능에 따른 UpdateSql 클래스를 생성하는 방식으로 새 기능에 개방적인 동시에,

   ​    다른 클래스들은 닫아놓는 방식으로 수정에는 폐쇄적임)

>**새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다.**
**이상적인 시스템이라면 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지는 않는다.**

<br>

### 변경으로부터 격리

```
Concrete Class (구체적 클래스) : 상세한 구현(코드) 포함
Abstract Class (추상 클래스) : 개념만 포함
```

상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠짐 

=> 인터페이스와 추상클래스를 이용해 구현이 미치는 영향을 격리

<br>

**예시**: Portfolio클래스는 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산함.

=> 우리 테스트 코드는 시세 변화에 영향을 받음

=> 5분마다 값이 달라지는 API로 테스트 코드를 짜기란 쉽지않음

**Solution**

```java
// StockExchange 인터페이스는 주식 기호를 받아 현재 주식 가격을 반환한다는 "추상적인 개념"을 표현
public insterface StockExchange {
	Money currentPrice(String symbol);
}

public Portfolio {
	private StockExchange exchange;
	public Portfolio(StockExchange exchange) {
		this.exchange = exchange;
	}
	// ...
}

// 추상화로 인해 실제로 주가를 얻어오는 출처나 얻어오는 방식 등과 같은 "구체적인 사실"을 모두 숨긴다.
public class PortfolioTest {
	private FixedStockExchangeStub exchange;
	private Portfolio portfolio;

	@Before
	protected void setUp() throws Exception {
		exchange = new FixedStockExchangeStub();
		exchange.fix("MSFT", 100);	// (테스트) 마이크로소프트 주식 1주를 구입하면 언제나 100불을 반환
		portfolio = new Portfolio(exchange);
	}

	@Test
	public void GivenFiveMSFTTotalShouldBe500() throws Exception {
		portfolio.add(5, "MSFT");	// 마이크로소프트 주식 5주를 사면
		Assert.assertEquals(500, portfolio.value());	// 500불과 일치하는지 확인하는 테스트 코드 작성 가능
	}
}

// => 
```

이와 같이 시스템의 결합도를 낮추면 => 유연성과 재사용성도 더욱 높아짐, 각 요소를 이해하기도 더 쉬워짐

=> 클래스 설계 원칙 DIP: Dependency Inversion Principle 을 만족하게 됨

**=> 본질적으로 클래스는 상세한 구현이 아니라 추상화에 의존해야한다는 의미**



<br>

<br>

용어 정리

```
[ SRP ]
SRP(Single Responsibility Principle)는 작성된 클래스는 하나의 기능만을 가지며 클래스가 제공하는 모든 서비스는 그 하나의 책임을 수행하는데 집중되어 있어야 한다는 것이다. SRP를 적용하면 각 클래스가 책임지는 영역이 확실해지므로, 한 클래스에서 변경사항이 요구될 때, 다른 클래스에 주는 연쇄작용을 없앨 수 있다. 

[ OCP ]
OCP(Open Closed Principle)는 소프트웨어의 구성요소(클래스, 함수, 컴포넌트 등)은 확장에는 열려있고, 변경에는 닫혀있어야 한다는 것이다. 즉, 주어진 클래스에 대한 요구사항의 변경이나 추가가 필요한 상황에서 기존 구성요소는 수정이 일어나지 말아야 하며, 확장이 용이해야 한다는 의미이다.

[ DIP ]
DIP(Dependency Inversion Principle)는 구조적 디자인에서 발생하던 하위 레벨 모듈의 변경이 상위 레벨 모듈의 변경을 요구하는 것을 끊는다는 의미입니다. 실제 사용관계는 바뀌지 않으며 추상을 매개로 메세지를 주고받음으로써 관계를 최대한 느슨하게 만듭니다.

출처: https://mangkyu.tistory.com/45
```


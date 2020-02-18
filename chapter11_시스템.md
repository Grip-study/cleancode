# 시스템

적절한 추상화와 모듈화 <br>
=> 깨끗한 코드를 작성하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다. 


## 1. 시스템 제작과 시스템 사용을 분리하라
소프트웨어 시스템은 
- (애플리케이션 객체를 제작하고 의존성을 서로 연결하는) 준비 과정
- (준비 과정 이후에 이어지는) 런타임 로직 

을 분리해야 한다. 

```
public Service getService() {
    if (service == null) {
        service = my ServiceImpl(...);
    }
    return service;
}
```
초기화 지연 (lazy initialization) , 계산 지연 (lazy evaluation)
- 실제로 필요할 때까지 객체를 생성하지 않기 떄문에 불필요한 부하가 걸리지 않는다. (애플리케이션 시작 시간이 빨라짐)
- 어떤 경우에도 null pointer를 반환하지 않는다. 

## => (하지만!!) 문제점 
- MyServiceImpl이 모든 상황에 적합한 객체일까? 
- getService 메서드를 포함한 클래스가 전체 문맥을 (이 시점에 MyServiceImpl이 필요하다는) 알 필요가 있는가?


=> 설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다. <br>
=> 또한 주요 의존성을 해소하기 위한 방식, 즉 전반적이며 일관적인 방식도 필요하다. 
<br><br><br>


## 1.1. Main 분리 : 애플리케이션이 객체 생성에 전혀 관여하지 않을 경우
- 시스템 생성 <br>
    => main 또는 main이 호출하는 모듈
- 시스템 사용 <br>
    => 나머지 시스템. 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정

=> main에서 필요한 객체를 생성하고, 애플리케이션은 그 객체를 사용 <br>
=> 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 몰라도, 모든 객체가 적절히 생성되었다고 가정하고 실행한다. 
<br><br>

## 1.2. 팩토리 : 애플리케이션이 객체가 생성되는 시점을 결정 및 통제
- Factory Interface 와 FactoryImplementation 구현체
- FactoryImpl을 애플리케이션에 넘겨서, 애플리케이션이 객체 생성을 위해 사용

=> 애플리케이션은 객체가 생성되는 시점을 완벽하게 통제하지만, 생성하는 코드에 대해서는 알 수 없다. 
<br><br>

## 1.3. 의존성 주입 
제어 역전(Inversion of Control) 기법을 의존성 관리에 적용한 메커니즘
- 제어 역전 : 한 객체가 맡은 보조 책임을 새로운 객체(제 3자)에게 전적으로 떠넘긴다. <br>
    => 새로운 객체는 넘겨받은 책임만 맡기 때문에 단일 책임 원칙을 지킴
- 의존성 주입을 '부분적으로' 구현한 코드
    ```
    MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”)); 
    ```
    위 코드를 호출하는 쪽에서는 실제로 jndiContext의 lookup 메소드가 무엇을 리턴하는지에 대해 관여하지 않으면서 의존성을 해결한다. 

진정한 의존성 주입 ?
- 클래스가 의존성을 해결하려 시도하지 않는다. 클래스는 완전히 수동적이며, setter 메서드 또는 생성자 인수를 통해 의존성을 주입한다. 
- DI 컨테이너는 요청이 들어올 때마다 필요한 객체이 인스턴스를 만든 후 생성자 인수나 setter를 사용해 의존성을 설정한다. 
<br><br><br><br>



## 2. 확장
관심사를 적절히 분리해 관리한다면 소프트에어 아키텍처는 점진적으로 발전할 수 있다. 
```
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    // Business logic...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    
    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    // EJB container logic
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }
    
    // The rest had to be implemented but were usually empty:
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```
문제점 >>
- 비즈니스 로직이 EJB2 컨테이너에 타이트하게 연결되어 있다. Entity를 만들기 위해 컨테이너 타입을 subclass하고 필요한 lifecycle 메서드를 구현해야 한다.
- 실제로 사용되지 않을 테스트 객체의 작성을 위해 mock 객체를 만드는 데에도 무의미한 노력이 많이 든다. EJB2 구조가 아닌 다른 구조에서 재사용할 수 없는 컴포넌트를 작성해야 한다.
- OOP 또한 등한시되고 있다. 상속도 불가능하며 쓸데없는 DTO(Data Transfer Object)를 작성하게 만든다.

## 2.1. 횡단(cross cutting) 관심사
- 이론적으로는 독립된 형태로 구분될 수 있지만 실제로는 코드에 산재하기 쉬운 부분들

일부 영역에서 관심사를 거의 완벽하게 분리한다. <br>
예를들어 트랜잭션, 보안, 일부 영속적인 동작은 소스코드가 아니라 배치 기술자에서 정의한다. <br>
(영속성과 같은 관심사는 애플리케이션의 자연스러운 객체 경계를 넘나든다. 모든 객체가 전반적으로 동일한 방식을 이용하게 만들어야 한다.) 

AOP 의 관점(aspect) 
- 특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다.

<br><br>

## 2.2. 관점과 관련된 자바 메커니즘
### 2.2.1. 자바 프록시
개별 객체나 클래스에서 메서드 호출을 감싸는 경우에 적합하다.
하지만 JDK에서 제공하는 동적 프록시는 인터페이스만 지원한다. 
```
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] {Bank.class},
    new BankProxyHandler(new BankImpl())
);
```
- 프록시로 감쌀 인터페이스 Bank
- 비즈니스 논리를 구현하는 POJO BankImpl 
- InvocationHandler 를 넘겨 프록시에 호출되는 Bank 메서드 구현

=> 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다. 또한, 프록시는 시스템단위로 실행 지점을 명시하는 매커니즘도 제공하지 못한다.

### 2.2.2. 순수 자바 AOP 프레임워크 (AspectJ를 사용하지 않음)
- 스프링 AOP, JBoss AOP 등
- 스프링은 비즈니스 논리를 POJO로 구현한다. POJO는 순수하게 도메인에 초점을 맞춘다.
- POJO는 엔터프라이즈 프렝미워크에 (다른 도메인에도) 의존하지 않는다. 따라서 테스트가 개념적으로 더 쉽고 간단하다.
- 상대적으로 단순하기 때문에 사용자 스토리를 올바로 구현하기 쉬우며 미래 스토리에 맞춰 코드를 보수하고 개선하기 편하다.
- 프로그래머는 설정 파일이나 API를 사용해 필수적인 애플리케이션 기반 구조를 구현한다. 
- 여기에는 영속성, 트랜잭션, 보안, 캐시, 장애조치 등과 같은 횡단 관심사도 포함된다.
- 이런 선언들이 요청에 따라 주요 객체를 생성하고 서로 연결하는 등 DI 컨테이너의 구체적인 동적을 제어한다.
```
<beans>
    ...
    <bean id="appDataSource" class="org.apache.commons.dbcp.BasicDataSource"/>

    <bean id="bankDataAccessObject" class="com.example.banking.persistence.BankDataAccessObject" dataSource-ref="appDataSource"/>

    <bean id="bank" class="com.example.banking.model.Bank" dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```
- Bank 도메인 객체는 자료 접근자 객체(DAO.DataAccessObject)인 bankDataAccessObject로 프록시되었으며,
- BankDataAccessObject는 JDBC 드라이브 자료 소스 appDataSource로 프록시되었다.
- 클라이언트는 Bank 객체에 getAccounts()를 호출한다고 믿지만, 실제로는 Bank POJO의 기본 동작을 확장한 중첩 DCORATOR 객체 집합의 갖아 외곽과 통신한다.
- 필요하다면 트랜잭션, 캐싱 등에도 DECORATOR를 추가할 수 있다. 

### 2.2.3. AspectJ 관점
- 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장
```
@Aspect 
public class SampleAspect { 

    @Before("execution(* com.devkuma.spring.aop.SampleAopBean.*(..))") 
    public void before() { 
        System.out.println("before:"); 
    } 

    @After("execution(* com.devkuma.spring.aop.SampleAopBean.*(..))") 
    public void after() { 
        System.out.println("after:"); 
    }

}

```


### 테스트 주도 시스템 아키텍처 구축
관점으로 (혹은 유사한 개념으로) 관심사를 분리하는 방식은 그 위력이 막강하다.
애플리케이션 도메인 논리를 POJO로 작성할 수 있다면, 즉 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다. 
- 소프트웨어는 나름대로 형체가 있지만, 소프트웨어 구조가 관점을 효과적으로 분리한다면 극적인 변화가 경제적으로 가능하다.
- 아주 단순하면서도 멋지게 분리된 아키텍처로 소프트웨어 프로젝트를 진행해 결과물을 빠르게 출시한 후, 기반 구조를 추가하며 조금씩 확장해 나갈 수 있다. 
- 고도의 자료 캐싱, 보안, 가상화 등을 이용해 아주 높은 가용성과 성능을 효율적이고도 유연하게 달성했다.
- 설계가 최대한 분리되어 각 추상화 수준과 범위에서 코드가 적당히 단순하기 때문이다.
- 최선의 시스템 구조는 각기 POJO 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

### 의사 결정을 최적화하라
- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕분에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기 쉬워진다. 또한 결정의 복잡성도 줄어든다. 

### 명백한 가치가 있을 때 표준을 현명하게 사용하라
- 표준을 사용하면 아이디어와 컴포턴트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다.
- 하지만 떄로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.
- 표준의 안좋은 예. EJB2

### 시스템은 도메인 특화 언어가 필요하다
- DSL. Domain Specific Language
- 간단한 스크립트 언어나 표준 언어로 구현한 API
- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 의사소통 간극을 줄여준다. 
- SQL, CSS, Regex, HQL, ...

<br><br>

## 결론
- 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨린다.
- 도메인 논리가 흐려지면 버그가 숨어들기 쉬워지고, 스토리를 구현하기 어려워진다.
- 기민성이 떨어지면 생산성이 낮아져 TDD가 제공하는 장점이 사라진다. 
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다. 


=> 그러려면 POJO를 작성하고 관점 혹은 관점과 유사항 메커니즘을 사용해 각 구현 관심사를 분리해야 한다. <br>
시스템을 설계하든 개별 모듈을 설계하든 실제로 돌아가는 가장 단순한 수단을 사용해야 한다. 
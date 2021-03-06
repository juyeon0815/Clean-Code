## Chapter 11. 시스템

- 높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법



### 도시를 세우려면?

- 혼자서 관리는 무리다.

  - 그럼에도 불구하고 잘 돌아가는 이유

    - 각 분야를 관리하는 팀이 있기 때문에
    - 적절한 추상화와 모듈화 때문에

    -> 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 **구성요소**는 효율적으로 돌아간다.

- 깨끗한 코드를 구현하면 낮은 추상화 수준에서 **관심사를 분리**하기 쉬워진다.



### 시스템 제작과 시스템 사용을 분리하라

> 소프트웨어 시스템은 애플리케이션 객체를 제작하고 의존성을 서로 연결하는 **준비 과정**과 준비과정 이후에 이어지는 **런타임 로직**을 분리해야 한다.



```java
public Service getService() {
    if (service == null)
        service = new MyServiceImpl(...);
    return service;
}
```

[장점]

- 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다.
- 어떤 경우에도 null 포인터를 반환하지 않는다.

[단점]

- getService() 메서드가 MyServiceImpl과 생성자 인수에 명시적으로 의존한다.
- MyServiceImpl이 무거운 객체라면 단위 테스트에서 getService 메서드를 호출하기 전에 적절한 테스트 전용 객체를 service 필드에 할당해야 한다.
- 일반 런타임 로직에다 객체 생성 로직을 섞어놓은 탓에 모든 실행 경로도 테스트해야한다.
  - 즉, 단일 책임 원칙 위반

[문제점]

- MyServiceImpl이 모든 상황에 적합한 객체인지 모른다.
- 초기화 지연 기법을 수시로 사용하면 모듈성은 저조하며 중복이 심각하다.



체계적이고 탄탄한 시스템을 만들고 싶다면 모듈성을 깨서는 절대로 안된다.

설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.



##### Main 분리

- 시스템 생성과 시스템 사용을 분리하는 방법

- 생성과 관련된 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정

- 애플리케이션은 그저 객체를 사용할 뿐, main이나 객체가 생성되는 과정을 전혀 모른다는 뜻이다.



##### 팩토리

- 객체가 생성되는 시점을 애플리케이션이 결정할 필요가 생길 때 **추상 팩토리 패턴**을 사용한다.
  - 참고)  https://kingchan223.tistory.com/284?category=894627

- 애플리케이션은 인스턴스가 생성되는 구체적인 방법을 모른다.
- 그럼에도 애플리케이션은 인스턴스가 생성되는 시점을 완벽하게 통제하며, 필요하다면 애플리케이션에서만 사용하는 생성자 인수도 넘길 수 있다.



##### 의존성 주입

> **사용**과 **제작**을 분리하는 강력한 메커니즘
> **제어 역전 기법**을 의존성 관리에 적용한 메커니즘

- 의존성 주입

  - ex ) 군인이 총을 가지고 있는 것을 나타내보자

  ```java
  //Gun.java
  public class Gun {
      ...
  }
  
  //Soldier.java
  public class Soldier() {
      private Gun gun;
      
      public Soldier() {
          gun = new Gun(); //Soldier생성자에게 Gun 클래스와의 의존관계를 애플리케이션 단에서 직접 설정
      }
  }
  ```

  

  [의존성 주입을 적용한 코드]

  ```java
  //Gun.java
  @Component //Bean으로 등록
  public class Gun {
      ...
  }
  
  //Soldier.java
  public class Soldier {
      @Autowired //Gun 타입의 Bean을 주입
      private Gun gun;
  }
  ```

  

- 제어의 역전
  - 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다.
  - 새로운 객체는 넘겨받은 책임만 맡으므로 **단일 책임 원칙**을 지키게 된다.

- **진정한 의존성 주입**은 클래스가 의존성을 해결하려 시도하지 않는다. 대신 의존성을 주입하는 방법으로 설정자 메서드나 생성자 인수를 제공한다.



##### 확장

> 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

- 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다.
- 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다.

=> 반복적이고 점진적인 애자일 방식의 핵심이다.

- TDD, 리팩터링, 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.



```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```



```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    //비즈니스 논리
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
    public abstract  getAccounts();
    public abstract void setAccounts(Collection accounts);
    public abstract void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    //EJB 컨테이너 논리
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) {....}
    public void ejbPostCreate(Integer id) {....}
    //
}
```

- 비즈니스 논리는 EJB2 애플리케이션 컨테이너에 강하게 결합된다.
- 비즈니스 논리가 큰 컨테이너와 밀접하게 결합된 탓에 독자적인 단위 테스트가 어렵다.



##### 횡단(cross-cutting) 관심사

- EJB2 아키텍처는 일부 영역에서 관심사를 거의 완벽하게 분리한다.
  - 원하는 트랜잭션, 보안, 영속적인 동작은 소스 코드가 아니라 배치 기술자에서 정의

- 모듈화되고 캡슐화된 방식으로 영속성 방식을 구상할 수 있다.





### 자바 프록시

- 자바 프록시는 단순환 상황에 적합하다.
  - 개별 객체나 클래스에서 메서드 호출을 감싸는 경우

```JAVA
// Bank.java
import java.util.*;

// 은행 추상화
public interface Bank {
    Collection<Account> getAccount();
    void setAccounts(Collection<Account> accounts);
}

//BankImpl.java
import java.util.*;

//추상화를 위한 POJO구현
public class BankImpl implements Bank {
    private List<Account> accounts;
    
    public Collction<Account> getAccounts(){
        return accounts;
    }
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for(Account account : accounts) {
            this.accounts.add(account);
        }
    }
}

//BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

//프록시 API가 필요한 "InvocationHandler"
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;
    
    public BankProxyHandler(Bank bank) {
        this.bank = bank;
    }
    
    //InvocationHandler에 저으이된 메서드
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if(methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            return bank.getAccounts();
        }
        else if(methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(back.getAccounts());
            return null;
        }else {
            ...
        }
    }
    
    //세부사항
}

//다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
	Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl()));
)
```

- 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다
- 프록시는 시스템 단위로 실행 지점을 명시하는 메커니즘을 제공하지 않는다.



### 순수 자바 AOP 프레임워크

- POJO는 엔터프라이즈 프레임워크에 의존하지 않기때문에 테스트가 개념적으로 더 쉽고 간단하다.
- 사용자 스토리를 올바로 구현하기 쉬우며 미래 스토리에 맞춰 코드를 보수하고 개선하기 편하다.
- EJB2에 비해 EJB3 코드가 훨씬 더 깨끗하며, 코드를 테스트하고 개선하고 보수하기 쉬워졌다.



### 의사 결정을 최적화하라

- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 가능한 마지막 순간까지 결정을 미루는 방법이 최선이다.
  - 최대한 정보를 모아 최선의 결정을 내리기 위해서
  - 성급한 결정은 고객 피드백을 더 모으고, 프로젝트를 더 고민하고, 구현 방안을 더 탐험할 기회가 사라진다.

> POJO 시스템 덕분에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기 쉬워진다.
>
> 또한 결정의 복잡성도 줄어든다.





### 명백한 가치가 있을 때 표준을 현명하게 사용하라

- 표준을 사용하면 
  - 아이디어와 컴포넌트를 재사용하기 쉽다.
  - 적절한 경험을 가진 사람을 구하기 쉽다.
  - 좋은 아이디어를 캡슐화하기 쉽다.
  - 컴포넌트를 엮기 쉽다.
- 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못하며, 원래 표준을 제정한 목적을 잊어버리기도 한다.



### 시스템은 도메인 특화 언어가 필요하다

- DSL (Domain-Specific Language)

  - 간단한 스크립트 언어나 표준 언어로 구현한 API
  - DSL로 짠 코드는 도메인 전문가가 작성한 구조적인 산문처럼 읽힌다.

  - 좋은 DSL은 의사소통 간극을 줄여준다.

  - 도메인 전문가가 사용하는 언어로 도메인 논리를 구현하면 도메인을 잘못 구현할 가능성이 줄어든다.

  - 효과적으로 사용한다면 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다.





### 결론

- 시스템은 역시 깨끗해야 한다.
- 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨린다.
  - 제품 품질 저하 + TDD가 제공하는 장점 사라짐
- 모든 추상화 단계에서 명확한 의도를 표현하기 위해 POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 구현 관심사를 분리해야한다.
- 시스템을 설계하든 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다는 사실을 명심하자.
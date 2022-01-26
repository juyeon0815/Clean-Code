## Chapter 3. 함수

- 의도를 분명히 표현하는 함수를 어떻게 구현할 수 있을까?
- 함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?



### 작게 만들어라!

- 함수를 만드는 규칙은 **작게! 더 작게!** 이다.



##### 블록과 들여쓰기

- if 문 / else 문 / while 문 등에 들어가는 블록은 **한 줄**이어야 한다.

- 바깥을 감싸는 함수가 작아질 뿐 아니라, 블록 안에서 호출하는 함수 이름을 적절히 짓는다면 코드를 이해하기도 쉬워진다.

  => 중첩 구조가 생길만큼 함수가 커져서는 안된다.



### 한 가지만 해라!

> **함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**

- 위에서 말하는 한 가지란?
  - 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한가지 작업만 한다.
  - 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.



### 함수 당 추상화 수준은 하나로!

- 함수가 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.

- 추상화 수준이 높다? <-> 낮다?
  - 정보를 많이 숨겼다는 것 <-> 정보가 많이 드러났다는 것

- **한 함수 내에 추상화 수준이 동일하지 않으면 코드를 읽는 사람이 헷갈린다.**



##### 위에서 아래로 코드 읽기 : 내려가기 규칙

- 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
- 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 번에 한 단계씩 낮아진다. = 내려가기 규칙
- 일련의 TO(~ 하려면) 문단을 읽듯이 프로그램이 읽혀야 한다.



### Switch 문

- 본질적으로 switch 문은 N가지를 처리하기 때문에 '한 가지' 작업만 하는 switch 문을 만들기 어렵다.
- **다형성**을 이용하여 switch 문을 저차원 클래스에 숨겨 반복하지 않는 방법이 있다.

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED :
            return calculateCommissionedPay(e);
        case HOURLY :
            return calculateHourlyPay(e);
        case SALARIED :
            return calculateSalariedPay(e);
        default :
            throw new InvalidEmpoyeeType(e.type);
    }
}
```

- 문제점
  - 함수가 길다. 
    - 새 직원 유형을 추가하면 더 길어진다.
  - '한 가지' 작업만 수행하지 않는다.
  - SRP(Single Responsibility Principle) 위반한다. 
    - SRP :  모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 한다.
    - 코드를 변경할 이유가 여럿이기 때문에
  - OCP(Open Closed Principle) 위반한다. 
    - OCP : 소프트웨어 개체는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.
    - 새 직원 유형을 추가할 때마다 코드를 변경하기 때문에
  - 위 함수와 구조가 동일한 함수가 무한정 존재



- 해결방법
  - switch 문을 추상 팩토리에 꽁꽁 숨긴다.
  - 팩토리는 switch 문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성하고, Employee 인터페이스를 거쳐 함수가 호출되며, 다형성으로 인해 실제 파생 클래스의 함수가 실행된다.

```java
public abstract class Employee {
    public abstract boolean isPayDay();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}

public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type){
            case COMMISSIONED :
                return new ComissionedEmployee(r);
            case HOURLY :
                return new HourlyEmployee(r);
            case SALARIED :
                return new SalariedEmployee(r);
            default :
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```



### 서술적인 이름을 사용하라!

- 함수가 하는 일을 좀 더 잘 표현한다면 훨씬 더 좋은 이름이다.
- 길고 서술적인 이름이 짧고 어려운 이름보다 좋다.
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.



### 함수 인수

- 함수에서 이상적인 인수 개수는 0개
- 1개(단항), 2개(이항) / 3개(삼항)은 가능한 피하는게 좋고 / 4개(다항)은 특별한 이유가 필요하나 이유가 있어도 사용하면 안된다.

```java
includeSetupPageInto(new PageContent);

includeSetupPageInto(); //위에보다 훨씬 더 이해하기 쉬움
```

- 단항 함수

  - 인수에 질문을 던지는 경우
    - ex) boolean fileExists("MyFile")

  - 인수를 뭘가로 변환해 결과를 반환하는 경우
    - ex) InputStream fileOpen("MyFile")

  - 에빈트 함수

    - ex ) void passwordAttemptFailedNtimes(int attempts);

      -> 입력 인수만 있고 출력 인수는 없다.
      -> 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다.
      -> 이벤트라는 사실이 코드에 명확히 드러나야 한다.

- 이항 함수
  - 이항 함수를 사용했을 때 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항 함수로 바꾸도록 노력해야한다.
- 삼항 함수
  - 이항 함수보다 훨씬 더 이해하기 어렵다.
  - 순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어난다.
  - *assertEquals(1.0, amount, 001) 은 왜 괜찮은건가..?*

- 동사와 키워드
  - 함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름은 필수
  - 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야함
    - ex ) write(name) -> writeField(name)
  - 함수 이름에 키워드 추가
    - ex) assertEquals -> assertExpectedEqualsActual(expected, actual)



### 부수 효과를 일으키지 마라!

```java
public class UserValidator {
  private Cryptographer cryptographer;
  
  public boolean checkPassword(String userName, String password) {
    User user = UserGateway.findByName(userName);
    if (user != User.NULL) {
      String codedPhrase = user.getPhraseEncodedByPassword();
      String phrase = cryptographer.decrypt(codedPhrase, password);
      if ("Valid Password".equals(phrase)) {
        Session.initialize();
        return true;
      }
    }
    return false;
  }
}
```

- checkPassword 함수이름만 봐서는 세션을 초기화한다는 사실이 들어나지 않아 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험이 있다.
- 그러므로 checkPassword -> checkPasswordAndInitializeSession 이 훨씬 좋다.



명령과 조회를 분리하라!

```java
public boolean set(String attribute, String value) {}
```

```java
if (set("username", "unclebob")) {}
```

- set이라는 단어가 동사인지 형용사인지 분간하기 어렵다.
- 개발자는 set을 동사로 의도했지만 if문에 넣고 보면 형용사로(~하다면) 느껴진다.



```java
if (attributeExists("username")) {
  setAttribute("username", "unclebob");
}
```

- 명령과 조회를 분리해 혼란을 없앤다면 위와 같은 문제를 해결할 수 있다.



### 오류 코드보다 예외를 사용하라!

- Try/Catch 블록 뽑아내기

  ```java
  public void delete(Page page) {
    try {
      deletePageAndAllReferences(page);
    } catch (Exception e) {
      logError(e);
    }
  }
  
  private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
  }
  
  private void logError(Exception e) {
    logger.log(e.getMessage());
  }
  ```

  - try/catch 블록은 코드 구조에 혼란을 일으키며, 정상 동작과 오류처리 동작을 뒤섞는다.
    - try/catch 블록을 별로 함수로 뽑아내는 편이 좋다.

  - 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.



### 구조적 프로그래밍

- 함수는 return 문이 하나여야 한다.
- 루프 안에서 break 나 continue를 사용해선 안 되며, goto는 절대로 안된다.



### 함수를 어떻게 짜죠?

- 처음에는 길고 복잡할 수 있으나 지속적으로 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거하고, 메서드를 줄이고 순서를 바꿔가는 노력을 통해 좋은 함수를 짤 수 있다.



### 결론

- 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다.

- 작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬워진다.

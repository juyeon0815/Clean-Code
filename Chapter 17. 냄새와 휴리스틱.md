## Chapter 17. 냄새와 휴리스틱

### 주석

#### [C1 : 부적절한 정보]

- 다른 시스템에 저장할 정보는 주석으로 적절하지 못하다.
- 작성자, 최종 수정일, SPR(Software Problem Report) 번호 등과 같은 메타 정보만 주석으로 넣는다.
- 주석은 코드와 설계에 기술적인 설명을 부연하는 수단



#### [C2 : 쓸모 없는 주석]

- 오래된 주석, 엉뚱한 주석, 잘못된 주석
- 쓸모 없어질 주석은 아예 달지 않는 편이 가장 좋다.
- 쓸모 없어진 주석은 재빨리 삭제하는 편이 가장 좋다.
- 코드와 무관하게 혼자서 따로 놀며 코드를 그릇된 방향으로 이끈다.



#### [C3 : 중복된 주석]

- 코드만으로 충분한데 구구절절 설명하는 주석
  - ex) i++ // i 증가
- 주석은 코드만으로 다하지 못하는 설명을 부언한다.



#### [C4 : 성의 없는 주석]

- 주석을 달 참이라면 시간을 들여 최대한 멋지게 작성한다.
- 간결하고 명료하게 작성한다.



#### [C5 : 주석 처리된 코드]

- 주석으로 처리된 코드는 얼마나 오래된 코드인지, 이 코드가 중요한 코드인지 아닌지 알 길이 없다.
- 그럼에도 불구하고 누군가에게 필요하거나 다른 사람이 사용할 코드라 생각하여 삭제하지도 않는다.
- 해당 코드는 읽는 사람을 헷갈리게 만들고 코드를 오염시키므로 **주석으로 처리된 코드는 즉각 지워버려라.**



### 환경

#### [E1 : 여러 단계로 빌드해야 한다]

- 빌드는 간단히 한 단계로 끝나야 한다.
- 소스 코드 관리 시스템에서 이것저것 따로 체크아웃할 필요가 없어야 한다.
- 불가해한 명령이나 스크립트를 잇달아 실행해 요소를 따로 빌드할 필요가 없어야 한다.
- 시스템에 필요한 파일을 찾느라 여기저기 뒤적일 필요가 없어야 한다.
- 한 명령으로 전체를 체크아웃해서 한 명령으로 빌드할 수 있어야 한다.



#### [E2 : 여러 단계로 테스트해야 한다]

- 모든 단위 테스트는 한 명령으로 돌려야 한다.
- 모든 테스트를 한 번에 실행하는 능력을 아주 중요하며 빠르고, 쉽고, 명백해야 한다.



### 함수

#### [F1 : 너무 많은 함수]

- 함수의 인수 개수는 작을수록 좋다.
  - 없는게 가장 좋다.
  - 4개 이상은 특별한 이유가 있어도 사용하면 안 된다. (P50쪽 참조)



#### [F2 : 출력 인수]

- 일반적으로 독자는 인수를 입력으로 간주한다.
  - ex ) appendFooter(s) : 인수 s는 입력인수인지 출력인수인지 판단하기 어렵다.
    - public void appendFooter(StringBuffer report) <- 함수 선언부를 확인하면 출력인수라는 것을 알수있다.
- 출력 인수는 가급적 피하는게 좋으며, 함수에서 상태를 변경해야 한다면 함수가 속한 객체의 상태를 변경하는 방식을 택한다.



#### [F3 : 플래그 인수]

- 플래그 인수는 혼란을 초래하므로 피해야 마땅하다.



#### [F4 : 죽은 함수]

- 호출하지 않는 함수는 삭제한다.
- 죽은 코드는 낭비이므로 과감히 삭제한다.



### 일반

#### [G1 :  한 소스 파일에 여러 언어를 사용한다]

- 소스 파일 하나에 언어 하나만 사용하는 방식이 가장 좋다.
- 소스 파일에서 언어 수와 범위를 최대한 줄이도록 노력해야 한다.



#### [G2 : 당연한 동작을 구현하지 않는다]

- 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다.
- 만약 당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 함수 이름만으로 함수 기능을 예상하기 어렵다. = 코드를 일일이 살펴야 한다.



#### [G3 : 경계를 올바로 처리하지 않는다]

- 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 작성하라.



#### [G4 : 안전 절차 무시]

- 컴파일러 경고 일부를 꺼버리면 빌드가 쉬워질지 모르지만 끝없는 디버깅에 시달린다.
- 실패하는 테스트 케이스를 나중으로 미루는 태도는 신용카드가 공짜 돈이라는 생각만큼 위험하다.



#### [G5 : 중복]

- 코드에서 중복을 발견할 때마다 추상화할 기회로 간주해, 중복된 코드를 하위 루틴이나 다른 클래스로 분리하라.
  - 구현이 빨라지고 오류가 적어진다.
- 똑같은 코드가 여러 차례 나오는 중복은 **간단한 함수로 교체**한다.
- 조건을 거듭 확인하는 중복은 **다형성으로 대체**한다.
- 알고리즘이 유사하나 코드가 서로 다른 중복은 **TEMPLATE METHOD 패턴**이나 **STRATEGY 패턴**으로 중복을 제거한다.

- 어디서든 중복을 발견하면 없애라.



#### [G6 : 추상화 수준이 올바르지 못하다]

- 모든 저차원 개념은 파생 클래스에 넣고, 고차원 개념은 기초 클래스에 넣는다.
  - 세부 구현과 관련한 상수, 변수는 파생 클래스에, 기초 클래스는 구현 정보를 몰라야한다.

- 잘못된 추상화 수준은 거짓말이나 꼼수로 해결하지 못한다.
- 잘못된 추상화를 임시변통으로 고치기는 불가능하다.



#### [G7 : 기초 클래스가 파생 클래스에 의존한다]

- 기초 클래스는 파생 클래스를 아예 몰라야 마땅하다.
- 기초 클래스와 파생 클래스를 다른 JAR 파일로 배포한다면 독릭접인 개별 컴포넌트 단위로 시스템을 배치할 수 있다.
  - 컴포넌트를 변경할 경우에 해당 컴포넌트만 다시 배치하면 된다. 즉, 시스템을 유지보수하기가 한결 수월해진다.



#### [G8 : 과도한 정보]

- 클래스가 제공하는 메서드 수가 작을수록, 함수가 아는 변수 수가 작을수록, 클래스에 들어있는 인스턴스 변수 수가 작을수록 좋다.

- 인터페이스를 매우 작게, 정보를 제한하여 결합도를 낮춰라



#### [G9 : 죽은 코드]

- 실행되지 않는 코드
  - 불가능한 조건을 확인하는 if문, 아무도 호출하지 않는 함수 등
- 시스템에서 제거해라.



#### [G10 : 수직 분리]

- 변수와 함수는 사용되는 위치에 가깝게 정의한다.

- 비공개 함수는 쉽게 눈에 띄도록 처음으로 호출한 직후에 정의한다.



#### [G11 : 일관성 부족]

- 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다.
- 선택한 표기법은 신중하게 따르며, 메서드 이름에따라 다른 메서드도 유사한 이름을 사용한다,
- 일관성있는 코드는 읽고 수정하기 쉬워진다.



#### [G12 : 잡동사니]

- 비어있는 기본 생성자, 아무도 사용하지 않는 변수/함수, 정보를 제공하지 않는 주석 등은 쓸데 없이 코드만 복잡하게 만든다.
- 그러므로 제거하자.



#### [G13 : 인위적 결합]

- 직접적인 상호작용이 없는 두 모듈 사이에서 일어난다.
- 함수, 상수, 변수를 선언할 때는 올바른 위치를 고민하여 알맞은 위치에 선언한다.



#### [G14 : 기능 욕심]

- 클래스 메서드는 자기 클래스와 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져서는 안된다.
- 기능 욕심은 한 클래스의 속사정을 다른 클래스에 노출하므로, 제거하는 편이 좋다.



#### [G15 : 선택자 인수]

- 선택자 인수는 목적을 기억하기 어려울 뿐 아니라 각 선택자 인수가 여러 함수를 하나로 조합한다.

```JAVA
public int calculateWeeklyPay(boolean overtime) {
    int tenthRate = getTenthRate();
    int tenthsWorked = getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime);
    int straightPay = straightTime * tenthRate;
    double overtimePay = (int)Math.round(overTime * overtimeRate);
    return straightPay + overtimePay;
}
```

- calculateWeeklyPay(false)라는 코드를 발견할 때마다 어떤 의미인지 생각해야 한다.



```java
public int straightPay() {
    return getTenthsWorked() * getTenthRate();
}

public int overTimePay() {
    int overTimeTenths = Math.max(0, getTenthsWorked() - 400);
    int overTimePay = overTimeBonus(overTimeTenths);
    return straightPay() + overTimePay;
}

private int overTimeBonus(int overTimeTenths) {
    double bonus = 0.5 * getTenthRate() * overTimeTenths;
    return (int) Math.round(bonus);
}
```

- 인수를 넘겨 동작을 선택하는 대신 하나의 함수를 작은 함수 여럿으로 쪼개는게 좋다.



#### [G17 : 잘못 지운 책임]

- 코드를 배치하는 위치는 중요하다.
- 개발자에게 편한 함수에 배치할 경우, 함수 이름을 제대로 지어야 한다.



#### [G18 : 부적절한 static 함수]

- 좋은 예로 Math.max(double a, double b)가 있다.
  - Math.max 메서드는 재정의할 가능성이 전혀 없다.
- 반드시 static 함수로 정의해야한다면 재정의할 가능성은 없는지 생각해봐야 한다.



#### [G20 : 이름과 기능이 일치하는 함수]

```JAVA
Date newDate = date.add(5);
```

- 해당 코드만 봐서는 5가 시간인지, 일수인지 구분하기 어려움
- 구분할 수 있는 명확한 이름으로 바꾸자



#### [G21 : 알고리즘을 이해하라]

- 구현이 끝났다고 선언하기 전에 함수가 돌아가는 방식을 확실히 이해하는지 확인하라.
- 기능이 뻔히 보일 정도로 함수를 깔끔하고 명확하게 재구성해라.



#### [G22 : 논리적 의존성을 물리적으로 드러내라]

- 한 모듈이 다른 모듈에 의존한다면 물리적인 의존성도 있어야 한다.

```JAVA
public class HourlyReporter {
    private HourlyReportFormatter formatter; //넘어온 정보 출력
    private List<LineItem> page;
    private final int PAGE_SIZE = 55;
    
    public HourlyReporter(HourlyReportFormatter formatter) {
        this.formatter = formatter;
        page = new ArrayList<LineItem>();
    }
    
    public void generateReport(List<HourlyEmployee> employees) {
        for (HourlyEmployee e : employees) {
            addLineItemToPage(e);
            if(page.size() == PAGE_SIZE)
                printAndClearItemList();
        }
        if(page.size()>0)
            printAndClearItemList();
    } 
    
    private void printAndClearItemList() {
        formatter.format(page);
        page.clear();
    }
    
    private void addLineItemToPage(HourlyEmployee e) {
        LineItem item = new LineItem();
        item.name = e.getName();
        item.hours = egetTenthsWorked() / 10;
        item.tenths = e.getTenthsWorked() % 10;
        page.add(item);
    }
    
    public class LineItem {
        public String name;
        public int hours;
        public int tenths;
    }
}
```

- PAGE_SIZE를 HourlyReporter 클래스가 알 필요가 없다.
  - 잘못 지원 책임에 해당

- HourlyReporter 클래스는 HourlyReportFormatter가 페이지 크기를 알 거라 가정
  - 논리적 의존성

- HourlyReportFormatter에 getMaxPageSize() 라는 메서드를 추가
  - 논리적인 의존성이 물리적인 의존성으로 바뀐다.
- HourlyReport 클래스는 PAGE_SIZE 상수 대신 getMaxPageSize() 메서드 호출



#### [G23 : If/Else 혹은 Switch/Case 문보다 다형성을 사용하라]

- switch문을 사용하는 이유는 손쉬운 선택이기 때문이다. 그러므로 switch를 선택하기 전에 다형성을 먼저 고려해보자.
- 선택 유형 하나에는 switch 문을 한번만 사용한다. 같은 선택을 수행하는 다른 코드에서는 다형성 객체를 생성해 switch 문을 대신한다.



#### [G25 : 매직 숫자는 명명된 상수로 교체하라]

- 일반적으로 코드에서 숫자를 사용하지 말라는 규칙

```JAVA
double milesWalked = feetWalked/5280.0;
int dailyPay = houlryRate*8;
double circumference = radius*Math.PI*2;
```

- FEET_PER_MILE은 5280이 잘 알려진 고유한 숫자라 독자가 금방 알아볼 수 있다.
- PI인 3.1415926535... 같은 숫자 역시 독자가 금방 알아볼 수 있으나 오류가 발생할 가능성이 너무 크므로 Math.PI 사용



#### [G26 : 정확하라]

- 코드에서 뭔가를 결정할 때에는 정확히 결정한다.
  - 호출하는 함수가 NULL을 반환할지도 모른다면 NULL을 반드시 점검
  - 동시에 갱신할 가능성이 있다면 적절한 잠금 매커니즘 구현



#### [G28 : 조건을 캡슐화하라]

- 조건의 의도를 분명히 밝히는 함수로 표현하라

```JAVA
if (sholudBeDeleted(timer))
    
if (timer.hasExpired() && !timer.isRecurrent())
```



#### [G29 : 부정 조건은 피하라]

- 부정 조건은 긍정 조건보다 이해하기 어렵다.
- 가능한 긍정 조건으로 표현한다.



#### [G30 : 함수는 한가지만 해야 한다]

- 한 가지만 수행하는 좀 더 작은 함수 여럿으로 나눠야 한다.

```JAVA
public void pay() {
    for (Employee e : employees) {
        if(e.isPayday()) {
            Money pay = e.calculatePay();
            e.deliveryPay(pay);
        }
    }
}
```

- 위의 코드는 세가지의 일을 수행한다.
  - 직원 목록 반복문 돌기
  - 직원의 월급일 확인하기
  - 해당 직원에게 월급 지급하기

```java
public void pay() {
    for(Employee e : employees) {
        payIfNecessary(e);
    }
}

private void payIfNecessary(Employee e) {
    if(e.isPayday()) {
        calculateAndDeliveryPay(e);
    }
}

private void calculateAndDeliveryPay(Employee e) {
    Money pay = e.calculatePay();
    e.deliveryPay(pay);
}
```

- 좀 더 작은 함수로 나눠 각 함수에서 한 가지 임수만 수행하도록 변경



#### [G31 : 숨겨진 시간적인 결합]

- 함수를 짤 떄에는 함수 인수를 적절히 배치해 함수가 호출되는 순서를 명백히 드러낸다.

```JAVA
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;
    
    public void dive(String reason) {
        saturateGradient();
        reticulateSplines();
        diveForMoog(reason);
    }
}
```

- 호출순서가 변경될 경우에 UnsaturatedGradientException 오류가 발생할 수 있다.



```java
public class MoogDiver {
    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines = reticulateSplines(gradient);
        diveForMoog(splines,reason);
    }
}
```

- 연결 소자를 생성해 시간적인 결합 노출

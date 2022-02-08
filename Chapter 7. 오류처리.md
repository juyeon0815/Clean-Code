## Chapter 7. 오류처리

- 오류 처리는 프로그램에 반드시 필요한 요소 중 하나
- 깨끗한 코드와 오류 처리는 연관성이 있다
- 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다



### 오류 코드보다 예외를 사용하라

```java
public class DeviceController {
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        if(handle != DeviceHandle.INVALID) {
            retrieveDeviceRecord(handle);
            
            if(record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
            logger.log("Device suspended. Unable to shut down");
        	}
        } else {
            logger.log("Invalid handle for: "+ DEV1.toString());
        }
    }
}
```

- 함수를 호출한 즉시 오류를 확인해야하기 때문에 호출자 코드가 복잡해진다
- 차라리 예외를 던지는 편이 낮다



```java
public class DeviceController {
    public void sendShutDown() {
       try {
           tryToShutDown();
       } catch(DeviceShutDownError e) {
           logger.log(e);
       }
    }
    
    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }
    
    private DeviceHandle getHandle(DeviceID id) {
        throws new DeviceShutDownError("Invalid handle for : "+ id.toString());
    }
}
```

- 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리함으로써 코드 품질 향상



### Try-Catch-Finally 문부터 작성하라

- try 블록 안 코드는 어느 시점에서든 실행이 중단된 후 catch 블록으로 넘어갈 수 있다
- try 블록에서 무든 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다



[잘못된 파일 접근 시도]

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
    } catch(Exception e){
        throw new StorageException("retrieval error" , e);
    }
    return new ArrayList<RecordGrip>();
}

* 에외 유형을 좁힘으로써 실제로 FileInputStream 생성자가 던지는 예외를 잡아낸다.
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        
        //나머지 논리 추가
        
        stream.close();
    } catch(FileNotFoundException e){
        throw new StorageException("retrieval error" , e);
    }
    return new ArrayList<RecordGrip>();
}
```

- 예외를 일으키는 테스트 케이스를 작성한 후 통과할 수 있는 코드를 작성한다면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다



### 미확인 예외를 사용하라

- C# , C++은 확인된 예외를 지원하지 않는다

  - 확인된 예외란?

    - 잘못된 코드가 아닌 잘못된 상황에서 발생하는 예외

    - ### 파일 열기와 같이 정확한 코드로 구현했음에도, 외부 환경(파일이 없는 상황 등)에 따라 발생 가능

    - 예외처리를 구현하지 않으면 컴파일 에러 발생 (컴파일 시 확인해서 확인된 예외)

    - RuntimeException 이외의 예외들 ex) FileNotFoundException, SQLException

  - 미확인 예외란?

    - 프로그램 로직의 오류로 인한 예외
    - RuntimeException을 상속하는 예외

- 확인된 예외는 OCP(Open Closed Principle) 위반
  - 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다
  - 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어나므로 최하위 함수에서 던지는 예외를 알아야 한다 = 캡슐화 깨짐

- 아주 중요한 라이브러리를 작성한다면 확인된 예외는 예상되는 모든 예외를 사전에 처리할 수 있다



### 예외에 의미를 제공하라

- 예외를 던질 때는 오류가 발생한 원인과 위치를 찾기 쉽게 전후 상황을 충분히 덧붙인다
- 오류 메세지에 정보를 담아 예외와 함께 던진다



### 호출자를 고려해 예외 클래스를 정의하라

```java
ACMEPort port = new ACMEPort(12);

try{
    port.open();
} catch(DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch(ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch(GMXError e) {
    reportPortError(e);
    logger.log("Device response excepiton");
} finally {
    
}
```

- 심한 코드 중복 발생



[개선한 코드]

```java
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch(PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    
}

public class LocalPort {
    private ACMEPort innerPort;
    
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
        try {
            innerPort.open();
        } catch(DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch(ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch(GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
}
```

- 호출하는 라이브러리 API 감싸면서 예외 유형 하나 반환
  - 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다
  - 나중에 다른 라이브러리로 갈아타도 비용이 적다
  - 감싸기 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기 쉬워진다
  - 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다



### 정상 흐름을 정의하라

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getToal();
} catch(MealExpenseNoFound e) {
    m_total += getMealPerDiem();
}

//위와 같은 예외는 논리를 따라가기 어렵게 만든다
//특수한 상황을 처리할 필요가 없게 만든다면 코드가 더 간결해진다
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        //기본 값으로 일일 기본 식비 반환
    }
}
```



### null을 반환하지 마라

```java
public void registerItem(Item item) {
    if(item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if(registry != null) {
            ....
        }
    }
}
```

- 만약 null이라면 NullPointException이 발생 할 것이다
- 사용하려는 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 **예외를 던지거나 특수 사례 객체를 반환**하는 방법 고려



```java
List<Employee> employees = getEmployees(); 
//null이 아닌 빈 리스트를 반환하는 것도 하나의 방법!
for(Employee e : employees) {
    totaolPay += e.getPay();
}

//미리 정의된 읽기 전용 리스트 반환
public List<Employee> getEmployees() {
    if (직원이 없다면)
        return Collections.emptyList();
}
```



### Null을 전달하지 마라

- 메서드로 null을 전달하는 코드는 최대한 피한다

```java
public class MetricsCalculator{
    public double xProjection(Point p1, Point p2) {
        return (p2.x - p1.x) * 1.5;
    }
}

//인수로 null을 전달한다면? : NullPorinterExcepiotn 발생
calculator.xPorjection(null, new Point(12,13));

//해결 방법

// 1. 새로운 예외 유형 만들어 던지기 : InvalidArgumentException을 잡아내는 처리가 필요
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if(p1 == null || p2 == null) {
            throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x - p1.x) * 1.5;
    }
}

// 2.assert문 사용
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null : "p1 should not be null";
        assert p2 != null : "p2 should not be null";
        return (p2.x - p1.x) * 1.5;
    }
}

```



### 결론

- 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다
- 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 높아진다
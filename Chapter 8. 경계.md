## Chapter 8. 경계

### 외부 코드 사용하기

- Map은 다양한 인터페이스로 수많은 기능을 제공

  - Map이 제공하는 기능성과 유연성은 유용하다는 장점 존재
  - Map은 객체 유형을 제한하지 않기 때문에 어떤 객체 유형도 추가할 수 있다

  ```java
  Map sensors = new HasMap();
  Sensors s (Sensors)sensors.get(sensorId);
  ```

  - Sensor라는 객체를 담는 Map을 생성하여 해당 Sensor 객체를 가져온다

    - 해당 코드의 의도를 알아차리기 쉽지 않다

    

  ```java
  Map<String, Sensor> sensors = new HashMap<Sensor>();
  ...
  Sensor s = sensors.get(sensorId);
  ```

  - 제네릭스를 사용하면 코드 가독성을 높일 수 있다
  - 하지만 사용자가 필요하지 않는 기능까지 제공한다는 문제점 존재
    - Map 인터페이스가 변할 경우, 수정할 코드가 많아진다

  

  [개선된 코드]

  ```java
  public class Sensors {
      private Map sensors = new HashMap();
      
      public Sensor getById(String id) {
          return (Sensor) sensors.get(id);
      }
     ...
  }
  ```

  - Map을 Sensors 안에 넣음으로써 Map 인터페이스가 변하더라도 프로그램에는 영향을 미치지 않는다
  - Sensors 클래스는 프로그램에 필요한 인터페이스만 제공하기 때문에 코드는 이해하기 쉽고 오용은 어려워진다



### log4j 익히기

```java
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.removeAllAppenders();
    logger.addAppender(new ConsoleAppender(
    	new PatternLayout("%p %t %m%n"),
        ConsoleAppender.SYSTEM_OUT));
    logger.info("hello");
}
```



### 학습 테스트는 공짜 이상이다

- 학습 테스트는 이해도를 높여주는 정확한 실험이다
- 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다
- 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다



### 아직 존재하지 않는 코드를 사용하기



### 깨끗한 경계

- 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다

- 엄청난 시간과 노력과 재작업을 요구하지 않는다

- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자

- 새로운 클래스로 경계를 감싸거나 아니면 ADAPTER 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자

  => 코드 가동석이 높아지며, 경계 인터페이스를 사용하는 일관성도 높아지며, 외부 패키지가 변경되었을 경우에도 수정해야할 코드도 줄어든다
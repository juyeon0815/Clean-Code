## Chapter 4. 주석

- 잘 달린 주석은 그 어떤 정보보다 유용하다.
- 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게 만든다.
- 오래되고 조잡한 주석은 거짓과 잘못된 정보를 퍼뜨려 해약을 미친다.
- 프로그래밍 언어 자체가 표현력이 풍부하다다면, 우리에게 프로그래밍 언어를 치밀하게 사용해 의도를 표현할 능력이 있다면 주석은 필요하지 않는다.



### 주석은 나쁜 코드를 보완하지 못한다

- 코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.
- 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.



### 좋은 주석

- 정말로 좋은 주석은, 주석을 달지 않을 방법을 찾아낸 주석

- 법적인 주석

  ```
  // Copyright (C) 2003,2004,2005 by Object Mentor, Inc. All rights reserved.
  // GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.
  ```

- 정보를 제공하는 주석

  ```java
  // 테스트 중인 Responder 인스턴스를 반환한다.
  protected abstract Responder responderInstance();
  ```

  - 함수 이름에 정보를 담는 편이 더 좋다.

  - 함수 이름을 responderBeingTested로 바꾸면 주석이 필요 없어진다.

  ```java
  // kk:mm:ss EEE, MMM dd, yyyy 형식이다.
  Pattern timeMatcher = Pattern.compile(
  	"\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*"
  );
  ```

  - 날짜를 변환하는 클래스를 만들어 코드를 옮겨주면 더 깔끔하고 주석이 필요 없어진다.

- 의미를 명료하게 밝히는 주석

  - 때때로 모호한 인수나 반환값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다.

  - 일반적으로 인수나 반환값 자체를 명확하게 만들면 더 좋겠지만, 인수나 반환값이 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.

- 결과를 경고하는 주석

  - 다른 프로그래머에게 결과를 경고할 목적으로 주석 사용

    ```java
    // 여유 시간이 충분하지 않다면 실행하지 마십시오.
    public void _testWithReallyBigFile() {
      writeLinesToFile(10000000);
      
      response.setBody(testFile);
      response.readyToSend(this);
      assertSubString("content-Length: 1000000000", responseString);
      assertTure(bytesSent > 1000000000);
    }
    ```

    - JUnit4가 나오기 전에는 메서드 이름 앞에 _ 기호를 붙이는 방법이 일반적인 관례
    - 요즘에는 @Ignore 속성을 이용해 테스트 케이스를 꺼버린다.

    ```java
    public static SimpleDateFormat makeStandardHttpDateFormat() {
      // SimpleDateFormat은 스레드에 안전하지 못하다.
      //따라서 각 인스턴스를 독립적으로 생성해야 한다.
      SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
      df.setTimeZone(TimeZone.getTimeZone("GMT"));
      return df;
    }
    ```



- TODO 주석
  - '앞으로 할 일'을 //TODO 주석으로 남겨두면 편하다.
  - TODO 주석은 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술



### 나쁜 주석

- 주절거리는 주석

  - 의무감 혹은 마지못해 주석을 단다면 시간낭비이다.
  - 충분한 시간을 들여 최고의 주석을 달도록 노력해야한다.

  ```java
  public void loadProperties() {
    try {
      String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
      FileInputStream propertiesStream = new FileInputStream(propertiesPath);
      loadedProperties.load(propertiesStream);
    } 
    catch (IOException e) {
      // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다.
    }
  }
  ```

  - 이해가 안 되어 다른 모듈까지 뒤져야 하는 주석은 독자와 제대로 소통하지 못하는 주석이다.
  - 그런 주석은 바이트만 낭비할 뿐이다.

- 같은 이야기를 중복하는 주석

  ```java
  // this.closed가 true일 때 반환되는 유틸리티 메서드다.
  // 타임아웃에 도달하면 예외를 던진다.
  public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if(!closed) {
      wait(timeoutMillis);
      if(!closed) {
        throw new Exception("MockResponsesender could not be closed");
      }
    }
  }
  ```

  - 주석이 코드보다 더 많은 정보를 제공하지 못한다.

- 의무적으로 다는 주석

  ```
  /**
   *
   * @param title CD 제목
   * @param author CD 저자
   * @param tracks CD 트랙 숫자
   * @param durationInMinutes CD 길이 (단위 : 분)
   */
  ```

  - 코드만 헷갈리게 만들며, 거짓말할 가능성을 높이고, 잘못된 정보를 제공한 여지만 만든다.

- 닫는 괄호에 다는 주석

  - 닫는 괄호에 주석을 달아야겠다는 생각에 든다면 대신에 함수를 줄이려 시도하자.

- 주석으로 처리한 코드

  - 주석으로 처리된 코드는 "이유가 있어 남겨 놓은거겠지?", "중요하니까 지우면 안되겠지?" 라는 생각에 지우기를 주저한다.

    => 질 나쁜 와인병 바닥에 앙금이 쌓이듯 쓸모 없는 코드가 점차 쌓여간다.

  - **주석으로 처리한 코드는 그냥 삭제하라**

- 전역 정보
  - 주석을 달아야 한다면 큰처에 있는 코드만 기술하라.
  - 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.
## Chapter 12. 창발성

### 창발적 설계로 깔끔한 코드를 구현하자

- 모든 테스트를 실행한다.
- 중복을 없앤다.
- 프로그래머 의도를 표현한다.
- 클래스와 메서드 수를 최소로 줄인다.

-> 위의 네 가지 설계 규칙은 소프트웨어 설계 품질을 크게 높여준다.



### 단순한 설계 규칙 1:모든 테스트를 실행하라

- 설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다.

- 테스트가 불가능한 시스템은 검증도 불가능하다.

- 검증이 불가능한 시스템은 절대 출시하면 안 된다.

- 테스트가 가능한 시스템을 만들려고 노력하면 설계 품질이 높아지며, 크기가 작고 목적을 하나만 수행하는 클래스가 나온다.

- 테스트 케이스를 많이 작성할수록 개발자는 DIP와 같은 원칙을 적용하고 의존성 주입, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춘다.

  => 설계 품질을 더욱 높아진다.



### 단순한 설계 규칙 2 ~ 4:리팩터링

- 리팩터링 단계에서는 소프트웨어 설계 품질을 높이는 기법이라면 적용해도 괜찮다.
- 단순한 설계 규칙 중 나머지 3개를 적용해 중복을 제거하고, 프로그래머 의도를 표현하고, 클래스와 메서드수를 최소로 줄이는 단계이기도 하다.



### 중복을 없애라

- 중복은 추가 작업, 추가 위험, 불필요한 복잡도를 뜻하기 때문에 설계에서 커다란 적이다.



```JAVA
int size() {}
boolean isEmpty() {}


boolean isEmpty() {
    return 0 == size();
}
```

- isEmpty 메서드에서 size매서드를 이용하면 코드를 중복해 구현할 필요가 없어진다.



```java
public void scaleToOneDimension(float desireDimension, float imageDimension) {
    if(Math.abs(desireDimension - imageDimension) < errorThreshold)
        return;
    float scalingFactor = desireDimension / imageDimension;
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    RenderdOp newImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
    image.dispose();
    System.gc();
    image = newImage;
}
public synchronized void rotate(int degress) {
    RenderdOp newImage = ImageUtilities.getRotatedImage(image, degrees);
    image.dispose();
    System.gc();
    image = newImage;
}
```

- 일부코드 동일



```java
public void scaleToOneDimension(float desireDimension, float imageDimension) {
    if(Math.abs(desireDimension - imageDimension) <errorThreshold)
        return;
    float scalingFactor = desireDimension / imageDimension;
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
  	replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}
public synchronized void rotate(int degress) {
    replaceImage(ImageUtilities.getRotatedImage(image, degress));
}

public void replaceImage(RenderedOp newImage) {
    image.dispose();
    System.gc();
    image = newImage;
}
```

- replaceImage 메서드를 통해 공통된 코드를 빼면 중복을 줄일 수 있다.
- 더 나아가 replaceImage 메서드를 다른 클래스로 옮기면 가시성이 높아진다.
- 소규모 재사용은 시스템 복잡도를 극적으로 줄여준다.



- **TEMPLATE METHOD 패턴**은 고차원 중복을 제거할 목적으로 자주 사용하는 기법이다.

```JAVA
public class VacationPolicy {
    public void accrueUSDivisionVacation(){
        //지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        ...
        //휴가 일수가 미국 최소 법정 일수을 만족하는지 확인하는  코드
        ...
        //휴가 일수를 급여 대장에 적용하는 코드
        ...
    }
    
    public void accrueDUDivisionVaction(){
        //지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        ...
        //휴가 일수가 유럽연합 최소 법정 일수을 만족하는지 확인하는  코드
        ...
        //휴가 일수를 급여 대장에 적용하는 코드
        ...
    }
}
```

- 위의 코드는 최소 법정 일수를 계산하는 코드만 제외하면 거의 동일하다.



```java
abstract public class VacationPolicy {
    public void accrueVaction() {
        calculateBaseVacationHours();
        alertForLegalMinimums();
        applyToPayroll();
    }
    
    private void calculateBaseVacationHours() { ... };
    abstract protected void alertForLegalMinimums();
    private void applyToPayroll() { .... }
}

public class USVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        //미국 최소 법정 일수를 사용한다.
    }
}

public class EUVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        //유럽연합 최소 법정 일수를 사용한다.
    }
}
```

- 최소 법정 일수를 계산하는 코드를 빼 중복 제거



### 표현하라

- 자신이 이해하는 코드는 짜기는 쉽지만, 코드를 유지보수할 사람이 코드를 짜는 사람만큼이나 문제를 깊이 이해할 가능성은 희박하다.
- 시스템이 점차 복잡해지면서 유지보수 개발자가 시스템을 이해하느라 보내는 시간은 점점 늘어나고 동시에 코드를 오해할 가능성도 커진다.
- 개발자가 코드를 명확하게 짤수록 다른 사람이 그 코드를 이해하기 쉬워지므로 개발자의 의도를 분명히 표현해야 한다.
  - 좋은 이름 선택하기
  - 함수와 클래스 크기 줄이기
  - 표준 명칭 사용하기
    - EX) COMMAND나 VISITOR과 같은 표준 패턴을 사용해 구현할 경우 클래스 이름에 패턴 이름 넣어주기
  - 단위 테스트 케이스 꼼꼼히 작성하기



### 클래스와 메서드 수를 최소로 줄여라

- 클래스와 메서드 크기를 줄이자고 조그만 클래스와 메서드를 수없이 만들면 득보다 실이 많아진다.
- 가능한 실용적인 방식을 택한다.
- 클래스와 함수 수를 줄이는 작업도 중요하지만, 테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요하다.
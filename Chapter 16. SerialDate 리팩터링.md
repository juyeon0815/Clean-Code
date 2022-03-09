## Chapter 16. SerialDate 리팩터링

- 자바에서 날짜를 표현하는 다양한 클래스를 제공하지만,  하루 중 시각/ 시간대에 무관하게 특정 날짜만 표현하기 위해 SerialDate 재정의



### 첫째, 돌려보자

- SerailDateTests라는 클래스는 단위 테스트 케이스 몇 개를 포함
  - 하지만 모든 경우를 점검하지 않음
  - ex) MonthCodeToQuarter(넘어온 달이 속한 분기 반환) 메서드 실행 X
- 현재 실행하는 단위 테스트는 50%정도, 코드를 리팩토링하면서 모든 테스트케이스를 통과하게 끔 손 볼 예정



```java
// stringToMonthCode(문자열을 달 코드로 변환)

if((result<1) || (result>12)) {
    result = -1;
    for(int i=0; i<monthNames.legnth;i++) {
        if(s.equalsIgnoreCase(shortMonthNames[i])) {
            result = i + 1;
            break;
        }
        if(s.equalsIgnoreCase(monthNames[i])) {
            result = i + 1;
            break;
        }
    }
}

```

- 대소문자 구분 없이 테스트케이스를 모두 통과해야하므로 equalsIgnoreCase로 변경



##### [P491- 318행]

```JAVA
assertEquals(d(1, JANUARY, 2005), getFollowingDayOfWeek(SATURDAY, d(25, DECEMBER, 2004)));
```

- getFollowingDayOfWeek 메서드에 있는 버그를 들어냄

  - 2004년 12월 25일 : 토요일, 2005년 1월 1일 : 다음 토요일

    하지만, 테스트를 돌렸을 경우 해당 메서드가 12월 25일을 다음 토요일로 반환

  - if(baseDOW > targetWeekday) -> if(baseDOW **>=** targetWeekday)로 변경



##### [P464 - 705행]

getNearestDayOfWeek(tragetDOW, base) : 기준에 가장 근접한 날짜 반환

- tragetDOW : 목표로 삼은 주 중 일자
- base : 기준 날짜

```java
int delta = targeDOW - base.getDayOfWeek();
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
if (adjust > 3)
    adjust -= 7;

return SerialDate.addDays(adjust, base);
```

- 수정 전 코드는 가장 가까운 날짜가 미래면 실패
  - 경계 조건 오류 (if문)



### 둘째, 고쳐보자

##### [P448 - 1행]

- 소스 코드 제어 도구를 사용하므로 변경 이력 삭제



##### [P449 - 61행]

```java
import java.text.DateFormatSymbols;
import java.text.SimpleDateFormat;
-> import java.text.*
    
import java.util.Calendar;
import java.util.GregorianCalendar;
-> import java.util.*
```

- import문 줄이기



##### [P450 - 86행]

- 클래스 선언 부분
- 클래스 이름이 SerialDate인 이유는 해당클래스가 인련번호를 사용해서 구현했기 때문에

```java
/**
 * Returns the serial number for the date, where 1 January 1900 = 2 (this
 * corresponds, almost, to the numbering system used in Microsoft Excel for
 * Windows and Lotus 1-2-3).
 *
 * @return the serial number for the date.
 */
```

- '일련번호' 라는 용어가 서술적이지 못한 이름이라고 생각
- SerialDate라는 이름의 추상화 수준이 올바르지 못하다고 생각
  - 저자는 Date/Day라는 이름이 좋으나, 자바 라이브러리에는 Date, Day 클래스가 있기 때문에 DayDate라는 이름을 사용하기로함



##### [DayDate 클래스가 MonthConstants를 상속하는 이유?]

- MonthConstants는 달을 정의하는 상수 모음에 불가
  - 상수 클래스를 상속하는 것은 옛날 자바 프로그래머들이 많이 쓰던 기교 = 바람직하지않다.
  - enum으로 정의해야 마땅

```java
public abstract class DayDate implements Comparable, Serializable {
    public static enum Month {
        JANUARY(1),
        FEBRUARY(2),
        MARCH(3),
        APRIL(4),
        MAY(5),
        JUNE(6),
        JULY(7),
        AUGUST(8),
        SEPTEMBER(9),
        OCTOBER(10),
        NOVEMBER(11),
        DECEMBER(12);

        Month(int index) {
            this.index = index;
        }

        public static Month make(int monthIndex) {
            for (Month m : Month.values()) {
                if (m.index == monthIndex)
                    return m;
            }
            throw new IllegalArgumentException("Invalid month index " + monthIndex);
        }
        public final int index;
    }
}
```

- enum으로 변경하게 되면 달을 int로 받던 메서드는 Month로 받을 수 있게 됨
- isValidMonthCode 메서드 불필요
- monthCodeToQuarter과 같은 오류 검사 코드 불필요



##### [P450 - 91행]

- serialVersionUID : 직렬화 제어
- 해당 변수를 선언하지 않으면 모듈을 변경할 때마다 변수 값이 자동으로 달라짐



##### [P450 - 97, 100행]

- DayDate 클래스가 표현할 수 있는 최초와 초후 날짜를 의미

```java
public static final int EARLIEST_DATE_ORDINAL = 2;      // 1/1/1900
public static final int LATEST_DATE_ORDINAL = 2958465;  // 12/31/9999
```

- 좀 더 깔끔하게 표현하기 위해 코드 변경
- EARLIEST_DATE_ORDINAL가 2인 이유는 엑셀에서 날짜를 표시하는 방법과 관련 (P492 - SpreadsheetDate 클래스 참고)
- 해당 변수는 SpreadsheetDate의 구현과 관련
  - SpreadsheetDate 클래스로 옮겨져야 한다.



##### [P450 - 104, 107행]

- MINIMUM_YEAR_SUPPORTED와 MAXIMUM_YEAR_SUPPORTED 두 변수를 SpreadsheetDate 클래스로 옮기려고 했으나, RleativeDayOfWeekRule 클래스에서도 두 변수를 getDate로 넘어온 인수 year가 올바른지 확인할 목적으로 사용함을 알 수 있었다.
- DayDate 자체를 훼손하지않으면서 구현 정보를 전달할 방법이 필요하다.
  - 일반적으로 기반 클래스는 파생 클래스를 몰라야 바람직하기 때문에 **ABSTRACT FACTORY 패턴**을 적용해 해결

```JAVA
public abstract class DayDateFactory {
    private static DayDateFactory factory = new SpreadsheetDateFactory();
    public static viud setInstance(DayDateFactory factory) {
        DayDateFactory.factory = factory;
    }

    protected abstract DayDate _makeDate(int ordinal);
    protected abstract DayDate _makeDate(int day, DayDate.Month month, int year);
    protected abstract DayDate _makeDate(int day, int month, int year);
    protected abstract DayDate _makeDate(java.util.Date date);
    protected abstract int _getMinimumYear();
    protected abstract int _getMaximumYear();

    public static DayDate makeDate(int ordinal) {
        return factory._makeDate(ordinal);
    }

    public static DayDate makeDate(int day, DayDate.Month month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(int day, int month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(java.util.Date date) {
        return factory._makeDate(date);
    }

    public static int getMinimumYear() {
        return factory._getMinimumYear();
    }

    public static int getMaximumYear() {
        return factory._getMaximumYear();
    }
}
```

- createInstance 메서드를 좀 더 서술적인 이름인 makeDate로 변경



```JAVA
public class SpreadsheetDateFactory extends DayDateFactory {
    public DayDate _makeDate(int ordinal) {
        return new SpreadsheetDate(ordinal);
    }

    public DayDate _makeDate(int day, DayDate.Month month, int year) {
        return new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(int day, int month, int year) {
        rturn new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(Date date) {
        final GregorianCalendar calendar = new GregorianCalendar();
        calendar.setTime(date);
        return new SpreadsheetDate(
            calendar.get(Calendar.DATE),
            DayDate.Month.make(calendar.get(Calendar.MONTH) + 1),
            calendar.get(Calendar.YEAR));
    }

    protected int _getMinimumYear() {
        return SpreadsheetDate.MINIMUM_YEAR_SUPPORTED;
    }

    protected int _getMaximumYear() {
        return SpreadsheetDate.MAXIMUM_YEAR_SUPPORTED;
    }
}
```

- MINIMUM_YEAR_SUPPORTED와 MAXIMUM_YEAR_SUPPORTED 변수를 적절한 위치인 SpreadsheetDate로 이동



##### [P451 - 140행]

- 변수 이름만으로 의미가 확실하므로 주석 삭제
- 적절한 접근 제어자 사용
- 사용하지 않는 변수 제거
- 변수가 한 곳에서만 사용될 경우,  변수가 사용되는 가까운 위치로 이동



##### [P451 - 162행 ~ P452 - 205행, P452 - 190~205행]

- 상수 enum으로 변환

```java
public enum WeekInMonth { //P451 - 162행 ~ P452 - 205행
    FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
    public final int index;

    WeekInMonth(int index) {
        this.index = index;
    }
}
```



##### [P454 - 259행]

- for 루프 안에 if문 두번을 || 연산사를 사용해 if문 하나로 변경
- stringToWeekdayCode 메서드 -> Day로 이동



##### [P454 - 272~286행]

```JAVA
public enum Day {
    MONDAY(Calendar.MONDAY),
    TUESDAY(Calendar.TUESDAY),
    WEDNESDAY(Calendar.WEDNESDAY),
    THURSDAY(Calendar.THURSDAY),
    FRIDAY(Calendar.FRIDAY),
    SATURDAY(Calendar.SATURDAY),
    SUNDAY(Calendar.SUNDAY);

    public final int index;
    private static DateFormatSymbols dateSymbols = new DateFormatSymbols();

    Day(int day) {
        index = day;
    }

    public static Day make(int index) throws IllegalArgumentException {
        for (Day d : Day.values())
            if (d.index == index)
                return d;
        throw new IllegalArgumentException(
            String.format("Illegal day index: %d.", index));
    }

    public static Day parse(String s) throws IllegalArgumentException {
        String[] shortWeekdayNames = 
            dateSymbols.getShortWeekdays();
        String[] weekDayNames =
            dateSymbols.getWeekdays();
        
        s = s.trim();
        for (Day day : Day.values()) {
            if (s.equalsIgnoreCase(shortWeekdayNames[day.index])) ||
                s.equalsIgnoreCase(weekDayNames[day.index])) {
                return day;
            }
        }
        throw new IllegalArgumentException(
            String.format("%s is not a valid weekday string", s));
    }

    public String toString() {
        return dateSymbols.getWeekdays()[index];
    }
}
```

- weekDayCodeToString(넘어온 주 중 일자를 표현하는 문자열 반환) 메서드를 Day로 옮기고 toSring이라 명명

## Chapter 14. 점진적인 개선

- 점진적인 개선을 보여주는 사례 연구
- 출발은 좋았으나 확장성이 부족했던 모듈 소개
- 모듈을 개선하고 정리하는 단계



```java
public static void main(String[] args) {
    try {
        Args arg = new Args("l,p#,d*", args);
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d');
        executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.printf("Argument error: %s\n", e.errorMessage());
    }
}
```

- 생성자에서 ArgsException이 발생하지 않는다면 명령행 인수의 구문 분석을 성공했으며 Args 인스턴스에 질의를 던져도 좋다.
- 형식 문자열이나 명령행 인수 자체에 문제가 있다면 ArgsException 발생



### Args 구현

```java
//14-2 Args.java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
import java.util.*;

public class Args {
    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;
    
    public Args(String schema, String[] args) throws ArgsException {
        marshalers = new HashMap<Character, ArgumentMarshaler>();
        argsFound = new HashSet<Character>();
        
        parseSchema(schema);
        parseArgumentStrings(Arrays.asList(args));
    }
    
    private void parseSchema(String schema) throws ArgsException {
        for(String element : schema.split(","))
            if(element.length() > 0)
                parseSchemaElement(element.trim());
    }
    
    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if(elementTail.length()==0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if(elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if(elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if(elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else if(elementTail.equals("[*]"))
            marshalers.put(elementId, new StringArrayArgumentMarshaler());
        else
            throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
        
    }
    
    private void validateSchemaElementId(char elementId) throws ArgsException {
        if(!Character.isLetter(elementId))
            throw new ArgsExcepion(INVALID_ARGUMENT_NAME, elementId, null);
    }
    
    private void parseArugmentStrings(List<String> argsList) throws ArgsException{
        for(currentArgument = argsList.listIterator(); currentArgument.hasNext();){
            String argString = currentArgument.next();
            if(argString.startWith("-")) {
                parseArgumentCharacters(argString.substring(1));
            }else {
                currentArgument.previous();
                break;
            }
        }
    }
    
    private void parseArgumentCharacters(String argChars) throws ArgsException {
        for(int i=0; i<argChars.length(); i++)
            parseArgumentCharacter(argChars.charAt(i));
    }
    
    private void parseArgumentCharacter(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null) {
            throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
        }else {
            argsFound.add(argChar);
            try {
                m.set(currentArgument);
            } catch (ArgsException e) {
                e.setErrorArgumentId(argChar);
                throw e;
            }
        }
    }
    
    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
    
    public int nextArgument() {
        return currentArgument.nextIndex();
    }
    
    public boolean getBoolean(char arg) {
        return BooleanArgumentAarshaler.getValue(marshalers.get(arg));
    }
    
    public String getString(char arg) {
        return StringArgumentAarshaler.getValue(marshalers.get(arg));
    }
    
    public int getInt(char arg) {
        return IntArgumentAarshaler.getValue(marshalers.get(arg));
    }
    
    public Double getDouble(char arg) {
        return DoubleArgumentAarshaler.getValue(marshalers.get(arg));
    }
    
    public String[] getStringArray(char arg) {
        return StringArrayArgumentAarshaler.getValue(marshalers.get(arg));
    }
}
```



```java
//14-3 ArgumentMarshaler.java
public interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
}
```



```java
//14-4 BooleanArgumentMarshaler.java
public class BooleanArgumentMarshaler implements ArgumentMarshaler {
    private boolean booleanValue = false;
    
    public void set(Iterator<String> currentArgument) throws ArgsException {
        booleanValue = true;
    }
    
    public static boolean getValue(ArgumentMarshaler am) {
        if(am!=null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler) am).booleanValue;
        else
            return false;
    }
}
```



```java
//14-5 StringArgumentMarshaler.java
import static com.objectmentor.utilities.args.ArgsExcepiton.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";
    
    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_STRING);
        }
    }
    
    public static String getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler) am).stringValue;
        else
            return "";
    }
}
```



```java
//14-6 IntegerArugmentMarshaler.java
import static com.objectmentor.utilities.args.ArgsExcepiton.ErrorCode.*;

public class IntegerArugmentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;
    
    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INBALID_INTEGER, parameter);
        }
    }
    
    public static int getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler) am).intValue;
        else
            return 0;
    }
}
```



```java
//14-7 ArgsException.java
import static com.objectmentor.utilities.args.ArgsExcepiton.ErrorCode.*;

public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;
    
    public ArgsException() {}
    
    public ArgsException(String message) {super(message);}
    
    public ArgsException(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }
    
    public ArgsException(ErrorCode errorCode, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }
    
    public ArgsException(ErrorCode errorCode, 
                         char errorArgumentId, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }
    
    public char getErrorArgumentId() {
        return errorArgumentId;
    }
    
    public void setErrorArgumentId(char errorArugmentId) {
        this.errorArgumentId = errorArgumentId;
    }
    
    public String getErrorParameter() {
        return errorParameter;
    }
    
    public void setErrorParameter(String errorParameter) {
        this.errorParameter = errorParameter;
    }
    
    public ErrorCode getErrorCode() {
        return errorCode;
    }
    
    public void setErrorCode(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }
    
    public String errorMessage() {
        ....
    }
    
    public enum ErrorCode {
        OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT,
        INVALID_ARGUMENT_NAME,
        MISSING_STRING,
        MISSING_INTEGER, INVALID_INTEGER,
        MISSING_DOUBLE, INVALID_DOUBLE
    }
}
```



**깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다**.

**일단 프로그램이 '돌아가면' 다른 업무로 넘어가는 행동은 자살 행위이다.**



### Args: 1차 초안

- 코드는 돌아가지만 엉망인 코드
- 많은 인스턴스 변수
- 지저분한 코드에 기여하는 'TILT'와 같은 희한한 문자열, HashSets, TreeSets, try-catch-catch 블록



[14-9]

- 코드가 간결하고 단순하며 이해하기 쉬운 코드



[14-10]

- 앞에 14-9 코드에 String과 Integer 인수 유형 두 개만 추가했을 뿐인데 코드가 엄청나게 지저분해졌다.
- 코드는 통제를 벗어나기 시작했다.



##### 그래서 멈췄다

- 새 인수 유형을 추가하려면 주요 지점 세 곳(parse, set, get)에다 코드를 추가해야 한다.



##### 점진적으로 개선하다

- 테스트 주도 개발(TDD) 기법 사용

  - 언제 어느 때라도 시스템이 돌아가야 한다는 원칙
  - 시스템을 망가뜨리는 변경을 허용X

- ArgumenetMarshaler 클래스의 골격 추가

  ```java
  private class ArgumentMarshaler {
      private boolean booleanValue = false;
      
      public void setBoolean(boolean value) {
          booleanValue = value;
      }
      
      public boolean getBoolean() {return booleanValue;}
      
  }
  
  private class BooleanArgumentMarshaler extends ArgumentMarshaler {}
  
  private class StringArgumentMarshaler extends ArgumentMarshaler {}
  
  private class IntegerArgumentMarshaler extends ArgumentMarshaler {}
  ```



- Boolean 인수 유형 -> ArgumentMarshaler 유형

  ```java
  //14-8(1차 초안)
  private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
  
  //변경
  private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
  
  private void parseBooleanSchemaElement(Char elementId) {
      booleanArgs.put(elementId, new BooleanArgumentMarshaler());
  }
  
  private void setBooleanArg(char argChar, boolean value) {
      booleanArgs.get(argChar).setBoolean(value);
  }
  
  public boolean getBoolean(char arg) {
      return falseIfNull(booleanArgs.get(arg).getBoolean());
  }
  ```

  - 변경한 코드에서 args로 'y'를 넘긴다면 NullPointerException 발생

  

  ```java
  public boolean getBoolean(char arg) {
      return booleanArgs.get(arg).getBoolean();
  }
  ```

  - falseIfNull제거



​		p272 왜 독자적인 변수에 저장해유?



### String 인수


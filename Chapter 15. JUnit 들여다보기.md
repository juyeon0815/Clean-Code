## Chapter 15. JUnit 들여다보기

### JUnit 프레임워크

```java
// 15-1 ComparisonCompactorTest.java
package junit.tests.framework;

import junit.framework.ComparisonCompactor;
import junit.framework.TestCase;

public class ComparisonCompactorTest extends TestCase {
    public void testMessage() {
        String failure = new ComparisonCompactor(0, "b", "c").compact("a");
        assertTrue("a expected:<[b]> but was:<[c]>".equals(failure));
    }
    
    public void testStartSame() {
        String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
        assertTrue("expected:<b[a]> but was:<b[c]>", failure);
    }
    
    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertTrue("expected:<[a]b> but was:<[c]b>", failure);
    }
    
    public void testSame() {
         String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertTrue("expected:<...[b]...> but was:<...[d]...>", failure);
    }
    ....
}
```



```java
// 15-2 ComparisonCompactor.java
package junit.framework;

public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    
    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;
    
    public ComparsionCompactor(int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }
    
    public String compact(String message) {
        if(fExpected == null || fActual == null || areStringEqual())
            return Assert.format(message, fExpected, fActual);
        
        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }
    
    private String compactString(String source) {
        String result = DELTA_START + source.subString(fPrefix, source.length() - 
                                                       fSuffix +1) + DELTA_END;
        if(fPrefix > 0 )
            result = computeCommonPrefix() + result;
        if(fSuffix > 0)
            result = result + computeCommonSuffix();
        return result;
    }
    
    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for (; fPrefix < end; fPrefix++) {
            if(fExpected.charAt(fPrefix) != fActual.charAt(fPrefix))
            break;
        }
    }
    
    private void findCommonSuffix() {
        int expectSuffix = fExpected.length()-1;
        int actualSuffix = fActual.length()-1;
        for (;
            actualSuffix>=fPrefix && expectedSuffix >=fPrefix;
            actualSuffix--, expectedSuffix--){
            if(fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix))
                break;
        }
        fSuffix = fExpected.length()-expectedSuffix;
    }
    
    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPIS : "") + fExpected.substring(Math.max(0,fPrefix - fContextLength), fPrefix);
    }
    
    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length()-fSuffix+1+fContextLength, fExpected.length());
        return fExpected.substring(fExpected.length()-fSuffix+1, end)+ (fExpected.length()-fSuffix+1<fExpected.length()-fContextLength ? ELLIPSIS : "");
    }
    
    private boolean areStringEqual() {
        return fExpected.equals(fActual);
    }
}
```

- 명확한 변수이름
  - 접두어 f를 제거함으로써 지역변수와 멤버변수의 이름이 동일해졌다.  명확한 변수 이름으로 변경



```java
// 15-3 ComparisonCompactor.java
package junit.framework;

public class ComparisonCompactor {
    private int ctxt;
    private String s1;
    private String s2;
    private int pfx;
    private int sfx;
    
    public ComparsionCompactor(int ctxt, String s1, String s2) {
        this.ctxt = ctxt;
        this.s1 = s1;
        this.s2 = s2;
    }
    
    public String compact(String msg) {
        if (s1 == null || s2 == null || s1.equals(s2))
            return Assert.format(msg, s1, s2);
        
        pfx = 0;
        for(; pfx<Math.min(s1.length(), s2.length()); pfx++){
            if(s1.charAt(pfx) != s2.charAt(pfx))
                break;
        }
        int sfx1 = s1.length()-1;
        int sfx2 = s2.length()-1;
        for(; sfx2 >= pfx && sfx1>=pfx; sfx2--, sfx1--) {
            if(s1.charAt(sfx1) != s2.charAt(sfx2))
                break;
        }
        sfx = s1.length()-sfx1;
        String cmp1 = compactString(s1);
        String cmp2 = compactString(s2);
        return Assert.format(msg, cmp1, cmp2);
    }
    
    private String compactString(String s) {
        String result = "[" + s.substring(pfx, s.length() - sfx + 1) + "]";
        if(pfx > 0)
            result = (pfx > ctxt ? "..." : "") + s1.substring(Math.max(0,pfx-ctxt), pfx)+result;
        if(sfx > 0)
            int end = Math.min(s1.length() - sfx + 1 + ctxt, s1.length());
            result = result + (s1.substring(s1.length() - sfx + 1, end) + (s1.length() - sfx + 1 < s1.length() 0 ctxt ? "..." : ""));
    }
    return result;
}
```

- 변수 이름에 범위를 명시할 필요가 없다.
  - 접두어 f는 중복되는 정보

- compact 함수 시작부에 캡슐화되지 않는 조건문
  - 적절한 이름을 붙여 메서드로 뽑아내기
    - 메서드 명은 부정문보다는 긍정문
    - 새 이름에 인수를 고려하면 가독성↑



```java
// 위의 내용을 반영한 코드

....
    private String compactExpected;
	private String compactActual;
....
    public String formatCompactedComparison(String message) {
    	if(canBeCompacted()) { // 캡슐화되지 않는 조건문 메서드로 뽑기
            compactExpectedAndActual();
            return Assert.format(message, compactExpected, compactActual);
        } else {
            return Assert.format(messgae, expected, actual);
        }
	}

	private void compactExpectAndActual() { //압축만 수행
        prefixIndex = findCommonPrefix();
        suffixIndex = findCommonSubfix(prefixIndex);
        compactExpected = compactString(expected);
        compactActual = compactString(actual);
    }

	private int findCommonPrefix() {
        int prefixIndex = 0;
        int end = Math.min(expected.length(), actual.length());
        for(; prefixIndex < end; prefixIndex++) {
            if(expected.charAt(prefixIndex) != actual.charAt(prefixIndex))
                break;
        }
        return prefixIndex;
    }

	private int findCommonSubfix(int prefixIndex) {
        int expectedSuffix = expected.length() - 1;
        int actualSuffix = actual.length() - 1;
        for(; actualSuffix >= prefixIndex && expectedSuffix >= prefixInde; actualSuffix--, expectedSuffix--) {
            if(expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
                break;
        }
        return expected.length() - expectedSuffix;
    }
```

- findCommonSubfix()는 findCommonPrefix()가 prefixIndex를 계산한다는 사실에 의존
  - 만약 순서가 뒤바뀐다면 다른 결과 도출
  - 시간 결합을 외부에 노출하고자 findCommonSubfix에 prefixIndex를 인수로 넘김



```java
// 위의 코드를 다르게 바꾼 코드

private void compactExpectAndActual() { //압축만 수행
        findCommonPrefixAndSubfix();
        compactExpected = compactString(expected);
        compactActual = compactString(actual);
    }

	private void findCommonPrefix() {
        int prefixIndex = 0;
        int end = Math.min(expected.length(), actual.length());
        for(; prefixIndex < end; prefixIndex++) {
            if(expected.charAt(prefixIndex) != actual.charAt(prefixIndex))
                break;
        }
    }

	private void findCommonPrefixAndSubfix() {
        findCommonPrefix(); // 여기서 호출
        int expectedSuffix = expected.length() - 1;
        int actualSuffix = actual.length() - 1;
        for(; actualSuffix >= prefixIndex && expectedSuffix >= prefixInde; actualSuffix--, expectedSuffix--) {
            if(expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
                break;
        }
        suffixIndex = expected.length() - expectedSuffix;
    }
```

- findCommonSuffix -> findCommonPrefixAndSubfix로 바꾸고 findCommonPrefixAndSubfix에서 findCommonPrefix를 호출함으로써 호출하는 순서가 더욱 분명해졌다.



```java
// 지저분한 findCommonPrefixAndSubfix 코드 리팩토링

	private void findCommonPrefixAndSubfix() {
        findCommonPrefix();
        int suffixLength = 1;
        for(; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if(charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
                break;
        }
        suffixIndex = suffixLength;
    }

	private char charFromEnd(String s, int i) {
        return s.charAt(s.length()-i);
    }

	private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() - suffixLength < prefixLength ||
            expected.legnth() - suffixLength() < prefixLength;
    }
```

- 코드 변경을 통해 suffixIndex가 접미어 길이라는 사실이 드러남
- suffixIndex는 1에서 시작 = **computeCommonSuffix + 1 이 등장하는 이유**



```java
// computeCommonSuffix + 1 -> suffixLength로 코드 변경

public class ComparisonCompactor {
   ...
   private int suffixLength;
   ...
   
   private void findCommonPrefixAndSubfix() {
        findCommonPrefix()
        suffixLength = 0;
       
         for(; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if(charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
                break;
        }
    }
    private char charFromEnd(String s, int i) {
        return s.charAt(s.length()-i-1);
    }

	private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() - suffixLength <= prefixLength ||
            expected.legnth() - suffixLength() <= prefixLength;
    }
   ...
    private String compactString(String source) {
        String result = DELTA_START + source.subString(fPrefix, source.length() - 
                                                       suffixLength) + DELTA_END;
        if(prefixLength > 0 )
            result = computeCommonPrefix() + result;
        if(suffixLength > 0)
            result = result + computeCommonSuffix();
        return result;
    }
    ...    
    private String computeCommonSuffix() {
        int end = Math.min(expected.length()-suffixLength+ContextLength, expected.length());
        return expected.substring(expected.length()-suffixLength, end)+ (expected.length()-suffixLength<expected.length()-ContextLength ? ELLIPSIS : "");
    }
}
```

- if(suffixLength > 0) : suffixLength 가 1씩 감소했으므로 해당 코드의 연산자는 >에서 >=로 바꿔야하지만 >=로 바꿨을 경우에 'suffixLength는 언제나 1이상이다' 라는 것과 모순
  - **이전 코드가 틀린 코드였다!**



```java
// compactString 리팩토링

private String compactString(String source) {
    return computeCommonPrefix() + DELTA_START + source.substring(prefixLength + source.legnth() - suffixLength) + DELTA_END + computeCommonSuffix();
}
```

- compactString()는 단순히 문자열 조각만 결합하는 메서드





### 15-5 최종 리팩토링 코드

- 나눴던 메서드를 fomatCompactedComparison()에 다시 합침
- shouldNotBeCompacted 조건도 돌려놓음
- 코드를 리팩터링 하다보면 원래 했던 변경을 되돌리는 경우가 흔함



### 결론

- 세상에 개선이 불필요한 모듈은 없다.
- 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.
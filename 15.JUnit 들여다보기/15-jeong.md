# 15. JUnit 들여다보기

생성일: 2022년 3월 6일 오후 5:20

- 이 장에서는 JUnit 프레임워크에서 가져온 코드를 평가한다.
- 살펴볼 ComparisonCompactor 모듈은 두 문자열을 받아 차이를 반환한다. 아래 테스트 코드를 읽어보면 모듈의 기능을 파악할 수 있다.

- ComparisonCompactorTest

```java
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
        assertEquals("expected:<b[a]> but was:<b[c]>", failure);
    }

    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertEquals("expected:<[a]b> but was:<[c]b>", failure);
    }

    public void testSame() {
        String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertEquals("expected:<ab> but was:<ab>", failure);
    }

    public void testNoContextStartAndEndSame() {
        String failure = new ComparisonCompactor(0, "abc", "adc").compact(null);
        assertEquals("expected:<...[b]...> but was:<...[d]...>", failure);
    }

    public void testStartAndEndContext() {
        String failure = new ComparisonCompactor(1, "abc", "adc").compact(null);
        assertEquals("expected:<a[b]c> but was:<a[d]c>", failure);
    }

    public void testStartAndEndContextWithEllipses() {
        String failure = new ComparisonCompactor(1, "abcde", "abfde").compact(null);
        assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
    }

    public void testComparisonErrorStartSameComplete() {
        String failure = new ComparisonCompactor(2, "ab", "abc").compact(null);
        assertEquals("expected:<ab[]> but was:<ab[c]>", failure);
    }

    public void testComparisonErrorEndSameComplete() {
        String failure = new ComparisonCompactor(0, "bc", "abc").compact(null);
        assertEquals("expected:<[]...> but was:<[a]...>", failure);
    }

    public void testComparisonErrorEndSameCompleteContext() {
        String failure = new ComparisonCompactor(2, "bc", "abc").compact(null);
        assertEquals("expected:<[]bc> but was:<[a]bc>", failure);
    }

    public void testComparisonErrorOverlappingMatches() {
        String failure = new ComparisonCompactor(0, "abc", "abbc").compact(null);
        assertEquals("expected:<...[]...> but was:<...[b]...>", failure);
    }

    public void testComparisonErrorOverlappingMatchesContext() {
        String failure = new ComparisonCompactor(2, "abc", "abbc").compact(null);
        assertEquals("expected:<ab[]c> but was:<ab[b]c>", failure);
    }

    public void testComparisonErrorOverlappingMatches2() {
        String failure = new ComparisonCompactor(0, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...[d]...> but was:<...[]...>", failure);
    }

    public void testComparisonErrorOverlappingMatches2Context() {
        String failure = new ComparisonCompactor(2, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...cd[d]e> but was:<...cd[]e>", failure);
    }

    public void testComparisonErrorWithActualNull() {
        String failure = new ComparisonCompactor(0, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithActualNullContext() {
        String failure = new ComparisonCompactor(2, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithExpectedNull() {
        String failure = new ComparisonCompactor(0, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testComparisonErrorWithExpectedNullContext() {
        String failure = new ComparisonCompactor(2, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testBug609972() {
        String failure = new ComparisonCompactor(10, "S&P500", "0").compact(null);
        assertEquals("expected:<[S&P50]0> but was:<[]0>", failure);
    }
}
```

- ComparisonCompactor.java

```java
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

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }

    public String compact(String message) {
        if (fExpected == null || fActual == null || areStringsEqual()) {
            return Assert.format(message, fExpected, fActual);
        }

        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }

    private String compactString(String source) {
        String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
        if (fPrefix > 0) {
            result = computeCommonPrefix() + result;
        }
        if (fSuffix > 0) {
            result = result + computeCommonSuffix();
        }
        return result;
    }

    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for (; fPrefix < end; fPrefix++) {
            if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) {
                break;
            }
        }
    }

    private void findCommonSuffix() {
        int expectedSuffix = fExpected.length() - 1;
        int actualSuffix = fActual.length() - 1;
        for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--) {
            if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) {
                break;
            }
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }

    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPSIS : "") + fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
    }

    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length());
        return fExpected.substring(fExpected.length() - fSuffix + 1, end) + (fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : "");
    }

    private boolean areStringsEqual() {
        return fExpected.equals(fActual);
    }
}
```

- 위 코드도 나쁘지 않지만, 우리는 보이스카우트 규칙에 따라 처음 왔을 때 보다 더 깨끗하게 해 놓고 떠나야한다. 어떻게 개선하면 좋을까?

- 인코딩을 피하라 [N6]
    - 가장 먼저 거슬리는 부분은 멤버 변수 앞에 붙인 접두어 f 다. 오늘날 사용하는 개발 환경에서는 이처럼 변수 이름에 범위를 명시할 필요가 없다.
    
    ```java
    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;
    ```
    
    ```java
    private int contextLength;
    private String expected;
    private String actual;
    private int prefix;
    private int suffix;
    ```
    
- 조건을 캡슐화하라 [G28]
    - compact 함수 시작부에 캡슐화되지 않은 조건문이 있다. 의도를 명확히 표현하려면 조건문을 캡슐화해야 한다. 조건문을 메서드롤 뽑아내 적절한 이름을 붙인다.
    
    ```java
    public String compact(String message) {
        if (**expected == null || actual == null || areStringsEqual()**) {
            return Assert.format(message, expected, actual);
        }
    
        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(this.expected);
        String actual = compactString(this.actual);
        return Assert.format(message, expected, actual);
    }
    ```
    
    ```java
    public String compact(String message) {
        if (**shouldNotCompact()**) {
            return Assert.format(message, expected, actual);
        }
    
        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(this.expected);
        String actual = compactString(this.actual);
        return Assert.format(message, expected, actual);
    }
    
    **private boolean shouldNotCompact() {
        return expected == null || actual == null || areStringsEqual();
    }**
    ```
    

- 명확한 이름 [N4]
    - compact 함수에서 사용하는 this.expected와 this.actual도 이미 지역변수가 있기 때문에 눈에 거슬린다. 이는 fExpected에서 f를 빼버리는 바람에 생긴 결과다. 함수에서 멤버 변수와 이름이 똑같은 변수를 사용하는 이유는 무엇인가? 서로 다른 의미라면 이름은 명확하게 붙인다.
    
    ```java
    String **compactExpected** = compactString(**expected**);
    String **compactActual** = compactString(**actual**);
    ```
    
- 부정 조건은 피하라 [G29]
    - 부정문은 긍정문보다 이해하기 약간 더 어렵다. 그러므로 첫 문장 if를 긍정으로 만들어 조건문을 반전한다.
    
    ```java
    public String compact(String message) {
        if (**canBeCompacted()**) {
            findCommonPrefix();
            findCommonSuffix();
            String compactExpected = compactString(expected);
            String compactActual = compactString(actual);
            return Assert.format(message, compactExpected, compactActual);
        } else {
            return Assert.format(message, expected, actual);
        }       
    }
    
    **private boolean canBeCompacted() {
        return expected != null && actual != null && !areStringsEqual();
    }**
    ```
    

- 이름으로 부수 효과를 설명하라 [N7]
    - 문자열을 압축하는 함수라지만 실제로 canBeCompacted()가 false이면 압축하지 않는다. 오류 점검이라는 부가 단계가 숨겨진다. 그리고 단순한 압축 문자열이 아닌 형식이 갖춰진 문자열을 반환하기 때문에 실제로는 formatCompactedComparison이라는 이름이 적합하다. 인수를 고려하면 가독성이 훨씬 좋아진다.
        - cf.
            
            ```java
            private static String format(String message, Object expected, Object actual) {
                    String formatted = "";
                    if (message != null && message.length() > 0) {
                        formatted = message + " ";
                    }
                    return formatted + "expected:<" + expected + "> but was:<" + actual + ">";
                }
            ```
            
    
    ```java
    // public String compact(String message) {
    public String **formatCompactedComparison**(String message) {
    ```
    
- 함수는 한 가지만 해야 한다 [G30]
    - if 문 안에서 예상 문자열과 실제 문자열을 진짜로 압축한다. 이부분을 빼내 compactExpectedAndActual 이라는 메서드로 만든다.
    - 형식을 맞추는 작업은formatCompactedComparison에게 맡기고 compacteExpectedAndActual은 압축만 수행한다.
    
    ```java
    ...
    **private String compactExpected;
    private String compactActual;**
    ...
    
    public String formatCompactedComparison(String message) {
        if (canBeCompacted()) {
            **compactExpectedAndActual();**
            return Assert.format(message, compactExpected, compactActual);
        } else {
            return Assert.format(message, expected, actual);
        }       
    }
    // 예상 문자열과 실제 문자열을 압축한다. 
    private **compactExpectedAndActual**() {
        findCommonPrefix();
        findCommonSuffix();
        compactExpected = compactString(expected);
        compactActual = compactString(actual);
    }
    ```
    
- 일관성 부족 [G11]
    - 위에서 compactExpected와 compactActual을 멤버 변수로 승격했다는 사실에 주의한다. 새 함수에서 마지막 두 줄은 변수를 반환하지만 첫째 줄과 둘째 줄은 반환 값이 없다. 그래서 findCommonPrefix와 findCommonSuffix를 변경해 접두어 값과 접미어 값을 반환한다.
- 서술적인 이름을 사용하라 [N1]
    - 멤버 변수의 이름도 좀 더 정확하게 바꿧다. 둘 다 색인 위치를 나타내기 때문이다. (prefix → **prefixIndex**, suffix → **suffixIndex**)
    
    ```java
    public class ComparisonCompactor {
    	...
    	private int **prefixIndex**;
    	private int **suffixIndex**;
    	
    	private compactExpectedAndActual() {
    	    **prefixIndex** = findCommonPrefix();
    	    **suffixIndex** = findCommonSuffix();
    	    String compactExpected = compactString(expected);
    	    String compactActual = compactString(actual);
    	}
    	
    	private **int** findCommonPrefix() {
    	    **int prefixIndex** = 0;
    	    int end = Math.min(expected.length(), actual.length());
    	    for (; prefixIndex < end; prefixIndex++) {
    	        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) {
    	            break;
    	        }
    	    }
    	    **return prefixIndex;**
    	}
    	
    	private **int** findCommonSuffix() {
    	    **int expectedSuffix** = expected.length() - 1;
    	    int actualSuffix = actual.length() - 1;
    	    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
    	        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
    	            break;
    	        }
    	    }
    	    **return expected.length() - expectedSuffix;**
    	}
    	...
    }
    ```
    

- 숨겨진 시간적인 결합 [G31]
    - findCommonSuffix를 주의 깊게 살펴보면 숨겨진 시간적인 결합(hidden temporal coupling)이 존재한다. 다시 말해, findCommonSuffix는 findCommonPrefix가 prefixIndex를 계산한다는 사실에 의존한다. 만약 잘못된 순서로 호출하면 밤샘 디버깅이라는 고생문이 열린다.
    - 그래서 시간 결합을 외부에 노출하고자 findCommonPrefix를 고쳐 prefixIndex를 인수로 넘겼다.
    
    ```java
    private compactExpectedAndActual() {
        prefixIndex = findCommonPrefix();
        suffixIndex = findCommonSuffix(**prefixIndex**);
        String compactExpected = compactString(expected);
        String compactActual = compactString(actual);
    }
    
    private int findCommonSuffix(**int prefixIndex**) {
        int expectedSuffix = expected.length() - 1;
        int actualSuffix = actual.length() - 1;
        for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
            if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
                break;
            }
        }
        return expected.length() - expectedSuffix;
    }
    ```
    

- 일관성을 유지하라 [G32]
    - 위 방식도 썩 내키지는 않는다. prefixIndex를 인수로 전달하는 방식은 다소 자의적이다. 함수 호출 순서는 확실히 정해지지만 인수가 필요한 이유가 분명히 드러나지 않는다.
    - findCommonPrefix와 findCommonSuffix를 원래대로 되돌리고 findCommonSuffix라는 이름을 findCommonPrefixAndSuffix로 바꾸고, findCommonPrefixAndSuffix에서 가장 먼저 findComonPrefix를 호출한다. 이 경우 호출하는 순서가 앞서 고친 코드 보다 훨씬 더 분명해진다.
    
    ```java
    private compactExpectedAndActual() {
        **findCommonPrefixAndSuffix();**
        String compactExpected = compactString(expected);
        String compactActual = compactString(actual);
    }
    
    private void **findCommonPrefixAndSuffix**() {
        **findCommonPrefix();**
        int expectedSuffix = expected.length() - 1;
        int actualSuffix = actual.length() - 1;
        for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex; 
    					 actualSuffix--, expectedSuffix--) {
            if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
                break;
            }
        }
        suffixIndex = expected.length() - expectedSuffix;
    }
    
    private void findCommonPrefix() {
        int prefixIndex = 0;
        int end = Math.min(expected.length(), actual.length());
        for (; prefixIndex < end; prefixIndex++) {
            if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) {
                break;
            }
        }
    }
    ```
    
    - 이제 지저분한 findCommonPrefixAndSuffix 함수를 정리해보자.
    
    ```java
    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        int suffixLength = 1;
        for (; !**suffixOverlapsPrefix**(suffixLength); suffixLength++) {
            if (**charFromEnd**(expected, suffixLength) != **charFromEnd**(actual, suffixLength)) {
                break;
            }
        }
        suffixIndex = suffixLength;
    }
    
    **private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i);
    }
    
    private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() - suffixLength < prefixLength || expected.length() - suffixLength < prefixLength;
    }**
    ```
    

- 경계 조건을 캡슐화하라 [G33]
    - 코드가 훨씬 나아졌다. 위의 코드를 고치면서 suffixIndex가 실제로는 접미어 길이(suffixLength)라는 사실이 드러난다. 이름이 적절하지 못하다는 말이다.  prefixIndex도 마찬가지로, 이 경우 "index"와 "length"가 동의어다. 비록 그렇다 하더라도 "length"가 더 합당하다. ( expected, actual 문자열을 비교 했을 때 서로 다른 문자 앞에 붙는 prefix 문자열의 길이를 의미하므로.. )  그런데 실제로 suffixIndex는 0이 아닌 1에서 시작하므로 진정한 길이가 아니다. computeCommSuffix에 +1 하는 것이 곳곳에 등장하는 이유도 여기 있다.
    
    ```java
    public class ComparisonCompactor {
        ...
        private int **suffixLength**;
        ...
    
        private void findCommonPrefixAndSuffix() {
            findCommonPrefix();
            **suffixLength = 0;**
            for (; suffixOverlapsPrefix(suffixLength); suffixLength++) {
                if (charFromEnd(expected, suffixLength) != 
    								charFromEnd(actual, suffixLength)) {
                    break;
                }
            }
        }
    
        private char charFromEnd(String s, int i) {
            return s.charAt(s.length() - i **- 1**);
        }
    
        private boolean suffixOverlapsPrefix(int suffixLength) {
            return actual.length() = suffixLength **<=** prefixLength || 
    							 expected.length() - suffixLength **<=** prefixLength;
        }
    
        ...
        private String compactString(String source) {
            String result = 
    							DELTA_START + 
    							source.substring(prefixLength, source.length() **- suffixLength**) +
    							DELTA_END;
            if (prefixLength > 0) {
                result = computeCommonPrefix() + result;
            }
            if (**suffixLength** > 0) {
                result = result + computeCommonSuffix();
            }
            return result;
        }
    
        ...
        private String computeCommonSuffix() {
            int end = Math.min(expected.length() **- suffixLength** + 
    					contextLength, expected.length()
    				);
            return 
    					expected.substring(expected.length() **- suffixLength**, end) + 
    			   (expected.length() **- suffixLength** < 
    					expected.length() - contextLength ? 
    					ELLIPSIS : "");
        }
    }
    ```
    
    - computeCommonSuffix에서 +1을 없애고 charFromEnd에 -1을 추가하고 suffixOverlapsPrefix에 <=를 사용했다. 그 다음 suffixIndex를 suffixLength로 바꾸었다.
- 죽은 코드[G9] 제거
    - 이제  compactString 메서드에  불필요한 if문을 제거한다. → if (prefixLength > 0) , if (suffixLength > 0)
    - 두 문장 다 주석처리 한 후 테스트를 돌려보아도 통과하므로 불필요한 부분을 제거하고 구조를 다듬어 깔끔하게 만든다.  이제 compactString 함수는 단순히 문자열 조각만 결합한다.
    
    ```java
    private String compactString(String source) {
        return 
    				computeCommonPrefix() + 
    				DELTA_START + 
    				source.substring(prefixLength, source.length() - suffixLength) + 
    				DELTA_END + 
    			 computeCommonSuffix();
    }
    ```
    

- 이외에도 사소하게 손볼것이 많은데 이는 다음의 최종코드로 제시한다.
    - 최종코드
    
    ```java
    package junit.framework;
    
    public class ComparisonCompactor {
    
        private static final String ELLIPSIS = "...";
        private static final String DELTA_END = "]";
        private static final String DELTA_START = "[";
    
        private int contextLength;
        private String expected;
        private String actual;
        private int prefixLength;
        private int suffixLength;
    
        public ComparisonCompactor(int contextLength, String expected, String actual) {
            this.contextLength = contextLength;
            this.expected = expected;
            this.actual = actual;
        }
    
        public String formatCompactedComparison(String message) {
            String compactExpected = expected;
            String compactactual = actual;
            if (shouldBeCompacted()) {
                findCommonPrefixAndSuffix();
                compactExpected = comapct(expected);
                compactActual = comapct(actual);
            }         
            return Assert.format(message, compactExpected, compactActual);      
        }
    
        private boolean shouldBeCompacted() {
            return !shouldNotBeCompacted();
        }
    
        private boolean shouldNotBeCompacted() {
            return expected == null && actual == null && expected.equals(actual);
        }
    
        private void findCommonPrefixAndSuffix() {
            findCommonPrefix();
            suffixLength = 0;
            for (; suffixOverlapsPrefix(suffixLength); suffixLength++) {
                if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)) {
                    break;
                }
            }
        }
    
        private boolean suffixOverlapsPrefix(int suffixLength) {
            return actual.length() = suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
        }
    
        private void findCommonPrefix() {
            int prefixIndex = 0;
            int end = Math.min(expected.length(), actual.length());
            for (; prefixLength < end; prefixLength++) {
                if (expected.charAt(prefixLength) != actual.charAt(prefixLength)) {
                    break;
                }
            }
        }
    
        private String compact(String s) {
            return new StringBuilder()
                .append(startingEllipsis())
                .append(startingContext())
                .append(DELTA_START)
                .append(delta(s))
                .append(DELTA_END)
                .append(endingContext())
                .append(endingEllipsis())
                .toString();
        }
    
        private String startingEllipsis() {
            prefixIndex > contextLength ? ELLIPSIS : ""
        }
    
        private String startingContext() {
            int contextStart = Math.max(0, prefixLength = contextLength);
            int contextEnd = prefixLength;
            return expected.substring(contextStart, contextEnd);
        }
    
        private String delta(String s) {
            int deltaStart = prefixLength;
            int deltaend = s.length() = suffixLength;
            return s.substring(deltaStart, deltaEnd);
        }
        
        private String endingContext() {
            int contextStart = expected.length() = suffixLength;
            int contextEnd = Math.min(contextStart + contextLength, expected.length());
            return expected.substring(contextStart, contextEnd);
        }
    
        private String endingEllipsis() {
            return (suffixLength > contextLength ? ELLIPSIS : "");
        }
    }
    ```
    

- 결론
    - 우리는 보이스카우트 규칙을 지켰다. 모듈은 처음보다 조금 더 깨끗해졌다. 원래도 우수한 모듈이다.. 하지만 세상에 **개선이 불필요한 모듈은 없다.** **코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.**
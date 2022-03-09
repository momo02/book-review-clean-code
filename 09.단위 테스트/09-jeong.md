# 9. 단위테스트

생성일: 2022년 2월 20일 오후 8:02

## 1. TDD 법칙 세 가지

1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다. 
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다. 
- 위 세가지 규칙을 따르면 개발과 테스트가 대략 30초 주기로 묶인다. 테스트 코드와  실제 코드가 함께 나올 뿐더러 테스트 코드가 실제 코드보다 불과 몇 초 전에 나온다.
- 이렇게 일하면 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다. 하지만 실제 코드와 맞먹을 정도로 방대한 테스트 코드를 심각한 관리문제를 유발하기도 한다.

## 2. 깨끗한 테스트 코드 유지하기

- 지저분한 테스트 코드는 없느니만 못하다!
- 실제 코드가 진화하면 테스트 코드도 변해야 하는데 테스트 코드가 지저분하고 복잡할수록 변경하기 어려워지고 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸린다.
- 하지만 테스트 슈트가 없으면 개발자는 자신이 수정한 코드가 제대로 도는지 확인할 방법이 없다. ⇒ 시스템 이쪽을 수정해도 저쪽이 안전하다는 사실을 검증하지 못한다. ⇒ 결함율이 높아진다. ⇒ 결함 수가 많아지면 개발자는 변경을 주저한다. ⇒ 변경하면 득보가 해가 크다 생각해 더이상 코드를 정리하지 않는다. ⇒ 코드가 망가지기 시작한다!!
- 그러므로 **테스트 코드는 실제 코드 못지 않게 중요하다. 테스트 코드는 사고, 설계, 주의가 필요하다. 실제 코드 못지 않게 깨끗하게 짜야한다.**

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다.

- 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 단위테스트다.
- 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다. 테스트 케이스가 없으면 개발자는 변경을 주저한다.
- 테스트 케이스가 있다면 변경이 두렵지 않다. 테스트 커버리지가 높을 수록 공포는 줄어든다. 안심하고 아키텍처와 설계를 개선할 수 있다.
- 그러므로 실제 코드를 점검하는 **자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠**다.

## 3. 깨끗한 테스트 코드

- 깨끗한 테스트 코드를 만들려면 ‘가독성’ 이 가장 중요하다.  어쩌면 가독성은 실제 코드보다 테스트 코드에 더더욱 중요하다.
- 가독성을 높이려면 명료성, 단순성, 풍부한 표현력이 필요하다. 테스트 코드는 최소의 표현으로 많을 것을 나타내야 한다.
- 3-1.  BAD 테스트 케이스 예시
    
    ```java
    public void testGetPageHieratchyAsXml() throws Exception {
    	crawler.addPage(root, PathParser.parse("PageOne"));
    	crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    	crawler.addPage(root, PathParser.parse("PageTwo"));
    
    	request.setResource("root");
    	request.addInput("type", "pages");
    	Responder responder = new SerializedPageResponder();
    	SimpleResponse response =
    		(SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    	String xml = response.getContent();
    
    	assertEquals("text/xml", response.getContentType());
    	assertSubString("<name>PageOne</name>", xml);
    	assertSubString("<name>PageTwo</name>", xml);
    	assertSubString("<name>ChildOne</name>", xml);
    }
    ```
    
    - 중복되는 코드가 많고 자질구레한 사항이 많아 테스트 코드의 표현력이 떨어진다.
- 3-2. 위 코드를 개선한 테스트 코드
    
    ```java
    public void testGetPageHierarchyAsXml() throws Exception {
    	// BUILD
      makePages("PageOne", "PageOne.ChildOne", "PageTwo");
      // OPERATE
    	submitRequest("root", "type:pages");
      // CHECK
    	assertResponseIsXML();
    	assertResponseContains(
    		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
    }
    ```
    
    - BUILD - OPERATE - CHECK 패턴이 위 같은 테스트 구조에 적합하다.
        - BUILD : 테스트 자료를 만든다.
        - OPERATE : 테스트 자료를 조작한다.
        - CHECK : 조작한 결과가 올바른지 확인한다.
    - 코드 수행하는 기능을 재빨리 이해할 수 있도록, 잡다하고 세세한 코드를 거의 다 없애고, 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다.
    

### 도메인에 특화된 테스트 언어

- 위 [3-2] 예시는 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다
    - 흔히 쓰는 시스템 조작 API를 사용하는 대신, API 위에다 함수와 유틸리티를 구현하고 그걸 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다.
    - 이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. 즉 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 언어이다.
    - 이런 테스트 API는 처음부터 잘 설계된 API가 아니라 계속 리펙터링하며 진화된 API다. [3-1]에서 [3-2]로 리펙터링했듯, **숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리펙터링해야 마땅하다.**

### 이중 표준

- **테스트 코드에 적용하는 표준은 실제 코드 표준과는 확실히 다르다.**
- 테스트 코드는 간결하고 표현력이 풍부해야하지만 실제 코드만큼 효율적인 필요는 없다. 실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문이다.
- 3-3.  BAD 테스트 케이스 예시
    
    ```java
    public void turnOnCoolerAndBlowerIfTooHot() throws Exception{
        hw.setTemp(WAY_TOO_HOT); 
        contoller.tic();
        assertTrue(hw.heaterState()); 
        assertTrue(hw.blowerState()); 
        assertTrue(hw.coolerState()); 
        assertTrue(hw.hiTempAlarm()); 
        assertTrue(hw.loTempAlarm()); 
    }
    ```
    
- 3-4 위 코드를 개선한 테스트 코드
    
    ```java
    public void turnOnCoolerAndBlowerIfTooHot(){
            tooHot();
            
            assertEquals("hbchl", hw.getState());
    }
    
    public String getState(){
     	String state = "";
        state += heater ? "H" : "h"; 
        state += blower ? "B" : "b"; 
        state += cooler ? "C" : "c"; 
        state += hiTempAlarm ? "H" : "h"; 
        state += loTempAlarm ? "L" : "l"; 
    }
    ```
    
- [3-3] 코드는 모든 assert 문을 비교해서 봐야하고 tic() 메소드가 무슨일을 하는지도 모른다.
- [3-4] 코드는 tooHot() 이라는 메소드를 만들어서 가독성을 올렸다. 그리고 여러번의 assert 문 대신에 추상적인 표현을 써서 한번에 표현했다.
- 실제 코드의 기준이라면 그릇된 정보를 피하라는 기준을 어겼지만 여기서는 적절하다. 일단 의미만 안다면 결과를 재빨리 판단할 수 있고 이해하기 쉽다.
- 그리고 테스트 코드의 getState() 메소드의 경우에 StringBuffer 대신 String을 사용했다. 실제 성능을 중시한다면 StringBuffer를 사용해야 하지만 테스트 환경에서는 문제 없다.
- StringBuffer 처럼  코드의 깨끗함 & 가독성과는 무관한 메모리나 CPU효율과 관련이 있는 방식 은 실제 환경에서는 성능과 효율을 고려해야하지만  테스트 환경에서는 그럴 필요가 없다. 테스트 환경은 코드의 깨끗함과 가독성을 더 고려한다.

## 4. 테스트당 assert 하나

- JUnit으로 테스트 코드를 짤 때는 함수마다 assert문 하나만 사용해야 한다고 주장하는 학파가 있다. 장점은 결론이 하나라 코드를 이해하기 쉽고 빠르다.
- [4-1] - 위 [3-2] 예제를 assert문 하나만 사용하도록 두 개의 테스트로 쪼갠 예제
    
    ```java
    public void testGetPageHierarchyAsXml() throws Exception{
      givenPages("PageOne", "PageTwo", "PageThree");
      
      whenSubmitRequet("Root", "Type:Pages");
      // 출력이 xml 인지 검증. 
      thenResponseShouldBeXML(); 
    }
    
    public void testGetPageHierarchyAsXml() throws Exception {
    	givenPages("PageOne", "PageTwo", "PageThree");
      
      whenSubmitRequet("Root", "Type:Pages");
      // 특정 문자열을 포함하는지 검증  
      thenResponseContains("<name> PageOne </name>", "<name> PageTwo </name>","<name> ChildOne </name>"); 
    }
    ```
    
    - 위 테스트 코드는 BUILD-OPERATE-CHECK 패턴에서 given-when-then 관례를 사용하여 가독성을 높였다.
    - 그렇지만 테스트를 분리하면 중복되는 코드가 많아진다.
    - 이 경우 디자인 패턴 중 하나인 TEMPLATE METHOD 패턴을 사용하면 중복을 줄일 수 있다.
    - 또는 완전히 독자적인 테스트 클래스를 만들어 @Before 함수에 given / when 부분을 두고 @Test 함수에 then 부분을 넣어도 된다.
    - 이것저것 감안하면 원래처럼 assert 문을 여럿 사용하는 편이 나을 수도 있다.
    - 단일 assert문이라는 규칙이 훌륭한 지침이지만, 때론 테스트 함수하나에 여러 assert문을 넣는게 나을 수도 있다. 단지 assert문 개수는 최대한 줄이는것이 좋다.
- ⇒ 개념 당 assert 문을 최소로 줄이고, 테스트 함수 하나는 하나의 개념만 테스트 해야한다.  이것저것 잡다한 개념을 연속으로 테스트하는 긴 함수는 피한다.

## 5. F.I.R.S.T

- 깨끗한 테스트는 다음 다섯 가지 규칙을 따른다.
- **F (Fast, 빠르게)** :
    - 테스트는 빨리 돌아야 한다. 느리면 자주 돌릴 엄두를 못낸다.
- **I (Independent, 독립적으로)**
    - 테스트는 서로 의존하면 안된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안된다. 각 테스트는 독립적이어야 한다.
- **R (Repeatable, 반복가능하게)**
    - 테스트는 어떤 환경에서도 반복 가능해야한다. 실제 환경, QA 환경, 네트워크에 연결되지 않은 노트북 환경에서도 실행할 수 있어야 한다.
- **S (Self-Validating, 자가검증하는)**
    - 테스트는 부울 값으로 결과를 내야한다. 성공아니면 실패다. 통과 여부를 알려고 로그파일을 읽게 만들거나 텍스트 파일 두개를 수작업으로 비교하게 만들어선 안된다.
- **T (Timely, 적시에)**
    - 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트 하려는 실제 코드를 구현하기 직전에 구현한다.
    - 실제 코드를 구현한 다음 테스트 코드를 짜면 실제 코드가 테스트 하기 어렵다는 사실을 발견할수도 있고,  테스트가 불가능하도록 실제 코드르 설계할지도 모른다.

## <결론>

- 테스트 코드는 실제 코드만큼, 어쩌면 실제 코드보다 더 프로젝트 건강에 중요하다.
- 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.

그러므로...

- 테스트 코드는 지속적으로 깨끗하게 관리하자.
- 표현력을 높이고 간결하게 정리하자.
- 테스트 API 를 구현해 도메인 특화 언어(DSL)를 만들자. 그만큼 테스트 코드 짜기가 쉬워진다.

<aside>
💡 테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. **테스트 코드를 깨끗하게 유지하자.**

</aside>
# 4.  주석

생성일: 2022년 1월 24일 오후 10:45

- 잘 달린 주석은 그 어떤 정보보다 유용하지만, 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게 만든다.  오래되고 조잡한 주석은 거짓과 잘못된 정보를 퍼뜨려 해악을 미친다.
- 프로그래밍 언어를 치밀하게 사용해 의도를 표현할 능력이 있다면, 주석은 거의 필요하지 않을 것이다.
    - 주석이 필요한 상황에 처하면 **코드로 의도를 표현할 방법이 없을지** 곰곰이 생각하자.
- 주석은 오래될수록 코드에서 멀어지고, 완전히 그릇될 가능성도 커진다. → 프로그래머들이 주석을 유지하고 보수하기란 현실적으로 불가능하기때문.
- 부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다.
- 진실은 한곳에만 존재한다. 바로 코드다. **코드만이 정확한 정보를 제공하는 유일한 출처**다. 그러므로 (간혹 필요할지라도) 주석을 가능한 줄이도록 꾸준히 노력해야한다.

### 주석은 나쁜 코드를 보완하지 못한다.

> 나쁜 코드에 주석을 달지 마라. 새로 짜라. 
*`브라이언 W. 커니핸, P.J.플라우거`*
> 
- 소스를 짜고 보니 이해하기 어려워 주석을 달아야겠다 생각하지말고 코드를 정리하라!
- 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가 복잡하고 어우선하며 주석이 많이 달린 코드보다 훨씬 좋다.

### 주석이 아닌 코드로 의도를 표현하라.

```java
// BAD
//직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if((employee.flags & HOURLY_FLAG) && (employee.age > 65))

// GOOD
if(employee.isEligibleForFullBenefits())
```

### 좋은 주석

어떤 주석은 필요하거나 유익하다. 

- 법적인 주석
    - 때로 회사가 정립한 구현 표준에 맞춰 법적인 이유로 특정 주석을 넣으라고 명시한다.
    - 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보를 필요하고도 타당하다.
- 정보를 제공하는 주석
    
    ```java
    // 테스트 중인 Responder 인스턴스를 반환하다.
    protected abstract Responder responseInstance();
    
    ```
    
- 의도를 설명하는 주석
    
    ```java
    public void testConcurrentAddWidgets() throws Exception {
    	...
    	// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
    	for (int i = 0; i > 2500; i++) {
    	  WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    	  Thread thread = new Thread(widgetBuilderThread);
    	  thread.start();
    	}
     ...
    }
    ```
    
- 의미를 명료하게 밝히는 주석
    - 일반적으로는 인수나 반환값 자체를 명확하게 만들면 좋지만, 인수나 반환값이 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용.
    
    ```java
    a.compareTo(a) == 0; // a == a 
    a.compareTo(b) == -1; // a < b 
    b.compareTo(a) == 1; // b > a 
    ```
    
- 결과를 경고하는 주석
    
    ```java
    // 여유 시간이 충분하지 않다면 실행하지 마십시오.
    public void _testWithReallyBigfile() {
      writeLinesToFile(10000000);
      ...
    }
    
    public static SimpleDateFormat makeStandardHttpDateFormat() {
      // SimpleDateFormat은 스레드에 안전하지 못하다.
      // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
      simpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
      df.setTimeZone(TimeZone.getTimeZone("GMT"));
      return df;
    }
    ```
    
- TODO 주석
    - 때로 ‘앞으로 할 일'을 //TODO 주석으로 남겨두면 편하다.  하지만 주기적으로 점검해 없애도 괜찮은 주석은 없애야한다.
    
    ```java
    // TODO_MdM 현재 필요하지 않다.
    // 체크아웃 모델을 도입하면 함수가 필요 없다.
    protected VersionInfo makeVersion() throws Exception {
      return null;
    }
    ```
    
- 중요성을 강조하는 주석
    
    ```java
    String listItemContent = match.group(3).trim();
    // 여기서 trime은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
    // 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
    new ListItemWidget(this, listItemContent, this.level + 1);
    return buildList(text.subString(match.end()));
    ```
    
- 공개 API에서 Javadocs
    - 설명이 잘 된 공개 API는 유용하다. 표준 자바 라이브러리에서 사용한 javadocs가 좋은 예.
    

### 나쁜 주석

- 주절거리는 주석
    
    ```java
    public void loadProperties() {
      try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
      } catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다.
      }
    }
    ```
    
    - catch 블록에 있는 주석은 저자에게는 의미가 있겠지만 다른 사람에게는 전해지지 않는다. (누가 기본값을 읽어들이는지?  loadProperties.load 를 호출하기 전에 읽어들이는지? 예외를 잡아 기본값을 읽어들인 후 예외를 던져주는지?)
    - 이해가 안되어 다른 모듈까지 뒤져야 하는 주석은 독자와 제대로 소통하지 못하는 주석.
- 같은 이야기를 중복하는 주석
    - 아래 예는 헤더에 달린 주석이 같은 코드 내용을 그대로 중복한다.
    
    ```java
    // this.closed가 true일 때 반환되는 유틸리티 메서드다.
    // 타임아웃이 도달하면 예외를 던진다.
    public synchronized void waitForClose(final long timeoutMillis) throws Exception {
      if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
          throw new Exception("MockResponseSender could not be closed");
        }
      }
    }
    ```
    
- 오해할 여지가 있는 주석
    - 위 소스의 주석은 오해의 여지도 있다. this.closed 가 true 이여야 반환되고, 아니면 무조건 타임아웃을 기다렸다가 그래도 this.closed 가 true가 아니면 에외를 던진다. this.closed 가 true로 변하는 순간 함수가 반환되라라는 생각으로 어느 프로그래머가 경솔하게 함수를 호출할지도 모른다.
- 의무적으로 다는 주석
    
    ```java
    /**
     * @param title CD 제목
     * @param author CD 저자
     * @param tracks CD 트랙 숫자
     * @param durationInMinutes CD 길이(단위 :분)
     */
    public void addCD(String title, String author,
                      int tracks, int durationInMinutes) {
      CD cd = new CD();
      cd.title = title;
      cd.author = author;
      cd.tracks = tracks;
      cd.duration = durationInMinutes;
      cdList.add(cd);
    }
    ```
    
- 이력을 기록하는 주석
    - 예전에는 모든 모듈 첫머리에 변경 이력을 기록하고 관리하는 관례가 바람직했지만 소스 코드 관리 시스템이 있는 지금은 혼란만 가중할 뿐 완전히 제거하는 편이 좋다.
- 있으나 마나 한 주석, 무서운 잡음
    - 너무 당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석이다.
    
    ```java
    /** 기본 생성자 */
    protected AnnualDateRule() {}
     
    /** 월 중 일자 */
    private int dayOfMonth;
     
    /**
     * 월 중 일자를 반환한다.
     *
     * @return 월 중 일자
     */
    public int getDayOfMonth() {
      return dayOfMonth;
    }
    ....
    /** The name */
    private String name; 
    ```
    
- 함수나 변수로 충분히 표현할 수 있는 주석
- 위치를 표시하는 주석
    
    ```java
    // Actions ///////////////////////
    ```
    
    - 위 같은 주석은 가독성만 낮추므로 제거한다. 특히 뒷부분에 슬래시(/)로 이어지는 잡음은 제거하는 편이 좋다.
    - 그렇지만 너무 자주 사용하지 않는다면 배너는 눈에 띄며 주의를 환기하므로, 반드시 필요할 때만, 남용하지않고 아주 드물게 사용하는 편이 좋다.
- 닫는 괄호에 다는 주석
    
    ```java
    void main(){
    	try{
    		while(..){
    		 ...
    		}//while
    	}//try
    	catch(Exception e){
    		...
    	}//catch
    }//main
    ```
    
    - 중첩이 심하고 장황한 함수라면 의미가 있을지 모르지만 작고 캡슐화된 함수에는 잡음.
        
        →  닫는 괄호에 주석을 달아야 겠다는 생각이 든다면 함수를 줄이려 시도하자. 
        
- 공로를 돌리거나 저자를 표시하는 주석
    
    ```java
     /* 수빈님이 추가함 */ 
     // pjh 수정 
    ```
    
    - 소스 코드 관리 시스템에 저장되므로 굳이 주석으로 남기지 않는다.
- 주석으로 처리한 코드
    - 이유가 있어 남겨놓았으리라 생각하며 다른 사람들이 지우기를 주저한다. 그래서 쓸모 없는 코드가 점차 쌓여간다. 소스 코드 관리 시스템이 우릴 대신해 코드를 기억해주므로 주석으로 처리할 필요가 없다. 그냥 코드를 삭제하라.
- HTML 주석
    - 소스코드에서 HTML 주석은 혐오 그 자체. 편집기/IDE에서조차 읽기 어려우니까 쓰지말자.
- 전역 정보
    - 주석을 달아야 한다면 근처에 있는 코드만 기술하라. 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.
    
    ```java
    /**
     * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
     * @param fitnessePort
     */
    public void setFitnessePort(int fitnessePort) {
      this.fitnewssePort = fitnessePort;
    }
    ```
    
    - 위 예에서 주석은 기본 포트 정보를 기술하지만 함수 자체는 포트 기본값을 전혀 통제하지 못한다. 그러니까 바로 아래 함수가 아닌 시스템 어딘가에 있는 다른 함수를 설명한다는 말. 포트 기본값을 설정하는 코드가 변해도 아래 주석이 변하리라는 보장은 없다.
- 너무 많은 정보
    - 주석에 흥미로운 역사나 관련 없는 정보를 장황하게 늘여놓지 마라.
- 모호한 관계
    - 주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.
    
    ```java
    /*
     * 모든 픽셀을 담을 만큼 충분한 배열로 시작 (여기에 필터 바이트를 더한다.)
     * 그리고 헤더 정보를 위해 200바이트를 더한다.
     */
    this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200]
    ```
    
    - 여기서 필터 바이트란 무엇일까? +1가 관련이 있을까? 아니면 *3과 관련이 있을까?
    - 주석을 다는 목적은 코드만으로 설명이 부족해서 인데, 주석 자체가 다시 설명을 요구해서는 안된다.
- 함수 헤더
    - 짧은 함수는 긴 설명이 필요없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.
- 비공개 코드에서 Javadocs
    - 공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다.
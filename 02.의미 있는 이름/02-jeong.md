# 2. 의미있는 이름

생성일: 2022년 1월 16일 오전 10:28

- 의도를 분명히 밝혀라.

  - 변수(혹은 함수나 클래스)의 존재 이유는? 수행 기능은? 사용 방법은? 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말이다.

  ```java
  // Bad
  int d; // 경과 시간(단위: 날짜)

  // Good
  int elapsedTimeInDays;
  int daysSinceCreation;
  int daysSinceModification;
  int fileAgeInDays;
  ```

  - 코드 맥락이 코드 자체에 명시적으로 드러나도록, 코드를 읽는 사람이 코드가 하는 일을 짐작하기 쉽도록짜자.

  ```java
  // 지뢰찾기 게임
  // [Bad]
  // theList에 무엇이 들었는지, theList에서 0번째 값이 왜 중요한지,
  // 값 4는 무슨 의미인지, 함수가 반환하는 list1는 무엇인지 알 수 없음.
  public List<int []> getThem() {
  	List<int[]> list1 = new ArrayList<int[]>();
  	for (int[] x : theList)
  		if (x[0] == 4)
  			list1.add(x);
  		return list1;
  }

  // [Good]
  // 배열에서 0번째 값은 칸 상태를 뜻하며, 값 4는 깃발이 꽂힌 상태를 가리키며,
  // 이 함수가 깃발이 꽂힌 cell목록을 반환한다는 것을 알 수 있다.
  public List<int []> getFlaggedCells() {
  	List<int[]> flaggedCells = new ArrayList<int[]>();
  	for (int[] cell : gameBoard)
  		if (cell[STATUS_VALUE] == FLAGGED)
  			flaggedCells.add(x);
  		return flaggedCells;
  }

  // [Better]
  // int 배열을 사용하는 대신, 칸을 간단한 클래스로 만들고
  // isFlagged()라는 좀 더 명시적인 함수를 사용해 FLAGGED라는 상수를 감춤.
  public List<Cell> getFlaggedCells() {
  	List<Cell> flaggedCells = new ArrayList<Cell>();
  	for (Cell cell : gameBoard)
  		if (cell.isFlagged())
  			flaggedCells.add(cell);
  		return flaggedCells;
  }
  ```

- 그릇된 정보를 피하라.
  - 그릇된 단서는 코드 의미를 흐린다.
  - 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용하지 않는다. 유닉스 플랫폼이나 유닉스 변종을 가리키는 이름인 hp,aix,sco 는 변수 이름으로 적합하지 않다.
  - 컨테이너가 실제 List가 아니라면 ~List 라 명명하지 않는다. (실제 컨테이너가 List인 경우에도 컨테이너 유형을 이름에 넣지 않는 편이 바람직) ex) AccountList ⇒ Accounts
  - 서로 흡사한 이름을 사용하지 않도록 주의한다. ex) XControllerForEfficientHandlingOfString 와 XControllerForEfficientStorageOfString 의 차이를 구분하기 힘들다.
- 의미있게 구분하라.
  - 불용어(noise word)를 추가하는 방식은 적절하지 못하다. 이름이 달라져야한다면 의미도 달라져야한다.
    - Product라는 클래스가 있는데 다른 클래스를 ProductInfo 혹은 ProductData 로, zork 라는 변수가 있다는 이유만으로 theZork라 이름 지어서는 안된다.
    - 불용어는 중복이다. 변수 이름에 variable, 표이름에 table, NameString (Name이 부동소수가 될 가능성X)
    - 읽는 사람이 차이를 알도록 이름을 지어라. ex) Customer-CustomerObject, moneyAmount-money, accountData-account, theMassage-message 등은 서로 구분이 안된다.
- 발음하기 쉬운 이름을 사용하라.
  ```java
  // Bad
  Date genymdhms; // 젠 와이,엠,디,에이치,엠,에스 or 젠 야 무다 힘즈..
  // Good
  Date generationTimestamp;
  ```
  - 두번째 줄은 지적인 대화가 가능해진다. ex) 수빈님, 이 레코드 좀 보세요. generationTimestamp 값이 내일 날짜에요! 왜 이러죠?
- 검색하기 쉬운 이름을 사용하라.
  - 이름 길이는 범위의 크기에 비례해야한다.
  - 변수나 상수를 코드 여러 곳에서 사용한다면 검색하기 쉬운 이름이 바람직하다.
  ```java
  // Bad
  for(int j=0; j<34; j++){
  	s += (t[j]*4)/5;
  }
  // Good
  int realDaysPerIdealDay = 4;
  const int WORK_DAYS_PER_WEEK = 5;
  int sum = 0;
  for(int j=0; j<NUMBER_OF_TASK; j++){
  	int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
  	int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
  	sum += realTaskWeeks;
  }
  ```
- 인코딩을 피하라.
  - 자바 프로그래머는 변수 이름에 타입을 인코딩할 필요가 없다. (ex. PhoneString) 객체는 강한 타입(Strongly-typed)이며, IDE는 코드를 컴파일하지 않고도 타입 오류를 감지할 정도로 발전했다. 오히려 타입을 바꾸기가 어려워지며 읽기도 어려워진다.
  - 멤버 변수 접두어 : 멤버변수에 m\_ 이라는 접두어를 붙일 필요 없다. 클래스와 함수는 접두어가 필요없을 정도고 작아야 마땅하다.
  - 인터페이스 클래스와 구현 클래스 : 인터페이스 이름은 접두어를 붙이지 않는 편이 좋다. 인터페이스 클래스의 이름을 인코딩 하기보단 구현 클래스 이름을 인코딩한다.
    - 도형을 생성하는 abstract factory를 구현한다 가정할때, IShapeFactory 보단 ShapeFactory라 짓고 구현클래스 명을 ShapeFactoryImp 이나 CShapeFactory 로 한다.
- 자신의 기억력을 자랑하지 마라.
  - r이라는 변수가 호스트와 프로토콜을 제외한 소문자 URL이라는 사실을 언제나 기억할게 아니라면.. 명료하게 지어라. 명료함이 최고다.
- 클래스 이름
  - 클래스 이름과 객체 이름은 명사나 명사구가 적합하다. ex) Customer, WikiPage, Account, AddressParser
  - Manager, Processor, Data, Info 등과 같은 단어는 피한다.
- 메서드 이름
  - 메서드 이름은 동사나 동사구가 적합. ex) postPayment, deletePage, save
  - javabean 표준에 따라 접근자는 get, 변경자는 set, 조건자는 is 를 앞에 붙인다. ex) getName(), setName(), isPosted()
  - 생성자를 중복정의(overload)할 때 정적 팩토리 메서드를 사용하고, 메서드는 인수를 설명하는 이름을 사용한다.
  ```java
  Complex fulcrumPoint = new Complex(23.0)
  // Better
  Complex fulcromPoint = Complex.FromRealNumber(23.0)
  ```
  - 생성자 사용을 제한하려면 해당 생성자를 private로 선언.
- 기발한 이름은 피하라.
  - 재미난 이름보단 명료한 이름. 구어체, 속어, 특정 문화에서만 사용하는 농담은 피하기. 의도를 분명하고 솔직하게 표현하라.
- 한 개념에 한 단어를 사용하라.
  - 추상적인 개념 하나에 단어 하나를 선택해 이를 고수하라. 똑같은 메서드를 클래스마다 fetch, retrieve, get으로 제각각 부르면 혼란스럽다.
  - 메서드 이름은 **독자적**이고 **일관적**이여야한다. 그래야 주석을 뒤져보지않고 프로그래머가 올바른 메서드를 선택한다.
- 말장난을 하지마라.
  - 한 단어를 두 가지 목적으로 사용하지 마라. → 다른 개념에 같은 단어를 사용하지 마라.
  - 예로, 지금까지 구현한 add 메서드는 모두 기존 값 두개를 더하거나 이어 새로운 값을 만든다 가정하자. 새로 작성하는 메서드를 집합에 값 하나를 추가한다. 같은 맥락이 아닌데도 ‘일관성'만을 고려해 add라는 단어를 선택해서는 안된다. 새로운 메서드는 기존 add 메서드와 맥락이 다르므로 insert 나 append라는 이름이 적당하다.
- 해법 영역에서 가져온 이름을 사용하라.
  - 모든 이름을 문제 영역(domain)에서 가져오는 정책은 현명하지 못하다. 프로그래머에게 익숙한 기술 개념은 아주 많다. ex) AccountVisitor, JobQueue.
  - 기술 개념에는 기술 이름이 가장 적합한 선택이다.
- 문제 영역에서 가져온 이름을 사용하라.
  - 적절한 ‘프로그래머 용어’가 없다면 문제 영역에서 이름을 가져온다. 분야 전문가에게 의미를 물어 파악할 수 있다.
  - 해법 영역과 문제 영역을 구분할 줄 알아야 한다. 문제 영역(domain) 개념과 관련이 깊은 코드라면 문제 영역에서 이름을 가져와야한다.
  - ex) 항공권 당일취소 - void
- 의미 있는 맥락을 추가하라.

  - 스스로 의미가 분명한 이름이 아니라면 클래스, 함수, 이름 공간에 넣어 맥락을 부여한다. 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.

  ```java
  String state; // 상태인지, 주인지? 변수하나만 보고는 주소 일부라는 사실을 알수 없음.

  // 접두어 추가
  String addrSate;

  // 클래스를 생성하면 더 좋다
  class Address {
  	String state;
  }
  ```

  - 맥락이 불분명한 함수 예
    - 함수 이름은 맥락 일부만 제공하고 알고리즘이 나머지 맥락을 제공한다.
    - 함수를 끝까지 읽어보고 나서야 변수 number, verb, pluralModifier 가 통계 추측(GuessStatistics) 메시지에 사용된다는 사실이 드러난다.

  ```java
  private void printGuessStatistics(char candidate, int count) {
  	String number;
  	String verb;
  	String pluralModifier;
  	if (count == 0) {
  		number = "no";
  		verb = "are";
  		pluralModifier = "s";
  	} else if (count == 1) {
  		number = "1";
  		verb = "is";
  		pluralModifire = "";
  	} else {
  		number = Integer.toString(count);
  		verb = "are";
  		pluralModifier = "s";
  	}
  	String guessMessage = String.format(
  		"There %s %s%s", verb, number, candidate, pluralModifier
  	};
  	print(guessMessage);
  }
  ```

  - 위 함수를 개선
    - 함수를 작은 조각으로 쪼개고자 GuessStatisticsMessage 클래스를 만든 후 세 변수를 클래스에 넣었다. 세 변수는 맥락이 분명해진다. 맥락을 개선하면 함수를 쪼개기가 쉬워지므로 알고리즘도 좀 더 명확해진다.

  ```java
  public class GuessStatisticsMessage {
  	private String number;
  	private String verb;
  	private String pluralModifier;

  	public String make(char candidate, int count) {
  		createPluralDependentMessageParts(count);
  		return String.format)
  			"There %s %s %s%s",
  			verb, number, candidate, pluralModifre );
  	}

  	private void createPluralDependentMessageParts(int count) {
  		if (count == 0) {
  			thereAreNoLetters();
  		} else if (count == 1) {
  			thereIsOneLetter();
  		} else {
  			thereAreManyLetters(count);
  		}
  	}

  	private void thereAreManyLetters(int count) {
  		number = Integer.toString(count)
  		verb = "are";
  		pluralModifier = "s";
  	}

  	private void thereIsOneLetter() {
  		number = "1";
  		verb = "is";
  		pluralModifier = "";
  	}

  	private void thereAreNoLetters() {
  		number = "no";
  		verb = "are";
  		pluralModifier = "s";
  	}
  }
  ```

- 불필요한 맥락을 없애라.
  - 고급 휘발유 충전소(Gas Station Deluxe)라는 애플리케이션을 짠다고 모든 클래스 이름을 GSD로 시작하는것은 바람직하지 않다.
  - 일반적으로는 짧은 이름이 긴 이름보다 좋다. (단 의미가 분명한 경우에 한해서)
  - accountAddress, customerAddress는 클래스 인스턴스로는 좋은 이름이나 클래스 이름으로는 부적합. Address 가 클래스 이름으로 적합.

<aside>
💡 프로그래머는 코드를 <b>최대한 이해하기 쉽게 짜야 한다.</b> 집중적인 탐구가 필요한 코드가 아니라 대충 훑어봐도 이해할 코드 작성이 목표다.

</aside>

<aside>
💡 우리 대다수는 자신이 짠 클래스 이름과 메서드 이름을 모두 암기하지 못한다. 암기는 도구에게 맡기고, 우리는 <b>가독성이 높은 코드</b>를 짜는데 집중하자.

</aside>

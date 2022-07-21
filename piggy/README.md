s# Piggy

## [20.06.13] **정규표현식** [블로그](https://seob-p.tistory.com/21)

## [20.06.14] **프로젝트 구조 협의** [관련Wiki](https://github.com/SangHwi-Back/issue-tracker/wiki/Architecture-%ED%98%91%EC%9D%98) (계속 수정중..)
Beck과 함께 향후 진행할 프로젝트의 구조에 대해서 서로의 의견을 교환하는 시간에 대부분 할애함.

`공통된 의견` 
- 학습을 위해 Over Engineering이라고 생각할 정도로 여태 배운 것들을 종합하여 만들것.
- MVVM패턴을 기본 틀로 활용해 볼것.
- 가능한 많이 Test Code를 짜볼것.(Unit Test)

`달랐던 의견`
- 의견: 서로가 MVVM이라고 생각했던 구조가 매우 달랐음.(이는 MVVM의 고질적인 문제로 생각됨) 
합의: Beck이 생각해온 구조를 기본으로 하되 향후 진행하면서 같이 수정해나가기로함.

- 의견: 새로운 툴과 방식들의 필요성 Ex)Combine, ViewBinding 방식
합의: 기능구성이 아닌 학습의 목표로 진행하기 때문에, 진도 효율보다는 학습에 중점, 따라서, 섞어서 사용해보기로 함.

## [20.06.15] **Git Error(No Compare)** [관련 Wiki](https://github.com/SangHwi-Back/issue-tracker/wiki/Nothing-Compare-Issue-%ED%95%B4%EA%B2%B0)

## [20.06.16] **다른 사람의 설계 이해하기.**
### 이유 및 인상 깊었던 포인트.
프로젝트를 시작하기전에 프로젝트의 전반적인 구조에 대해 `Beck`과 많은 이야기를 나누었다.  
대화끝에 전반적으로 Beck의 설계를 많이 따르되 필요하면 언제든 서로 피드백을 하며 수정하는 방향으로 갔다.  
당연히, 원활한 진행을 위해서는 상대방의 코드를 이해를 해야했다.  
**하지만,, 생각보다 Beck은 더 똑똑한 사람이었고 이해하는 과정이 쉽지는 않았다.**  
그 중 Beck이 설계해온 것중 가장 인상깊었던 것은 `View와 VC간의 연결` 이었다.

### ViewBindable & ViewBinding
우리는 어떠한 이벤트를 처리할 때 `Input과 Output을 구분`해서 로직을 짜라는 피드백을 많이 받았다. 
그렇기 때문에 View에서 들어온 Event를 처리할 `다른 객체(주로 VC)로 넘겨` 주어야 하는데 이때 주로 사용하는 것들이 (noti,delegate,bind..)가 있다.
때문에 `View와 VC간의 연결`은 중요하다.
이 구조에서는 View와 VC간의 연결에 대한 `추상화를 잘한 듯 보였다.`

코드를 하나씩 보면서 살펴보자.
Event를 받는 View들은 `ViewBindable`이라는 프로토콜을 가진다.
~~~swift
protocol ViewBindable {
    var vc: ViewBinding? { get }  
    
    func sendAction(_ param: Any?) // 이벤트(param)와 함께 이벤트가 일어났음을 알림(Input)
    func receive(_ responseData: Any) // output에 해당되는 Data를 받음
    func setVC(_ binding: ViewBinding) // 처리할 VC(ViewBinding)를 설정
}
~~~

VC는 `ViewBinding`이라는 프로토콜을 가진다.
이 함수는 Event가 전달 될때마다 실행되며, 실행될때마다 VC가 가지고 있는 Model에게 로직을 넘겨준다.
~~~swift
protocol ViewBinding {
    func inputViewEvent(_ target: ViewBindable, _ param: Any?)
}
~~~

VC는 `ViewModel`을 가지고 있고 binding되어 있다.
target(View)에게 새로 들어온 data를 recevie하라고 한다.
~~~swift
vm = LoginViewModel { data, target in
    if let data = data {
        target.receive(data)
    }
}
~~~
`inputViewEvent`이벤트가 일어 날때마다 binding된 vm에게 `request`(로직처리해~)를 보낸다
~~~swift
 func inputViewEvent(_ target: ViewBindable, _ param: Any?) {
        self.vm?.request(target, param: param)
 }
~~~

Model에서는 reqeust가 들어올때마다 data(param)와 함께 `output이라는 클로저를 실행`한다.
실행된 클로저는 이 ViewModel과 `binding된 VC가 클로저를 실행`하게 한다.
클로저안에는 VC가 가지고 있는 View가 data를 `receive하는 함수가 실행`된다.
결국, `View가 output을 받게된다.`
~~~swift
final class LoginViewModel: CommonViewModel {
    var output: (Any?, ViewBindable) -> Void
    
    init(_ output: @escaping (Any?, ViewBindable) -> Void) {
        self.output = output
    }
    
    func request(_ bindable: ViewBindable, param: Any?) {
        self.output(param, bindable)
        print("request!")
    }
}
~~~

### 기존 구조(delegate)와 비교

기존에 Delegate를 사용했을 때와 비교를 한다면 아마 이런 그림이지 않을까? 생각이든다.  
View와 VC사이의 소통이 모두 `protocol에 정의된 함수로 추상화` 되었다.

![](https://velog.velcdn.com/images/piggy_seob/post/337a0bc5-5ff5-4279-8774-c75afabf551e/image.png)

### 느낀점 및 배운점
- 이번 구조는 내 생각에 delegate와 Bind를 섞은 구조?에 가깝다고 생각이 들었다.   
setVC 부분은 delegate패턴과 비슷한것 같고 VM과 연결하는 부분은 binding이라 생각했기 때문이다.
- View와 VC간의 연결을 Protocol로 추상화하는 것이 신기했고 아직까지 Test코드를 짜지는 않았지만 훨씬 더 용이 할 것으로 생각이 된다.
- 다른사람이 짠 구조를 파악하기는 쉽지 않다. 특히 이렇게 추상화와 Binding등이 있다면..

## [20.06.17] 한 주 회고

- 초반
이번 주는 시작전부터 뭔가 의미가 부여되는 느낌으로 시작되었다.
코드스쿼드 마지막 프로젝트가 시작되는 주 이기도 하면서, 정말로 이제 얼마 안남았다고 생각하니 괜히 더 열심히 해야할 것 같았기 때문이다. 
JK가 조급해 하지 말라고 하셨지만, 나를 누구보다 잘 아는 사람은 나다. 
나는 조급함을 가지지 않으면 완전 해이해질게 뻔하다.

- 중반
초반의 그 느낌을 그대로 정말 코드스쿼드에 처음 들어왔을때의 열쩡으로 아침에 조금 더 일어나고 잘때 조금 더 늦게 잤다. 몸은 피곤했지만, **뭔가 하고있다** 라는 느낌이 좋았다.
하지만, 이 짓을 매일은 못하겠다는 생각이 들었던게 수요일 밤부터였다.
아침에 눈이 너무 안떠졌기 때문이다. 

- 현재(금) ~ 주말
쉬고 싶다는 생각이 간절해 졌다.
평일에 열심히 한 만큼 주말에 정말 잘 쉬지 않으면 다음주에는 지칠것 같다는 생각이 든다.
주말에는 쉬엄쉬엄하면서 다음 주 평일을 위한 휴식을 취해야 겠다.


## [22.06.20] 설계 vs 구현에 대한 생각
### 설계? 구현?
내 경험상, 어떤 프로젝트를 시작할 때 `항상 고민이 되는 부분`이 있다.
프로젝트의 설계와 구현의 `비율`이다.
여기서 말하는 `설계`는 MVC, MVVM 과 같은 디자인 패턴 뿐만 아니라, 팀(2인이상이라면)의 그라운드 룰, 목표기간 등등.. `실제로 제품의 코드를 만드는 것 외의 부분`이다.
반면에, `구현`은 실제로 제품의 기능을 만드는 부분이다.

### 올바른 비율
`설계`가 없으면 프로젝트의 유지 및 보수가 힘들어 질 것이고, `구현`이 없다면 설계는 그냥 계획일 뿐일 것 이다.
설계와 구현을 동시에 잘 할 수 있다면 정말 좋겠지만 그러기는 정말 쉽지않다.
(천재들은 가능하겠지만 나는 그랬다.)
우리의 시간은 한정적이기 때문에 `어느 정도의 설계를 하고 시작`을 할지는 항상 큰 고민이었다.
특히나, 팀으로하는 경우에는 모든 이해관계자들과 얘기를 해야하기 때문에 기능은 하나도 개발하지 않았는데 시간이 훅 가는 경험을 할 수도 있다.

그래서, 구현과 설계의 `올바른 비율`이 필요하다.

### 나의 생각
~~~
항상 정답은 없죠 -🧔🏻‍-
~~~
정답이 딱 정해진 것은 아니지만, 공감이 가는 말이 있다.
`Quick & Dirty`라는 말이다.
일단은 Quick 하지만 Dirty 하게 만든 다음에, 조정을 해나가는 것이다.
회사에서 일을 했거나 하는 경험은 없지만, 현재까지 경험으로 비루어 봤을때 조금씩 마음이 가는 단어이다.
왜냐하면, 우리가 하는 프로젝트도 결국에는 무엇인가 `제품을 만들기 위한 과정`이다.
제품이 나오지 않으면 `설계는 의미가 없다`. 는게 현재까지 나의 생각이다.
실제 현업에서는 어떻게 이 비율을 맞추고 있을지 궁금하다.

## [22.06.21] 민감정보 관리하기
### 민감한 정보?
이제까지 프로젝트를 하면서는 딱히 보안에 신경 쓸 일이 많이 없었다.
하지만, OAuth를 써 먹어야하는 이번 미션에서 사용자의 개인정보를 보호하는 것은 매우 중요해졌다.
당연히 사용자의 정보를 가져올 수 있는 민감한 API Key값들은 매우 민감한 정보가 된다.
특히, github같이 Open된 사이트에는 절대로 이런 Key값들이 올라가면 안될 것이다.

### 관리법
그래서 어케하누??
키 관리가 어려운 이유는 쩃든 우리는 이 key값을 사용해야 한다는 것이다.
그렇게 때문에 관리법이란, 결국 이 `Key를 쓰긴 쓸건데 어디에 어떻게 두고 공유할거니?` 라는 말과 같다고 생각한다.

이를 관리하기 위한 방법에는 여러가지가 있겠지만.. 내가 조사한 것은 한 4가지쯤 된다.
(실제로 해본건 2개정도지만..)
~~~
기본 전제는 gitignore로 key값은 git에 추적이 안되게 하고 시작한다.
~~~
1. **Key가 들어있는 파일을 팀원들끼리 Key가 들어있는 파일을 공유하고 상대경로에 넣는다.**
    - 가장 심플한 방법이지만, 민감한 정보를 가진 파일을 주고받는다는 것 자체가 딱히 보안적으로 좋아보이지는 않는다는 생각이 든다.
    <br>
2. **파일을 주고 받지않고, Plist를 하나더 만들어서 Key값들 추가 Bundle을 Extension해서 사용한다.[참고](https://nareunhagae.tistory.com/44)**
    - Key값들이 좀 많을 때 Plist의 이름으로 어떤 Key값인지 유추 할 수 있다는 장점이 있는 것 같다. 
    <br>
3. **xcconfig파일에 Key값을 저장, plist를 이용 사용한다.**[참고](https://ios-development.tistory.com/660)
    - Plist에 직접적으로 Key값을 적지 않아도 되서, 위 방식과 함께쓰면 한번더 꼬을 수 있어서 좋은 것 같다.
    - xcconfig는 Key값을 숨기는것 외에도 프로젝트가 커졌을때 여러 프로젝트의 환경값을 관리할때도 사용한다고 한다.
    <br>
4. **코드 생성 도구를 이용, key를 암호화해서 관리한다[참고](https://nshipster.co.kr/secrets/)**
    - 이 방식은 특이하게 gitignore도 하지않고, Key값을 말그대로 암호화 해서 관리한다.
    - 참고 사이트에서 아직까지 이해가 잘 되지않음..
    
조사 결과를 개인적인 견해를 붙여서 정리해봤다. (⚠️틀릴수도 있음 주의!⚠️)
아직까지 하나씩 해보지 않아서 참고사이트를 확인하면서 비교해야겠다.(언젠간..하겠지..)

## [22.06.22] 오늘의 git 삽질
Beck과 상의끝에 민감 정보를 관리하기 위한 방법중 plist를 사용하는 방식을 택하기로 했다.
![](https://velog.velcdn.com/images/piggy_seob/post/a746da05-ed0c-4428-aedd-f32e97852026/image.png)

Plist는 위와같이 구성이 되어있고 Key값에 맞는 Value를 넣어주면 된다.
이 과정에서 우리는 git repo에는 Value가 들어가 있지 않은 상태로 push를 하고 싶었다.
그래야 매번 파일을 생성안해도 되고, 좀더 명확하게 필요한 값들을 바로 알수 있다고 생각했기 때문이다.
근데 gitignore를 설정을 해버리면 plist파일 자체가 repo에 추가가 안되기 때문에,
이를 처리할 방법을 생각해야 했다.
그래서 떠올린 방법이 이거였다.

~~~
아. 일단 gitignore를 하기 전에 value가 없는  plist를 push하고, 
그다음에 gitignore를 하면 될거야~ 그럼 로컬에서 셋팅한 값들은 추적이 안될것이고 push도 안되겠지.
~~~

하지만 이 방법은 Beck과 같이 해본 결과 plist를 repo에 올리는 것까지는 성공했으나 그 뒤에 Local에서 셋팅한 값들을 자꾸 git이 추적 하는 문제를 발생시켰다.(gitignore에 추가했음에도..)
그래서 구글링을 해보니 gitignore가 안먹을때에는 .. `Cache를 지우면 된다고 한다.`

설레는 맘으로 Cache를 지우는 명령어를 수행한 뒤에 확인을 해보니 Local에서 더이상 plist를 추적하지 않았다.
기쁜 맘으로 다른 작업 후 push를 했으나... 이번에는 `repo에 잘 들어가있던 plist가 사라져버렸다.`
한참을 해메다가, git명령어를 친 뒤 log를 확인해보니.. `delete mode`라는게 찍혀있었다.
![](https://velog.velcdn.com/images/piggy_seob/post/8b3eb136-75f6-499c-a61c-5f9f80eee9b6/image.png)

이를 단서로 구글링을 해보니.. Cache를 제거하는 명령어를 활용하면, git repo에 올린 파일 or 폴더를 지울 수도 있다는 것이다..! 
![](https://velog.velcdn.com/images/piggy_seob/post/8ec51ba2-d95a-4eed-9c4c-320802717c4b/image.png) [참고블로그](https://mygumi.tistory.com/103)
나는 그저 Cache된 파일을 지는것이기 때문에 아무런 관계가 없는줄 알았는데..


결국, gitignore를 정상 작동하게 하려고 Cache를 무분별하게 지웠던 것이 독이되었다.
해결하는 과정에서 진이 빠져버려 지금은 그냥 repo에 plist를 안올려놨다.
이를 해결하기 위해서는 다른 방법으로 접근해야겠다.(내일 ㅎㅎ..)

### 배운점
구글은 `신`이다. 모든 답을 다 알고있다.
하지만, 그 답들은 하나하나 `독립적`이다.
따라서, 이해를 하지 못하고 답을 구하면 결국 이렇게 삽질을 하게된다..

## [22.06.23] AutoLayout 다시보기.

### AutoLayout
AutoLayout이란 상당히 편한 기능이다.
다양한 크기의 스마트폰에 우리가 원하는 UI를 적절하게 보여줄 수 있다.
하지만, 이런 AutoLayout을 잘 못 사용하면 오히려 UI가 이상하게 보일 수 있다.

### 문제상황
사실 이번 플젝을 진행하면서 맞닥드린 문제였다.

> 대충.. title Label에 맞춰서 얼만큼 떨어지게 만든 AutoLayout..

~~~swift
titleLabel.snp.makeConstraints {
    let insetValue = 120.0
    $0.top.equalToSuperview().inset(insetValue)
    $0.centerX.equalToSuperview()
}

userInfoInputStackView.snp.makeConstraints {
    $0.top.equalTo(titleLabel.snp.bottom).offset(72.0)
    $0.leading.trailing.equalToSuperview()
}
~~~

아이폰 8(내가 생각하기에 요즘볼수 있는 가장 작은 스크린)로 시뮬레이터를 돌리니 `Issue Tracker title이 잘려서 보임`을 알 수 있었다. 
왜냐하면, 내가 아무생각없이 하드코딩한 120.0과 같은 숫자들이 작은 스크린에서는 매우 큰 숫자가 될 수 있고 `설정해준 layout을 처리할수 없을만큼 큰 숫자인 경우`  AutoLayout들을 적절히 처리되지 못하고 이렇게 이상한 UI가 나온다.

![](https://velog.velcdn.com/images/piggy_seob/post/055e5046-8c16-42ed-a62f-ea9aed29af1d/image.png)

특히나, `하나하나 Layout이 맞물려 있는 상황`에서는 지금처럼 안좋은 UI를 만들어낼 수 있다.

### 내가 생각한 해결법
물론 내가 AutoLayout을 잘못 사용한 것 일 수도 있지만, 위와같은 문제를 해결하기 위한 방법으로는 `현재 Screen의 크기를 가지고와야만 해결 할 수있다고 생각했다.`

1. UIScreen.main.bound로 현재 스크린의 크기를 가져온다.
2. 스크린에서 UI가 위치해야할 비율을 구한다.(다행히 우리는 기획서가 있다.)
3. 스크린의 크기에서 비율을 곱한 곳에 UI를 위치한다.

~~~swift
// 비율 from 기획서
private struct LayoutRatio {
static let titleTopInsetRatio: CGFloat = 165 / 812
static let userInfoInputStackViewInsetRatio: CGFloat = 312 / 812

static let loginButtonTopInsetRatio: CGFloat = 430 / 812
static let loginButtonLeadingInsetRatio: CGFloat = 96 / 375

...

}
// 스크린 크기
let screenHeight = UIScreen.main.bounds.height
let screenWeight = UIScreen.main.bounds.width

// AutoLayout적용
titleLabel.snp.makeConstraints {
    $0.top.equalToSuperview()
        .inset(screenHeight * LayoutRatio.titleTopInsetRatio)
    $0.centerX.equalToSuperview()
}
        
userInfoInputStackView.snp.makeConstraints {
  $0.top.equalToSuperview()
        .inset(screenHeight * LayoutRatio.userInfoInputStackViewInsetRatio)
  $0.leading.trailing.equalToSuperview()
}
~~~

![](https://velog.velcdn.com/images/piggy_seob/post/fa968b70-0b70-4d85-b189-e5ad112d9656/image.png)

이제 잘나온다!
물론 View안에 Label의 크기나 버튼안의 이미지크기도 비율을 설정해야 완벽하겠지만..

### 배운점
이제까지 그냥 당연하게 생각했던 Autolayout도 `잘` 하려면 쉽지 않다는걸 깨달았다.
안다고 생각하는 것도 조금더 주의깊게 한번더 보자.

## [22.06.24] 주간 회고

### 주간회고
1. 서울에 왔다.
원래는 수료식날에만 서울에 올라오려고 했는데 장관님 찬스로 숙소가 생기는 바람에 일찍 오게 되었다.서울에 온게 별로 실감은 되지 않는다(코딩한다고 숙소에 박혀있으니..) 심지어 여기는 동대문 근처인데 창밖을 바라보니, 몇몇 건물뺴고는 김해시내보다도 골목길이 무섭게 생겼다(진지.) 좋은 점도 있었다. 서울사람들은 다 냉정하고 깍쟁이같은 인식이 있었는데, 일면식도 없는 집주인(처음보는 서울사람)이 비오는날 우산이 없는 나를 보며 먼저 말을 걸어주셨고,우산을 흔쾌히 빌려주셨다. 서울사람들도 `케.바.케 || 사.바.사`인가 보다.

2. 손목이 아프다.
예전부터 손목에 통증이 있었는데, 요새들어 왼쪽손목이 정말 시~큼하게 아프다.
검색을 해도 뭐 평소에 조심하는것 밖에는 답이 없는거 같은데.. 더 심해지면 병원에 가봐야겟다.

3. 집중이 잘 안된다.
분명 저번주 까지만 해도 집중할거라고 불태웠는데, 수료가 다가오니 오히려 붕뜨는 느낌이다.그래도 마무리는 아름답고 깨끗히 맺어보려고 노력한다.

## [22.06.27] DIContainer의 Flow?

### DIContainer?
DIContainer 어디선가 들어본 그녀석을 이번에 사용해보기로 했다..
내가 이해한 DIContainer의 사용 이유는 이렇다.
우리가 Protocol로 의존성을 역전 시키고, 구체 타입을 밖에서 주입해줌으로써 의존성 주입을 해준다고 해도 어디선가는 구체타입을 알고 있어야하며, 생성 및 주입 해야한다.
아무리 추상화를 한다고 해도 말이다.
그렇기 떄문에 어짜피 구체타입을 어딘가 알아야한다면, DIContainer라는 것을 만들어서 그곳에 구체타입에 대한 의존성과 생성로직을 몽땅 넣어버리고 관리하자.라고 생각했고,DIConatiner가 작동하는 Flow는 다음과 같다고 생각했다.

### DIContainer의 Flow
내가 잘 못 만든것일 수도 있지만, 대부분의 예제에서 DIContainer가 작동하는 Flow는 이랬다.

등록
~~~swift
// UseCase 등록
func registUseCaseDependencies() {
    UseCaseContainer.shard.regist(instance: GithubLoginUseCase(model: GitHubLoginModel()))
    UseCaseContainer.shard.regist(instance: AppleLoginUseCase(model: AppleLoginModel()))
}
~~~

사용
~~~swift
private var githubLoginUseCase = UseCaseContainer.shard.resolve(type: GithubLoginUseCase.self)
private var appleLoginUseCase = UseCaseContainer.shard.resolve(type: GithubLoginUseCase.self)
~~~

UseCase는 Model들을 인자로 받아서 init된다.
init이 될때에는 구체적인 Model타입을 알아야하니 이 로직을 DIContainer를 이용해 분리한다.
특히, DIContainer는 전역적으로 사용되는 경우가 많기 때문에 shared라는 싱글톤을 사용하곤 하는데, 이것이 Flow에서 내가 생각하지 못했던 한 단계를 더 만들어 버렸따.

### 리뷰어님의 피드백과 문제점.
발단은 리뷰어님의 피드백에서 시작되었다..

>그리고 한 가지 궁금한게, AppDelegate에서 register하고 이후의 플로우에서 따로 GithubLoginUseCase와 AppleLoginUseCase를 메모리에서 해제시키는 코드는 찾지 못했는데요. 그렇게 되면 LoginUseCase는 앱을 사용하는 내내 싱글톤으로 메모리에 살아있는게 되는 걸텐데 이는 비효율적이라서, 혹시 처리하고 계신 곳이 없다면 처리가 필요할 것 같습니다.

메모리...요?
DIContainer는 앱 실행할때 몽땅 regist하고 쓰는게 아니었나...?


## [22.06.28] DIContainer의 메모리 문제를 해결해보자.

### 현재 구조의 문제점.
현재 나의 DIContainer의 문제점은 앱을 실행할때 모두 구체타입을 `생성`해서 Dictionary에 넣어두는 것이다.
생성되어 regist된 구체타입들은 싱글톤에서 영원히 살 것이고, 이는 당연히 메모리 측면에서 비효율적이게 된다.

### 혼돈의 카오스
이걸 어떻게 해결해야하나??
사용한 후에는 값을 없애야하나??
그러면 없앤 다음에 다시 쓰고싶을때는 어떻게 regist하지??
지금은 AppDelegate에서 regist하는데.. 그러면 AppDelegate에서 regist를 하면 안되나?
그러면 결국 생성로직을 분리하겠다는 초기 취지랑 안맞게 되는데??
머릿속에 혼돈이 가득한 가운데 ..
~~~
구체 객체가 아닌 구체 객체를 return 하는 factory closure를 넣어서 register 해요. -Eddy God-
~~~

### 클로저를 활용하자!
**갓 에디 가라사대 클로저를 활용하라 하심에 죄인은 코드에 적용해보기 시작한다.**

Dictionary에 넣던 Value값을 Value값을 생성하는 클로저로 바꾸어 보았다.
이제 Value값에 `생성`이 된 타입들이 없기 때문에 메모리상의 문제를 제거했다.

~~~swift
 typealias Creator = () -> Value
 ...
 var map: [ObjectIdentifier: Creator] = [:]
~~~

Regist
생성된 구체타입을 따르는 프로토콜을 리턴하는 Closure를 regist한다.
~~~swift
mutating func regist<T: UseCaseResponsible>(type: T.Type, make: @escaping Creator) {
    let identifier = ObjectIdentifier(type)
    map[identifier] = make
}
~~~

Resolve
Regist된 Creator 클로저를 실행해서 프로토콜을 따르는 객체를 생성.
~~~swift
func resolve<T: UseCaseResponsible>(type: T.Type) -> Value? {
  let identifier = ObjectIdentifier(type)
  guard let useCaseCreator = map[identifier] else { return nil }
  return useCaseCreator()
}
~~~

실제구현부
Regist: Type을 키값으로 구체타입을 리턴하는 Closure를 regist
~~~swift
func registUseCaseDependencies() {
    UseCaseContainer.shared.regist(type: GithubLoginUseCase.self) {
        return GithubLoginUseCase(model: GitHubLoginModel())
    }
~~~

Resolve: Type을 키값으로 Creator를 실행한 값 가져오기 및 사용
~~~swift
private var githubLoginUseCaseCreator = UseCaseContainer.shared.resolve(type: GithubLoginUseCase.self)
~~~

~~~swift
func request(_ bindable: ViewBindable, param: Any?) {
  guard let loginType = param as? LoginType else { return }

  switch loginType {
  case .gitHub:
      guard let useCase = githubLoginUseCase else { return }
      useCase.request { loginURL in
          self.output(loginURL, bindable)
      }
~~~

### 배운점
- DIContainer에 대해 조금 더 알게 됬다.
- 역시나 로직이 좀더 복잡해져서 안그래도 보기 힘들었던 DI였는데.. 더 복잡해진기분이다.
- 거마워요 에디.

## [22.06.29] Protocol Extension 이용하기

### Protocol Extension
당연하게도 Struct나 Class 처럼, Protocol도 Extension이 가능하다.
다만, `Protocol의 Extension은 Struct나 Class와는 조금 다르다.`

예제를 통해 살펴보자.
여기 변수와 메서드를 하나씩 가진 Protocol이 있고 이를 따르는 Class A가 있다.
~~~swift
protocol SomeProtocol {
    var a: String { get }
    func printSomthing()
}

class A: SomeProtocol {  //Type 'A' does not conform to protocol 'SomeProtocol'
    
}
~~~
여기서 SomeProtocol에 선언한 변수나 함수를 구현하지 않는다면 컴파일러는 protocol을 준수해달라고 징징거리기 시작할것이다.

하지만, 프로토콜의 Extension에 함수와 변수를 선언한다면 에러가 나지않는다.

Protocol에는 추상화된 값을 넣지만, Protocol의 Extension에는 실제 구현부를 넣어야 하고 이는 마치 `Class를 상속받은거 처럼`(물론 완벽히 같진 않다.)
해당 Protocol을 채택하는 객체에게 Extension의 기능을 가지게 한다.
~~~swift
protocol SomeProtocol {
//    var a: String { get }
//    func printSomthing()
}

extension SomeProtocol {
    var a: String {
        return "2022 iOS Members 🔥"
    }
    func printSomething() {
        print("Fighting!")
    }
}

class A: SomeProtocol { // No compile Error
    
}
~~~

### 사용 예시
Protocol의 Extension을 활용하는 방법은 무궁무진해서 다 적기 힘들지만, 내가 느끼기에 좀더 활용성이 높아 보이는 것은 `associatedtype` 활용할 때이다.

예를 들어서 SomeProtocol에 `associatedtype`이 있는데 이 타입에 따라 protocol을 채택하는 객체들의 행동을 다르게 주고싶다. 고 한다면..

~~~swift
import Foundation
protocol SomeProtocol {
    associatedtype Value
}

extension SomeProtocol where Value == String {
    var a: String {
        return "2022 iOS Members 🔥"
    }
    func printSomething() {
        print("\(a) Fighting!")
    }
}

extension SomeProtocol where Value == Int {
    var a: String {
        return "2022 iOS Members 🔥"
    }
    func printSomething() {
        print("\(a) Forever!")
    }
}

class A: SomeProtocol {
    typealias Value = String
}

class B: SomeProtocol {
    typealias Value = Int
}

let a = A()
let b = B()

a.printSomething() // 2022 iOS Member 🔥 Fighting!
b.printSomething() // 2022 iOS Member 🔥 Forever!

~~~

- 장점?
지금은 아주 단순한 예제이기 때문에 그냥 구현 코드만 구체타입에서 Protocol로 옮긴것 처럼 보이지만 만약, 구체타입안에 제네릭이 있는 좀 복잡한 구조라면 associatedType과 함꼐 코드의 효율성을 높일 수 있을 것 같다.


## [22.07.05] RxSwift 시작하기.

그놈이다.
채용공고를 눈팅할 때마다 우대사항으로 있던놈.
이제는 진짜 배울때가 된 것 같아 예전에 사놓고 안 본 듀토리얼을 제 나름대로 해석해서 써봤습니다.

### RxSwift란?
Rx + Swift가 합쳐진 말이다.
Swift는 알겠고.. Rx는?
Rx는 다시 Reactive Extension의 줄임말인데, `관찰 가능한 시퀸스`를 사용하여 `비동기식 및 이벤트 기반 프로그램`을 구성하기 위한 라이브러리라고 한다.
관찰 가능한 시퀸스? 비동기식 및 이벤트 기반? 🤯 뜻을 생각해 보면서 구성요소를 살펴보자.

### RxSwift의 구성요소
- Observable - Rx의 기반
위 에서 Rx의 정의 중에 `관찰 가능한` 시퀸스라고 하였다.
이를 대충 영어로 번역하면 Observable일 것이기 때문에 이러한 이름이 지어지지 않았나 싶다.
`시퀸스란`, 개개인의 타입을 하나씩 순회할 수 있는 타입으로 Swift의 Array와 흡사하다.
따라서, `Observable이란 관찰 가능한 배열`의 형태를 띈 무엇인가구나~ 라고 생각했다.
기본 형태는 이런 모양이며 다음과 같은 기능을 한다.
  ~~~swift
  Observable<T>
  ~~~
1. Generic T형태의 데이터 SnapShot을 전달 할 수 있는 일련의 이벤트를 `비동기적으로 생성`
2. 3가지 형태의 Event를 생성한다.
    ~~~swift
    enum Event<Element> {
        case next(Element)            // 시퀸스의 다음 element
        case error(Swift.Error)        // 시퀸스가 실패했을때
        case completed                // 시퀸스가 성공적으로 끝났을때
    }
    ~~~
3. 관찰가능한(observable) Event를 생성하면 관찰자(observer)가 보는 방식대로 진행 되게 된다.

- Operator - 연산자
    말 그대로 연산자이다.
    좀더 복잡한 연산을 도와주어 Element들을 가공하는데 도움을 준다.
    마치 +, -, * 등을 조합해서 사칙연산을 하듯이.
    Observable이 Swift의 Array와 같은 시퀸스이기 때문에, map, filter 등과 같이 Array에 주로 쓰던 연산자들이 대표적이다.

- Schedular ≒ GCD?
    DispatchQueue와 비슷한 역할을 하지만, 훨씬 쓰기 쉽게 만들어 놨다고 함.
    이미 많은 Schedular를 잘 만들어 놔서 개발자가 직접 Schedular를 생성할 일은 잘 없다.
    자주쓰는 DispatchQueue구문들 예를 들어 DispatchQueue.main.aysnc 와 같은 것들을 미리 정의해 놓은게 아닌가 싶다.
![](https://velog.velcdn.com/images/piggy_seob/post/e38d9234-ab02-4e47-b149-e637ce389524/image.png) 
[이미지 출처](https://www.raywenderlich.com/13285844-rxswift-reactive-programming-with-swift-update-now-available)


## [22.07.06] RxSwift(trait)

### Trait
Rxswift의 Observable중에는 가독성을 위해서 좀더 좁은 범위의 Observable이 있다.
이런 Observable들을 trait라고 한다.
Single, Maybe, Completable이 바로 trait다.
기존의 ObserVable에 asSingle(), asMaybe() 등의 메서드를 사용하면 타입을 바꿀수도 있다. (단, Competeable은 안됨.)

### Single
Single은 Success 혹은 Error 이벤트를 방출하는 Observable이다.
여기서 Success는 Observable의 OnNext와 Completed를 합친 것과 같다.
`정확히 한 가지 요소만 방출할 때 유리함.`
Ex) 디크스에서 파일 가져오기와 같이 작업이 Success냐 Error냐 나누어지는 작업을 할때
- 주의
single을 subscribe를 통해 이벤트가 두 번이상 방출되는지 확인하면 Error가 날 수 있음.

### Maybe
Single과 비슷하지만 아무런 값도 방출하지 않는 Completed 이벤트가 추가되어 있다.
`그럼 Single만 쓰면 되지않나..??`
예를 들어 사진을 가지고 있는 포토앨범이 있다고 쳐보자.
그곳에서 만든 앨범명은 UserDefault에 저장이 된다.
해당 앨범명으로 앨범을 열고 작업을 하게 될 것이다.
이 때, 앨범명이 있다면 아무런 값을 방출하지 않는 Completed를 방출하면 될것이고,
앨범명이 UserDefault에 없다면 새로운 값과 함께 Success를 방출하면 될 것이다.
이렇게 해당 이벤트에 값을 방출 할 수도 있고, 안하고 성공만 표시할수 있기 때문에(그래서 maybe인가?) 흐름을 제어 할 수 있다고 한다.

### Completable
Completed 혹은 Error만을 방출한다.
따라서, 어떠한 값을 방출하지는 않는다.
`값을 방출하지 않는데 쓸대가 있나..?`
단순히, 작업의 성공여부를 판단하여 일의 순서를 정할때 사용하기 좋다고 한다.
예를 들어 background에서 작업이 완료가 되면 자동저장이 되는 기능이 있다고 해보자.
이 때는 어떠한 값도 방출이 될 필요는 없다.
단순히 저장이 되었으면 되었으면 저장이 된 노티를 띄우고, 뭔가 에러가 났다면 에러 노티를 띄우면 된다.

- 주의
Completed는 생성을 하고 싶다면 create 메서드를 이용해서 생성할 수 밖에 없다.

[220707] DP(Dynamic Programming)

### Dynamic Programming(동적계획법)
동적 계획법이란.. 큰문제를 작은 문제로 나누어 해결하는 알고리즘 중에 하나이며,
`하나의 문제를 단 한번만 풀도록 해서 효율성을 높이는 알고리즘`이다.
처음에 봤을때 답도 없던 이녀석을 이해하기 위해 피보나치의 수 문제를 통해, 조금이나마 이해하려 시도해봤다.

### 피보나치의 수
피보나치의 수는 정의보다는 숫자를 보면 이해하기 쉽다.
먼저 숫자 0 과 1을 준다. [0, 1]
그리고 다음 숫자는 0 과 1을 더한 숫자 1을 넣는다. [0, 1, 1]
다음 숫자는 1과 1을 더한다 [0, 1, 1, 2] 
...
이런 식으로 N번째 피보나치 숫자는 N-2 + N-1 번째 숫자를 더한 값으로 생성되는 배열이다. 

### 피보나치의 수 구하는 방법
N번째 피보나치 숫자를 구하고 싶다고 했을 때 구할 수 있는 방법은 3가지가 있다.
1. 재귀
~~~swift
 //recursive
func solution(_ n: Int) -> Int {  
    if n == 0 || n == 1 {        // 이 값이 나올때 까지 계속 재귀를 돌것임.
        return n
    }
    let n = solution(n-1) + (solution(n-2)) // 재귀도는 부분
    return n
}
~~~
위 방식은 단순하지만?(처음엔 이것도 어려웠다) 시간복잡도가 N의 제곱이라는 아주 안좋은 수치를 가지고 있다.

2. recursive with DP
~~~swift
 //recursive Dynamic(top down)             // Call stack Depth Error 주의
var fiboArray = [0, 1]
func solution(_ n: Int) -> Int {
    if n == 1 || n == 0 {
        return n
    }

    if n < fiboArray.count {    // 이 구문이 실행이 된다는 것은 fiboArray에 0,1 활용할 값이 있다는 뜻
        return fiboArray[n]
    }

    let n = solution(n - 1) + solution(n - 2)
    fiboArray.append(n)            // 새로운 값이 있다면 추가
    return n
}
~~~
재귀함수에 DP를 섞은 모습이다.
재귀와 매우 흡사하나 fiboArray라는 배열이 생겼고 배열에서 값을 리턴하는 것을 볼 수 있다.
`DP는 한번 계산한 것을 다시 계산 하지 않음으로써 효율을 높이는 알고리즘`이라고 했다.
fiboArray에 결과값들을 저장하고 그 값들을 활용해서 시간복잡도를 낮추는 것이다.
이렇게 fiboArray같은 곳에 값을 저장하는 것을 `메모제이션`이라고도 한다더라~
이를 통해 시간복잡도는 O(N)까지 내려간다.
큰값부터 구해서 내려오기 떄문에 top down 이라고도 하며, 이로인해 Call stack Error가 날수있다.


3. for loop with DP
~~~swift
// Dynamic with for loop(bottom up)
func solution(_ n: Int) -> Int {
    if n == 1 || n == 0 {
        return n
    }
    var fiboArray:[Int] = [0, 1]

    for i in 2...n {
        fiboArray.append((fiboArray[i-1] + fiboArray[i-2]))
    }

    return fiboArray[n]
}
~~~
재귀를 사용하지 않고 구현하는 방법이다.
재귀와 함께 쓰는 DP와 다르게 작은 값부터 구해나간다는 것이 특징이다.

### 그래서 둘중에 누가 더 좋은데(빠른데)?
이는 가르치는 사람마다 조금씩 달랐다.
누구는 bottom up이 좋다고하고~ (왜냐하면 재귀를 돌때 Stack할당 이라든지 부가 작업들이 많아서) 누구는 정답은 없고 상황마다 다르다고 하시더라.

사실 아직까지 문제에 잘 적용할 수 있을 자신은 없다.
문제를 많이 풀어보는 수밖에..


## [220707] 코테포기자였던 사람이 일주일 공부해보고 느낀점.

### 수학
내가 개발자가 되기 전에는 약간의 선입견이 있었다.
그 중 하나가 `개발자는 수학을 잘해야한다.` 였다.
그래서 개발자의 진로를 결정하기 전에도 상당한 고민이 있었다.
하지만, 막상 시작을 해보니 내가 걱정한 만큼 수학적 지식을 요구하지는 않았다.
물론 알면 좋겠지만, 그렇다고 해서 수학을 다시 공부를 해야겠다고 느껴진 적은 없었다.
하지만, `알고리즘은 달랐다.`
특히, 코딩테스트와 같이 시간이 정해져있는 경우 `수학적 지식을 얼마나 잘 정리하고 있느냐`가 해결시간에 많은 영향을 끼침을 확인 할 수 있었다.
한 예로 최근에 최소공배수와 최대공약수를 구하는 문제를 푼적이 있는데, 공식을 아는 사람과 모르는 사람(나)의 코드 길이나 가독성은 매우 차이가 났다.
심지어 `공식을 모르니 다른 사람의 코드를 이해하기도 어려웠다.`

### 이론보단 실전
다이나믹 프로그래밍이 어쩌구 DFS가 어쩌구 이론을 설명해줄때는 이제 뭐든 적용할 수 있을것만 같았다.
하지만, 코테를 내는 `출제자들은 대놓고 어떤 기법을 사용하라고 말해주지 않는다.`
인형뽑기가 어쩌구 ~ 미로가 어쩌구~ 식으로 최대한 숨기려 애쓴다.
그렇기 때문에 이론을 안다고 되는게 아니라 많은 문제를 풀어봐야한다.
어떻게 보면, `프로그래밍이란 세상의 문제를 해결하기 위해 있는 것`이니까 당연한 것 같다.

### 모르면 답을 봐라
~~~
나는 하루종일 잡았는데 이게 쉽다고? -> 코테우짜노.. -> 응 코테 안보면 그만이야~
~~~
내가 코테를 포기했었던 결정적 이유 중 하나였는데, 하루종일 문제를 잡고 있어도 답이 안나오는데 막상 다른 사람의 풀이를 봤더니 엄청 간단한 문제 였을때의 허탈함 때문이었다.
`이 시간에 다른 공부를 했으면 뭐를 해도 더했겠다~` 라는 생각이 들때 정말 허탈했다.
지금은 시간을 정해놓고 그래도 안풀리면 그냥 풀이를 본다.(풀이봐도 모르겠는건 비밀)
기업들은 수많은 지원자들을 거르는 허들로 코딩테스트를 우리에게 제공한다.
허들만 잘 넘으면 됬지, 어떤 포즈로 허들을 넘을 지를 고민하는 것은 미련하다고 생각이 된다.

## [220711] 내가 쓴글 다시보기: 메모리와 Array

이번에 어떻게 운좋게 면접기회가 생겼다.
이력서에 내 블로그를 기입해뒀는데, 옛날 글을 보니 기억이 하나도 기억이 나지않았다.
분명 내가쓴글인데.. 질문에 대답을 못하면 낭패니 다시 살펴보자.

### Swift에서 메모리와 Array
Apple개발 문서를 기준으로 Array를 어떻게 설명했는지 메모리와 관련지어서 살펴봤다.

~~~swift
@frozen struct Array<Element>
~~~
frozen Struct로 Array가 구성이 되어있다.
여기서 @frozen이란 라이브러리를 만들때 주로 사용하는 것으로 최적화를 위해 사용하는 것으로 구조체나 열거형에 사용이된다고 한다.
이렇게 frozen이 붙은 녀석들은 열거형의 케이스 또는 구조체의 저장된 인스터스 프로퍼티를 추가, 제거, 또는 재정렬로 선언을 변경 할 수 없다.

(extension으로 연산프로퍼티를 추가하는 것은 가능해보인다.)

### Array의 메모리 크기
Swift의 Array는 안의 내용물을 보관하기 위해 특정 양의 메모리를 예약한다.
예약된 메모리를 초과하게 된다면 더 큰 영역 할당 및 복사(Array는 Struct니까)를 하게 되는데, 이때 할당되는 크기는 이전 Array의 크기의 배수로 커진다.

엥?? 딱맞는 사이즈를 할당하면 되지, 왜 두배나 되는값을 할당하지? 낭비가 심한데??
라고 생각할 수 있지만, 우리가 메모리를 할당할 때는 두가지를 생각해야한다.
1. 메모리 할당이 실행되는 빈도
2. 메모리 할당이 되는 크기

예를 들어서 5의크기를 가진 배열이 하나의 요소가 더 추가가 되서 6이라는 공간이 필요했다고 해보자. 이때 딱맞는 크기의 메모리를 할당하면 어떻게 될까?
array에 요소가 들어갈때마다 메모리를 할당하는 프로세스가 실행이 될것이다.
하지만, 미리 배열의 크기를 두배로 늘려놓는다면?
메모리가 낭비는 될수 있을지언정 할당이 실행되는 빈도가 급격히 낮아진다.
공식문서에서는 이 전략들을 적절히 사용해서 성능을 평균화 한다고 설명이 되어있다.
즉, 메모리 비용을 감수하더라도 할당의 빈도를 낮추는 전략을 택한 것이다.
(아마도 2배가 적절하다고 느꼇나보다)

### Custom하게 메모리 할당 크기 설정하기
하지만, 우리가 설정한 배열에 추가되는 값의 크기를 대충 알고있다면?
굳이 두배로 늘리지않고 추가되는 값의 크기만 늘리면 되지 않겠는가?
역시나 애플은 이러한 작업도 가능하게 해놓았다.
reserveCapcity(_:)를 사용하면 된다.
~~~swift
 reserveCapcity(_:)
~~~

### 배열안의 요소가 클래스인경우
Array는 값타입이기 때문에 copy를 하면 복사가 된다.
~~~swift
var numbers = [1, 2, 3, 4, 5]
var numbersCopy = numbers
numbers[0] = 100
print(numbers)
// Prints "[100, 2, 3, 4, 5]"
print(numbersCopy)
// Prints "[1, 2, 3, 4, 5]"

//값이 복사가 되어 원본의 변형이 사본에는 영향을 안끼침을 확인가능.
~~~

Array안에 클래스(참조타입)가 있으면 어떻게 될까?
물론 값타입과는 다르게 작동이 된다.
왜냐하면, 배열안의 값들은 배열 외부에 있는 개체에 대한 참조이기 때문이다.
~~~swift
// An integer type with reference semantics
class IntegerReference {
    var value = 10
}
var firstIntegers = [IntegerReference(), IntegerReference()]
var secondIntegers = firstIntegers

// Modifications to an instance are visible from either array
firstIntegers[0].value = 100
print(secondIntegers[0].value)
// Prints "100"

// Replacements, additions, and removals are still visible
// only in the modified array 첫번째 배열만 바꿔보기.
firstIntegers[0] = IntegerReference() //새로운 IntegerReference를 firstIntegers의 첫번째 요소로 넣는다.
print(firstIntegers[0].value) //당연히 새로운 값이 들어왔으므로 default값인 10
// Prints "10"    
print(secondIntegers[0].value) //원래 가지고있던 참조인 100
// Prints "100"  //값이 다르다.
~~~


### Copy On Write
표준 라이브러리에 있는 collecion들은 모두 COW를 사용한다.
Array는 값타입이지만, 수정전까지는 같은 스토리지를 공유한다.(여기서 스토리지가 아마 힙?)
수정이 일어날 때 수정 중인 Array는 고유하게 소유된 자체 스토리지로 교체를하고 이 복사본은 자동으로 수정이된다. (같은 스토리지를 참조하던게 분리된다는 뜻)
예를 들어서 numbers라는 Array를 복사한 두개의 Array가 있다고 하고, 원본 Array인 numbers의 요소를 수정하는 과정이다.
COW에 따라서 numbers와 firstCopy, secondCopy는 같은 스토리지를 공유하고 있다.
이때 수정이 일어나게 되면, numbers는 고유한 새로운 스토리지로 교체를 하게 된다.

이 과정이.. 그래서 뭐? 중요해?
약간?
왜냐하면, 반대로 참조하는 CopyArray들이 없으면 위와 같이 새로운 스토리지로 교체하는 작업이 일어나지 않기 때문이다.
배열이 자신 혼자 고유하게 있다면 굳이 새로운 스토리지로 교체할 필요가 없다.
그냥 똑같은 스토리지에 값만 바꾸면 되니까..

~~~swift
var numbers = [1, 2, 3, 4, 5]
var firstCopy = numbers
var secondCopy = numbers

// The storage for 'numbers' is copied here
numbers[0] = 100
numbers[1] = 200
numbers[2] = 300
// 'numbers' is [100, 200, 300, 4, 5]
// 'firstCopy' and 'secondCopy' are [1, 2, 3, 4, 5]
~~~

## [220712] 가면증후군(Imposter Syndrome) 

### 가면 증후군(Imposter Syndrome)
개발하면서 이런 생각을 한적이 아마 나뿐만 아니라 다들 있을 것 같다.
아.. 저사람은 나랑 같이 시작했는데 나보다 훨씬 낫네.
아.. 나는 개발자가 될만큼 똑똑한거 같지 않은데?
아.. 사실 나는 정말 아는게 없는데 내 진짜 실력이 들통나면 어쩌지?

이런 걱정으로 비롯되는게 `가면 증후군`이라고 한다.
```
자신의 성공이 실력이 아니라 순전히 운으로 얻어졌다고 생각해서 주변 사람들을 속여왔다고 불안해 하는 심리
사회적으로 충분히 인정 받는 자리에 있음에도 언제 나의 가면이 벗겨질지 모른다는 불안에 계속 사로잡혀 있는 상태.
```

영상에 따르면 거의 70%에 해당하는 개발자들이 이러한 경험을 한다고 한다.

### 왜 개발자는??
개발자는 할게 많다.
기술적 뿐만 아니라 연차가 쌓일 수록 리더쉽등의 커뮤니케이션도 잘해야되고 영어도 잘해야한다.
해야될게 애초에 많다 보니 자신이 너무 부족하다고 느끼는 경우가 많다는 것이다.

### 팁
1. 가면증후군은 70%이상의 사람들이 겪는다.
    - 내면에 수많은 부정적인 목소리들에 끝없이 말리지 말고 이건 모두가 겪는 과정이다. 라고 생각을 하고 객관적으로 왜 내가 이런 생각이 들었는지 돌아본다.
    

2. 우리는 항상 다른사람의 하이라이트만 본다.
    - 다른사람의 잘난 부분은 잘 보인다. 하지만 그 사람이 그만큼의 퍼포먼스를 보여주기까지 보여지지 않는 곳에서 노력한 것은 잘 보이지 않는다. 저 사람은 그냥 잘하는 사람이다.. 타고난 사람이다 라고 생각하기 보단 저사람이 저렇게 되기까지 얼마나 열심히 했을까~ 로 생각을 전환해보자
    

3. 모든걸 다 알 수 없다는 것을 인정하자.
    - 개발지식은 매일 새롭고 방대하다. 시니어들도 자신의 분야가 아닌 것들은 잘 모른다. 모르는게 있는게 자연스러운거라고 생각하자.

4. 성장의 길을 택하자
    - 아이러니하게도 사람들은 성장할 때 이러한 불안함을 느낀다. 자신이 알고 익숙한 곳에 머물러있는 사람들은 굳이 이런 감정을 느낄 필요가 없다. 이 불안함은 성장을 위한 진통이라고 생각하자. 

공부하기 싫어서 유튜브를 뒤적거리다가 [엘리님 유튜브](https://www.youtube.com/watch?v=hU4kULhOdNE)에서 꽤나 공감이 많이 가는 영상이 올라와서 조금 정리 해봤습니다.
시간나면 한번씩 보시면 좋겠네요~

## [220713] 내가 쓴 글 다시보기: MetaType

### Type이란?
하나의 Type이란 `어떤 공통된 속성들을 가진 것을 묶어서 모아놓은 것`이라 생각한다.
예를 들어 MBTI도 하나의 타입이 될수 있다.
외향적인 사람들한테는 너 'E(타입)지??'
내향적인 사람들한테는 너 'I(타입)지?' 하는 것 처럼 말이다.

### Meta + Type?
(구)FaceBook. 은 아니고 `뒤, 넘어서, 스스로` 등등의 의미를 가지고 있는데 이 중 Swift에서 쓰는 Meta Type은 `스스로`라는 의미가 가장 알맞다고 생각했다.

Meta Type은 Type 그 스스로, 그 자체를 의미하는 타입이라고 생각한다.

### 사용처
타입의 타입? 타입을 추상화? 말도 어려운 얘를 어디다가 쓰냐?
`타입 그 자체를 비교해야할 때` 주로 사용이된다.

예를 들어서 MBTI가 E로 시작하는 사람들만 들어올 수 있는 인싸파티가 있다고 해보자.
MBTI가 하나의 타입이라고 한다면, `MBTI를 가지고 있는 사람들은 하나의 인스턴스`라고 할 수 있다.
INTP이라는 타입은 하나밖에 없지만, INTP 타입을 가지고 있는 사람은 여러명일 수 있기 때문이다.

따라서, 이 파티의 입구에 있는 가드는 사람(인스턴스)을 판단하는게 아니라, 그 `사람의 타입 자체를 비교`해야한다.

Swift에서는 ObjectiveIdentifier라는 고유한 String값으로 변환 시켜주기도한다.
(주의 - 오직 클래스의 instance나 MetaType만 변환 가능)
~~~swift
class A { }
struct B { }

let a = A()
let b = B()
let c = B.self

ObjectIdentifier(a) // No error
ObjectIdentifier(b) // Error!
ObjectIdentifier(c) // No Error
~~~

위 값은 타입별로 고유한 값이기 때문에 이를 이용해 Dictionary나 Switch Case문을 사용하면 굳이 Enum타입을 더 만들지 않아도 효율적으로 값을 찾거나 로직을 분기할 수 있다.

### 추가로 알게 된 사실
Meta Type은 recursive하다.  
따라서 이런게 가능함.  
~~~swift
struct A {
    let name: String
}


let a = A.self
let b = A.Type.self
let c = A.Type.Type.self
print(ObjectIdentifier(a))
print(ObjectIdentifier(b))
print(ObjectIdentifier(c))

// ObjectIdentifier(0x000000010559c2a8)
// ObjectIdentifier(0x00000001db069cb8)
// ObjectIdentifier(0x00000001db069cd0)
~~~

## [220714] 내가쓴글 다시보기: iOS 뷰가 그려지는 과정

### Run Loop
View가 그려지는 과정을 이해하기 전에는 `Run Loop`라는 개념을 알아야한다.
Run Loop는 input Source들이 관리되는 프로그래밍적인 인터페이스이다.
Input Source에는 사용자의 입력(마우스, 키보드..) 등이 있다.
* Timer또한 Input Source는 아니지만 처리된다.

Thread들은 각각 자신의 Run Loop를 가지고 있으며, Thread를 생성할 때 자동으로 생성이 된다. 하지만, 자동으로 실행이 되지는 않는다.(Main Run Loop제외)

실행된 Run Loop는 Input Source(비동기 이벤트) Timer(동기 이벤트)를 처리한다.
여기서 우리가 잘 봐야할 것은 Main Thread가 가지고 있는 Main Run Loop이다.
이 Run Loop가 UI처리를 담당한다.

### Update Cycle
![](https://velog.velcdn.com/images/piggy_seob/post/f1b19cc7-4fa6-4810-bcd4-6203b163f781/image.png)

Run Loop로 들어온 이벤트들은 바로 처리가 되는게 아니라 Event Queue에 추가가 된다.

Event Queue에 있는 Event를 Application 객체가 하나씩 가져와 앱내 object들에게 Event를 dispatch한다.
object들은 입력을 해석하여 handler를 호출하는데 이 때, 개발자들이 작성한 코드들이 호출된다.
여기서 **모든** handler들이 호출이 되고 나면 Main Run Loop로 시스템 제어권이 넘어오는 시점이 생기는데 이때가 Update Cycle이 시작되는 시점이다.

Update Cycle에서 iOS시스템은 View와 관련된 작업들을 업데이트하기 시작한다.

Constraint -> Layout -> Display

즉, UI가 바뀌는 시점이 바로 이 시점이며 `모든 handler들이 호출되고 나면` 시작된다고 했으므로 실제로 UI가 그려지는 시점과 handler(우리의 코드)가 실행된 시점은 약간의 차이가 있다.
(나는 한번도 못느꼇는데? 초당 60번의 Update Cycle이 발생된다고 함. ㅇㅇ)

아무리 초당 60번의 Update Cycle이 발생한다고 해도 엄연히 차이가 있기 때문에,
애니메이션같이 실시간으로 View의 속성을 바꾸고 그 기준으로 다시 그려야하는 경우, Cycle을 관리할 필요가 있다.

### 요약 
Event -> EventQueue -> Application object -> core object -> handler() -> (모든 handler return시) UpdateCycle !

## [220715] 금주 회고

### 일어난 일
이번주는 꽤나 이벤트들이 많았다.
망쳤다고 생각했던 코테가 합격됬다고 연락이 왔고, 잊고 살았던 무신사에서 연락도 왔다.
때문에, 잔잔히 공부하고 8월,9월부터 지원해야지~ 라는 나의 계획은 조금씩 틀어졌다.
알고리즘 공부, 개인 프로젝트.. 열심히 계획세웠었는데;;
(하지만, 기분이 썩 나쁘지는 않았다~ 합격메일은 언제나 기분이 좋다)

### 배운 것
- DiffableDataSouce & Compositional Layout
DiffableDataSource와 Compositional Layout을 저번 에어비엔비 프로젝트때 사용을 했었는데, 정말 놀랍게도 다까먹었었다. 무신사 과제덕에 다시 공부할 기회? 를 얻었다.

- Kingfisher
Kingfisher는 이미지를 편하게 가져올수 있는 라이브러리다.
예전에 정말 코딩을 처음시작했을때 듀토리얼로 배운게 전분데,
이번에 다시 보게됬다.
신기한건 그때에 비해서 좀더 라이브러리가 잘읽힌다는 것이다.
그떄는 이해 못했던.. 비동기가 어쩌구~ URLSession이 어쩌구를 다 읽을 수 있다.
전보다 조금은 나아졌다는 생각에 기분이 조금 좋았다.
- 궁금증
	kingfisher는 UIKit을 import하는 라이브러리인데 URLSession을 이용해 이미지를 비동기로 가져오기까지한다.
    그렇다면.. 우리가 이제껏 했던 View와 Model(network)의 분리가 불가능한걸까?? 

### 앞으로의 계획

무계획이 최고의 계획이다라는 명언이 있다.

이번주를 표현하면 딱 이말이 어울리지 않나 싶다.
담주 계획은 담주에 생각하는 걸로!

## [220718] HTTP vs HTTPS piggy

면접을 준비하다가 HTTP와 HTTPS의 차이점을 물어보는 질문을 받았다.
이 질문은 되게 베이직한 질문 같은데 막상 말하려니 입이 떨어지지 않았다.
하자. 공부

### 정의
`HTTP(HyperText Transfer Protocol)`는 W3 상에서 정보를 주고받을 수 있는 프로토콜이다. 주로 HTML 문서를 주고받는 데에 쓰인다. 주로 TCP를 사용하고 HTTP/3부터는 UDP를 사용하며, 80번 포트를 사용한다 - 위키

HTTPS(HyperText Transfer Protocol over Secure Socket Layer) HTTP의 보안이 강화된 버전이다
HTTPS는 소켓 통신에서 일반 텍스트를 이용하는 대신에, SSL이나 TLS 프로토콜을 통해 세션 데이터를 암호화한다. 따라서 데이터의 적절한 보호를 보장한다. HTTPS의 기본 TCP/IP 포트는 443이다.
HTTPS를 사용하는 웹페이지의 URI는 http://대신 https://로 시작한다. - 위키

### 등장배경
HTTP는 서버에서 브라우저로 전송되는 정보가 암호화가 안된다는 단점이 있었다.
이는 사용자들의 정보가 취약하다는 말이므로 수정이 필요했다.

### SSL + TLS
이를 해결하기 위해 SSL(보안 소켓 계층?)을 HTTP에 적용 시키게 되었다.
SSL 인증서는 사용자가 사이트에 제공하는 정보를 `암호화`한다.
이렇게, 전송된 데이터는 중간에 누군가 해킹하더라도 암호화 되어있어서 해독하기 힘들 것이다.

결국, HTTP + SSL은 서버와 브라우저 사이에 안전하게 암호화된 연결을 만들 수 있게 도와주고 서버 브라우저가 민감한 정보를 주고 받을 때 이것이 도난 당하는 것을 막아준다.

**TLS**
TLS(Transport Layer Security)는 데이터가 전송중에 수정되거나 손실되는 것을 방지하고, 사용자가 자신이 의도하는 웹사이트와 통신하고 있다고 인증할수 있는 기능이라고 한다.
[구글워크스페이스](https://support.google.com/a/answer/100181?hl=ko)에 따르면 TLS가 SSL보다 더 업그레이드 된 버전이며 SSL과 TLS를 모두 SSL이라고 부르기도 한다고 한다.

### 장점
보안상의 장점이 있는것은 알겠는데.. 그럼 보안상 장점밖에없나?
별로 중요하지 않은 데이터를 사용하는 웹사이트는 http를 사용하면 되는거 아닌가?
사실 몇가지 장점이 더있다.
- SEO(검색엔진 최적화, search engine optimization)
검색엔진이 더 잘 검색할 수 있도록 최적화 하는 것인데.. 보안이 높은 HTTPS사이트에 사용자들이 몰리는 것도 자연스럽지만, 구글검색엔진은 보안성이 높은 HTTPS에 가산점을 준다고 한다. 

- 가속화된 모바일 페이지(AMP, Accelerated Mobile Pages)
AMP를 만들기 위해서는 HTTPS를 사용해야만 한다고 한다.
AMP는 구글에서 만든 모바일 기기에서 콘텐츠를 빠르게 로딩하기 위해 만든 것인데, HTML에서 필요없는 부분을 제거했다고 한다. (이 부분은 다음에 더..)

[참고: http vs https](https://blog.wishket.com/http-vs-https-%EC%B0%A8%EC%9D%B4-%EC%95%8C%EB%A9%B4-%EC%82%AC%EC%9D%B4%ED%8A%B8%EC%9D%98-%EB%A0%88%EB%B2%A8%EC%9D%B4-%EB%B3%B4%EC%9D%B8%EB%8B%A4/)

## [220719] 면접 후기
내생의 첫 면접, 그 후기.

### 알고있다 생각한 것의 역습
면접 질문중에 가장 어려움을 겪었던것은 아예 모르는 것이 아니라, 알고 있다고 생각했던 것 들 이었다.
예를 들면, http가 뭐냐, 스레드가 뭐냐..  
그냥 당연하게 쓰던 말들을 설명을 하려하니까 말이 좀처럼 나오지 않았다.  
누가 그랬다. 내가 완벽히 이해했다는 것은 지나가던 어린아이를 붙잡고도 설명할 수 있어야 한다고..   
그런면에서 나는 아직 이 것들을 잘 몰랐던 것이구나.. 라는 것을 깨달았다.

### 향후 계획
- 내가 아는 것은 말로 내뱉는 연습을 해보자.
- 배운 것은 남들에게 공유를 해보자.(조금 더 정리가 될 것이다) 
- 면접은 이번이 끝이 아니다. 질문과 답변들을 내 나름대로 정리해 보쟈.

## DiffableDataSource

### DiffableDataSource
얘는 대체 뭘까?
The object you use to manage data and provide cells for a collection view. - 애플
TableView 혹은 CollectionView의 데이터를 관리하고 cell을 제공하기 위한 object.
근데 이미 우리는 dataSource를 사용하고 있지 않나?
이 친구는 어째서 나오게 된걸까?

### 탄생 배경
애플의 표현에 따르면 단하나의 Truth Data를 가질 수 있게 하기 위해서라고 한다.

우리가 일반적인 DataSource를 이용해서 CollectionView를 구현할 때를 살펴보자.
DataSource에 있는 함수들을 이용해서 우리는 View를 그리게 된다.
![](https://velog.velcdn.com/images/piggy_seob/post/43ca44bf-d53d-469c-9115-c2473889accf/image.png)

이 함수들은 어떠한 Model 값에 의존해서 view를 그리게 된다.
여기서 중요한 점은 UI가 가지고 있는 값과 Model이 가지고 있는 값이 차이가 날수 있다는 것이다. 이때 우리가 가끔 봤던 에러가 발생하게 된다.

![](https://velog.velcdn.com/images/piggy_seob/post/0fe16ccf-e444-48e1-a542-85dee445ba63/image.png)

centralize된 truth가 없다.

그래서 보통 이런 에러는 `reloadData`라는 메서드를 이용해서 Controller가 가지고 있는 값(보통 DataSoure를 Controller에서 구현하므로)과 UI가 가지고 있는 값을 맞춰준다. 
이 방법이 나쁜 것은 아니라고 애플도 인정했지만, 이보다 더 나은 방법을 제시하기 위해 나온 것이 DiffableDataSource이다.

### 사용 방법(기본)
가장 기본적인 DiffableDataSoure를 만들어보자.

![](https://velog.velcdn.com/images/piggy_seob/post/832a1e23-42a1-4a99-9803-5543170c2083/image.png)

**1. SectionIdentifierType과 itemIdentifierType 정의**
먼저, DiffableDataSource는 `제네릭 타입을 두개 정의` 해야한다.
여기서 정의되는 제네릭 타입은 `Section과 Item이다.`
Section은 말그대로 CollectionView의 Section이고, Item은 이 DataSource를 채울 (우리가 Cell에 채울) Model이다.
이때 Section에 넣는 타입과 Item은 모두 `Hashable해야한다.`
DiffableDataSource의 원리는 현재 상태를 SnapShot을 찍은다음 비교해서 적용하는 원리인데, 값이 고유하지않으면(hashable)하지 않으면 `현재의 값과 바꿀 값을 비교할 수 없기 때문이다.`
(마치 Equatable Protocol을 채택하지 않으면 값을 비교할 수 없는것처럼.)
보통 Enum 값을 이용해서 SectionType을 정의하는게 보편적이다.

~~~swift
enum SectionType {
    case main
}

struct Person: Hashable {
    let name: String
    let age: Int
}

var diffableDataSource: UICollectionViewDiffableDataSource<SectionType, Person>!
~~~

**2. Cell provider 정의**
이제 Cellprovider라는 클로저를 정의 해주어야 한다.
얘는 우리가 알던 CollectionView의 cellForItemAt을 정의해준다고 생각하면 편하다.
따라서, 코드도 매우 흡사하다.
여기서 중요한 점은 우리가 미리 정의한 `ItemIdentifier로 넣은 값이 클로저의 인자로 들어온다는 것`이다.
만약에, ItemIdentifierType으로 String값을 넣었다면 Person이 아닌 String타입이 매개변수로 들어오게 된다.

Ex)
~~~swift
self.diffableDataSource = UICollectionViewDiffableDataSource<SectionType, Person>(collectionView: collectionView) {
    collectionView, indexPath, person in // Item으로 PersonType을 넣었기 때문에 person이 인자로 들어옴.
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PersonCell", for: indexPath)
            as? PersonCell else { return UICollectionViewCell()}
    cell.config(name: person.name, age: person.age)
    return cell
}
~~~

**3. SnapShot 정의**
위에서 DiffableDataSource의 원리가 SnapShot으로 비교하는 것이라고 했다.
snapShot을 만들고 item과 Section을 추가해보자.
~~~swift
  func snapShot(people: [Person]) {
      var snapShot = NSDiffableDataSourceSnapshot<SectionType, Person>()              // 빈 SnapShot 생성
      snapShot.appendSections([.main])    // Section 추가(미리 지정한 SectionType만 가능)
      snapShot.appendItems(people)        // Item 추가.(미리 지정한 ItemType)
      self.diffableDataSource.apply(snapShot, animatingDifferences: true)        // 적용
  }
~~~

이제 이 함수에 매개변수를 넣어주면 될 성공이다.
기존에 있던 Controller Extension에 있던 DataSource로직은 과감히 지워도 된다!

### 사용 방법(심화)
하지만, 위방법은 뭔가 아쉽다. CollectionView를 사용하는 이유는 뭔가... 생각해보면 Section별로 다른 Item과 flow를 주고싶을때?
Compositional Layout과 함께 써보자.


### 장점 및 단점

### 비교

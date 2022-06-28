# Piggy

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



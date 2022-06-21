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


## [22.06.18] 설계 vs 구현에 대한 생각
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


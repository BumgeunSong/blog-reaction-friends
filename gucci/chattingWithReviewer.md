# [참고 PR ](https://github.com/codesquad-members-2022/issue-tracker/pull/26)

# 첫번째 질문. 화면 전환 로직 담당 어떻게 ? -> 코디네이터 패턴

* [추천 레포](https://github.com/AndreyPanov/ApplicationCoordinator/blob/master/ApplicationCoordinator/Application/ApplicationCoordinator.swift)
 이용할 수 있습니다:)
* [추천 블로그](https://zeddios.medium.com/coordinator-pattern-bf4a1bc46930)
```
fileprivate var onboardingWasShown = false
fileprivate var isAutorized = false

fileprivate enum LaunchInstructor {
  case main, auth, onboarding
```

# 두번째 질문. 추상화를 하면 테스트가 좋아지는 이유
질문: 
test에서 var issueRepository: IssueTrackinRepository!
이렇게 만들었을 때, 실제 서버에 요청하는 레포지토리와 테스트용으로 요청하는 Mock 서버 레포지토리를 갈아끼우기 좋아서 그러는 것일까요?

두두: 
두번째 질문에 대해서 답변 남길게용~

갈아 끼우는 것도 있지만, 테스트 작성해야 할 때 기본 원칙이 외부 의존성이 생기면 사이드 이팩트가 발생할 여지가 있기 때문에 가급적 외부 의존성이 생기지 않도록 테스트 작성해야 합니당~

Repository의 구체적인 구현으로 테스트 하는 경우 네트워크 영향을 받을 수 밖에 없고, 테스트 하려는 로직이 정말 그 객체만이 가지고 있는 로직만 테스트 하는 건지 모호해지기 때문입니당..!

마침 예전에 토이 플젝으로 작성해둔 테스트 코드가 있는데요, 현재 현업에서도 외부 의존성 받지 않게 Mock, Stub, Faker 등 활용해서 작성하고 있습니다~!

* [추천레포](https://github.com/mienne/iOS-Github-User-Search/tree/master/GithubUserSearchTests/Search)
* 추천라이브러리
  * 퀵 QUICK
    * 애플에서 지원해주는 XCTest 프레임워크와 다르게
개인적인 생각으로 테스트 케이스 작성하기가 편해용~
descrube, it 이러한 표현들을 이용해서 ㅎㅎ
그룹 짓기도 편하고, 찾아보시면 도움 많이 되실거에요 ㅎㅎ
  * 님블 NIMBLE
    * Nimble 같은 경우는 테스트 메서드를 좀더 편하게 제공해주는 라이브러리라 생각해도 됩니다~

* ViewModel 작성할 때도 구체적인 값보다 인터페이스 의존될 수 있도록 작성해야 테스트 하기 수월해져요~

* 그레서 -Input, -Output 프로토콜이나 Action, State 표현하는 케이스도 있는거 같고 ㅎㅎ!

* [추천 레포](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/MoviesScene/MovieDetails/ViewModel/MovieDetailsViewModel.swift)

* [추천 레포](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVMTests/Presentation/MoviesScene/MovieDetailsViewModelTests.swift)

* 바인딩하는 부분은 작성하신 것처럼 클로저로 하셔도 되용 ㅎㅎ~

* 예시처럼 Observable 객체 만들어서 하실 필요 없긴 해요~

* 그리고 테스트 코드 같은 경우 Alamofire 오픈소스가 테스트 잘짜여져 잇어요 ㅎㅎ

* [알라모파이어](https://github.com/Alamofire/Alamofire/tree/master/Tests)

* 깔끔한 바인딩 방법
  * RxSwift
  * Combine

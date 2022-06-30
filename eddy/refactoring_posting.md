## Massive View Controller를 해체해보자 - 리팩토링 후기
(하지만 결국 해체되는 건 나였다?)

iOS 개발을 배우면서 짧은 시간 내에 다양한 프로젝트를 진행해봤다. 

처음 보는 기능을 접하면, 당장 구현하기에 급급해진다. 어떻게든 돌아가게 만들고, 코드를 푸시하는데 집중한다. 일단 이것부터 하고 고치자. 이것부터 하고... 시야가 좁아진다.

하지만 그 과정을 거치면서 느꼈다. 실력이 늘기 위해서는, 더 나은 방법은 없을지 고민해야 한다. 적당히 되는 수준에서, 한 뎁스, 두 뎁스를 내려간다. 겉으로 별로 바뀌는 건 없다. 하지만 배우는 게 많다. 

뎁스를 챙기려면, 집중할 부분을 선택해야 한다. 모든 걸 다 파볼 수는 없다. 방향을 잃지 않으려면 처음부터 내가 실력을 쌓고 싶은 부분을 의식적으로 설정해야 한다. 

마치 무술가가, 똑같은 발차기를 계속 반복 연습하듯이 말이다. 


![kick](https://i.pinimg.com/originals/d5/9d/08/d59d08aaef8301e34b0b32f122c21e94.jpg)


> 당신이 요즘 연습하고 있는 발차기는 무엇인가?

이번 프로젝트에서 나의 '발차기'는 **'단위 테스트와 리팩토링'**이었다. 

객체 지향적인 코드, 역할이 잘 나눠진 코드를 만들기 위해서 코드를 여러번 뒤집었다.

기능은 단순했지만, 총 3번이나 갈아엎는 대공사를 했다. 전채 개발 시간의 50% 이상을 리팩토링에 사용한 것 같다. 그만큼 삽질도 많이 했다. 

글이 꽤 길다. 하지만 iOS 개발을 하다보면 흔히 발생하는 상황들이다. 이 글을 다 읽고나면, 여러분도 **코드 구조를 개선하는 힌트를 분명 얻을 수 있을 것**이다. 

# 기능 소개

리팩토링할 코드가 하는 일을 간단하게 이해하고 넘어가자.

에어비앤비 숙박 검색. 위치, 날짜, 가격 조건을 설정한다. 거기에 맞는 숙박 리스트를 받아온다.

이 중 오늘 얘기할 부분은 '위치 검색' 기능이다.

1. 검색 탭이 활성화되면, 사용자 현재 위치 근처의 장소들이 추천된다.
2. 검색어를 입력하면 애플 맵을 사용해 검색어 자동완성이 뜬다.
3. 검색어를 선택하면 2가지 선택지가 있다. 해당 검색어에 해당하는 결과가 하나뿐인 경우는 해당 결과로 위치 검색을 종료한다. 결과가 여러 개인 경우 다시 한번 상세 검색을 띄운다.

![location-search](https://user-images.githubusercontent.com/17468015/173227973-4daf7840-bc63-47f8-abfd-f4dd029a82c8.gif)

# 리팩토링

## 개요

했던 리팩토링 작업을 요약하면 3가지 파트로 나눌 수 있다.

1. ViewController에서 데이터 소스와 데이터 로딩 분리하기.
2. `UICollectionViewDelegate`를 DataSource에 합치기.
3. 객체 간 의존성 추상화하고 DI Container로 주입하기.

> 물론 실제로는 이렇게 깔끔하고 논리적으로 진행되지 않았다. 좌충우돌의 연속이었을 뿐... 하지만 설명을 위해서 다듬어봤다.


## 1. ViewController에서 데이터 소스와 데이터 로딩 분리하기

iOS MVC는 Massive View Controller라는 아주 오래된 농담이 있다. 

하지만 뷰 컨트롤러가 거대해지는 이유는 MVC의 문제가 아니다. 역할 분리에 신경을 쓰지 않았기 때문이다. 

MVP든, MVVM이든 다른 패턴이라도 역할 분리를 잘 하지 못하면 거대한 객체는 또 등장한다. 

맨 처음 구현한 위치 검색 기능은, 그야말로 전형적인 거대 뷰 컨트롤러였다. 

`LocationSearchViewController`라는 하나의 객체가 다음의 역할을 모두 하고 있었다.

- SearchBar, CollectionView에 대한 설정과 레이아웃 세팅
- CollectionView의 Delegate와 DataSource 역할
- MapKit API와 서버 API에서 데이터 불러오기.

> **자, 이제 거대 뷰 컨트롤러를 해체해보자!**

![tuna](https://cdn.isusanin.com/news/photo/201701/28481_7258_2417.jpg)
(출처: 수산인신문)

### 1. DataSource, Delegate 분리

위치 검색 기능을 담당하는 `LocationSearchViewController` (이하 VC) 는 `UICollectionView` (이하 컬렉션 뷰)를 가지고 있다. 

컬렉션 뷰는 데이터를 띄우거나 인터랙션이 있을 때 `UICollectionViewDataSource` (이하 DataSource)와 `UICollectionViewDelegate`(이하 Delegate)에 무엇을 해야할지 물어본다. 

현재는 VC가 DataSource와 Delegate 메서드까지 직접 구현하고 있다.

![](https://velog.velcdn.com/images/eddy_song/post/e54d20a7-36a7-4841-8d02-4a8bbab16a7b/image.png)

이 두가지 역할을 별도 객체로 분리해보자. 

**DataSource를 역할별로 분리하기.**

DataSource를 분리하는 것은 단순한데, 그 안에도 복잡한 부분이 있었다.

위치 검색은 처음에는 추천 장소를 보여주다가, 검색이 시작되면 검색 키워드를 보여준다. 그리고 상세 검색 때는 상세 검색 데이터를 보여준다.

기존의 DataSource 메서드는 이 변화를 조건문으로 처리하고 있었다.

```swift
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    if isSearching{
        return searchResultData.count
    }
    return recommendationData.count
}
```

`isSearching` 이라는 변수를 사용해서 셀의 갯수를 지정했다.

`isSearching`은 검색 바 입력 처리에서 바꿔주는 `Bool` 값이다. 처음에는 이런 식의 코드였다. 

```swift
func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
		if searchText == "" {
		    isSearching = false
		    searchResultData.removeAll()
		    collectionView.reloadData()
		}
		else {
		    isSearching = true
		    searchCompleter.queryFragment = searchText
		}
}
```

조건문을 사용하면 기능이 추가될 때 코드를 수정해야한다. 좋지 않은 방법이다. 

또 각 단계별로 Datasource를 분리해서 역할을 더 뚜렷하게 만들 필요가 있었다.

추천, 자동완성, 상세검색을 모두 별도의 DataSource로 분리했다. 

아래는 추천 데이터를 담당하는 `RecommendationDataSource` 객체다.


```swift
class RecommendationDataSource: NSObject, UICollectionViewDataSource {

    private var recommendationData = [Place]()
    private var didLoadData: () -> Void

    init(didLoadData: @escaping () -> Void) {
        self.didLoadData = didLoadData
        super.init()

        let location = Location.makeRandomInKR()
        let recommendator = DefaultRecommendator()

        recommendator.recommend(for: location) { place in
            guard let place = place else { return }
            self.recommendationData = place
            didLoadData()
        }
    }

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        recommendationData.count
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: PlaceCell.reuseIdentifier, for: indexPath) as? PlaceCell else { return UICollectionViewCell() }
        let data = recommendationData[indexPath.item]
        cell.setPlaceCell(data)
        return cell
    }
}
```

이런 식으로 DataSource를 총 3개로 분리했다. 

응집도가 훨씬 높은 객체를 만들 수 있었다. 추천, 자동완성, 최종 검색 각 단계에 따라서 서로 완전히 다른 인스턴스 변수를 참조하기 때문이다.

![diagram_1](https://raw.githubusercontent.com/adnpark/daily-devlog/f0abac63dec4c01144b872613ab174ca84e64a2b/eddy/separate_datasource.png)

이제 상위 VC는 이벤트(검색 시작, 장소 선택)에 따라 서로 다른 Datasource를 갈아끼운다. 하나의 콜렉션뷰를 재활용하면서도, 데이터를 채우는 역할은 분리했다.

마찬가지로 Delegate도 분리했다. Delegate는 Cell에 대한 터치가 들어왔을 때 이벤트를 처리한다.  

그런데 Delegate 분리는 문제가 발생했다. 깔끔한 DataSource 분리와 달랐다. 여기에 대해 다음 챕터에 다시 설명하도록 하자.


### 2. 데이터 로딩 모델 분리

각 DataSource는 외부에서 데이터를 불러온다. 추천 데이터는 **서버 API**, 자동완성과 검색 결과는 **MapKit API**를 사용한다.

현재 코드는 DataSource가 직접 MapKit과의 소통과 네트워크 요청을 처리하고 있었다.

어떻게 하고 있었을까? 아래 코드를 보자.

```swift
import UIKit
import MapKit

class SearchCompletionDataSource: NSObject, UICollectionViewDataSource {

    private var searchResultData = [MKLocalSearchCompletion]()
    private var searchCompleter = MKLocalSearchCompleter()
    private var didLoadData: () -> Void

    init(didLoadData: @escaping () -> Void) {
        self.didLoadData = didLoadData
        super.init()
        searchCompleter.delegate = self
    }

	// (...) Datasource 관련 메서드 생략

	// 검색어를 MKLocalSearchCompleter에게 알려준다.
    func setQueryFragment(_ queryFragement: String) {
        searchCompleter.queryFragment = queryFragement
    }
}

extension SearchCompletionDataSource: MKLocalSearchCompleterDelegate {
	// MKLocalSearchCompleter는 자동완성 결과를 delegate로 전달한다.
	// Datasource는 결과를 인스턴스 변수에 저장하고, 상위 객체에 로딩 완료를 알린다.
    func completerDidUpdateResults(_ completer: MKLocalSearchCompleter) {
        searchResultData = searchCompleter.results
        didLoadData()
    }
}
```

DataSource 역할이면서, 외부 API에 데이터를 요청하는 역할을 같이 하고 있다. 

객체 역할을 더 작게 쪼개고 싶어서 데이터 로딩 부분을 분리해냈다.

이쯤 되면 약간 고민이 되기 시작한다. 정말 이 역할을 꼭 분리해야하는가?

사실 누구나 '단일 책임 원칙'을 안다. 한 객체가 한 역할만 하게 해야한다. 

그런데 어디까지가 하나의 역할인가? 그건 보는 사람마다 다르다. Data를 요청해 받아오는 로딩(Loading)과, View에 전달해주는 소싱(Sourcing)은 별도 역할이라고 봐야할까?

어쩌면 지나친 분리일 수도 있겠다 싶었다.

또 하나 고민해본 옵션이있다. DataSource를 컨트롤러 객체가 담당하지 않고, 모델 계층이 담당하도록 하면 어떨까?

ViewController에서 분리해놓고 보니 Datasource는 모델과 거의 비슷한 역할을 한다는 생각이 들었기 때문이다.

> DataSource는 사실 모델에 가깝지 않은가?

하지만 리서치 결과 그게 아니라는 사실을 알았다. DataSource는 TableView, CollectionView에 데이터를 채우는 역할이다. 자연스럽게 `UIKit`에 의존한다. `UITableViewCell` 같은 타입 정보를 알아야 한다. 다시 말해, `import UIKit`이 들어갈 수밖에 없다.

MVC 패턴에서는 뷰와 모델의 관심사를 분리해야한다는 원칙이 있다. 모델은 UIKit에 완전히 독립적이어야 한다.

현재는 기능이 단순하기 때문에 데이터 소싱과, 데이터 로딩을 같이 하는 게 이상해보이지 않는다. 하지만 장기적으로 일관성 있는 계층 구조를 유지하기 위해서는 둘의 분리가 필요하다는 결론이 나왔다.

Data를 요청해 받아오는 로딩(Loading)과, View에 전달해주는 소싱(Sourcing)을 별도 역할로 분리했다. 

'컨트롤러' 계층에 속하는 DataSource 객체와 '모델' 계층에 속하는 Data Loading 객체로 분리했다. 

이제 DataSource는 데이터를 저장하지 않는다. 모델에게 요청해서, 뷰로 전달해줄 뿐이다.

![diagram_2](https://raw.githubusercontent.com/adnpark/daily-devlog/f0abac63dec4c01144b872613ab174ca84e64a2b/eddy/separate_dataloading.png)

### ⚠️ Value type을 캡처하는 클로저 

이 때 한 가지 문제가 발생한다.

데이터 로딩 모델은 `struct`로 만들었다. `struct`는 value type이 된다. value type은 불변성 덕분에 부수효과가 발생하지 않고, Stack에서 다루기 때문에 효율적이다. 

별다른 이유가 없으면 기본적으로 value type으로 타입을 정의해준다.

여기에 데이터를 비동기로 요청하는 메서드를 추가했다. 완료되면 자신의 인스턴스 변수에 저장한다.


![error_1](https://raw.githubusercontent.com/adnpark/daily-devlog/f0abac63dec4c01144b872613ab174ca84e64a2b/eddy/mutating_self_error.png)

탈출 클로저는 self 파라미터를 캡처할 수 없다는 에러가 뜬다. 

우리는 `getRecommendation()` 이라는 함수에 인자로 클로저를 넘긴다. 

탈출 클로저는 해당 스코프가 끝난 뒤에 나중 어떤 시점에 실행된다. 그때 이 클로저는 받아온 데이터를 저장하기 위해 DefaultRecommendator 모델을 기억(캡처)한다.

> 탈출 클로저가 Value type을 캡처하고 있다.

Value type은 할당 시 참조가 아닌 값 전체를 복사한다. 클로저가 value type을 캡처하면 복사가 일어난다.

클로저 안에 캡처되는 `DefaultRecommendator` 인스턴스는 실제 클로저를 파라미터로 넘기는 `DefaultRecommendator`  인스턴스와 '다른' 인스턴스다.

따라서 나중에 이 `DefaultRecommendator`를 찾아와서 값을 저장해준다는 게 말이 되지 않는다. 

에러는 이 말을 하고 있었다.

특정 스코프가 완료(return)된 후에 목표한 데이터에 접근하려면 어떻게 해야할까? 

**참조 타입(Reference type)으로 만들어줘야 한다.** 참조 타입은 힙에 저장되고 참조가 복사된다. '고유성'을 갖는다. 다른 스코프에서도 식별가능하다.

데이터를 참조 타입으로 만드는 방법은 2가지가 있다.


1. 데이터를 요청하는 `DefaultRecommendator` 타입을 `class`로 바꾸는 것
2. `DefaultRecommendator`의 프로퍼티이자 데이터를 저장하는 `recommendationData`를 `class`로 감싸는 것

첫번째 방법이 가장 단순해보인다. 가장 먼저 떠오르는 해결책이다. 

> struct로 하니까 안 된다고? 그럼 class로 바꿔.

하지만 여기서 우리가 클로저에서 캡처하고자 하는 목표는 `DefaultRecommendator`가 아니라 변수인 `recommendationData`다.

물론 1번으로 문제는 해결할 수 있다. 
`DefaultRecommendator`가 `recommendationData`를 갖고 있다. `DefaultRecommendator`를 `class`로 바꿔 Heap에 저장되게 한다. `recommendationData`도 자연스럽게 힙에 저장된다. 

하지만 참조 타입의 단점인 부수효과(Side effect)의 범위가 '필요 이상으로' 넓어진다. 

만약 `DefaultRecommendator`에 또다른 인스턴스 변수가 존재한다면, 그 데이터도 레퍼런스 타입이 된다. 우리는 가급적 Value type을 많이 쓰고 싶다!

예측가능성, 스레드안전성, 테스트 용이성 등 Value type이 가지는 장점을 유지하기 위해서다. 

따라서 Reference type이 되는 범위, 부수 효과가 생기는 범위는 가급적 최소화하는 게 좋다. 

2번째 방법을 선택하자.

`recommendationData`는 배열이다. 배열(Array)은 값 타입이다. 참조 타입으로 바꾸기 위해 값을 감싸는 별도의 객체를 만들어줘야 한다.

이 객체 이름을 `Box`라고 하자.

```swift
class Box<T> {
    init(value: T) {
        self.value = value
    }

    var value: T
}
```

`Box`는 참조 타입이다. 클로저는 참조를 캡처한다. 나중에 요청한 함수 스코프가 종료된 후에도 데이터 응답을 넣어줄 수 있다.

동시에 `DefaultRecommendator` 타입은 `struct`로 그대로 유지할 수 있다.


## 2. Delegate 분리 문제 해결하기

여기까지 하고, 얼마 후 다시 코드를 봤다. 

그런데 실행 흐름이 이해가 되지 않았다. 일관되지 않은 실행 흐름과 참조가 이곳저곳에 등장했다.

> 분명 분리를 시켰는데 왜 더 복잡해진 거지?

원인은 'Delegate의 분리'였다.

챕터 1에서 우리는 DataSource와 Delegate를 모두 VC에서 분리시켰다.

UIKit에서 DataSource는 주로 컬렉션 뷰의 출력 흐름을 담당하고, Delegate는 입력 흐름을 담당한다. 

좀 더 구체적으로 말하면, 지금 Delegate는 컬렉션 뷰의 셀 하나가 '선택'됐을 때 처리 로직을 담당하고 있다.

문제는 이 셀 선택의 처리 로직은 다른 객체에 많이 의존한다는 점이다.

상황에 따라 셀이 선택되면 이런 일을 해야 한다.
- DataSource가 가진 배열에 `IndexPath`로 접근해서, 선택된 셀에 해당하는 데이터 가져오기.
- `MapKit`으로 LocalSearch를 실행하고, 결과를 받아 상세 검색 담당 DataSource에게 넘겨주기.
- `NavigationBar`에 들어있는 `SearchBar`의 텍스트 초기화/변경하기.
- `NavigationBar`에 새로운 뷰 컨트롤러 추가하기 (화면 전환)

가만히 살펴보자. 자기 혼자 하는 일을 하나도 없다. 전부 다른 객체를 참조한다.

그런데 이런 의존성이 높은 객체를 급하게 분리하다보니 이상한 상황이 벌어졌다. Delegate가 콜렉션 뷰와 네비게이션, 그리고 다음에 등장할 뷰 컨트롤러까지 알게 됐다.

아래 코드에서 생성자(init)를 유심히 보자. 콜렉션 뷰와 네비게이션 컨트롤러가 넘겨진다...?

```swift
class DetailSearchDelegate: NSObject, UICollectionViewDelegate {

    let searchDateVC = SearchDateViewController()
    let navigationController: UINavigationController?
    let collectionView: UICollectionView?

    init(navigation: UINavigationController, collectionView: UICollectionView){
        self.navigationController = navigation
        self.collectionView = collectionView
    }

    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {

        guard let datasource = self.collectionView?.dataSource as? DetailSearchLocationDataSource else { return }
        let mapItem = datasource.searchResultData[0]
        guard let place = PlaceFactory.makePlace(with: mapItem) else { return }
        searchDateVC.queryParameter?.place = place
        self.navigationController?.pushViewController(searchDateVC, animated: true)
    }
    // ...
```

아래 `didSelectItemAt`에선 dataSource의 searchResultData까지 꺼내오고 있다. 닷(.)을 2개나 써서 말이다. (디미터 법칙 위반) 

3개의 Delegate가 다 이런 식으로 다른 객체를 참조하고 있었다. 실행 흐름이 매우 복잡해질 수 밖에.

현재 상태를 다이어그램으로 요약하면 다음과 같다.

![before_diagram](https://user-images.githubusercontent.com/17468015/173004810-5c99ae45-9158-4fd7-ad26-fd0e52f41fd8.png)

> 이제 리팩토링으로 다시 개선해보자!


### 1. Delegate와 DataSouce를 합쳐서 하위 Controller로 만들기

앞서 말했듯이, Delegate는 무언가를 하기 위해서 DataSource 혹은 ViewController에 의존한다.

`didSelectItemAt`에서 넘겨받은 IndexPath를 가지고 DataSource에게 이 Index가 어떤 데이터인지 알아낸다. 

그 다음으로 흔한 작업. 알아낸 데이터를 다음 VC로 넘겨주고, 다음 VC를 화면에 띄우는 것이다. 이때 다음 화면을 담당하는 VC와 화면을 띄우기 위한 NavigtaionController를 아는 것은 상위 뷰 컨트롤러다.

즉 Delegate의 실제 구현은 ViewController나 Datasource와 관련성이 매우 높다. 

그러다보니 과도한 의존 관계가 생겨난 것이다. 이걸 끊기 위해선 위임이 필요하다. Delegate는 정보 전달만 하고, 작업은 ViewController나 DataSource에게 시켜야겠지.

하지만 그러면 문제가 있다. Delegate를 분리한 이유가 없어진다. 어차피 자기 혼자 처리하는 일도 없는데 왜 별도 객체를 만드느냔 말이야.

그럼 어떻게 해야할까?

뭘 어떻게 해. 합쳐야지.

Delegate를 떼어냈다가 오히려 실행 흐름이 복잡해졌다. 물론 지금 생각하면 Delegate 자체 때문이라기보다는 Delegate를 분리하면서 역할 위임을 제대로 안했기 때문이다. 

하지만 객체를 작게 분리하는 게 항상 좋은 게 아니라는 점을 깨달았다.  쉽게 생각하면 무조건 분리가 좋은 것 같다.

오히려 같은 데이터를 빈번하게 참조하는 메서드는, 데이터를 공유할 수 있는 같은 객체에 둬야 한다. 그래야 객체의 응집도가 높아진다.

Delegate를 합칠 때는 2가지 선택지가 있었다.

* DataSource에 합치기
* ViewController에 합치기.

더 많이 관련이 있는 쪽에 합치는 게 정답이겠지. 

현재 상황에서 Delegate 메서드는 양쪽 다 의존하는 정도가 비슷했다.

DataSource는 3개로 작게 나누어놓았기에 비교적 역할이 간단했다. ViewController는 이미 DataSource를 전환하고, 화면을 바꾸는 로직이 많이 들어있었다. 

그래서 DataSource와 Delegate를 합치기로 했다.

적절한 이름이 뭘지 고민했다. (DataSource와 Delegate를 합쳤으니 DelegateDataSource??)

생각해보니 그냥 Controller라고 하면 될 것 같았다. View와 Model 사이를 중재하는 역할이 전부다. 전형적인 Controller 역할이다. 보통 Controller라고 하면 ViewController가 일반적이지만, ViewController만 Controller라는 법은 없다.

아래는 둘을 합쳐서 만든 `LocationSearchRecommendationController` 객체다.


```swift
class LocationSearchRecommendationController: NSObject, UICollectionViewDataSource {

func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        guard let place = recommendator?[indexPath.item] as? Place else {
            return
        }
        self.delegate?.didSelectPlace(place)
    }
	  // ... 생략
}
```

어지러운 외부 참조가 사라졌다. 셀 선택시 Index의 데이터를 파악하기가 쉽다.


### 2. 하위 컨트롤러에서 상위 컨트롤러에게 작업 위임하기

위의 함수 안을 살펴보자. self.delegate를 호출하고 있다. 이건 왜 추가됐을까?

아까 전에 Delegate는 DataSource를 참조하는 것 외에도, 화면 전환 로직도 실행한다고 했다.

하지만 화면 전환에 관련된 NavigationBar, SearchBar, ViewController는 이 `LocationSearchRecommendationController`의 역할이라고 하기 어렵다. 

이 하위 컨트롤러의 역할은 단순하다. '추천 장소' 데이터를 관리한다. 추천 데이터를 모델에서 가져온다. 뷰에 뿌려준다. 뷰에서 셀이 선택되면, 그 장소가 어디인지 알려준다. 

그러면 그 외의 모든 로직은? 
위임한다.

하위 컨트롤러와 상위 뷰 컨트롤러를 Delegate pattern으로 연결한다. 추천 데이터 담당 컨트롤러는 delegate 프로토콜에 정의된 `delegate?.didSelectPlace(place)` 을 호출한다. 그리고 작업을 끝낸다. 

상위 ViewController의 이름은 `LocationSearchViewController`다. 이 VC는 하위 컨트롤러의 Delegate 프로토콜인 `LocationSearchRecommendationControllerDelegate`를 채택한다. 그리고 데이터가 로딩/선택되었을 때 해야할 일을 정의한다.


```swift
protocol LocationSearchRecommendationControllerDelegate: AnyObject {
    func didLoadData()
    func didSelectPlace(_ place: Place)
}
```

```swift
extension LocationSearchViewController: LocationSearchRecommendationControllerDelegate {
    func didLoadData() {
        DispatchQueue.main.async {
            self.collectionView.reloadData()
        }
    }

    func didSelectPlace(_ place: Place) {
        DispatchQueue.main.async {
            self.moveToNextVC(with: place)
        }
    }
}
```

### 여기서 잠깐, Delegate vs Closure

위의 코드에서 ViewController에게 작업을 위임하는 방법으로 Delegate 패턴을 썼다. 

> 하지만 이게 최선일까?  

Delegate 대신 Closure를 쓰는 방법도 있다. 

Delegate와 Closure는 객체간 1:1 커뮤니케이션을 가능하게 하고, 둘 다 발신하는 객체가 수신하는 객체를 몰라도 된다. 

이 둘은 약한 결합을 만든다. 다시 말해 수신 객체가 바뀌어도 발신 객체의 코드는 변화하지 않는다.

즉, 둘 다 우리가 원하는 바를 달성할 수 있다.

이 둘의 장단점은 무엇일까? 이렇게 설명해보자.

> Delegate의 장점: 써야할 코드가 많다.
> Delegate의 단점: 설명하는 코드가 많다.

> Closure의 장점: 써야할 코드가 적다.
> Closure의 단점: 설명하는 코드가 적다.


Closure는 문법이 간단해서 쓰기 쉽다. 웬만하면 Closure로 다 해결이 가능하다. 

하지만 Closure는 작업 단위로만 구성된다. 별도의 문법이나 설명이 없다. **비슷한 Closure가 많아지면, 점점 복잡해지고 읽기가 어려워진다.**

Delegate는 Protocol 타입을 추가로 구현한다. 써야할 코드가 더 많고, 약간 번거롭다. 

하지만 두 객체 간 커뮤니케이션이 protocol에 정의된다. 따라서 커뮤니케이션 흐름이 여러개 있을 때, 좀 더 정돈된 코드를 만들 수 있다. **Protocol 타입을 추가로 작성하기가 번거롭지만, 바로 그 점 때문에 복잡한 상황에서 가독성이 더 좋다.**

그래서 내가 Delegate와 Closure를 결정하는 기준이 있다. 

> 커뮤니케이션이 하나인가, 2개 이상인가.

하나라면 간단하게 Closure를, 2개 이상으로 많아지면 Delegate를 쓴다.

Apple이 구현한 API도 다소 비슷한 구석이 있다. 네트워크 요청을 수행하는 URLSession DataTask를 예로 들어보자. 

URLSession은 네트워크 요청 결과를 처리할 때 dataTask와 커뮤니케이션을 한다. 

간단한 **클로저**를 콜백으로 넘기는 방법과, **`URLSessionDataTaskDelegate`**를 구현하는 방법 2가지가 있다.


이 둘의 차이는 커뮤니케이션이 한 번이냐, 여러번이냐다. 콜백으로 넘기는 클로저는 최종 완료시에 한번 실행된다. 단순한 커뮤니케이션 흐름이다.

URLSessionDelegate는 여러 이벤트가 있다. willCacheResponse, willPerformHTTPRedirection, didCompleteWithError 등. 커뮤니케이션 흐름이 복잡하다. 그래서 Delegate를 쓰지 않았을까 생각한다. 



## 결론

위치 검색 기능으로 다시 돌아오자. 

하위 컨트롤러는, 데이터가 로딩/선택 됐을 때 작업을 상위 컨트롤러에게 위임한다. 

즉, 2개 이상의 커뮤니케이션 흐름을 가진다. 

게다가 하나의 ViewController가 3개의 컨트롤러를 가진다. 자칫 흐름이 매우 복잡해질 수 있다. 따라서 비록 써야할 코드는 많지만, Delegate 패턴을 쓰는 게 더 깔끔했다. 

코드를 리팩토링한 후 구조는 이렇게 변했다.

![before](https://user-images.githubusercontent.com/17468015/173010397-bb277c9f-6a9a-4246-bf66-4598bf070e90.png)

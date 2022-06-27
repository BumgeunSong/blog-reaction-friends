# Metatype

> [외국 블로그](https://swiftrocks.com/whats-type-and-self-swift-metatypes) 를 번역한 글입니다. 제가 직접 허락을 맡진 않았지만 다른 블로거가 허락을 맡아서 원래 글의 주소를 표시하면 번역을 해도 좋다고 하더군요. 부디 자비가 저에게도 오기를

## .self, .Type, .Protocol, .Self 등을 알아보자.

아.. 메타 타입. 이건 내가 매일 쓰면서도 인터뷰에서 내 목숨이 달렸다해도 설명하지 못하는 내용 중에 또 다른 하나였다. 

메타타입은 스위프트에서 꽤나 유용한데, 분명 여러 번 사용해왔다.
불행하게도 그들이 코드에서 꽤나 이상하게 생겨서, 이 점은 메타타입이 진짜로 무엇인가에 대해 이해하려고 노력할 때 약간의 혼란을 야기할 수 있다.

나도 이 접미사들이 어떻게 당신을 괴롭힐 지 아는 사람중에 하나지만, 당신이 그 사이의 차이점을 한번만 알아두면 그들은 사실 꽤나 직관적이다. 하지만 알아보기에 앞서, 잠시 물러나보자. 

## 메타타입이란 무엇인가?

만약 너가 애플 개발자 문서를 둘러보면, 메타타입은 타입의 타입이라고 정의되어 있다. 잠시만, `String` 이 타입이 아니라고? 스트링이 이미 타입인데, 스트링의 타입의 타입이 될 수 있는 건 대체 무어란 말인가? `SuperString`??

이론적으로는 이상하게 들리겠지만, 이는 우리가 언어를 사용하기 쉽게 하도록 우리에게 이러한 디테일을 숨기는 스위프트의 문법에 익숙해진 탓이다. 메타타입을 이해하기 위해서, 타입으로서 보는 것을 멈추고 그들을 **인스턴스와 클래스**로 바라봐야한다. 

다음과 같은 스니펫을 고려해보세요. `SwiftRocks() 와 : SwiftRocks` 를 어떻게 정의할겁니까?

```swift
struct SwiftRocks {
	static let author = "Bruno Rocha"
	func postArticle(name: String) { } 
}
let blog: SwiftRocks = SwiftRocks()
```
`SwiftRocks()`는 객체라고 할 수 있고 그리고 `SwiftRocks`는 그것의 타입이다 라고 말할 수 잇겠습니다. 그러나 `SwiftRocks()`는 인스턴스고 `: SwiftRocks`는 **그 자체로 그 인스턴스의 타입을 대표**하는 것이라고 말하려고 노력해보세요. 결국, `postArticle()`이라는 인스턴스 메서드엔 `blog`에서 접근 가능하지만, 클래스 프로퍼티인 `author`에는 접근하지 못합니다.

이제,  `author`에 어떻게 접근할까요? 가장 흔한 방법은 `SwiftRocks.author`를 통한 방법일겁니다. 이는 직접 문자열을  반환합니다만, 이 방법은 지금 만큼은 잊어버리라고 부탁하고싶네요. 다른 방법이 있을까요?

`type(of: blog).author` 가 있겠네요! 

맞아요. 이것도 맞습니다. `type(of:)`는 객체를 클래스 프로퍼티에 접근할 수 있는 무언가로 변환 시켜주니까요. 그러나 단지 `type(of: blog)`를 호출해서 무슨 일이 일어나는지 보려고 시도해본적이 있습니까?

```swift
let something = type(of: blog) // SwfitRocks.Type
```
요상한 접미사중 하나네요! SwiftRocks의 타입이 `SwiftRocks.Type`입니다. 이는 `SwiftRocks.Type` 이것이 `SwiftRocks`의 **메타타입**이라는 거죠!

xcode의 코드 자동완성으로 something의 프로퍼티를 보면, 메타타입으로 참조를 하고 있는 덕분에 타입의 모든 클래스 프로퍼티나 메서드를 사용할 수 있게 되는 것을 볼 수 있습니다. `init()`을 포함해서요!

```swift
let author: String = something.author
let instance: SwiftRocks = something.init()
```
당신이 객체를 생성하는 메서드를 원할 때 이는 아주 유용합니다.(UITableView 셀이 어떻게 재사용하고, decodable 작업을 하는지와 같이 말이죠.) 클래스 속성에 접근하거나, 단지 전반적인 객체의 타입에 기반한 행동을 할 때도요.
일반적인 방식으로 이렇게 함으로써 메타타입을 인자값으로 넘길 수 있는 쉬운 방법입니다.
```swift
func createWidget<T: Widget>(ofType: T.Type) -> T {
	let widget = T.init()
	myWidgets.insert(widget)
	return widget
}
```
메타타입은 동일성 체크를 할 때도 사용될 수 있습니다. 이는 제가 개인적으로 발견한 방법인데요 팩토리를 디자인 할 때 사용합니다.
```swift
func create<T: BlogPost>(blogType: T.Type) -> T {
	switch blogType {
		case is TutorialBlogPost.Type:
			return blogType.init(subject: currentSubject)
		case is ArticleBlogPost.Type:
			return blogType.init(subject: getLatestFeatures().random())
		case is TipBlogPost.Type:
			return blogType.init(subject: getKnowledge().random())
		default:
			fatalError("Unknown blog kind!")
	}
}
```
어떤 타입이든 메타타입을 정의할 수 있습니다. 클래스, 구조체, 열거형 , 프로토콜을 포함해서 그것의 이름뒤에 `.Type`을 붙이면 되죠. 요약하자면 `SwiftRocks`는 인스턴스의 타입을 일컫는 반면에, 메타타입인 `SwiftRocks.Type`은 클래스 스스로의 타입을 나타냅니다. 이는 여러분이 `SwiftRocks`의 클래스 프로퍼티를 사용할 수 있게 해주죠.
타입의 타입이라는 말이 이제 좀 더 이해가 되시죠?

## type(of:) 다이나믹 메타 타입 vs .self 스태틱 메타타입

이제 type(of) 가 객체의 메타타입을 반환한다고하는데, 만약 내가 객체를 가지고 있지 않는다고 하면 어떻게 될까요? `create(blogType: TutorialBlogPost.Type)`을 호출 하려고 하면 xcode는 컴파일러 에러를 냅니다.

다시 말해, 이를 할 수 없는 이유는 `myArray.append(String)` 을 하지 못하는 이유와 같습니다. String은 타입의 이름이고, 값이 아니기 때문이죠! 메타 타입을 값으로 얻기 위해서, 타입의 이름 뒤에 `.self`라고 적어줘야합니다. 

혼란스럽게 들리겠지만, 이렇게도 볼 수 있겠습니다. : `String`이 타입인 것과 마찬가지고, "Hello World"는 인스턴스의 값인 것처럼, String.Type 은 타입이고, String.self 는 메타타입의 값입니다. 

```swift
let intMetatype: Int.Type = Int.self

let widget = createWidget(ofType: MyWidget.self)
tableView.register(MyTableViewCell.self, forReuseIdentifier: "myCell") // 셀의 타입을 등록하는 것이 아닌 타입의 값을 등록해야한다..?
```
.self 는 애플에선 스태틱 메타타입이라 부릅니다. 이는 객체의 컴파일 시점에 타입을 말하는 화려한 단어입니다. 당신은 이것을 기대하는 것보다 더 사용하고 있습니다. SwiftRocks.author 를 무시하라고 했던 말 기억하나요? 그 이유는 이것을 적는게, 실은 
`SwiftRocks.self.author`를 적는 것과 같습니다.

스태틱 메타타입은 스위프트에 어느 곳에나 존재합니다. 그리고 당신은 그들을 매번 암암리에 타입의 클래스 속성에 직접 접근할 때마다 사용하고 있습니다. 아마 흥미로운 점을 발견했을 수도 있는데, 테이블뷰의 레지스터에 사용되는 AnyClass 타입은 단지  AnyObject.Type의 별칭입니다.
```swift
public typealias AnyClass = AnyObject.Type
```
반대로, type(of)는 다이나믹 메타타입을 리턴할 것입니다. 이는 실제 객체의 런타입 타입의 메타타입입니다. (여기서 부터 몬말인지..?)

```swift
let myNum: Any = 1 // 컴파일 타임의 myNum의 타입은 Any이지만, 런타임 타입은 Int입니다. 
type(of: myNum) // Int.type
```
type(of:)의 실제 콘텐츠와 그것의 메타타입 반환 타입은 컴파일러 매직입니다.(?)
여기 그 메서드의 시그니처입니다.
```swift
func type<T, Metatype>(of value: T) -> Metatype { }
```
결론적으로, 객체의 하위 클래스가 문제가된다면, type(of)를 문제를 일으키는 서브 클래스의 메타타입에 접근하기 위해서 사용하세요. 그렇지 않으면, 단순히 스태틱 메타타입에 직접적으로 (원하는 타입의 이름).self 로 접근할 수 있습니다. 

메타타입의 흥미로운 속성은 그들이 재귀적이라는 것인데, 메타, 메타 타입이 가능하다는 거죠. `SwiftRocks.Type.Type` ... -_-
우리의 온화한 정신을 위해서, 그것들의 현재 메타타입의 확상을 적는 것이 불가능하기 때문에 당신은 이것들을 그렇게 할 수 없습니다. 

## Protocol 메타타입

모든 것을 프로토콜에 적용하기에 앞서, 그들은 중요한 차이점이 있습니다. 다음의 코드는 컴파일이 안될것입니다.
```swift
protocol MyProtocol { } 
let metatype: MyProtocol.Type = MyProtocol.self 	// 타입의 값 형식으로 변환이 불가하다.
```
이러한 이유는 프로토콜의 컨텍스트에 있습니다. 
MyProtocol.Type은 프로토콜 자체의 메타타입을 참조하지 않습니다. 그러나 어떠한 타입의 메타타입도 그 프로토콜을 상속합니다. 애플은 이를 `existential metatype`이라고 합니다. 

```swift
protocol MyProtocl { } 
struct MyType: MyProtocol { }
let metatype: MyProtocol.Type = MyType.self 	// 이제 작동합니다.
```
이 경우에서 메타타입은 오직 MyProtocol 클래스 속성과 메서드에 접근할 수 있습니다.  그러나 MyType 의 구현체가 호출될것입니다. 프로토콜 타입 그자체의 구체 메타타입을 얻기 위해서, .Protocol 접미사를 사용할 수 있습니다. 다른 타입들에 .Type을 하는 것과 똑같습니다. 

```swift
let protMetatype: MyProtocol.Protocl = MyProtocol.self
```

상속을 받지 않은 프로토콜 그자체를 참조하기 때문에, protMetatype 으로 할 수 있는게 단순한 동일성 확인 말고는 없습니다. (protMetatype is MyProtocol.Protocol) . 내가 추측을 한다면, 프로토콜의 구체적인 메타타입의 목적은 컴파일러 측면 안에서의 프로토콜을 만드는 작업에 관련된다고 말씀 드릴 수 있을 것같습니다. 이것이 우리가 iOS 프로젝트에서 본적이 없는 이유입니다.

## 결론: 메타타입을 더 사용하세요!

메타 타입으로 타입을 표시하는 것은 여러분이 매우 지능적이고 타입에 안정된 일반 시스템을 빌드하는데 도움을 줍니다.  여기 우리가 어떻게 딥링크 핸들러에서 스트링을 직접적으로 다루는 것을 예방하는지에 대한 예제입니다. 
```swift
public protocol DeepLinkHandler: class {
	var handledDeepLinks: [DeepLink.Type] { get }
	func canHandle(deepLink: DeepLink) -> Bool
	func handle(deepLink: DeepLink)
}

public extension DeepLinkHandler {
	func canHandle(deepLink: DeepLink) -> Bool {
		let deepLinkTYpe = type(of: deepLink)
		// 안타깝게도 메타타입은 셋 에 들어갈 수가 없다. 해시어블을 따르지 않고 있어서.
		return handledDeepLinks.contains { $0.identifier == deepLinkType.identifier }
	}
}

class MyClass: DeepLinkHandler {
	var handledDeepLinks: [DeepLink.Type] {
		return [HomeDeepLink.self, PurchaseDeepLink.self]
	}

	func handle(deepLink: DeepLink) {
		switch deepLink {
		case let deepLink as HomeDeepLink:
			//
		case let deepLink as PurchaseDeepLink:
			//
		default: 
		//
		}
	}
}
```



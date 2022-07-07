# UICollectionView 톺아보기

## UICompositionalLayout

> A layout object that lets you combine items in highly adaptive and flexible visual arrangements.
>
> 높은 적응력과 시각적으로 유연하게 결합이 가능한 레이아웃 객체

UICompositionalLayout 이란 무엇일까. 예제 코드를 보고 따라서 쳐보기는 했지만, 제대로 살펴본적은 없어 여러가지를 시도해보며 온몸으로 느껴보려고 한다.

### 초기설정

**ViewController**

```swift
final class ViewController: UIViewController {

    private lazy var dataSource: UICollectionViewDataSource = NormalDataSource()
    private lazy var collectionView: UICollectionView = {
        let collectionView = UICollectionView(frame: .zero, collectionViewLayout: LayoutFactory.create())
        collectionView.register(MyCell.self, forCellWithReuseIdentifier: MyCell.identifier)
        collectionView.dataSource = self.dataSource
        collectionView.backgroundColor = .blue
        return collectionView
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        layoutCollectionView()
    }


}

// MARK: - Layout Section

private extension ViewController {
    func layoutCollectionView() {
        view.addSubview(collectionView)

        collectionView.snp.makeConstraints { make in
            make.edges.equalToSuperview()
        }
    }
}
```

**DataSource**

```swift
final class NormalDataSource: NSObject, UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return 3
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: MyCell.identifier, for: indexPath) as? MyCell else { return UICollectionViewCell() }

        switch indexPath.section {
        case 0:
            cell.configure(with: .red)
        case 1:
            cell.configure(with: .green)
        case 2:
            cell.configure(with: .yellow)
        default:
            cell.configure(with: .brown)
        }
        cell.configure(with: "\(indexPath.item)")
        return cell
    }

    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 3
    }
}
```

**CompositionalLayout**

```swift
enum LayoutFactory {
    static func create() -> UICollectionViewCompositionalLayout {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(0.5),
            heightDimension: .fractionalWidth(0.5)
        )

        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = .init(
            top: 10,
            leading: 10,
            bottom: 10,
            trailing: 10
        )

        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1),
            heightDimension: .fractionalWidth(0.5)
        )
      
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

        let section = NSCollectionLayoutSection(group: group)

        return UICollectionViewCompositionalLayout(section: section)
    }
}
```

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v1z65n9lj20eu0rq0tu.jpg)

### NSCollectionLayoutSize (NSCollectionLayoutDimension)

하나의 섹션에 여러개의 그룹,

하나의 그룹안에 여러개의 아이템이 있을때, 각 그룹, 아이템의 레이아웃을 설정할 수 있다.

높이와 너비에 각각 NSCollectionLayoutDimension을 적용 수 있고

NSCollectionLayoutDimension의 적용방식은 다음과 같다.

1. absolute - 절대적크기

   | itemSize.withDimension = absolute(50) | itemSize.withDimension = absolute(100) | itemSize.withDimension = absolute(150) |
   | ------------------------------------------------------------ | ------------------------------------------------------------ |---|
   | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v2fyp93bj20eu0rqmy6.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v2rhf1d0j20eu0rqjsm.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v2rw7agej20eu0rqmy8.jpg) |

2. estimated - 추정크기

   - estimate로 지정해주는 값은 초기값이며 runtime에 content size에 따라 크기가 결정

3. fractional - 상대적크기

   - item 또는 group이 **그려질 공간**에 대한 상대적 크기
   - group의 경우 collectionView의 높이와 너비 에 대한 상대적크기
   - item의 경우 group의 높이와 너비에 대한 상대적 크기


`groupSize.withDimension = .fractionalWidth(1)` 인경우 

| itemSize.withDimension = .fractionalWidth(0.1) | itemSize.withDimension = fractionalWidth(0.5) | itemSize.withDimension = fractionalWidth(1.0) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |---|
| ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v31sn80wj20eu0rqq3v.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v326iietj20eu0rq0tu.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v32j61rbj20eu0rq75a.jpg) |

​	`groupSize.withDimension = .fractionalWidth(0.5)` 인경우 

| itemSize.withDimension = .fractionalWidth(0.1)               | itemSize.withDimension = fractionalWidth(0.5)                | itemSize.withDimension = fractionalWidth(1.0)                |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v3cia5bcj20dm0qimxp.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v3cr8v5gj20eu0rq757.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v3d1yc81j20eu0rqaaw.jpg) |

### NSLayoutCollectionLayoutGroup

위 코드의 경우 

```swift
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
```

와 같이 subitems로 [NSCollectionalLayoutItem]을 사용하고있다.

하나의 그룹에 개수도 지정할수 있는데,

```swift
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: 3)
```

와 같이 subitem을 지정하고, count로 한 그룹내에 아이템 개수를 정해줄수 있다.

하나의 섹션에 6개의 아이템이 있다고 가정을 해보자.

```swift
let itemSize = NSCollectionLayoutSize(
  widthDimension: .fractionalWidth(0.5),
  heightDimension: .fractionalWidth(0.5)
)

let item = NSCollectionLayoutItem(layoutSize: itemSize)
item.contentInsets = .init(
  top: 10,
  leading: 10,
  bottom: 10,
  trailing: 10
)

let groupSize = NSCollectionLayoutSize(
  widthDimension: .fractionalWidth(1.0),
  heightDimension: .fractionalWidth(0.5)
)

let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

let section = NSCollectionLayoutSection(group: group)

return UICollectionViewCompositionalLayout(section: section)
```

레이아웃을 위와 같이 잡고, subItems만 변경 하게 되면 다음과 같다.

| (layoutSize: groupSize, subitems: [item]) 사용               | (layoutSize: groupSize, subitem: item, count: 3) 사용        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v3q3ykkzj20to1jgadl.jpg) | ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3v3qdikf5j20to1jg42j.jpg) |

itemSize의 너비를 바꾸어 주지 않아도, 하나의 그룹에 3개의 아이템이 들어간다고 지정해주면, group의 너비를 3개로 나누어 하나의 그룹에 3개의 아이템이 들어가도록 cell을 그려준다.

물론 count를 지정해주면 itemSize의 fractionalWidth값은 적용되지 않는다.



### Section별로 다른 Layout 적용하기

```swift
let collectionView = UICollectionView(frame: .zero, collectionViewLayout: LayoutFactory.create())

...

enum LayoutFactory {
    static func create() -> UICollectionViewCompositionalLayout {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(0.5),
            heightDimension: .fractionalWidth(0.5)
        )

        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = .init(
            top: 10,
            leading: 10,
            bottom: 10,
            trailing: 10
        )

        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalWidth(0.5)
        )
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
        let section = NSCollectionLayoutSection(group: group)

        return UICollectionViewCompositionalLayout(section: section)
    }
}

```

UICollectionViewCompositionalLayout 에는 다음과 같은 생성자가 있다.

`init(sectionProvider:)`

파라미터로 UICollectionViewCompositionalLayoutSectionProvider 타입을 받는데,

```swift
typealias UICollectionViewCompositionalLayoutSectionProvider = (Int, NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection?
```

이 타입은 Int, 와 NSCollectionLayoutEnvironment 를 받아 NSCollectionLayoutSection? 타입을 반환하는 클로저 타입이다.



다시 코드로 돌아가 사용해보도록 하자.

```swift
static func create() -> UICollectionViewCompositionalLayout {
  return UICollectionViewCompositionalLayout { (sectionNumber, _) -> NSCollectionLayoutSection? in
            let section: NSCollectionLayoutSection

            switch sectionNumber {
            case 0:
                section = createRedSection()
            case 1:
                section = createGreenSection()
            case 2:
                section = createYellowSection()
            default:
                section = createRedSection()
            }
            return section
        }
    }
```

기존에 create() 메소드에서 생성해주던 NSCollectionLayoutSection을 각각의 메소드가 리턴할 수 있도로 분리를 하고, 그 값을 이용해 CompositionalLayout을 생성할 수 있도록 바꾸어 주었다.

```swift
static func createRedSection() -> NSCollectionLayoutSection {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(0.5),
        heightDimension: .fractionalWidth(0.5)
    )

    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = .init(
        top: 10,
        leading: 10,
        bottom: 10,
        trailing: 10
    )

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .fractionalWidth(0.5)
    )
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
    let section = NSCollectionLayoutSection(group: group)

    return section
}

static func createGreenSection() -> NSCollectionLayoutSection {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(0.25),
        heightDimension: .fractionalWidth(0.5)
    )
		... (상동)
    return section
}

static func createYellowSection() -> NSCollectionLayoutSection {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .fractionalHeight(1.0)
    )
		... (상동)
    return section
}
```



<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h3v4zq21aog20gf0zk4qp.gif" width="300" height="600">



더 바꾸어보자

빨간색 영역은 2개씩 묶고,

초록색 영역은 4개씩 묶고, 

노란색 영역은 1개씩 묶어 오른쪽으로 스크롤이 가능하게 하고싶어 다음과 같이 변경했다.

```swift
 static func createRedSection() -> NSCollectionLayoutSection {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalWidth(0.5)
        )

      	...(나머지 상동)
   
        let group = NSCollectionLayoutGroup.vertical(layoutSize: groupSize, subitem: item, count: 2)
   
        ...
   
        section.orthogonalScrollingBehavior = .continuous
   
        return section
    }

    static func createGreenSection() -> NSCollectionLayoutSection {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalHeight(0.5)
        )
	      ...
        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(0.5),
            heightDimension: .fractionalWidth(0.5)
        )
      
      	...
      
        let group = NSCollectionLayoutGroup.vertical(layoutSize: groupSize, subitem: item, count: 2)
      
      	...
      
        section.orthogonalScrollingBehavior = .continuous

        return section
    }

    static func createYellowSection() -> NSCollectionLayoutSection {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalHeight(1.0)
        )

      	...
      
        section.orthogonalScrollingBehavior = .continuous
      
        return section
    }
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h3v5ljz066g20gf0zke81.gif" width="300" height="600">

### Group별로 다른 Layout 적용하기

위에서 살펴본 NSCollectionLayoutGroup.horizontal 의 subItems 는 [NSCollectionLayoutItem] 타입이다.

subItems에 여러개의 그룹을 넣을수 있는데,

이를 이용해 Section에 여러개의 그룹을 넣어보자.

```swift
static func createRedSection() -> NSCollectionLayoutSection {
  let firstItemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1.0),
    heightDimension: .fractionalWidth(1.0)
  )

  let firstItem = NSCollectionLayoutItem(layoutSize: firstItemSize)
  firstItem.contentInsets = .init(
    top: 10,
    leading: 10,
    bottom: 10,
    trailing: 10
  )

  let firstGroupSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(0.7),
    heightDimension: .fractionalHeight(1.0)
  )

  let firstGroup = NSCollectionLayoutGroup.vertical(layoutSize: firstGroupSize, subitem: firstItem, count: 1)

  let secondItemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1.0),
    heightDimension: .fractionalHeight(1.0)
  )

  let secondItem = NSCollectionLayoutItem(layoutSize: secondItemSize)
  secondItem.contentInsets = .init(
    top: 10,
    leading: 10,
    bottom: 10,
    trailing: 10
  )

  let secondGroupSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(0.3),
    heightDimension: .fractionalHeight(1.0)
  )

  let secondGroup = NSCollectionLayoutGroup.vertical(layoutSize: secondGroupSize, subitem: secondItem, count: 2)

  let containerGroupSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1.0),
    heightDimension: .fractionalHeight(0.5)
  )

  let containerGroup = NSCollectionLayoutGroup.horizontal(layoutSize: containerGroupSize, subitems: [firstGroup, secondGroup])

  let section = NSCollectionLayoutSection(group: containerGroup)

  section.orthogonalScrollingBehavior = .continuousGroupLeadingBoundary
  return section
}
```

코드를 살펴보면 일반적인 Group 두 개와 containerGroup 한 개, 총 3개의 그룹을 볼수 있다.

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3w20d8176j20eu0rqabb.jpg)

### ContentSize에 따른 dynamic cell (UILabel)

초록색 영역의 크기를 셀마다 다르게 바꾸어보자.

groupSize의 withDimension만 fractional, 나머지는 estimated로 바꾸어주었다.

```swift
static func createGreenSection() -> NSCollectionLayoutSection {
  let itemSize = NSCollectionLayoutSize(
    widthDimension: .estimated(1),
    heightDimension: .estimated(1)
  )

  let item = NSCollectionLayoutItem(layoutSize: itemSize)
  item.edgeSpacing = .init(
    leading: .fixed(10),
    top: .fixed(10),
    trailing: .none,
    bottom: .none
  )

  let groupSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1),
    heightDimension: .estimated(1)
  )

  let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
  let section = NSCollectionLayoutSection(group: group)

  return section
}
```

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3xb80fztyj20eu0rqabh.jpg)

estimated(1)로 설정해주어 초기 cell의 크기는 1로 그리지만, 내부 UILabel을 그릴때,  UILabel의 intrinsicContentSize에 맞추어 cell의 크기를 늘린다. ( 추측 )


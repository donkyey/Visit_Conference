# Let's Swift in 판교

---------------

### 느낌적인 느낌을 찾아서... 	- 네이버 웹툰 장수한님

##### UIView.isHidden

- 로직이 헷갈리는 경우가 종종 있다.
- true? false? 어떤거를 넣어야 하지?
- 그럼 Swift는? 왜 UIView.isHidden을 사용했을까?

##### titleLabel.isHidden = title.isEmpty

- 여기서는 문제가 없어요.
- 데이터 관점이기 때문에 이런 코드가 올 수 있었다.

- 여기서는 문제가 없어요

##### !

~~~swift
if !title.isEmpty {
	titleLabel.text = title
}
// 만약에, 아니면, 제목이, 비었으면

if title.isEmpty == false {
  title.Label.text = title
}
// 만약에, 제목이, 비었으면, 아니면

// 아래 코드가 더 가독성이 좋음
~~~

##### guard

~~~swift
guard title.isEmpty else { return }
// 이중부정

if title.isEmpty == false { return }
// 읽는데로

// guard보다는 if가 낫지 않을까?
~~~

- 그럼 guard는 필요 없을까?
  - early action(조기 탈출)
  - 로직을 조기 종료하는 코드가 없으면 빌드 에러가 나는 guard.
- Swift는 런타임이 아닌 빌드타임에 실수를 잡아내는데 중점을 둔 문법을 가지고 있다
  - 그렇기에 guard가 태어남

~~~swift
guard let nonOptionalTitle = title else
// nonOprionalTitle이 guard문 밖에서도 살아있음

if let nonOptionalTitle = title
// nonOptionalTitle은 if문 안에서만 살아있음
~~~

##### 코드를 읽는 순서, 해석, 그리고 가독성

- 코드를 읽을 때 부정이 미치는 영향은 매우 크다
- 부정을 사용할 때는 가장 마지막에 사용하는 것이 가독성이 좋다

##### 모든 기본 문법엔, 이유가 있습니다

- if, guard가 뭐가 다른지와 같이 기본 문법의 존재 이유에 대해 생각해보면 더욱 좋은 코드를 짤 수 있다.

##### Computed Property vs Method

- 동사라면 Property, 명사라면 Method

### 지저분한 Unwrapping 코드를 깔끔하게 정리해보자! - 네이버 웹툰 김형규님

##### Optional

- Nullability
- Stability 향상
- 하지만 귀찮음.....

##### Optional 지옥에서 벗어나자

- Non-Optional / Optional 데이터 분리

- Guard / if / coalescing unwrapping의 사용 구분

  - Early exit

  - 분기가 필요한 경우

  - ~~~swift
    if let name = name {
    	label?.text = name
    } else {
    	label?.text = "no name"
    }
    ~~~

  - Default value

- Collection 내의 Unwrapping 코드 줄이기

  - compactMap / map 활용

- 코드를 역할에 맞게 더 쪼개고, 그에 맞춰 unwrapping코드를 분산하기

  - 가독성을 높여준다

- Optional 변수를 unwrapping 하지 않고 사용할 방법이 없을까?

  - map / flatMap 사용하기
  - 값이 있을때만 작동...

##### Optional without Unwrap

~~~swift
func makeText(_ name: String) -> String {
	return "name is \(name)"
}

name.map(makeText)
~~~

- 함수 내의 불필요한 unwrapping 코드 삭제

##### 여러개의 Optional은?

- 여러개의 Optional이 input이 되는 경우도 map / flatMap 으로 unwrapping을 줄일 수 없을까?

- 결과

  ~~~swift
  func unwrap<A, B> (a: A?, b: B?) -> (A, B)? {
  	return a.flatMap { a -> (A, B)? in
  		b.flatMap { b -> (A, B)? in
  			(a, b)
  		}
  	}
  }
  ~~~

  - 반응은 좋지 않았어요 ㅠ

### [레거시 프로젝트에서 의존성 주입하기](https://github.com/devxoul/Pure)

#### 왜 개념만 알려주고 적용 방법은 아무도 알려주지 않는가

##### 테스트는 어렵다

- 현업에서 사용하는 코드가 테스트하기 좋게 짜여있지 않다

- Coupling이 강하다

  ~~~swift
  func fetchUser() -> Single<User> {
  	return Networking.shared.get("/user/1")
  }
  // 와이파이 있으면 통과, 없으면 실패
  // 잘못됐어요
  // 의존성을 약하게 해주어야해요
  // Production에서는 Networking을, Testing에서는 NetworkingStub로
  
  func fetchUser() -> Single<User> {
  	return self.networking.get("/user/1")
  }
  
  UserService(networking: Networking())
  // Production에서
  
  UserService(networking: NetworkingStub())
  // Testing에서
  ~~~

##### Compsition Root

- 의존성 그래프가 만들어지는 곳
- 프로그램의 진입점 -> AppDelegate

~~~swift
struct AppDependency {
	let window: UIWindow
}

extentsion AppDependency {
  static func resolve() -> AppDependency {
    return AppDependency {
      // ...
    }
  }
}
~~~

- Composition Root는 테스트 환경에서 실행되면 안돼요

##### 의존성 그래프에 속하지 못하는 이유?

- 정적 인터페이스에 의존한다

  - ~~~swift
    UserService.shared.fetchUser()
    // 여기에서
    init(userService: UserService)
    // 여기로
    ~~~

- 인스턴스를 직접 생성한다

  - 생성자로 팩토리를 전달받는다
  - 적어도 외부에서 주입할 수 있게 도ㅚㅁ

- 공통점?

  - 외존성이 외부에 위치
  - 필요에 따라 원하는 개체를 주입할 수 있다

### 민소네 실험실: 검증되지 않은 이상적이고 극단적인 iOS 프로젝트 구조 설계

##### 빌드 타임을 줄이기 위해 프레임워크 분리하기
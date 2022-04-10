# RxSwift
RxSwift를 사용하는 방법을 기초부터 세세하게 배웁니다

# 1. Hello, RxSwift!

## why Rx?

> ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.
***

_ReactiveX는 관찰 가능한 시퀀스를 사용하여 비동기 및 이벤트 기반 프로그램을 구성하기 위한 라이브러리입니다._
_잘 와닿지는 않지만 데이터 및 이벤트의 시퀀스를 지원하고 낮은 수준의 스레딩, 동기화, 스레드 안정성, 동시 데이터 구조 및 I/O 차단_
   
이라고 합니다.
또한 시간이 지남에 따라 데이터가 방출되고 그것을 바탕으로 작동합니다.

## 설치 방법

##### A. Cocoapods 설치 (m1 기준)
***
1. <pre><code>터미널을 이용하여 설치할 프로젝트 경로로 이동</code></pre>
2. <pre><code>pod init</code></pre>
3. <pre><code>open Podfile</code></pre>
4. <pre><code>텍스트 편집기에 설치할 팟 작성 후 저장</code></pre>   

<pre>
<code>
pod 'RxSwift'
pod 'RxCocoa'
pod 'RxRelay'
</code>
</pre>
5. 터미널에 <pre><code>arch -x86_64 pod install</code></pre> 입력
6. 설치 후 .xcworkspace 프로젝트로 열기

***
##### B. Add Packages 설치
> File -> Add Packages... > 검색창에 https://github.com/ReactiveX/RxSwift 입력 -> Up to Next Major Version

## Hello, RxSwift! 출력해보기

<pre>
<code>
import RxSwift

let disposeBag = DisposeBag()
Observable.just("Hello, RxSwift!")
    .subscribe { print($0) } // 반드시 subscribe를 해줘야 값이 Hello, RxSwift! 라는 String이 방출이 됩니다.
    .disposed(by: disposeBag) // 메모리에서 할당 해제
</code>
</pre>
> // Hello, RxSwift!

***

# 2. Observable   
## Observable을 만드는 두가지 방법

##### A. create 사용
<pre>
<code>
Observable<Int>.create { observer in
    observer.on(.next(0)) // 0이 담긴 Observable
    observer.onNext(1) // 1이 담긴 Observable
    
    observer.onCompleted() // observer에 값을 담는 작업이 끝나면 onCompleted()로 완료
    
    return Disposable.create() // // 마지막에 Disposables.create()를 하여 메모리에서 해제(반환타입은 Disposable)
}
</code>
</pre>

##### B. from 사용
<pre>
<code>
Observable<Int>.from([0, 1])
</code>
</pre>

* 지금은 Observable이 생성된 상태일 뿐이다   
* 즉, 현재 상태에서는 0, 1의 값이 방출이 되지 않는다   
* Observable은 이벤트가 어떤 순서로 전달되는지 정의할 뿐이다   
* Observable이 방출되는 시점은 언제일까?   
* observer가 obsubscribe을 구독하는 시점이다   
* 구독했을 때 0과 1이 순서대로 방출되고 이어서 onCompleted()이벤트가 전달된다   

#### just

<pre>
<code>
let disposeBag = DisposeBag()
let element = "ReactiveX"

Observable<String>.just(element) // just는 하나의 인자를 그대로 방출합니다 next(ReactiveX)
    .subscribe { element in print(element) }
    .disposed(by: disposeBag) 
    
Observable<String>.just(["RxSwift", "RxJava", "RxKotlin"]) // 배열도 그대로 방출합니다 next(["RxSwift", "RxJava", "RxKotlin"])
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)
</code>
</pre>

#### of

<pre><code>
let disposeBag = DisposeBag()
let rxSwift = "RxSwift"
let rxJava = "RxJava"
let rxKotlin = "RxKotlin"

Observable.of(rxSwift, rxJava, rxKotlin) // 두개 이상의 인자를 방출할 때는 of를 사용합니다
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)
    
Observable.of([rxSwift, rxJava], [rxKotlin, "RxJS"], ["RxRuby", "RxDart"]) // just와 동일하게 배열을 그대로 방출합니다
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)
</code></pre>

#### from

<pre><code>
let disposeBag = DisposeBag()
let reactiveX = ["RxSwift", "RxJava", "RxKotlin", "RxJS", "RxRuby", "RxDart"]

Observable.from(reactiveX) // 만약 배열 안에 요소들을 하나하나씩 순서대로 방출하고 싶다면 from을 사용합니다
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)
</code></pre>

> just, of는 인자를 그대로 방출합니다. 즉, String을 받으면 String을 그대로, Array를 받으면 Array를 그대로 방출합니다.([1, 2, 3] 이런식) 하지만 만약 배열 안 요소들을 하나씩 꺼내고 싶다면 just를 사용하면 배열 통째로가 아닌 배열 안의 요소들을 한개씩 순서대로 꺼낼 수 있습니다.

### filter

<pre><code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
    .filter { $0 % 2 == 0 }
    .subscribe { print($0) }
    .disposed(by: disposeBag)
</code></pre>

### flatMap

<pre><code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMap { $0.asObservable() } // flatMap은 subject를 Observable로 바꿔서 리턴합니다.
    .subscribe { print($0) }
    .disposed(by: disposeBag)
    
subject.onNext(a) // 1
subject.onNext(b) // 2
subject.onNext(111) // 111
subject.onNext(123) // 123
subject.onNext(9999) // 9999
</code></pre>

> flatMap은 먼저 어떤 값이 들어왔든 최종적으로 모든 Observable이 하나로 합쳐지고 onNext 할때마다 방출되는 항목들이 순서대로 subscribe에게 전달됩니다. flatMap은 보통 네트워크 요청을 구현할 때 자주 활용됩니다.(Json, Database)

### CombineLatest   

CombineLatest MarvelDiagram: [CombineLatestLink](https://reactivex.io/documentation/operators/combinelatest.html)

<pre><code>
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()

Observable.combineLatest(greetings, languages)
   .subscribe { print($0 + " " + $1) }
   .disposed(by: disposeBag)
   
greetings.onNext("하이") // 아무런 값을 출력하지 않습니다. 만약 구독과 동시에 값을 넘겨받고 싶다면 기본값을 주거나, BehaviorSubject(value: _)를 사용하면 됩니다
languages.onNext("Rx!") // 하이 Rx!

greetings.onNext("헬로우") // 헬로우 Rx! | 가장 최근에 전달받은 값을 subscribe에게 넘겨줍니다
languages.onNext("RxSwift!") // 헬로우 RxSwift!

greetings.onCompleted() // subscribe에 onCompleted() 메소드를 넘겨주지 않습니다
//greetings.onError(MyError.error) // 소스 Observable 중 하나라도 onError() 메소드가 전달되면 그 즉시 subscribe에 에러 이벤트를 전달하고 종료합니다

languages.onNext("RxJava!") // 헬로우 RxJava!
languages.onCompleted() // 모든 소스 Observable이 onCompleted() 메소드를 전달하면 이 시점에 completed 메시지가 전달됩니다
languages.onNext("RxKotlin!") // 따라서 RxKotlin!은 전달되지 않습니다
</code></pre>

### Binding

바인더는 UI Binding에서 사용되는 observer 입니다.   
바인더로 새로운 값을 전달하는 것(onNext)이라든지 onCompleted()를 사용하는 것은 가능하지만 onError() 메소드를 전달하는 것은 불가능합니다.   
그 이유는 UI를 관리하고 제어하는 operator인데 에러를 받게 된다면 앱이 크래쉬가 발생하거나 앱이 동작이 멈추는 것을 방지하기 위함입니다.   
또한 UI를 제어하는 것이기 때문에 Main Thread에서 실행되는 것을 보장합니다.   
***

아래 예시는 TextField에 작성한 String이 Label에 바로 업데이트 되는 간단한 앱입니다.
기존의 과정대로면 아래 과정처럼 UITextFieldDelegate 프로토콜을 추가해야 합니다.  
<pre><code>
var valueLabel = UILabel()
var valueField = UITextField()

extension ExampleController: UITextFieldDelegate {
    func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        guard let currentText = textField.text else { return }
        
        let finalText = (currentText as NSString).replacingCharacters(in: range, with: string)
        valueLabel.text = finalText
        
        return true
    }
}
</code></pre>

아래 예시는 rx의 binding을 사용한 코드입니다.
<pre><code>
import RxCocoa

var valueLabel = UILabel()
var valueField = UITextField()
let disposeBag = DisposeBag()

// valueField.rx.text
//    .observe(on: MainScheduler.instance)
//    .subscribe(onNext: { [weak self] str in
//       self.valueLabel.text = str
//    })
//    .disposed(by: disposeBag)

valueField.rx.text
   .bind(to: valueLabel.rx.text)
   .disposed(by: disposeBag)
</code></pre>
   
위의 코드 중간에 주석이 적용된 코드와 아래 코드는 동일하게 동작합니다.   
차이점이라고 한다면 코드가 더 짧아진 것을 볼 수 있어요.   
우리가 RxSwift를 사용하기 전에 GCD를 사용할 때는 DispatchQueue, OperatorQueue를 사용했습니다.   
하지만 RxSwift에서는 observe(on:)를 사용해 비동기적으로 멀티 쓰레드에서 동작하게 할 수 있습니다.   
그러나 이것보다 더 간단하게 적용이 가능한 코드가 bind(to:)를 사용해 멀티 쓰레드에서 동작함과 동시에 약한 참조(weak self)를 대신할 수도 있습니다.   

> 위에서 했던 Delegate 프로토콜을 추가하고 했던 과정보다 훨씬 간결합니다. 개발하면서 항상 이렇게 간결한 코드를 짤 수는 없겠지만 기존의 방식에 비하면 직관적이고 코드량도 적습니다.

### Obseravble의 생명주기   

##### Observable의 생명주기는 간단하게 아래와 같이 설명할 수 있습니다.

1. Create
2. Subscribe
3. onNext
4. onCompleted / onError
5. Disposed
   
> Observable(관찰대상)을 만들고, Observer는 관찰대상을 구독(subscribe)합니다. 그리고 Observable이 이벤트를 방출하면 Observer는 이벤트를 수신하고 그 이벤트를 수행합니다.

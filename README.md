# RxSwift
RxSwift를 사용하는 방법을 기초부터 세세하게 배웁니다

# 1. Hello, RxSwift!

## why Rx?

> ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.   
직역하자면 
***
_ReactiveX는 관찰 가능한 시퀀스를 사용하여 비동기 및 이벤트 기반 프로그램을 구성하기 위한 라이브러리입니다._
_잘 와닿지는 않지만 데이터 및 이벤트의 시퀀스를 지원하고 낮은 수준의 스레딩, 동기화, 스레드 안정성, 동시 데이터 구조 및 I/O 차단_
***
이라고 합니다.
또한 시간이 지남에 따라 데이터가 방출되고 그것을 바탕으로 작동합니다.

## 설치 방법

##### A. Cocoapods 설치 (m1 기준)
***
1. 터미널을 이용하여 설치할 프로젝트 경로로 이동
2. <pre><code>pod init</code></pre>
3. <pre><code>open Podfile</code></pre>
4. 텍스트 편집기에 설치할 팟 작성 후 저장
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

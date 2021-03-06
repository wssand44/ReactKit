<img src="https://avatars3.githubusercontent.com/u/8986128" width="36" height="36"> ReactKit
========

Swift Reactive Programming.


## How to install

See [Wiki page](https://github.com/ReactKit/ReactKit/wiki/How-to-install).


## Example

For UI Demo, please see [ReactKit/ReactKitCatalog](https://github.com/ReactKit/ReactKitCatalog).

### Key-Value Observing

```swift
// create signal via KVO
self.obj1Signal = KVO.signal(obj1, "value")

// bind signal via KVC (<~ as binding operator)
(obj2, "value") <~ self.obj1Signal

XCTAssertEqual(obj1.value, "initial")
XCTAssertEqual(obj2.value, "initial")

obj1.value = "REACT"

XCTAssertEqual(obj1.value, "REACT")
XCTAssertEqual(obj2.value, "REACT")
```

To remove signal-bindings, just release `signal` itself.

```swift
self.obj1Signal = nil   // release signal & its bindings

obj1.value = "Done"

XCTAssertEqual(obj1.value, "Done")
XCTAssertEqual(obj2.value, "REACT")
```

### NSNotification

```swift
self.signal = Notification.signal("MyNotification", obj1).map { notification -> NSString? in
    return "hello" // convert NSNotification? to NSString?
}

(obj2, "value") <~ self.signal
```

Normally, `NSNotification` itself is useless value for binding with other objects, so use [signal-operations](#signal-operations) e.g. `map(f: T -> U)` to convert it.

### Target-Action

```swift
// UIButton
self.buttonSignal = self.button.buttonSignal("OK")

// UITextField
self.textFieldSignal = self.textField.textChangedSignal()

^{ println($0) } <~ self.buttonSignal     // prints "OK" on tap

// NOTE: ^{ ... } = closure-first operator, same as `signal ~> { ... }`
^{ println($0) } <~ self.textFieldSignal  // prints textField.text on change
```

### Complex example

The example below is taken from

- [iOS - ReactiveCocoaをかじってみた - Qiita](http://qiita.com/paming/items/9ac189ab0fe5b25fe722) (well-illustrated)

where it describes 4 `UITextField`s which enables/disables `UIButton` at certain condition:

```swift
let usernameTextSignal = self.usernameTextField.textChangedSignal()
let emailTextSignal = self.emailTextField.textChangedSignal()
let passwordTextSignal = self.passwordTextField.textChangedSignal()
let password2TextSignal = self.password2TextField.textChangedSignal()

let anyTextSignal = Signal.any([usernameTextSignal, emailTextSignal, passwordTextSignal, password2TextSignal])

// create button-enabling signal via any textField change
self.buttonEnablingSignal = anyTextSignal.map { (values, changedValue) -> NSNumber? in

    let username: NSString? = values[0]?
    let email: NSString? = values[1]?
    let password: NSString? = values[2]?
    let password2: NSString? = values[3]?

    // validation
    let buttonEnabled = username?.length > 0 && email?.length > 0 && password?.length >= MIN_PASSWORD_LENGTH && password? == password2?

    // NOTE: use NSNumber because KVO does not understand Bool
    return NSNumber(bool: buttonEnabled)
}

// REACT: enable/disable okButton
(self.okButton, "enabled") <~ self.buttonEnablingSignal!
```

For more examples, please see XCTestCases.


## How it works

ReactKit is based on powerful [SwiftTask](https://github.com/ReactKit/SwiftTask) (JavaScript Promise-like) library, allowing to deliver multiple events (KVO, NSNotification, Target-Action, etc) continuously over time using its `progress` feature (`<~` operator in ReactKit).

Unlike many Reactive Extensions (Rx) libraries including [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) which has a basic concept of "hot" and "cold" signals, ReactKit gracefully integrated them into one **hot + paused (lazy) signal** `Signal<T>` class as well as **not adopting the concept of Disposable** but using `signal.cancel()` feature instead.

Also in ReactKit, Rx's `signal.subscribe(onNext, onError, onComplete)` method is interpreted as `signal.progress()` (`<~`) and `signal.then()`/`success()`/`failure()` separately. When they are called on needs, they will start lazy evaluation.


## Methods

### Signal Operations

- Instance Methods
  - `filter(f: T -> Bool)`
  - `filter2(f: (old: T?, new: T) -> Bool)`
  - `map(f: T -> U)`
  - `flatMap(f: T -> Signal<U>)`
  - `map2(f: (old: T?, new: T) -> U)`
  - `mapAccumulate(initialValue, accumulator)` (alias: `scan`)
  - `take(count)`
  - `takeUntil(signal)`
  - `skip(count)`
  - `skipUntil(signal)`
  - `merge(signal)`
  - `concat(signal)`
  - `startWith(initialValue)`
  - `zip(signal)`
  - `buffer(count)`
  - `bufferBy(signal)`
  - `delay(timeInterval)`
  - `throttle(timeInterval)`
  - `debounce(timeInterval)`
- Class Methods
  - `merge(signals)`
  - `merge2(signals)` (generalized method for `merge` & `combineLatest`)
  - `combineLatest(signals)`
  - `concat(signals)`
  - `zip(signals)`

### Helpers

- `asSignal(ValueType)` (WARNING: currently works for non-Optional only)
- `Signal.once(value)` (alias: `just`)
- `Signal.never()`
- `Signal.fulfilled()` (alias: `empty`)
- `Signal.rejected()` (alias: `error`)
- `Signal(values:)` (a.k.a Rx.fromArray)


## Dependencies

- [SwiftTask](https://github.com/ReactKit/SwiftTask)
- [SwiftState](https://github.com/ReactKit/SwiftState)


## References

- [Introducing ReactKit // Speaker Deck](https://speakerdeck.com/inamiy/introducing-reactkit) (ver 0.3.0)


## Licence

[MIT](https://github.com/ReactKit/ReactKit/blob/master/LICENSE)

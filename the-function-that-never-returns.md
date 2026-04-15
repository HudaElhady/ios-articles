## So what is the problem?

```swift
func divide(_ a: Int, by b: Int) -> Int {
    if b == 0 {
        fatalError("b input must not be zero")
    }
    return a / b
}

func greeting(_ name: String) -> String {
    guard !name.isEmpty
    else {
        return "Hello anonymous"
    }
    
    return "Hello \(name)"
}

```

We know the compiler before running the code, will check that every function is returning the value it promised to return in every single code branch

So how come the compiler is happy with both of the above functions? the `greeting` function makes sense it returns string in all branches

but the `divide` function clearly not, it promised to return Int and the `b == 0` branch doesn't, instead it has `fatalError()`

The compiler was ok because `fatalError` means this branch is a deadend, the control will never return back to the `divide` function caller instead the application will terminate

so the compiler knows how this path of code will end and so no point of adding return after `fatalError` since it won't be executed

Great, but wait how the compiler knew that the `fatalError` means that?

## Never

`fatalError` return type is `Never`, which means that this code will not return and will never hand the control back to the function caller `Never` actually is an enum with zero cases `enum Never {}` We know to construct an enum you just pick one of it's cases

```swift
enum Mode {
    case horizontal
    case vertical
}

var verticalMode = Mode.vertical
```

and so an empty enum like `Never`, can't be constructed and it makes sense because what's better return type to a code that promisises to never return than a type that can't be constructed

But `Never` is **not** enough to make the compiler happy, the code of the function itself should not hand the control back to the caller meaning the compiler then verifies that every code path inside the function actually never returns

for example this code won't compile, because this function promises to never return (-> Never) but nothing in code prevent it from returning

```swift
func greeting() -> Never {
    print("Hello")
}
```

**so What are the cases where** `Never` **would be a good fit?**

*   When we need some code branch to terminate the app, like in `fatalError`
    
*   When we need some code to just run forever
    

```swift
func runForever() -> Never {
    while true {
        
    }
}
```

*   When you are forced to return Result, but you know it will never fail
    

```swift
protocol DataProvider {
    associatedtype Failure: Error
    func fetchData() -> Result<String, Failure>
}

class MockDataProvider: DataProvider {
    func fetchData() -> Result<String, Never> {
        return .success("mock response")
    }
}
```

`Never` conforms to `Error`, so it satisfies the constraint the compiler is ok because it knows the error failure case will not happen

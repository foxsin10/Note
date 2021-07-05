# Combine issues

## Memory leak for `assiont(to:on)` method

when `Root` refers to an `ReferenceType`, the following method will cause retainSycle

see [here](https://forums.swift.org/t/does-assign-to-produce-memory-leaks/29546/13)

```swift
extension Publisher where Self.Failure == Never {
    public func assign<Root>(to keyPath: ReferenceWritableKeyPath<Root, Self.Output>, on object: Root) -> AnyCancellable
}
```

use this function instead

```swift
extension Publisher where Self.Failure == Never {
    public func assign<Root: AnyObject>(to keyPath: ReferenceWritableKeyPath<Root, Self.Output>, on object: Root) -> AnyCancellable {
        sink { [weak object] value in
            object?[keyPath: keyPath] = value
        }
    }
}
```

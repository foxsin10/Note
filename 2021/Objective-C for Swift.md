# Marcos for Swift Interface in Objective-C

## nonull and nullable

```Objective-C
#import <Foundation/Foundation/h>

NS_ASSUME_NONULL_BEGIN

NSString * _Nonnull const SKRocketSaturnV;

@interface SKMission: NSObject

@property (nonatomic, readonly, nullable) NSString *name;

- (instancetype)initWithName: (nullable NSString *)name;

- (BOOL)getResiurceValue:(id _Nullable * _Nonull)outValue error: (NSError **)error;

@end

NS_ASSUME_NONULL_END
```

// null_unspecified _Null_unspecified

## Type alias

before

```Objective-C
NSString *const SKRocketTitanII;
NSString *const SKRocketAtlas;
NSString *const SKRocketSaturnV;

NSInteger SKRocketStageCount(NSString *);
```

```swift
public let SKRocketTitanII: String
public let SKRocketAtlas: String
public let SKRocketSaturnV: String

public func SKRocketStageCount(_: String) -> Int
```

after

```Objective-C
typedef NSString *SKRocket NS_STRING_ENUM;

extern SKRocket const SKRocketTitanII;
extern SKRocket const SKRocketAtlas;
extern SKRocket const SKRocketSaturnV;

NSInteger SKRocketStageCount(SKRocket);
```

```swift
public struct SKRocket : Hashable, Equatable, RawRepresentable {
    public init(rawValue: String)
}
extension SKRocket {
    public static let titanII: SKRocket

    public static let atlas: SKRocket

    public static let saturnV: SKRocket
}

public func SKRocketStageCount(_: SKRocket) -> Int
```


## Initializer

before
```Objective-C
- (instancetype)initWithNameComponents: (NSPersonNameComponents *)name;
- (instancetype)initWithName: (NSString *)name;
```

```swift
public init(nameComponents name: PersonNameComponents)
public init(name: String)
public init()
```

after
```Objective-C
- (instancetype)initWithNameComponents: (NSPersonNameComponents *)name NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithName: (NSString *)name;
- (instancetype)init NS_UNAVAILABLE;
```
NS_DESIGNATED_INITIALIZER will make the second method as `convenience` to call first method to initialize instance

```swift
public init(nameComponents name: PersonNameComponents)
public convenience init(name: String)
```
NS_REFINED_FOR_SWIFT add `__` before func or property


## Naming

```Objective-C
- (NSSet<SKMission *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut NS_SWIFT_NAME(previousMissions(flownBy:))
```

```swift
func previousMissions(flowBy astronaut: SKAstronaut) -> Set<SKMission>
```

```Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH2 = 1,
    SKFuelKindC12H26 = 2
} 
NS_SWIFT_NAME(SKFuel.Kind);

MSString *SKFuelKindToString(SKFuelKing)
NS_SWIFT_NAME(
    string(from:)
);
NS_SWIFT_NAME(
    getter:SKFuelKind.description(self:)
)
```

```swift
extension SKFuel {
    public enum Kind: Int {
        case H2 = 0
        case CH4 = 1
        case C12H26 = 2
    }
}

public func string(from: SKFuel.Kind) -> String
extension SKFuel.Kind {
    public var description: String { get }
}
```

```Objective-C
const NSString *SKErrorDomain;

typedef NS_ERROR_ENUM(SKErrorDomain, SKErrorCode) {
    SKErrorLaunchAborted = 1,
    SKErrorLaunchOutOfRange,
    SKErrorWaterFlow
} 
```

```swift
public let SKErrorDomain: String
public struct SKError {
    public enum Code: Int { 
        case launchAborted = 1
        case launchOutOfRange 
        case waterFlow
    }

    public static var lanchAboted: SKError.Code { get }
    public static var launchOutOfRange: SKError.Code { get }
    public static var waterFlow: SKError.Code { get }

    public static var errorDomain: String { get }
}
```
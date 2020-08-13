# Phi Swift Style Guide

## Goals

Following this style guide should:

* Make it easier to read and begin understanding unfamiliar code.
* Make code easier to maintain.
* Reduce simple programmer errors.
* Reduce cognitive load while coding.
* Keep discussions on diffs focused on the code's logic rather than its style.

## Guiding Tenets

* This guide is in addition to the official [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). These rules should not contradict that document.
* These rules should not fight Xcode's <kbd>^</kbd> + <kbd>I</kbd> indentation behavior.
* We strive to make every rule lintable:
  * If a rule changes the format of the code, it needs to be able to be reformatted automatically (either using [SwiftLint](https://github.com/realm/SwiftLint) autocorrect or [SwiftFormat](https://github.com/nicklockwood/SwiftFormat)).
  * For rules that don't directly change the format of the code, we should have a lint rule that throws a warning.
  * Exceptions to these rules should be rare and heavily justified.


## Regarding Frameworks

Please do not import frameworks "just because". Try to do things natively, only importing frameworks. Remember that this is not a regular app. We need to be lightweight.

## Xcode Templates

[Clone Phi Xcode Templetes]() WIP

- Installation instructions in `README.md` file

## Naming

Using descriptive names makes code easier to read and understand. Use the Swift naming conventions described in the [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). Also refer to the Clean Code's chapter on naming for more examples. Remember what was said at the Comment's section and be aware that property/parameter/method names should be enough documentation. Make sure that their purpose can be fully understood purely by reading its name.

# Views

The views on the project should follow the following structure:

### Setup / ViewConfiguration

The initial setup of a view should happen inside a `setup()` method called inside the `UIView`'s init.

```swift
override init(frame: CGRect) {}
    super.init(frame: frame)

    setup()
}
```

-----

#### ViewConfiguration Protocol

> We use a protocol to better organize the hierarchy and settings of our views, both in ViewControllers and Custom Views

```swift
public protocol ViewConfiguration: AnyObject {
    func setupConstraints()
    func buildViewHierarchy()
    func configureViews()
    func setupViewConfiguration()
}

extension ViewConfiguration {
    public func setupViewConfiguration() {
        buildViewHierarchy()
        setupConstraints()
        configureViews()
    }

    public func configureViews() {
    }
}
```

- Usage sample

```swift
import UIKit
import PhiUIKit

public final class TitleAndDescriptionInfoView: UIView {
    // MARK: - Views
    
    // stackTexts container for texts: UIStackView
    private let stackTexts: UIStackView = {
        let stackTexts = UIStackView()
        stackTexts.axis = .vertical
        stackTexts.distribution = .fillProportionally
        stackTexts.alignment = .fill
        stackTexts.spacing = 10

        return stackTexts
    }()
    // titleLabel: PhiUILabel
    private let titleLabel: PhiUILabel = {
        let titleLabel = PhiUILabel(text: "")
        titleLabel.textColor = ColorPallet.green.color

        return titleLabel
    }()
    // descriptionLabel: PhiUILabel
    private let descriptionLabel = PhiUILabel(text: "")

    // MARK: - Initialization
    ///Create a stack with optional `String` title and `String` description
    public init() {
        super.init(frame: .zero)

        setupViewConfiguration()
    }

    @available(*, unavailable)
    public required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    /**
       Configuration for view
       - Parameters:
          - title: `String?` for any texts
          - description: `String` for any texts
    */
    public func configure(title: String?, description: String) {
        titleLabel.text = title
        descriptionLabel.text = description
    }
}

// MARK: - ViewConfiguration
extension TitleAndDescriptionInfoView: ViewConfiguration {
    public func setupConstraints() {
        stackTexts.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            stackTexts.topAnchor.constraint(equalTo: topAnchor),
            stackTexts.leadingAnchor.constraint(equalTo: leadingAnchor),
            stackTexts.trailingAnchor.constraint(equalTo: trailingAnchor),
            stackTexts.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }

    public func buildViewHierarchy() {
        stackTexts.addArrangedSubview(titleLabel)
        stackTexts.addArrangedSubview(descriptionLabel)

        addSubview(stackTexts)
    }
}
```


-----


The constraints of each subview should be added inside the setup of that subview. It’s important to always remember to add the subview to it’s superview before adding any constraints and pay attention to the order on which the setup methods are called.

```swift
private func setup() {
    setupTableView()
    setupEmptyState()
    setupLoadingView()
    setupSegmentedControl()
}
```

Actions of buttons and other views are configured in the setup method of that view.


## Assets & Strings

### Asset Enum

```swift
import UIKit

public protocol AssetProtocol {
    var name: String { get }
    var image: UIImage? { get }
}

public enum Asset: String, AssetProtocol {
    enum Close: String, AssetProtocol {
        case closeRounded
        case simpleClose

        public var name: String {
            return self.rawValue
        }

        public var image: UIImage? {
            return UIImage(named: self.rawValue, in: AssetKit.getBundle(), compatibleWith: nil)
        }
    }

    public var name: String {
        return self.rawValue
    }

    public var image: UIImage? {
        return UIImage(named: self.rawValue, in: AssetKit.getBundle(), compatibleWith: nil)
    }
}
```

### ***Important***

Import assets of icons such as `.pdf` vector, Make sure **Devices** is checked with `Universal` and **Scales** is marked as `Single scale`


### i18n Enum

```swift
protocol i18nProtocol {
    var text: String { get }

    func text(with complement: String...) -> String
}

enum i18n {

    // MARK: - General

    enum General: String, i18nProtocol {
        case yes
        case no
        case ok
        case back
        case cancel
        case `continue`
        case register
        /* ... */

        var text: String {
            return NSLocalizedString(self.rawValue, comment: "")
        }

        func text(with complement: String...) -> String {
            return String(format: NSLocalizedString(self.rawValue, comment: "%@"), arguments: complement)
        }
    }
```

* Usage

```swift
let message = i18n.General.questionSignOut.text
```
```
"GREETING"="Hello %@. Lorem ipsum";

```

> When you need to use string replacement use the `%@` wildcard in the string file. and implement as an example:

```swift
let message = i18n.General.greeting.text(with: "John Doe")
```


## Optionals

> The only time you should be using implicitly unwrapped optionals is with @IBOutlets. In every other case, it is better to use a non-optional or regular optional property. Yes, there are cases in which you can probably "guarantee" that the property will never be nil when used, but it is better to be safe and consistent. Similarly, don't use force unwraps.

Don't use `as!` or `try!`.

If you don't plan on actually using the value stored in an optional, but need to determine whether or not this value is `nil`, explicitly check this value against `nil` as opposed to using `if let` syntax.

```swift
// PREFERRED
if someOptional != nil {
    // do something
}

// NOT PREFERRED
if let _ = someOptional {
    // do something
}
```

Don't use `unowned`. You can think of `unowned` as somewhat of an equivalent of a `weak` property that is implicitly unwrapped (though unowned has slight performance improvements on account of completely ignoring reference counting). Since we don't ever want to have implicit unwraps, we similarly don't want `unowned` properties.

```swift
// PREFERRED
weak var parentViewController: UIViewController?

// NOT PREFERRED
weak var parentViewController: UIViewController!
unowned var parentViewController: UIViewController
```

* When unwrapping optionals, use the same name for the unwrapped constant or variable where appropriate.

```swift
guard let myValue = myValue else {
    return
}
```

* Use `XCTUnwrap` instead of forced unwrapping in tests.

```swift
func isEvenNumber(_ number: Int) -> Bool {
    return number % 2 == 0
}

// PREFERRED
func testWithXCTUnwrap() throws {
    let number: Int? = functionThatReturnsOptionalNumber()
    XCTAssertTrue(isEvenNumber(try XCTUnwrap(number)))
}

// NOT PREFERRED
func testWithForcedUnwrap() {
    let number: Int? = functionThatReturnsOptionalNumber()
    XCTAssertTrue(isEvenNumber(number!)) // may crash the simulator
}
```

## Protocols

When implementing protocols, there are two ways of organizing your code:

1. Using `// MARK:` comments to separate your protocol implementation from the rest of your code
2. Using an extension outside your `class`/`struct` implementation code, but in the same source file

Keep in mind that when using an extension, however, the methods in the extension can't be overridden by a subclass, which can make testing difficult. If this is a common use case, it might be better to stick with method #1 for consistency. Otherwise, method #2 allows for cleaner separation of concerns.

Even when using method #2, add `// MARK:` statements anyway for easier readability in Xcode's method/property/class/etc. list UI.

## Properties

* If making a read-only, computed property, provide the getter without the `get {}` around it.

```swift
var computedProperty: String {
    if someBool {
        return "I'm a mighty pirate!"
    }
    return "I'm selling these fine leather jackets."
}
```

* When using `get {}`, `set {}`, `willSet`, and `didSet`, indent these blocks.
* Though you can create a custom name for the new or old value for `willSet`/`didSet` and `set`, use the standard `newValue`/`oldValue` identifiers that are provided by default.

```swift
var storedProperty: String = "I'm selling these fine leather jackets." {
    willSet {
        print("will set to \(newValue)")
    }
    didSet {
        print("did set from \(oldValue) to \(storedProperty)")
    }
}

var computedProperty: String  {
    get {
        if someBool {
            return "I'm a mighty pirate!"
        }
        return storedProperty
    }
    set {
        storedProperty = newValue
    }
}
```

* You can declare a singleton property as follows:

```swift
class PirateManager {
    static let shared = PirateManager()

    /* ... */
}
```

  
## Using `guard` Statements  

> When unwrapping optionals, prefer guard statements as opposed to if statements to decrease the amount of nested indentation in your code.


```swift
// PREFERRED
guard let monkeyIsland = monkeyIsland else {
    return
}
bookVacation(on: monkeyIsland)
bragAboutVacation(at: monkeyIsland)

// NOT PREFERRED
if let monkeyIsland = monkeyIsland {
    bookVacation(on: monkeyIsland)
    bragAboutVacation(at: monkeyIsland)
}

// EVEN LESS PREFERRED
if monkeyIsland == nil {
    return
}
bookVacation(on: monkeyIsland!)
bragAboutVacation(at: monkeyIsland!)
```

> Don’t use one-liners for guard statements.

```swift
// PREFERRED
guard let thingOne = thingOne else {
    return
}

// NOT PREFERRED
guard let thingOne = thingOne else { return }
```

## Subview Creation with UIKit

All subviews should be created and configured using closures. Any setup that doesn’t depend on state or dynamic information must be done inside the closure. Do not use `lazy var` if not really necessary. Prefer to configure actions in action methods isolating their behavior, if you need to configure information to be loaded later use configure methods. 

```swift 
// PREFERRED
let confirmButton: RoundedButton = {
    let button = RoundedButton()
    button.setTitle(i18n.confirm, for: .normal)
    button.isEnabled = false

    return button
}()

// MARK: - Actions

private func actions() {
	confirmButton.addTarget(self, action: #selector(confirmButtonTapped), for: .touchUpInside)
	/** **/
}

// NOT PREFERRED
lazy var confirmButton: RoundedButton = {
    let button = RoundedButton()
    button.setTitle(i18n.confirm, for: .normal)
    button.isEnabled = false
	 button.addTarget(self, action: #selector(confirmButtonTapped), for: .touchUpInside)

    return button
}()
```

> Keep a blank line between the last configuration line and the return

## Documentation / Comments

### Documentation

If a function is more complicated than a simple O(1) operation, you should generally consider adding a doc comment for the function since there could be some information that the method signature does not make immediately obvious. If there are any quirks to the way that something was implemented, whether technically interesting, tricky, not obvious, etc., this should be documented. Documentation should be added for complex classes/structs/enums/protocols and properties. All `public` functions/classes/properties/constants/structs/enums/protocols/etc. should be documented as well (provided, again, that their signature/name does not make their meaning/functionality immediately obvious).

After writing a doc comment, you should option click the function/property/class/etc. to make sure that everything is formatted correctly.

Be sure to check out the full set of features available in Swift's comment markup [described in Apple's Documentation](https://developer.apple.com/library/tvos/documentation/Xcode/Reference/xcode_markup_formatting_ref/Attention.html#//apple_ref/doc/uid/TP40016497-CH29-SW1).

Guidelines:

* 160 character column limit (like the rest of the code).

* Even if the doc comment takes up one line, use block (`/** */`).

* Do not prefix each additional line with a `*`.

* Use the new `- parameter` syntax as opposed to the old `:param:` syntax (make sure to use lower case `parameter` and not `Parameter`). Option-click on a method you wrote to make sure the quick help looks correct.

```swift
class Human {
    /**
     This method feeds a certain food to a person.

     - parameter food: The food you want to be eaten.
     - parameter person: The person who should eat the food.
     - returns: True if the food was eaten by the person; false otherwise.
    */
    func feed(_ food: Food, to person: Human) -> Bool {
        // ...
    }
}
```

* If you’re going to be documenting the parameters/returns/throws of a method, document all of them, even if some of the documentation ends up being somewhat repetitive (this is preferable to having the documentation look incomplete). Sometimes, if only a single parameter warrants documentation, it might be better to just mention it in the description instead.

* For complicated classes, describe the usage of the class with some potential examples as seems appropriate. Remember that markdown syntax is valid in Swift's comment docs. Newlines, lists, etc. are therefore appropriate.

```swift
/**
 ## Feature Support

 This class does some awesome things. It supports:

 - Feature 1
 - Feature 2
 - Feature 3

 ## Examples

 Here is an example use case indented by four spaces because that indicates a
 code block:

     let myAwesomeThing = MyAwesomeClass()
     myAwesomeThing.makeMoney()

 ## Warnings

 There are some things you should be careful of:

 1. Thing one
 2. Thing two
 3. Thing three
 */
class MyAwesomeClass {
    /* ... */
}
```

* When mentioning code, use code ticks - \`

```swift
/**
 This does something with a `UIViewController`, perchance.
 - warning: Make sure that `someValue` is `true` before running this function.
 */
func myFunction() {
    /* ... */
}
```

* When writing doc comments, prefer brevity where possible.

### Other Commenting Guidelines

* Always leave a space after `//`.
* Always leave comments on their own line.
* When using `// MARK: - whatever`, leave a newline after the comment.

```swift
class Pirate {

    // MARK: - instance properties

    private let pirateName: String

    // MARK: - initialization

    init() {
        /* ... */
    }

}
```

## Protocol Conformance

In particular, when adding protocol conformance to a model or view, prefer adding a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class.

```swift
extention MyModel: SomeProtocol {
    func someProtocolRequiredMethod() -> Int {
        return 10
    }
}
```

For class protocols, use `: AnyObject` instead of `: class`. The latter is popular, but it's not supposed to be used.

## Functions With an Implicit Return

If the entire body of the function is a single expression, the function implicitly returns that expression. But prefer to use the explicit return.

```swift 
// PREFERRED
func anotherGreeting(for person: String) -> String {
    return "Hello, " + person + "!"
}

// NOT PREFERRED
func greeting(for person: String) -> String {
    "Hello, " + person + "!"
}
```

## Unused Code

Unused (dead) code should be removed. Don't worry about losing stuff, that's what Git is for :smile:



## Final
All classes or members of a class that are not meant to be overriden should be marked as final.


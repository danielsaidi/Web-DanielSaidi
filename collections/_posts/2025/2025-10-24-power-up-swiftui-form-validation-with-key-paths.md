---
title:  Power-Up SwiftUI Form Validation with Key Paths
date:   2025-10-24 06:00:00 +0000
tags:   swift swiftui

assets: /assets/blog/25/1024/
image:  /assets/blog/25/1024/image.jpg
image-show: 0

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3m3xrmbbsbk2p
toot: https://mastodon.social/@danielsaidi/115431167834695198
---

I struggled to start using keypaths in Swift, but I'm really glad that they're now a go-to tool to reduce repetition and increase readabily. Let's take a look at how to use keypaths with form validation.


## Form and request models

Consider that we have an observable user registration form model, that can be used with a SwiftUI
form, to let the form automatically observe changes in the model:

```swift
/// This type is a namespace for user registration types.
struct UserRegistration {}

extension UserRegistration {

    /// This observable type can be used to collect user registration data.
    class FormData: ObservableObject {

        typealias StringPath = ReferenceWritableKeyPath<UserRegistration.FormData, String>

        @Published var email = ""       // A valid email address
        @Published var password = ""    // At least 6 characters
        @Published var passwordConfirm = ""
        @Published var subscribeToNewsletter = false

        @Published var validationState = [StringPath: Bool]()
    }
}
```

Note that we have already defined a `StringPath` shorthand, which we can use to refer to any string value in our model, as well as a `validatedState` where we can store validation data. 

The idea is to use this class to collect data, then map it to this `Sendable` request type when all form data values are successfully validated:

```swift
public extension UserRegistration {

    /// This sendable type can be used as an API request value.
    struct RequestData: Sendable {

        public let email: String
        public let password: String
        public let subscribeToNewsletter: Bool
    }
}

public extension UserRegistration.FormData {

  func requestData() -> UserRegistration.RequestData? {
      guard isValid else { return nil } // <-- We'll add validation soon
      return .init(
          email: email,
          password: password,
          subscribeToNewsletter: subscribeToNewsletter
      )
  }
}
```

Let's now look at how key paths can help us streamline the validation process of our form data type.


## Form validation without key paths

If you have built form validation before, your may already be writing up the following in your head:

```swift
public extension UserRegistration.FormData {

    var isValid: Bool { isEmailValid && isPasswordValid }

    var isEmailValid: Bool { ... }
    var isPasswordValid: Bool { ... }
}
```

In this case, where we only have two properties, that would be perfectly fine. But imagine if we'd be collecting a user name and personal information as well:

```swift
public extension UserRegistration.FormData {

    var isValid: Bool { isEmailValid && isPasswordValid && ... }

    var isEmailValid: Bool { ... }
    var isPasswordValid: Bool { ... }
    var isFirstNameValid: Bool { ... }
    var isLastNameValid: Bool { ... }
    var isCityValid: Bool { ... }
    ...
}
```

As you can see, having individual properties like this doesn't scale, and would require us to add at least two properties for each new form value. 

Individual properties also increase the risk for bugs, as we will have to write more code to handle each form data property. Lets see how we can use keypath to improve things.

## Form validation with key paths

With keypaths, we can add a bunch of validation logic to our form data model with very little code:

```swift
public extension UserRegistration.FormData {

    var isValid: Bool {
        validationKeyPaths.allSatisfy(isValid)
    }

    var validationKeyPaths: [StringPath] {
        [\.email, \.password, \.passwordConfirm]
    }

    func isValid(_ path: StringPath) -> Bool {

        // Require that all values are non-empty
        let text = stringValue(for: path)
        guard text.hasContent else { return false }

        // Perform property-specific validation
        switch path {
        case \.email: return text.isValidEmail
        case \.password: return text == passwordConfirm
        }
    }

    func stringValue(for path: StringPath) -> String {
        self[keyPath: path].trimmed()
    }
}

private extension String {

    var isValidEmail: Bool {
        let regex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES %@", regex)
        let result = predicate.evaluate(with: self)
        print("\(self) -> \(result)")
        return result
    }
}
```

We can now make `requestData()` only return a value when the form data is completely valid:

```swift
func requestData() -> UserRegistration.RequestData? {
    guard isValid else { return nil }
    return .init(
        email: email,
        password: password,
        subscribeToNewsletter: subscribeToNewsletter
    )
}
```

## View-based form validation

With these additions, we can now let `SwiftUI` validate a form view as the user fills out the details:

```swift
public struct UserRegistrationForm: View, Localized {

    @StateObject private var formData = UserRegistration.FormData()

    public var body: some View {
        Section(translate(.user)) {
            TextField("Form.Email", text: $formData.email)
                .textContentType(.emailAddress)
                .foregroundStyle(textFieldColor(for: \.email))
            TextField("Form.Password", text: $formData.password)
                .textContentType(.newPassword)
                .foregroundStyle(textFieldColor(for: \.password))
            TextField("Form.PasswordConfirm", text: $formData.passwordConfirm)
                .textContentType(.newPassword)
                .foregroundStyle(textFieldColor(for: \.passwordConfirm))
        }
        Section {
            Toggle(
                "Form.SubscribeToNewsletter", 
                isOn: $formData.subscribeToNewsletter
            )
            .font(.footnote)
        }
    }
}

private extension UserRegistrationForm {

    func textFieldColor(
        for keyPath: UserRegistration.FormData.StringPath
    ) -> Color {
        formData.isValid(keyPath) ? .primary : .red
    }
}
```

Since the `formData` property is observable, it will automatically refresh the form as we fill it out. Text fields with invalid texts will be red, while the ones with valid text will be primary colored.

However, if we look at the code, it's quite repetetive. We create a text field for each property with a placeholder, a text binding, a content type and a foreground style. Let's simplify things.

We can gather all the logic in a single function by using the form data key path we did set up earlier:

```swift
private extension UserRegistrationForm {

    func textField(
        _ title: LocalizedStringKey,
        _ keypath: StringPath,
        _ type: UITextContentType
    ) -> some View {
        TextField(translate(title), text: $formData[dynamicMember: keypath])
            .textContentType(type)
            .foregroundStyle(textFieldColor(for: \.userName))
    }
}
```

By using `$formData[dynamicMember:]` we can create a text binding for the keypath. This lets us reduce the code in our form to this:

```swift
public struct UserRegistrationForm: View, Localized {

    @StateObject private var formData = UserRegistration.FormData()

    public var body: some View {
        Section(translate(.user)) {
            textField("Form.Email", \.email, .emailAddress)
            textField("Form.Password", \.password, .newPassword)
            textField("Form.PasswordConfirm", \.passwordConfirm, .newPassword)
        }
        Section {
            Toggle(
                "Form.SubscribeToNewsletter", 
                isOn: $formData.subscribeToNewsletter
            )
            .font(.footnote)
        }
    }
}
```

By combining key paths, extensions and functions, we have created way to validate forms in a way that scales when we add more properties, and that results in safer and more readable code.


## On-submit validation

Before we're done for now, there's one more thing to consider. The form will now constantly update itself as the user types, and mark all invalid text red.

This however means that as we start typing `daniel.saidi@gm` the email text field will be red from the first letter until the email address is correct. What if we want to wait until we try to submit the form?

To achieve this, we can easily add on-demand form validation. If we go back to our form data, you'll see that we actually prepared this from start:

```swift
extension UserRegistration {

    class FormData: ObservableObject {

        typealias StringPath = ReferenceWritableKeyPath<UserRegistration.FormData, String>

        @Published var email = ""
        @Published var password = ""
        @Published var passwordConfirm = ""
        @Published var subscribeToNewsletter = false

        @Published var validationState = [StringPath: Bool]()   // <-- TADA!
    }
}
```

We can now add a `validate()` function that populate this state as part of validating all properties:

```swift
extension UserRegistration.FormData {

    func validate() {
        validationKeyPaths.forEach {
            validationState[$0] = isValid($0)
        }
    }
}
```

In the code above, we iterate over all `validationKeyPaths` and set the `validationState` for each one.

We can now update the text field and color functions in our form, to call `validate()` when any text field is submitted, and use the `validationState` to determine the text field foreground color:

```swift
private extension UserRegistrationForm {

    typealias StringPath = UserRegistration.FormData.StringPath

    func textField(
        _ title: Localization.Key,
        _ keyPath: StringPath,
        _ type: UITextContentType
    ) -> some View {
        TextField(translate(title), text: $formData[dynamicMember: keyPath])
            .textContentType(type)
            .onSubmit(formData.validate)
            .foregroundStyle(textFieldColor(for: keyPath))
    }

    func textFieldColor(
        for keyPath: StringPath
    ) -> Color {
        let state = formData.validationState[keyPath]
        return state == false ? .red : .primary
    }
}
```

With this change, the text is marked red when we stop editing a text field. We could also move the validation call to a `submit` button, if we prefer to validate the form even later.


## Conclusion

In this post, we have taken a look at how key paths can be used to drastically reduce the amount of code we have to write, and make it much more readable as a direct result.

We could take this one step further and extract all validation logic to a protocol, e.g. `FormData` which would let us reuse the same logic across multiple form models.

We will probably also have to extend the form data model with support for more types than strings. If we do, it's probably wise to split up the logic in multiple protocols.

For instance a `StringFormData` protocol could handle strings, an `IntFormData` protocol could handle integers, and so on. We could then tie is all together with a `MultiTypeFormData` protocol.
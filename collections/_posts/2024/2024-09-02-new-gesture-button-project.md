---
title:  New GestureButton open-source project
date:   2024-09-02 06:00:00 +0000
tags:   swiftui open-source gestures

image:  /assets/sdks/gesturebutton-header.jpg
image-show: 0
---

{% include kankoda/data/open-source name="SwiftUIKit" %}
The GestureButton component that used to be part of the [{{project.name}}]({{project.url}}){% include kankoda/data/open-source name="GestureButton" %} has now been moved to a new, separate open-source project called [{{project.name}}]({{project.url}}).

![Header image]({{page.image}})

GestureButton can trigger gesture-specific actions. You can use it just like a regular SwiftUI `Button` and provide it with custom actions for any gesture you want to handle:

```swift
struct MyView: View {

    @State private var isPressed = false
    
    var body: some View {
        GestureButton(
            isPressed: $isPressed,
            pressAction: { print("Pressed") },
            releaseInsideAction: { print("Released Inside") },
            releaseOutsideAction: { print("Released Outside") },
            longPressAction: { print("Long Pressed") },
            doubleTapAction: { print("Double Tapped") },
            repeatAction: { print("Repeating Action") },
            dragStartAction: { value in print("Drag Started") },
            dragAction: { value in print("Drag \(value)") },
            dragEndAction: { value in print("Drag Ended") },
            endAction: { print("Gesture Ended") }
        ) { isPressed in
            Color.yellow // Add any label view here.
        }
    }
}
```

In iOS 17 and earlier, you can set `isInScrollView` to true when you render the button in a `ScrollView`, to avoid it messing with the scroll gesture. This is not needed in iOS 18.

Feel free to try out the very first version of [{{project.name}}]({{project.url}}) and let me know what you think.
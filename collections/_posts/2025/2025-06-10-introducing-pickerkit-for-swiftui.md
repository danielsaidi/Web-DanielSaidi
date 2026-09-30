---
title:  Introducing PickerKit for SwiftUI
date:   2025-06-10 07:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/25/0610/
# image:  /assets/blog/25/0610/image.jpg
image:  /assets/sdks/pickerkit-header.jpg

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3lr5sfscy4227
toot: https://mastodon.social/@danielsaidi/114652103712648073
---

{% include kankoda/data/open-source name="PickerKit" %}
Say hi to [{{project.name}}]({{project.url}}) - an open-source SwiftUI package that contains various image pickers, cameras, document scanners, file pickers, picker utilities, etc. for your everyday picker needs.

![PickerKit logo]({{page.image}})


## Available Features

The first PickerKit beta was just released, but already has many pickers and picker-related views:

* `Camera` can be used to take photos and handle them as images.
* `ColorPickerBar` adds a color picker to a horizontal or vertical bar with additional colors.
* `DocumentScanner` can be used to scan documents and handle them as images.
* `FilePicker` can be used to pick any file from the Files app.
* `ImagePicker` can be used to pick images from the user's photo library.
* `MultiPicker` can be used to pick multiple items in e.g. a list or form.

There are also models and utilities, like the `OptionalBinding` and `CancellableResult` models. 


## Camera

The `Camera` picker uses the `ImagePicker` with a `.camera` picker configuration, and can take pictures with the device camera:

```swift
Camera(
    isPresented: $isPickerPresented,
    action: { result in
        switch result {
        case .cancelled: break
        case .failure: break
        case .success(let image): // Handle image here...
        }
    }
)
```

If you provide a value for `isPresented`, the camera will automatically dismiss itself when it's done.


## DocumentScanner

The `DocumentScanner` picker wraps a `VNDocumentCameraViewController` and can scan one or multiple documents with the device camera:

```swift
DocumentScanner(
    isPresented: $isPickerPresented,
    action: { result in
        switch result {
        case .cancelled: break
        case .failure: break
        case .success(let scan): // Handle scan here...
        }
    }
)
```

If you provide a value for `isPresented`, the scanner will automatically dismiss itself when it's done.


## FilePicker

The `FilePicker` picker wraps a `UIDocumentPickerViewController` and can pick files from Files:

```swift
FilePicker(
    documentTypes: [.content],
    isPresented: $isPickerPresented,
    pickerConfig: { picker in
        picker.allowsMultipleSelection = true
    },
    action: { result in
        switch result {
        case .cancelled: break
        case .failure: break
        case .success(let urls): self.urls = urls
        }
    }
)
```

If you provide a value for `isPresented`, the picker will automatically dismiss itself when it's done.


## ImagePicker

The `ImagePicker` picker wraps a `UIImagePickerController` and can pick images in various ways:

```swift
ImagePicker(
    sourceType: .photoLibrary,
    isPresented: $isPickerPresented,
    pickerConfig: { picker in },
    action: { result in
        switch result {
        case .cancelled: break
        case .failure: break
        case .success(let image): images.append(image)
        }
    }
)
```

If you provide a value for `isPresented`, the picker will automatically dismiss itself when it's done.


## ColorPickerBar

<div class="grid col2">
    <img src="{{page.assets}}colorpickerbar-1.png" class="plain" width=250 />
    <img src="{{page.assets}}colorpickerbar-2.png" class="plain" width=250 />
</div>

The `ColorPickerBar` adds a `ColorPicker` to a horizontal or vertical bar with extra colors and actions:

```swift
ColorPickerBar(
    axis: .horizontal, 
    value: $color
)
.colorPickerBarConfig(.init(
    barColors: .colorPickerBarColors(withClearColor: true),
    opacity: true,
    resetButton: true,
    resetValue: .black
))
.colorPickerBarStyle(.init(
    animation: .bouncy,
    spacing: 10,
    colorSize: 30,
    selectedColorSize: 45
))
```

If you provide a value for `isPresented`, the picker will automatically dismiss itself when it's done.


## MultiPicker

<img src="{{page.assets}}multipicker.png" class="plain" width=250 />

A `MultiPicker` can be used to pick multiple `Identifiable` values in a `List`, `Form`, stack or grid:

```swift
MultiPicker(
    items: items,
    selection: $selection
) { item, isSelected in
    MultiPickerItem(isSelected: isSelected) {
        Text(item.name).tag(item)
    }
}
```

You can use any custom item view, and wrap it in a `MultiItemPickerView` to automatically add a checkmark to the selected values.



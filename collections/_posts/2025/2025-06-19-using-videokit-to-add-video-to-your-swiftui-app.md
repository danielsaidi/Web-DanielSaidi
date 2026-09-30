---
title:  Using VideoKit to add video to your SwiftUI app
date:   2025-06-19 07:00:00 +0000
tags:   swiftui open-source

assets: /assets/blog/25/0619/
image:  /assets/blog/25/0619/image.jpg

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3lrx6gos77227
toot: https://mastodon.social/@danielsaidi/114709248777804995
---

{% include kankoda/data/open-source name="VideoKit" %}
I just created a new open-source Swift Package called [{{project.name}}]({{project.url}}), which makes it easy to add video to your SwiftUI apps. It has a configurable `VideoPlayer` and a video-based splash screen.

![PickerKit logo]({{project.header}})

Unlike AVKit's VideoPlayer, VideoKit's ``VideoPlayer`` can be configured to great extent. It also lets you observe the current player time and can trigger an action when the player reaches the end.

VideoKit can also add customizable, video-based splash screens to your app. This makes it easy to add powerful launch effects, like we see in many video streaming apps.


## Video Player

To add video content to your app, just add a ``VideoPlayer`` with a URL to the video you want to play:

```swift
struct ContentView: View {

    var body: some View {
        VideoPlayer(videoURL: VideoPlayer.sampleVideoURL)
            .aspectRatio(16/9, contentMode: .fit)
    }
}
```

You can injecting a `time` binding, and trigger an action when the video stops playing:

```swift
struct ContentView: View {

    @State var isVideoPresented = false
    @State var videoTime = TimeInterval.zero

    var body: some View {
        Button("Play video") {
            isVideoPresented = true
        }
        .fullScreenCover(isPresented: $isVideoPresented) {
            VideoPlayer(
                videoURL: VideoPlayer.sampleVideoURL,
                time: $videoTime,
                didPlayToEndAction: { isVideoPresented = false }
            )
            .ignoresSafeArea()
        }
    }
}
```

You can inject a ``VideoPlayerConfiguration`` and a controller configuration to customize a player and its underlying controller:

```swift
struct ContentView: View {

    var body: some View {
        VideoPlayer(
            videoURL: VideoPlayer.sampleVideoURL,
            configuration: .init(autoPlay: false),
            controllerConfiguration: { controller in
                controller.showsPlaybackControls = false
            }
        )
    }
}
```

These options make it easy to add powerful video-based features to your app. 


## Video Splash Screen

VideoKit makes it easy to add a video-based splash screen that is automatically presented when an app launches, and dismissed when the embedded video stops playing.

To add a video splash screen to your app, just apply a ``.videoSplashScreen(videoURL:configuration:)`` view modifier to your app's root view:

```swift
struct ContentView: View {

    var body: some View {
        Text("Hello, world")
            .videoSplashScreen(
                videoURL: VideoPlayer.sampleVideoURL
            )
    }
}
```

You can pass in a ``VideoSplashScreenConfiguration`` to customize the splash screen:

```swift
struct ContentView: View {

    var body: some View {
        Text("Hello, world")
            .videoSplashScreen(
                videoURL: VideoPlayer.sampleVideoURL,
                configuration: .init(
                    dismissAnimation: .linear(duration: 2),
                    maxDisplayDuration: 2
                )
            )
    }
}
```

You can also customize the video player view, for instance to add a custom background view to it:


```swift
struct ContentView: View {

    var body: some View {
        Text("Hello, world")
            .videoSplashScreen(
                videoURL: VideoPlayer.sampleVideoURL,
                videoPlayerView: { videoPlayer in
                    Color.red
                    videoPlayer.aspectRatio(contentMode: .fit)
                }
            )
    }
}
```

This makes it easy to add customizable video splash screens to your app with a few lines of code.


## Sample Videos

VideoKit also has a ``SampleVideo`` type that can be used to test the player, and a list of sample videos that are parsed from a JSON file in the library.


## Conclusion

[VideoKit]({{project.url}}) is a new, open-source Swift package that aims to make it easy to add powerful video-based features to your SwiftUI apps. I hope that you'll like it.
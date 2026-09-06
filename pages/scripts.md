---
layout: default
title: Scripts
permalink: /scripts
---

# Scripts

A collection of scripts I use when working with my open-source projects. Feel free to copy and adapt them for your own use.

## Git Update

Save this as a script file, e.g. `update`, make it executable with `chmod +x update`, then run it from a folder with many sub-folder repositories:

```bash
#!/bin/bash

for dir in */; do
  if [ -d "$dir/.git" ]; then
    echo "Pulling $dir..."
    git -C "$dir" pull
  fi
done
```
&nbsp;

## Open-Source (HTTPS)

This script clones all my open-source projects using HTTPS. It skips any that are already cloned.

```
git clone https://github.com/danielsaidi/ApiKit.git; \
git clone https://github.com/danielsaidi/AppIconKit.git; \
git clone https://github.com/danielsaidi/BadgeIcon.git; \
git clone https://github.com/danielsaidi/DeckKit.git; \
git clone https://github.com/danielsaidi/EmojiKit.git; \
git clone https://github.com/danielsaidi/FlipKit.git; \
git clone https://github.com/danielsaidi/FontKit.git; \
git clone https://github.com/danielsaidi/GestureButton.git; \
git clone https://github.com/danielsaidi/MockingKit.git; \
git clone https://github.com/danielsaidi/OnboardingKit.git; \
git clone https://github.com/danielsaidi/PageView.git; \
git clone https://github.com/danielsaidi/PickerKit.git; \
git clone https://github.com/danielsaidi/PresentationKit.git; \
git clone https://github.com/danielsaidi/PrintingKit.git; \
git clone https://github.com/danielsaidi/QuickSearch.git; \
git clone https://github.com/danielsaidi/RichTextKit.git; \
git clone https://github.com/danielsaidi/ScanCodes.git; \
git clone https://github.com/danielsaidi/ScrollKit.git; \
git clone https://github.com/danielsaidi/StandardActions.git; \
git clone https://github.com/danielsaidi/StoreKitPlus.git; \
git clone https://github.com/danielsaidi/SwiftPackageScripts.git; \
git clone https://github.com/danielsaidi/SwiftUIKit.git; \
git clone https://github.com/danielsaidi/SystemNotification.git; \
git clone https://github.com/danielsaidi/TagKit.git; \
git clone https://github.com/danielsaidi/TextReplacements.git; \
git clone https://github.com/danielsaidi/VideoKit.git
```

## Open-Source (SSH)

This script clones all my open-source projects using SSH. It skips any that are already cloned.

```
git clone git@github.com:danielsaidi/ApiKit.git; \
git clone git@github.com:danielsaidi/AppIconKit.git; \
git clone git@github.com:danielsaidi/BadgeIcon.git; \
git clone git@github.com:danielsaidi/DeckKit.git; \
git clone git@github.com:danielsaidi/EmojiKit.git; \
git clone git@github.com:danielsaidi/FlipKit.git; \
git clone git@github.com:danielsaidi/FontKit.git; \
git clone git@github.com:danielsaidi/GestureButton.git; \
git clone git@github.com:danielsaidi/MockingKit.git; \
git clone git@github.com:danielsaidi/OnboardingKit.git; \
git clone git@github.com:danielsaidi/PageView.git; \
git clone git@github.com:danielsaidi/PickerKit.git; \
git clone git@github.com:danielsaidi/PresentationKit.git; \
git clone git@github.com:danielsaidi/PrintingKit.git; \
git clone git@github.com:danielsaidi/QuickSearch.git; \
git clone git@github.com:danielsaidi/RichTextKit.git; \
git clone git@github.com:danielsaidi/ScanCodes.git; \
git clone git@github.com:danielsaidi/ScrollKit.git; \
git clone git@github.com:danielsaidi/StandardActions.git; \
git clone git@github.com:danielsaidi/StoreKitPlus.git; \
git clone git@github.com:danielsaidi/SwiftPackageScripts.git; \
git clone git@github.com:danielsaidi/SwiftUIKit.git; \
git clone git@github.com:danielsaidi/SystemNotification.git; \
git clone git@github.com:danielsaidi/TagKit.git; \
git clone git@github.com:danielsaidi/TextReplacements.git; \
git clone git@github.com:danielsaidi/VideoKit.git
```

## Private Repos

These repos are private and require that you first register your SSH key with the Kankoda company.

### Websites

```
git clone git@github.com:danielsaidi/Appamini-Web.git Appamini; \
git clone git@github.com:danielsaidi/Web-DanielSaidi.git DanielSaidi; \
git clone git@github.com:Kankoda/Web.git Kankoda; \
git clone git@github.com:keyboardkit/Keyboardkit-Web.git KeyboardKit; \
git clone git@github.com:danielsaidi/Wally-Web.git Wally
```

### Apps

```
git clone git@github.com:danielsaidi/Appamini-App.git Appamini; \
git clone git@github.com:kankoda/EmojiPicker-App.git EmojiPicker; \
git clone git@github.com:kankoda/KankodaKit.git KankodaKit; \
git clone git@github.com:keyboardkit/KeyboardKit-App.git KeyboardKit; \
git clone git@github.com:danielsaidi/Lunchrrrrr-App.git Lunchrrrrr; \
git clone git@github.com:danielsaidi/OneTouchPaste-App.git OTP; \
git clone git@github.com:danielsaidi/Wally-App.git Wally; \
git clone git@github.com:danielsaidi/Vinylsamlaren-App.git Vinylsamlaren
```

### KeyboardKit

```
git clone git@github.com:keyboardkit/KeyboardKit.git release; \
git clone git@github.com:keyboardkit/KeyboardKit-Source.git src; \
git clone git@github.com:keyboardkit/KeyboardKit-Source-Android.git src-android; \
git clone git@github.com:keyboardkit/KeyboardKit-Documentation.git docs; \
git clone git@github.com:keyboardkit/KeyboardKit-Binaries.git binaries; \
git clone git@github.com:keyboardkit/KeyboardKit-Licenses.git licenses
```

### LicenseKit

```
git clone git@github.com:kankoda/LicenseKit.git release; \
git clone git@github.com:kankoda/LicenseKit-Source.git src; \
git clone git@github.com:kankoda/LicenseKit-Binaries.git binaries
```

### MediaKit

```
git clone git@github.com:kankoda/MediaKit.git release; \
git clone git@github.com:kankoda/MediaKit-Source.git src; \
git clone git@github.com:kankoda/MediaKit-Binaries.git binaries
```

### Vietnamese Input

```
git clone git@github.com:kankoda/VietnameseInput.git release; \
git clone git@github.com:kankoda/VietnameseInput-source.git src; \
git clone git@github.com:kankoda/VietnameseInput-binaries.git binaries
```
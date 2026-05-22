---
layout: default
title: Scripts
permalink: /scripts
---

# Scripts

A collection of scripts I use when working with my open-source projects. Feel free to copy and adapt them for your own use.

## Clone all open-source projects (HTTPS)

This script clones all my open-source projects using HTTPS. It skips any that are already cloned.

```
git clone https://github.com/danielsaidi/ApiKit.git; git clone https://github.com/danielsaidi/AppIconKit.git; git clone https://github.com/danielsaidi/BadgeIcon.git; git clone https://github.com/danielsaidi/DeckKit.git; git clone https://github.com/danielsaidi/EmojiKit.git; git clone https://github.com/danielsaidi/FlipKit.git; git clone https://github.com/danielsaidi/FontKit.git; git clone https://github.com/danielsaidi/GestureButton.git; git clone https://github.com/danielsaidi/MockingKit.git; git clone https://github.com/danielsaidi/OnboardingKit.git; git clone https://github.com/danielsaidi/PageView.git; git clone https://github.com/danielsaidi/PickerKit.git; git clone https://github.com/danielsaidi/PresentationKit.git; git clone https://github.com/danielsaidi/PrintingKit.git; git clone https://github.com/danielsaidi/QuickSearch.git; git clone https://github.com/danielsaidi/RichTextKit.git; git clone https://github.com/danielsaidi/ScanCodes.git; git clone https://github.com/danielsaidi/ScrollKit.git; git clone https://github.com/danielsaidi/StandardActions.git; git clone https://github.com/danielsaidi/StoreKitPlus.git; git clone https://github.com/danielsaidi/SwiftPackageScripts.git; git clone https://github.com/danielsaidi/SwiftUIKit.git; git clone https://github.com/danielsaidi/SystemNotification.git; git clone https://github.com/danielsaidi/TagKit.git; git clone https://github.com/danielsaidi/TextReplacements.git; git clone https://github.com/danielsaidi/VideoKit.git
```

## Clone all open-source projects (SSH)

This script clones all my open-source projects using SSH. It skips any that are already cloned.

```
git clone git@github.com:danielsaidi/ApiKit.git; git clone git@github.com:danielsaidi/AppIconKit.git; git clone git@github.com:danielsaidi/BadgeIcon.git; git clone git@github.com:danielsaidi/DeckKit.git; git clone git@github.com:danielsaidi/EmojiKit.git; git clone git@github.com:danielsaidi/FlipKit.git; git clone git@github.com:danielsaidi/FontKit.git; git clone git@github.com:danielsaidi/GestureButton.git; git clone git@github.com:danielsaidi/MockingKit.git; git clone git@github.com:danielsaidi/OnboardingKit.git; git clone git@github.com:danielsaidi/PageView.git; git clone git@github.com:danielsaidi/PickerKit.git; git clone git@github.com:danielsaidi/PresentationKit.git; git clone git@github.com:danielsaidi/PrintingKit.git; git clone git@github.com:danielsaidi/QuickSearch.git; git clone git@github.com:danielsaidi/RichTextKit.git; git clone git@github.com:danielsaidi/ScanCodes.git; git clone git@github.com:danielsaidi/ScrollKit.git; git clone git@github.com:danielsaidi/StandardActions.git; git clone git@github.com:danielsaidi/StoreKitPlus.git; git clone git@github.com:danielsaidi/SwiftPackageScripts.git; git clone git@github.com:danielsaidi/SwiftUIKit.git; git clone git@github.com:danielsaidi/SystemNotification.git; git clone git@github.com:danielsaidi/TagKit.git; git clone git@github.com:danielsaidi/TextReplacements.git; git clone git@github.com:danielsaidi/VideoKit.git
```

## Update all open-source projects

Save this as a script file, e.g. `update-all.sh`, make it executable with `chmod +x update-all.sh`, then run it from the folder where you cloned your projects.

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

# Private Repos

These repos are private and require that you first register your SSH key with the Kankoda company.

## Websites

```
git clone git@github.com:danielsaidi/web-danielsaidi.git danielsaidi; git clone git@github.com:kankoda/kankoda-web.git kankoda; git clone git@github.com:danielsaidi/wally-web.git wally; git clone git@github.com:danielsaidi/appamini-web.git appamini
```

## Apps

```
git clone git@github.com:kankoda/kankodakit.git kankodakit; git clone git@github.com:kankoda/emojipicker-app.git emojipicker; git clone git@github.com:danielsaidi/lunchrrrrr-app.git lunchrrrrr; git clone git@github.com:danielsaidi/onetouchpaste-app.git otp; git clone git@github.com:danielsaidi/wally-app.git wally; git clone git@github.com:danielsaidi/appamini-app.git appamini; git clone git@github.com:danielsaidi/vinylsamlaren-app.git vinylsamlaren
```

## KeyboardKit

```
git clone git@github.com:keyboardkit/keyboardkit.git release; git clone git@github.com:keyboardkit/keyboardkit-source.git src; git clone git@github.com:keyboardkit/keyboardkit-source-android.git src-android; git clone git@github.com:keyboardkit/keyboardkit-documentation.git docs; git clone git@github.com:keyboardkit/keyboardkit-binaries.git binaries; git clone git@github.com:keyboardkit/keyboardkit-licenses.git licenses; git clone git@github.com:keyboardkit/keyboardkit-app.git app; git clone git@github.com:keyboardkit/keyboardkit-web.git web
```

## LicenseKit

```
git clone git@github.com:kankoda/licensekit.git release; git clone git@github.com:kankoda/licensekit-source.git src; git clone git@github.com:kankoda/licensekit-binaries.git binaries
```

## MediaKit

```
git clone git@github.com:kankoda/mediakit.git release; git clone git@github.com:kankoda/mediakit-source.git src; git clone git@github.com:kankoda/mediakit-binaries.git binaries
```

## Vietnamese Input

```
git clone git@github.com:kankoda/vietnameseinput.git release; git clone git@github.com:kankoda/vietnameseinput-source.git src; git clone git@github.com:kankoda/vietnameseinput-binaries.git binaries
```
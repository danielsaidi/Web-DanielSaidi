---
title:  Publish DocC to GitHub Pages with GitHub Actions
date:   2024-03-10 06:00:00 +0000
tags:   swift docc automation git

assets: /assets/blog/24/0310/
image:  /assets/blog/24/0310.jpg

inspiration: https://maxxfrazer.medium.com/deploying-docc-with-github-actions-218c5ca6cad5

tweet:  https://x.com/danielsaidi/status/1766754008260104649?s=20
toot:   https://mastodon.social/@danielsaidi/112070621419993035
---

In this post, we'll see how we can use GitHub Actions to automatically build and publish a Swift Package's DocC docs to GitHub Pages, every time we push to a specific branch.

![GitHub Actions Logo]({{page.assets}}header.jpg)

The workflow presented in this post is **heavily** inspired from [@maxxfrazer]({{page.inspiration}})'s Medium post, so if you like this, make sure to jump over there and give it a like.


## Background

I used to have DocC generation as part of my standard [open-source](/opensource) workflow, where every new version required the code to pass linting, testing, and DocC generation.

For every new version, I would then take the generated docs and move them to a different folder that pushed to the `gh-pages` branch of that repository.

Since these docs can grow huge (some are around 300MB), I used `git amend` to only get a single commit for the `gh-pages` branch.

This has been tedious and error-prone, since each release has consisted of many manual steps. Since I have many [projects](/opensource), the amount of manual work addded up.

I was therefore very happy to see that GitHub now lets you use GitHub Actions to generate new GitHub Pages every time you push new changes. Let's see how it works.


## GitHub Settings

In your GitHub repository dashboard, go to `Settings` then select `Pages` in the side menu. 

![GitHub Settings]({{page.assets}}github.jpg)

Under `Build and deployment`, you can now select your GitHub Pages `Source`. Switch from `Deploy from a branch` to `GitHub Actions`.


## DocC Publish Workflow

With this, I could delete my `gh-pages` branch, remove all DocC code from my `Fastfile`, and replace the Fastlane script with this workflow, placed in `.github/workflows/docc.yml`:

```swift
name: DocC Runner

on:
  push:
    branches: ["main"]

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true
  
# A single job that builds and deploys the DocC documentation
jobs:
  deploy:
    environment:
      name: github-pages
      url: {% raw %}${{ steps.deployment.outputs.page_url }}{% endraw %}
    runs-on: macos-14
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Select Xcode 15.1
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: '15.1.0'
      - name: Build DocC
        run: |
          swift package resolve;

          xcodebuild docbuild -scheme ApiKit -derivedDataPath /tmp/docbuild -destination 'generic/platform=iOS';
          
          $(xcrun --find docc) process-archive \
            transform-for-static-hosting /tmp/docbuild/Build/Products/Debug-iphoneos/ApiKit.doccarchive \
            --output-path docs \
            --hosting-base-path 'ApiKit';
          
          echo "<script>window.location.href += \"/documentation/apikit\"</script>" > docs/index.html;
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'docs'
      - id: deployment
        name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

The job sets up the environment & token permissions, specifies that is must run on macOS 14 and Xcode 15.1, then builds and uploads a new DocC build.

The DocC build step currently only builds for iOS, but you can add more `xcodebuild` rows to include more platforms. It adds a redirect to the root folder, then deploys the `doc` folder.

The DocC build step also adds a JavaScript redirect to the root `index.html`, to redirect it to the generated documentation. Without this, the root page would just show a blank page.


## Build Runner Workflow

Besides this DocC workflow, I also have a `.github/workflows/build.yml` workflow that will trigger on every new push and pull request, and build the package for all platforms:

```swift
name: Build Runner

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
    
env:
  SCHEME: ApiKit

jobs:
  build:
    runs-on: macos-13
    steps:
      - uses: actions/checkout@v3
      - uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: '15.1.0'

      - name: Build iOS
        run: xcodebuild -scheme $SCHEME -derivedDataPath .build -destination 'generic/platform=iOS';
      - name: Build macOS
        run: xcodebuild -scheme $SCHEME -derivedDataPath .build -destination 'generic/platform=OS X';
      - name: Build tvOS
        run: xcodebuild -scheme $SCHEME -derivedDataPath .build -destination 'generic/platform=tvOS';
      - name: Build watchOS
        run: xcodebuild -scheme $SCHEME -derivedDataPath .build -destination 'generic/platform=watchOS';
      - name: Build visionOS
        run: xcodebuild -scheme $SCHEME -derivedDataPath .build -destination 'generic/platform=xrOS';

      - name: Test iOS
        run: xcodebuild test -scheme $SCHEME -derivedDataPath .build -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.2' -enableCodeCoverage YES;
```

The workflow has individual build steps for iOS, macOS, tvOS, watchOS and visionOS, to ensure that the package builds for all supported platforms.

If your package only supports one or a few platforms, make sure to adjust the build steps to only include the platforms that your package supports.


## Fastlane Version Bump Script

With these GitHub Action workflows in place, we still need to have a build & test runner in the Fastlane version script, to avoid faulty versions.

This is the new Fastlane file that I will use in all my open-source projects. It's basically just defining a version bump script (lane) that uses other lanes to ensure that the code is legit.

```swift
fastlane_version "2.129.0"

default_platform :ios


platform :ios do

  name = "PACKAGE_NAME"
  main_branch = "main"


  # Build ==================
  
  lane :build do |options|
    platform = options[:platform]
    sh("cd .. && xcodebuild -scheme " + name + " -derivedDataPath .build -destination 'generic/platform=" + platform + "';")
  end
  
  lane :build_all do
    build(platform: "iOS")
    build(platform: "OS X")
    build(platform: "tvOS")
    build(platform: "watchOS")
    build(platform: "xrOS")
  end


  # Test ==================
  
  lane :test_ios do
    sh("cd .. && xcodebuild test -scheme " + name + " -derivedDataPath .build -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.2' -enableCodeCoverage YES;")
  end


  # Version ================

  desc "Create a new version"
  lane :version do |options|
    version_validate

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end
  
  desc "Validate that the repo is valid for release"
  lane :version_validate do
    ensure_git_status_clean
    ensure_git_branch(branch: main_branch)
    swiftlint(strict: true)
    build_all
    test_ios
  end

end
```

It defines a `build` lane that build the package for any platform, as well as an iOS test lane.

The `version` lane calls `version_validate` to check that the git repo status is clean, that it's on the correct branch, and that `swiftlint` passes, then calls `build_all` and `test_ios`.


## Conclusion

With the new GitHub Action workflows, I don't have to manually handle DocC. Every push to `main` automatically publishes updated DocC docs to GitHub Pages.

I also get continuous build and test checks with the `build` workflow, which builds and tests all supported platforms on every new push to `main`.

I found GitHub Actions trickier to set up than e.g. Bitrise, but once in place it integrates well with pull requests and other open-source events.

I have migrated all [open-source projects](/opensource) from my manual workflow to this fully automated approach. It's very nice to have it in place, and saves me a lot of time.
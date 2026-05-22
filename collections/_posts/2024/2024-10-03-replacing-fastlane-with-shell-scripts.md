---
title:  Replacing Fastlane with Shell scripts
date:   2024-10-03 06:00:00 +0000
tags:   automation github

image:  /assets/blog/24/1003.jpg
image-show: 0

post_ghpages: /blog/2024/03/10/automatically-publish-docc-to-github-pages-with-github-actions
sdk: https://github.com/danielsaidi/SwiftPackageScripts

tweet: https://x.com/danielsaidi/status/1842178052056305687
toot:  https://mastodon.social/@danielsaidi/113249121421389909
---

After many, many years of great service, I'm looking to replace Fastlane with plain Shell script files, which I hope will result in faster builds and...less Ruby.

<!--![Header image]({{page.image}})-->

## TL;DR

This article goes in-depth on how to create Shell scripts to manage many parts of a Swift Package's lifecycle. If you're just after the scripts and basic info, have a look at the [SwiftPackageScripts]({{page.sdk}}) project.


## Background

I use Fastlane to build, test and generate new versions of my various [open-source projects](/opensource). While the setup has been quite complex earlier, the current one is pretty basic:

```bash
fastlane_version "2.129.0"

default_platform :ios

platform :ios do

  name = "EmojiKit"
  main_branch = "main"


  # Build ==================

  lane :build do |options|
    build_platform(platform: "iOS")
    build_platform(platform: "OS X")
    build_platform(platform: "tvOS")
    build_platform(platform: "watchOS")
    build_platform(platform: "xrOS")
  end

  lane :build_platform do |options|
    platform = options[:platform]
    sh("cd .. && xcodebuild -scheme " + name + " -derivedDataPath .build -destination 'generic/platform=" + platform + "';")
  end


  # Test ==================

  lane :test do
    test_platform(platform: "platform=iOS Simulator,name=iPhone 16")
  end

  lane :test_platform do |options|
    platform = options[:platform]
    sh("cd .. && xcodebuild test -scheme " + name + " -derivedDataPath .build -destination '" + platform + "' -enableCodeCoverage YES;")
  end


  # Version ================

  desc "Create a new version"
  lane :version do |options|
    version_validate
    version_build

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end
  
  desc "Validate that the SDK is ready for release"
  lane :version_validate do
    ensure_git_status_clean
    ensure_git_branch(branch: main_branch)
    swiftlint(strict: true)
  end
  
  desc "Validate that the repo is valid for release"
  lane :version_build do
    build
    test
  end

end
```

I used to have DocC (Apple's documentation tool for Swift software) scripts in here, but have moved that part to a GitHub Actions, as described in [this post]({{page.post_ghpages}}).

We will now convert these lanes to plain Shell scripts, step by step, to eventually be able to remove Fastlane and all things that come with it.


## Step 1 - Replacing the build lanes

To avoid having to use Fastlane to build the SDK, let's convert these `Fastfile` lanes to script files:

```ruby
lane :build do |options|
  build_platform(platform: "iOS")
  build_platform(platform: "OS X")
  build_platform(platform: "tvOS")
  build_platform(platform: "watchOS")
  build_platform(platform: "xrOS")
end

lane :build_platform do |options|
  platform = options[:platform]
  sh("cd .. && xcodebuild -scheme " + name + " -derivedDataPath .build -destination 'generic/platform=" + platform + "';")
end
```

Let's first create a `scripts/build_framework.sh` script to replace the `build_platform` lane. It just takes the code from inside `sh(...)`, removes the `cd ..` and adds some argument validation:

```bash
#!/bin/bash

# Verify that all required arguments are provided
if [ $# -ne 2 ]; then
    echo "Error: This script requires exactly two arguments"
    echo "Usage: $0 <TARGET> <PLATFORM>"
    exit 1
fi

TARGET=$1
PLATFORM=$2

xcodebuild -scheme $TARGET -derivedDataPath .build -destination generic/platform=$PLATFORM
```

To build a project called MyLib for iOS, we'd just have to write `bash scripts/build_platform.sh MyLib iOS`. The rest of the post assumes that you understand, and will not repeat how each script is called.

We can now create a build script that replaces the `build` lane, that builds a target for all platforms.

This will be a bit more complicated, since the `scripts/build.sh` script will locate the `build_framework` script in the same folder, make it executable, then call it for all platforms:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Verify that all required arguments are provided
if [ $# -eq 0 ]; then
    echo "Error: This script requires exactly one argument"
    echo "Usage: $0 <TARGET>"
    exit 1
fi

# Create local argument variables.
TARGET=$1

# Use the script folder to refer to other scripts.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
SCRIPT="$FOLDER/build_platform.sh"

# Make the script executable
chmod +x $SCRIPT

# A function that builds a specific platform
build_platform() {
    local platform=$1
    echo "Building for $platform..."
    if ! bash $SCRIPT $TARGET $platform; then
        echo "Failed to build $platform"
        return 1
    fi
    echo "Successfully built $platform"
}

# Array of platforms to build
platforms=("iOS" "macOS" "tvOS" "watchOS" "xrOS")

# Loop through platforms and build
for platform in "${platforms[@]}"; do
    if ! build_platform "$platform"; then
        exit 1
    fi
done

echo "All platforms built successfully!"
```

The reason for having two files, while we could just use the one-line from `build_platform` within the loop, is that it's nice to be able to build a single platform with a single script.

With this in place, we can remove the `Fastfile` lanes and call the `build` script from `version_build`:

```ruby
fastlane_version "2.129.0"

default_platform :ios

platform :ios do

  name = "EmojiKit"
  main_branch = "main"


  # Test ==================

  lane :test do
    test_platform(platform: "platform=iOS Simulator,name=iPhone 16")
  end

  lane :test_platform do |options|
    platform = options[:platform]
    sh("cd .. && xcodebuild test -scheme " + name + " -derivedDataPath .build -destination '" + platform + "' -enableCodeCoverage YES;")
  end


  # Version ================

  desc "Create a new version"
  lane :version do |options|
    version_validate
    version_build

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end

  desc "Validate that the SDK is ready for release"
  lane :version_validate do
    ensure_git_status_clean
    ensure_git_branch(branch: main_branch)
    swiftlint(strict: true)
  end

  desc "Validate that the repo is valid for release"
  lane :version_build do
    sh("cd .. && bash scripts/build.sh " + name)
    test
  end

end
```


## Step 2 - Replacing the test lanes

To avoid having to use Fastlane to test the SDK, we can repeat the above steps for these lanes:

```ruby
lane :test do
  test_platform(platform: "platform=iOS Simulator,name=iPhone 16")
end

lane :test_platform do |options|
  platform = options[:platform]
  sh("cd .. && xcodebuild test -scheme " + name + " -derivedDataPath .build -destination '" + platform + "' -enableCodeCoverage YES;")
end
```

Let’s first create a `scripts/test_framework.sh` script to replace the `test_platform` lane. Just like with the build script, we can rewrite the code in `sh(...)`:

```bash
#!/bin/bash

# Verify that all required arguments are provided
if [ $# -ne 2 ]; then
    echo "Error: This script requires exactly two arguments"
    echo "Usage: $0 <TARGET> <PLATFORM>"
    exit 1
fi

TARGET=$1
PLATFORM="${2//_/ }"

xcodebuild test -scheme $TARGET -derivedDataPath .build -destination "$PLATFORM" -enableCodeCoverage YES;
```

This file uses a workaround to map _ to spaces in the platform name, since calling it with spaces will cause the platform to be interpreted as multiple arguments.

We can now create a `scripts/test.sh` script that replaces the `test` lane. It will be as complicated as the `build` script, since it needs to work in the same way:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Verify that all required arguments are provided
if [ $# -eq 0 ]; then
    echo "Error: This script requires exactly one argument"
    echo "Usage: $0 <TARGET>"
    exit 1
fi

# Create local argument variables.
TARGET=$1

# Use the script folder to refer to other scripts.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
SCRIPT="$FOLDER/test_platform.sh"

# Make the script executable
chmod +x $SCRIPT

# A function that tests a specific platform
test_platform() {
    local platform=$1
    echo "Testing for $platform..."
    if ! bash $SCRIPT $TARGET $platform; then
        echo "Failed to test $platform"
        return 1
    fi
    echo "Successfully tested $platform"
}

# Array of platforms to test
platforms=("platform=iOS_Simulator,name=iPhone_16")

# Loop through platforms and build
for platform in "${platforms[@]}"; do
    if ! test_platform "$platform"; then
        exit 1
    fi
done

echo "All platforms tested successfully!"
```

With this in place, we can remove the `Fastfile` lanes and call the `test` script from `version_build`:

```ruby
fastlane_version "2.129.0"

default_platform :ios

platform :ios do

  name = "EmojiKit"
  main_branch = "main"

  desc "Create a new version"
  lane :version do |options|
    version_validate
    version_build

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end

  desc "Validate that the SDK is ready for release"
  lane :version_validate do
    ensure_git_status_clean
    ensure_git_branch(branch: main_branch)
    swiftlint(strict: true)
  end

  desc "Validate that the repo is valid for release"
  lane :version_build do
    sh("cd .. && bash scripts/build.sh " + name)
    sh("cd .. && bash scripts/test.sh " + name)
  end

end
```


## Step 3 - Replacing the version validation lanes

To move even more things out of `Fastfile`, we must create a Shell script variant for the convenient `ensure_git_status_clean` and `ensure_git_branch(branch: main_branch)` Fastlane scripts.

Let's replace these two git validation scripts with a single `scripts/version_validate_git.sh` script:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Verify that all required arguments are provided
if [ $# -eq 0 ]; then
    echo "Error: This script requires exactly one argument"
    echo "Usage: $0 <BRANCH>"
    exit 1
fi

# Create local argument variables.
BRANCH=$1

# Check if the current directory is a Git repository
if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "Error: Not a Git repository"
    exit 1
fi

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
    echo "Error: Git repository is dirty. There are uncommitted changes."
    exit 1
fi

# Verify that we're on the correct branch
current_branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$current_branch" != "$BRANCH" ]; then
    echo "Error: Not on the specified branch. Current branch is $current_branch, expected $1."
    exit 1
fi

# The Git repository validation succeeded.
echo "Git repository successfully validated for branch ($1)."
exit 0
```

The script uses `git rev-parse` to check that we're in a git repo, then `git diff-index` to check if we're on `HEAD`, then finally uses `git rev-parse` again to check if we're on the correct branch. 

We can then create a `version_validate_project.sh` script that performs other quality validations like linting, and running the build and test scripts:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Verify that all required arguments are provided
if [ $# -eq 0 ]; then
    echo "Error: This script requires exactly one argument"
    echo "Usage: $0 <TARGET>"
    exit 1
fi

# Create local argument variables.
TARGET=$1

# Use the script folder to refer to other scripts.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
BUILD="$FOLDER/build.sh"
TEST="$FOLDER/test.sh"

# A function that run a certain script and checks for errors
run_script() {
    local script="$1"
    shift  # Remove the first argument (script path) from the argument list

    if [ ! -f "$script" ]; then
        echo "Error: Script not found: $script"
        exit 1
    fi

    chmod +x "$script"
    if ! "$script" "$@"; then
        echo "Error: Script $script failed"
        exit 1
    fi
}

echo "Running SwiftLint"
if ! swiftlint; then
    echo "Error: SwiftLint failed."
    exit 1
fi

echo "Building..."
run_script "$BUILD" "$TARGET"

echo "Testing..."
run_script "$TEST" "$TARGET"

echo ""
echo "Project successfully validated!"
echo ""
```

We can now replace the Fastlane validations with `version_validate_git` & `version_validate_project`. We can also remove the `version_build` lane and move the build and test steps into `version`:

```ruby
fastlane_version "2.129.0"

default_platform :ios

platform :ios do

  name = "EmojiKit"
  main_branch = "main"

  desc "Create a new version"
  lane :version do |options|
    sh("cd .. && bash scripts/version_validate_git.sh " + main_branch)
    sh("cd .. && bash scripts/version_validate_project.sh" + name)

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end

end
```

If `version_validate_git` and `version_validate_project` successfully validate the git repo and project, the script bumps the version number, creates a new tag and then pushes all changes.

Let's now proceed with replacing these last version bump lane steps with a couple of Shell scripts.


## Step 4 - Replacing the version bump lanes

The next step is a little trickier, since we have to create a Shell script variant of the pretty awesome `version_bump_podspec` script, which bumps the current version and returns the new version number.

The script accepts a "bump type" which can be `major`, `minor`, `patch`, etc. This lets us bump a version in different ways, depending on the kind of version we want to create.

Although the script has the word `podspec` in its name, you use it with any Ruby file. For instance, my projects just have a `Version` file that defines the version number like this:

```ruby
Version::Number.new do |v|
  v.version = '1.0.0'
end
```

Still...why do we need a separate version file to track the version number when we already have git? We can remove this dependency to make things more flexible.

Instead of keeping the `Version` and `version_bump_podspec`, I will use git to get the latest version, then replace the bump script with a manual version number step.

Let's first add this `scripts/version_number.sh` script that returns the last semver conforming version:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Check if the current directory is a Git repository
if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "Error: Not a Git repository"
    exit 1
fi

# Fetch all tags
git fetch --tags > /dev/null 2>&1

# Get the latest semver tag
latest_version=$(git tag -l --sort=-v:refname | grep -E '^v?[0-9]+\.[0-9]+\.[0-9]+$' | head -n 1)

# Check if we found a version tag
if [ -z "$latest_version" ]; then
    echo "Error: No semver tags found in this repository" >&2
    exit 1
fi

# Print the latest version
echo "$latest_version"
```

This script fetches all tags, then uses a regex to pick the latest semantic version, for instance `1.2.3`. 

We can now create a `scripts/version_number_bump.sh` script that calls the `version_number` script, then displays the version number and waits for the user to enter which version number to bump to:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Use the script folder to refer to other scripts.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
SCRIPT="$FOLDER/version_number.sh"

# Get the latest version
VERSION=$($SCRIPT)

if [ $? -ne 0 ]; then
    echo "Failed to get the latest version"
    exit 1
fi

# Print the current version
echo "The current version is: $VERSION"

# Function to validate semver format, including optional -rc.<INT> suffix
validate_semver() {
    if [[ $1 =~ ^v?[0-9]+\.[0-9]+\.[0-9]+(-rc\.[0-9]+)?$ ]]; then
        return 0
    else
        return 1
    fi
}

# Prompt user for new version
while true; do
    read -p "Enter the new version number: " NEW_VERSION

    if validate_semver "$NEW_VERSION"; then
        break
    else
        echo "Invalid version format. Please use semver format (e.g., 1.2.3, v1.2.3, 1.2.3-rc.1, etc.)."
        exit 1
    fi
done
```

The semver validation function above accepts a `-rc.x` suffix, to let us create release candidates.

Before we proceed, lets pause and to take a step back to look at the current `Fastfile` setup:

```ruby
fastlane_version "2.129.0"

default_platform :ios

platform :ios do

  name = "EmojiKit"
  main_branch = "main"

  desc "Create a new version"
  lane :version do |options|
    sh("cd .. && bash scripts/validate_git.sh " + main_branch)
    sh("cd .. && bash scripts/validate_project.sh")
    sh("cd .. && bash scripts/build.sh " + name)
    sh("cd .. && bash scripts/test.sh " + name)

    type = options[:type]
    version = version_bump_podspec(path: 'Version', bump_type: type)
    git_commit(path: "*", message: "Bump to #{version}")
    add_git_tag(tag: version)
    push_git_tags()
    push_to_git_remote()
  end

end
```

We *could* replace the `type` and `version` lines with the script above, but we're actually ready to leave `Fastlane` behind altogether, so let's instead create Shell script versions for the last four lines:

```bash
git commit -am "Bump to #$NEW_VERSION"
git tag $NEW_VERSION
git push --tags
git push -u origin HEAD
```

However, since we no longer update the version in the `Version` file, we can remove the `git commit` and just have this:

```bash
git push -u origin HEAD
git tag $NEW_VERSION
git push --tags
```

Add these three lines to the `version_bump.sh` and it now looks like this:

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Use the script folder to refer to other scripts.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
SCRIPT="$FOLDER/version_number.sh"

# Get the latest version
VERSION=$($SCRIPT)

if [ $? -ne 0 ]; then
    echo "Failed to get the latest version"
    exit 1
fi

# Print the current version
echo "The current version is: $VERSION"

# Function to validate semver format, including optional -rc.<INT> suffix
validate_semver() {
    if [[ $1 =~ ^v?[0-9]+\.[0-9]+\.[0-9]+(-rc\.[0-9]+)?$ ]]; then
        return 0
    else
        return 1
    fi
}

# Prompt user for new version
while true; do
    read -p "Enter the new version number: " NEW_VERSION

    if validate_semver "$NEW_VERSION"; then
        break
    else
        echo "Invalid version format. Please use semver format (e.g., 1.2.3, v1.2.3, 1.2.3-rc.1, etc.)."
        exit 1
    fi
done

git push -u origin HEAD
git tag $NEW_VERSION
git push --tags
```

That's all we need to be able to replace the `version_bump_podspec` script with a flexible alternative.

As a bonus, we can also remove the `Version` file and `Fastlane` folder, and remove `Fastlane` from `.gitignore`, since we can now create a Shell script that replaces the entire `version` lane.


## Step 4 - Replacing the versioning lane

The next step is to replace the `version` lane with this `scripts/version_create.sh` script, which takes a target and a branch, then performs all required validations before calling `version_number_bump`:

```swift
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Verify that all required arguments are provided
if [ $# -ne 2 ]; then
    echo "Error: This script requires exactly two arguments"
    echo "Usage: $0 <BUILD_TARGET> <GIT_BRANCH>"
    exit 1
fi

# Create local argument variables.
BUILD_TARGET="$1"
GIT_BRANCH="$2"

# Use the script folder to refer to the platform script.
FOLDER="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
VALIDATE_GIT="$FOLDER/version_validate_git.sh"
VALIDATE_PROJECT="$FOLDER/version_validate_project.sh"
VERSION_BUMP="$FOLDER/version_number_bump.sh"

# A function that run a certain script and checks for errors
run_script() {
    local script="$1"
    shift # Remove the first argument (the script path)

    if [ ! -f "$script" ]; then
        echo "Error: Script not found: $script"
        exit 1
    fi

    chmod +x "$script"
    if ! "$script" "$@"; then
        echo "Error: Script $script failed"
        exit 1
    fi
}


# Execute the pipeline steps
echo "Starting pipeline for BUILD_TARGET: $BUILD_TARGET, GIT_BRANCH: $GIT_BRANCH"

echo "Validating Git..."
run_script "$VALIDATE_GIT" "$GIT_BRANCH"

echo "Validating Project..."
run_script "$VALIDATE_PROJECT" "$BUILD_TARGET"

echo "Bumping version..."
run_script "$VERSION_BUMP"

echo ""
echo "Version created successfully!"
echo ""
```

The script will call multiple scripts in sequence, to perform all the steps that we have created earlier. We are more explicit in our argument naming here, since this script is more general.

...and with that, we're done! We have now replaced `Fastlane` and the `Version` file with a couple of basic Shell scripts, which work both separatele or in combination with each other.

Not take a look at how to use these scripts and add them into the package continuous integration.


## Step 5 - Creating new package versions

We can now create new versions of our package with `bash scripts/version_create.sh [PackageName] [MainBranch]`. The script will validate, build and test, then create and bump a new version number.

To avoid having to type the project name and branch every time we, we can create a project-specific script file in the project root, that does this for us. Let's name it `version_create.sh` as well:

```bash
SCRIPT="scripts/version_create.sh"
chmod +x $SCRIPT
chmod +x version_create.sh
bash $SCRIPT PackageName MainBranch
```

With this file in place, we can create new versions by just typing `bash version_create.sh`. And since the file makes itself executable, you can just type `./version_create.sh` for all subsequent versions.


## Step 6 - Integrate with GitHub workflows

With these scripts in place, creating GitHub Action build runners is a breeze. For instance, this is a `.github/workflows/build.yml` file that is run on every push to the `main` branch:

```yml
name: Build Runner

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: macos-15
    steps:
      - uses: actions/checkout@v3
      - uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: '16.0'
      - name: Build all platforms
        run: bash scripts/build.sh ${% raw %}{{ github.event.repository.name }}{% endraw %}
      - name: Test iOS
        run: bash scripts/test.sh ${% raw %}{{ github.event.repository.name }}{% endraw %}
```

This file will run the `build` script to build the package for all platforms, then the `test` script to run unit tests on our selected platforms.

Note how we use `{% raw %}{{ github.event.repository.name }}{% endraw %}` to pass in the repository name to the scripts. This means that the script file can be project agnostic, and easily copied between projects.

We can also create a DocC runner, to automatically publish new documentation every time we push to `main`. But first, let's create a `scripts/docc.sh` script file that performs the operation:

```swift
#!/bin/bash

# Verify that all required arguments are provided
if [ $# -eq 0 ]; then
    echo "Error: This script requires exactly one argument"
    echo "Usage: $0 <TARGET>"
    exit 1
fi

TARGET=$1
TARGET_LOWERCASED=$(echo "$1" | tr '[:upper:]' '[:lower:]')

swift package resolve;

xcodebuild docbuild -scheme $1 -derivedDataPath /tmp/docbuild -destination 'generic/platform=iOS';

$(xcrun --find docc) process-archive \
  transform-for-static-hosting /tmp/docbuild/Build/Products/Debug-iphoneos/$1.doccarchive \
  --output-path .build/docs \
  --hosting-base-path "$TARGET";

echo "<script>window.location.href += \"/documentation/$TARGET_LOWERCASED\"</script>" > .build/docs/index.html;
```

This script builds DocC documentation, then transforms it for static hosting in the `.build/docs` build folder. The last line injects a redirect from the (empty) root page to the actual documentation root.

We can now create a `.github/workflows/docc.yml` that makes GitHub Actions build and publish DocC documentation on every push to the `main` branch:

```bash
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
  
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${% raw %}{{ steps.deployment.outputs.page_url }}{% endraw %}
    runs-on: macos-15
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - id: pages
        name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Select Xcode version
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: '16.0'
      - name: Build DocC
        run: bash scripts/docc.sh ${% raw %}{{ github.event.repository.name }}{% endraw %}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.build/docs'
      - id: deployment
        name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

This runner triggers the `scripts/docc.sh` script, then grabs the generated, transformed web content and uploads it as an artifact, after which it becomes published on GitHub Pages.


## Conclusion

This became a bit longer than intended, but I'm happy that I managed to replace the entire Fastlane setup with Shell scripts. This now run faster, with more flexibility. And less Ruby.

If you want to try this out, I have published it as a [this GitHub repo]({{page.sdk}}). You can just add the scripts and GitHub workflows to any Swift package to easily build, test, and create new versions of it.

I'd love to hear what you think of this approach, and if you think any of the scripts can be improved. Reach out in the comment section below, or comment on social media, using the links below.

Thank you for reading!
---
title: Automate setting up macOS
date:  2018-08-26 17:28:01 +0200
tags:	 macos automation
icon:  avatar
---

In this post, I'll describe how to automate setting up a new Mac with a terminal script that will install system software and applications, configure the computer etc. in mere minutes.


## Why automate?

I prefer to automate as many tasks as possible, to reduce the amount of repetitive manual work, reduce the risk of human error and to increase the overall reliability of a process. 

For developers, this involves unit testing, continuous integration, release management etc. while for testers it can involve automated UI testing etc.

However, one thing that I haven't automated until recently, is to set up a brand new Mac, which is time consuming and just not fun. It's tedious to remember all the tools and apps. 

Without automation, this can easily take a day, and you will then fill out the gaps for weeks. With automation, you can set up a new Mac in minutes.


## Tools on which I base my script

Before I show you the script I have put together, let
me go through some tools that I base it on. They are great and will simplify your life, so make sure to check them out.

* [Homebrew](https://brew.sh) is a package and dependency manager (one of several) for macOS.
* [Homebrew Cask](https://github.com/Homebrew/homebrew-cask) is a brew extension
that lets you install Mac apps with Homebrew.
* [Brew Bundle](https://github.com/Homebrew/homebrew-bundle) is a brew extension
that lets you define brew packages with a `Brewfile`.
* [Gem](https://rubygems.org/pages/download) is a package manager for Ruby-based
software, e.g. Fastlane.
* [NPM](https://www.npmjs.com) is a package manager for web-related software. I also use it for hybrid apps.

My script will first install these tools, then install everything else using the tools and scripts.


## Creating the script

Let's first create the main system script. It's modular, to make it easy to adjust it over time.

First, create a file called `setup.sh` and add the following code to it:

```bash
#!/bin/bash

while true; do
  if [[ $# == 0 ]]; then
    printf "\n\n***** Setup *****\n\n"

    printf "Available commands:\n\n"
    printf "\n"
    printf "      all:  Install everything\n"
    printf "     brew:  Install packages & apps from Brewfile\n"
    printf "   config:  Configure macOS\n"
    printf "      gem:  Install packages from Gemfile\n"
    printf "      npm:  Install npm packages from scripts/npm.sh\n"
    printf "      ssh:  Create & copy SSH key\n"
    printf "   system:  Install system software\n"
    printf "\n"
    printf "        q:  Quit/Exit.\n"
    printf "\n\n"

    read -p "Enter option: " response
    printf "\n"
    process_option $response
  else
    process_option $1
  fi
done
```

This prints a "main menu" with various options. As you can see, I have split it  into several modules, with an `all` option that installs everything.

The script lacks the `process_options` function that it refers to. Add this above `while true`:


```bash
process_option() {
  case $1 in
    'all')
      source scripts/config.sh
      source scripts/system.sh
      source scripts/apps.sh
      source scripts/npm.sh
      source scripts/fastlane.sh
      source scripts/ssh.sh
      break;;
    'apps')
      source scripts/apps.sh
      break;;
    'config')
      source scripts/config.sh
      break;;
    'fastlane')
      source scripts/fastlane.sh
      break;;
    'npm')
      source scripts/npm.sh
      break;;
    'ssh')
      source scripts/ssh.sh
      break;;
    'system')
      source scripts/system.sh
      break;;
      
    'q')
      break;;
    *)
      break;;
  esac
}
```

Each option calls another script in the `scripts` folder or runs a Terminal command. Each external file is simple. The `Brewfile` contains brew and cask dependencies, the `Gemfile` gem dependencies and the script files commands that you could also type manually.

Have a look at some examples from each file:


```bash
// scripts/system.sh

if ! command -v brew > /dev/null; then
    printf "[SYSTEM] Install Homebrew\n"
    ruby -e "$(curl --location --fail --silent --show-error https://raw.githubusercontent.com/Homebrew/install/master/install)"
else
    printf "[SYSTEM] Update Homebrew\n"
    brew update
fi
printf "\n"

printf "[SYSTEM] Install Cask\n"
brew tap caskroom/cask
printf "\n"

...
```

```bash
// scripts/config.sh

printf "[CONFIG] Finder, Show hidden files\n"
defaults write com.apple.finder AppleShowAllFiles -bool true
killall Finder -9
printf "\n"
```

```bash
// scripts/npm.sh

printf "[NPM] Installing TypeScript\n"
sudo npm install -g typescript
printf "\n"
```


```bash
// scripts/ssh.sh

read -p "[SSH] Create new SSH key (yes/no): " response
if test "$response" = "yes"; then
	printf "\n"
	read -p "Enter your e-mail: " ssh_email
	printf "\n"
    printf "[SSH] Creating ssh key\n"
    ssh-keygen -t rsa -b 4096 -C $ssh_email
fi
printf "\n"

printf "[SSH] Adding ssh key to ssh-agent\n"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
printf "\n"

printf "[SSH] Copying ssh key to pasteboard\n"
pbcopy < ~/.ssh/id_rsa.pub

printf "[SSH] Done\n"
printf "\n"
```


```bash
// Brewfile

cask_args appdir: "/Applications"

brew "mas"
cask "android-studio"
...
```


```bash
// Gemfile

# frozen_string_literal: true
source "https://rubygems.org"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }
gem 'cocoapods'
...
```


## Result

If I run the `all` option, it will take care of the following:

* Configuring OS X
* Installing package managers (Homebrew, Cask, RubyGems, NPM)
* Installing brew packages (Carthage, SwiftGen, SwiftLint etc.)
* Installing Mac applications (Chrome, Slack, Sketch etc.)
* Installing gem packages (CocoaPods, Fastlane, Jekyll etc.)
* Installing npm packages (Ionic, Gulp, TypeScript etc.)
* Setting up SSH (create a key, add to ssh-agent, copy to pasteboard)

Doing this manually would take a couple of hours. Now, it finishes in a matter of minutes.


## Download

I have a GitHub repo that you can fork and tweak to fit your needs. You can find it [here](https://github.com/danielsaidi/osx).
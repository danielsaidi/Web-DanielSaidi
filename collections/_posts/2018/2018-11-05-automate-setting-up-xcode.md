---
title: Automate setting up Xcode
date:  2018-11-05 15:00:00 +0200
tags:	 xcode automation
icon:  swift
---

In this post, I will show how to automate setting up Xcode using `Homebrew` and `Fastlane` in a way that is easy to extend if you need to automate more later on.

![Xcode](/assets/blog/xcode.png){:class="plain" width="250px"}


## Why automate?

I prefer to automate as many tasks as possible, to reduce the amount of repetitive manual work, reduce the risk of human error and increase the overall reliability of a process.

For a team, automation can streamline the setup of a developer environment and make it easy to follow conventions. For instance, `swiftlint` can help to enforce code conventions, code snippets can be used to generate comments and code blocks etc.


## External dependencies

When you need external dependencies to build and run a project, consider using package and dependency managers to setup dependencies with a single command.

For instance, we can use tools like Homebrew and Fastlane and compose them in a way that makes the setup process simple and painless.

For instance, you can add a `Brewfile` to your project root and add all required tools to it:

```bash
brew "carthage"
brew "swiftgen"
brew "swiftlint" 
```

Your can now install all these tools by typing a single command in the Terminal:

```bash
brew bundle
``` 

Although `homebrew` is a standard tool, you can always make it even more accessible with `Fastlane`. Just add this lane to your `Fastfile`:

```
desc "Setup Xcode"
lane :setup_xcode do |options|
  sh "cd .. && brew bundle"
end
```

Now, you can just type `fastlane setup_xcode` to install all tools. It may seem like a no-win, but the nice thing with this approach is that it can be extended to handle even more tasks.


## Linting

Linting is a way to automatically validate that code follows established conventions. To do this, you use a code analyzer tool called a linter.

`swiftlint` is a great Swift linter that triggers warnings and errors if the  code doesn't follow certain conventions. You can customize the standard setup by adding a `.swiftlint.yml` to the project root, in which you can ignore rules, tweak them, add new rules, etc.

The SwiftLint readme suggests that you add this `Run Script Phase` to your app target:

```bash
if which swiftlint >/dev/null; then
    swiftlint
else
    echo "SwiftLint does not exist, download from https://github.com/realm/SwiftLint"
fi
```

Since this is a critical tool, I prefer it to be mandatory, and therefore skip the if/else check:

```bash
swiftlint
```

This means that the app fails to build whenever SwiftLint isn't available, which means that it has gone from being an optional to a required tool.

Since Swift Packages don't have a build phase where you can add SwiftLint. I instead add it to certain lanes, to make it part of critical processes, like creating new versions.


## Xcode snippets

Xcode snippets can be used quickly add text you type often. For instance, I use it for `MARK` blocks, extension bodies, test templates, etc. 

Xcode snippets save a lot of time and make the code look the same across the codebase.

For instance, I have a snippet that creates a "plain" `// MARK - ` and have it defined in a file named `mark_plain.codesnippet`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>IDECodeSnippetCompletionPrefix</key>
  <string>mark_plain</string>
  <key>IDECodeSnippetCompletionScopes</key>
  <array>
    <string>All</string>
  </array>
  <key>IDECodeSnippetContents</key>
  <string>// MARK: - </string>
  <key>IDECodeSnippetIdentifier</key>
  <string>6A7BC3C9-32A4-4EE8-A8DC-7848BC0E40F3</string>
  <key>IDECodeSnippetLanguage</key>
  <string>Xcode.SourceCodeLanguage.Swift</string>
  <key>IDECodeSnippetTitle</key>
  <string>Mark</string>
  <key>IDECodeSnippetUserSnippet</key>
  <true/>
  <key>IDECodeSnippetVersion</key>
  <integer>2</integer>
</dict>
</plist>
```

Since snippets are powerful, I decided to automate how my team shares these snippets. It was very easy, since snippets are just text files. 

It therefore basically just involved setting up a shared folder with snippets files and writing a script that copies these files to the correct place.

For my personal projects, I just added a snippet folder to a setup project that I made public [here](https://github.com/danielsaidi/osx), then created a script that copies these files to the correct place. 

The copy script looks like this:

```bash
#!/bin/bash

path_src=Snippets
path_dst=~/Library/Developer/Xcode/UserData/CodeSnippets
mkdir $path_dst
for file in $path_src/*.codesnippet; do
  cp $file $path_dst/$(basename $file)
done
```

At work, where my team works on many projects that has shared conventions, I added a snippet folder to a shared repo, then added work-specific snippets to it.

After that, I extended `setup_xcode` to copy all these
snippets to their correct place as well:

```
desc "Setup Xcode"
  lane :setup_xcode do |options|
    setup_xcode_tools
    setup_xcode_snippets
  end

  desc "Setup Xcode Tools"
  lane :setup_xcode_tools do |options|
    sh "cd .. && brew bundle"
  end

  desc "Setup Xcode snippets"
  lane :setup_xcode_snippets do |options|
    snippets_path = File.expand_path('../Snippets/*.codesnippet')
    snippets_paths = [snippets_path].flatten
    snippets = snippets_paths.map { |f| f.include?("*") ? Dir.glob(f) : f }.flatten
    target_path = File.expand_path('~/Library/Developer/Xcode/UserData/CodeSnippets')
    FileUtils.cp_r(snippets, target_path, remove_destination: true)
  end
```

Running `fastlane setup_xcode` will now run `brew bundle` and copy snippets. This means that we have a way to set up Xcode, that we can easily extend whenever we need.


## Conclusion

Shell scripts, dependency managers and Fastlane are convenient ways to set up Xcode with a single command, which can help you standardize a personal or work environment.

Tools like `swiftlint` make it easy to enforce code conventions, while code snippets can generate code and comments that should follow a desired format.

Feel free to check out [my setup script](https://github.com/danielsaidi/osx) for some examples, scripts and code snippets.
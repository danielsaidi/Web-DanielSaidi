---
title:  Optimizing Claude Code's Xcode build token usage
date:   2026-09-30 06:00:00 +0100
tags:   ai

assets: /assets/blog/26/0930/
image: /assets/blog/26/0930/image.jpg
image-show: 0

article:    https://www.avanderlee.com/ai-development/reduce-token-usage-claude-code-codex-cursor/
swiftlee:   https://www.avanderlee.com
xcsift:     https://github.com/ldomaradzki/xcsift
---

This post is based on Antoine van der Lee's great article [How to reduce token usage in Claude Code, Codex, and Cursor]({{page.article}}) on his blog [SwiftLee]({{page.swiftlee}}). All credit goes to him. I'm just writing this down as a note to my future self.


## Background

In [his article]({{page.article}}), Antoine explains how AI agents like Claude Code spend a lot of tokens reading Xcode's verbose build output, which is mostly noise.

Antoine therefore uses [xcsift]({{page.xcsift}}) to compress the Xcode build output into a compact format that keeps errors, warnings and test results. You can install it with Homebrew:

```bash
brew install xcsift
```

Antoine tried adding instructions to use xcsift to his agents, but found that this was mostly ignored. Instead, he therefore uses hooks to automatically pipe all build commands through xcsift.


## The Claude Code Hook Script

While [Antoine's article]({{page.article}}) covers how to register hooks for Codex, Cursor, and Claude Code, I will only include a simpler script that's needed for Claude Code.

First, save this script as `~/.claude/hooks/xcode-output.py`. It runs before every Bash command, and rewrites `xcodebuild`, `swift build` and `swift test` commands to pipe their output through xcsift:

```python
#!/usr/bin/env python3
"""A Claude Code PreToolUse hook that pipes Xcode build output through xcsift."""

import json
import re
import sys

BUILD = re.compile(r"^(xcodebuild|swift\s+(build|test))\b")
FILTERS = ("xcsift", "xcbeautify", "xcpretty")
UNSAFE = re.compile(r"[|;<>`\n]|\$\(|(?<!&)&(?!&)")


def rewrite(command):
    """Return a rewritten command, or None if it should be left untouched."""
    if any(f in command for f in FILTERS) or UNSAFE.search(command):
        return None
    *prefix, build = [part.strip() for part in command.split("&&")]
    if not BUILD.match(build) or any(not p.startswith("cd ") for p in prefix):
        return None
    steps = prefix + [f"{build} 2>&1 | xcsift -f toon -w"]
    return "set -o pipefail && " + " && ".join(steps)


def main():
    data = json.load(sys.stdin)
    tool_input = data.get("tool_input", {})
    command = rewrite(tool_input.get("command", "").strip())
    if command is None:
        print("{}")
        return
    print(json.dumps({
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "allow",
            "permissionDecisionReason": "Piping build output through xcsift",
            "updatedInput": {**tool_input, "command": command},
        }
    }))


if __name__ == "__main__":
    main()
```

The script leaves commands untouched if they use a formatter, or if they contain pipes, redirects or other command chains that would be unsafe to rewrite.

The `set -o pipefail` makes builds still report failures, even though the output goes through xcsift.


## Registering the Hook

Register the hook as a `PreToolUse` hook for the `Bash` tool in `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/xcode-output.py",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

If your settings file already has a `hooks` section, just add the `PreToolUse` entry to it. Restart Claude Code, and every build command will now return compressed output.


## Conclusion

This agent hook can save a lot of tokens when working on Xcode projects with Claude Code. Make sure to read [Antoine's article]({{page.article}}) for more tips, and how to set up the same hook for Codex and Cursor.

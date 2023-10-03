---
categories:
- Code
date: "2022-07-22T00:00:00Z"
excerpt_separator: <!-- more -->
sub_title: A new experimental C++ successor by the big G
tags:
- carbon
- Google
- languages
title: Carbon Language
---

After revisiting three languages in the last three weeks from "Hello, World!" to advanced concepts. And also a little stint into lesser known languages, like [👑 nim](https://nim-lang.org/) now, something new: [https://github.com/carbon-language/carbon-lang](https://github.com/carbon-language/carbon-lang). 😊

<!--more-->

## Getting Started

```bash
# Install bazelisk using Homebrew.
$ brew install bazelisk

# Install Clang/LLVM using Homebrew.
# Many Clang/LLVM releases aren't built with options we rely on.
$ brew install llvm
$ export PATH="$(brew --prefix llvm)/bin:${PATH}"

# Download Carbon's code.
$ git clone https://github.com/carbon-language/carbon-lang
$ cd carbon-lang

# Build and run the explorer.
$ bazel run //explorer -- ./explorer/testdata/print/format_only.carbon
```

From the project's README it seems like most of Google is coding on Mac OSX and using brew. Even though it should be possible to use Fedora's built-in tools we follow the recommended path (even risking to mess up the system).

### Requirements

Homebrew using this [canonical install command](https://brew.sh/).

```bash
brew upgrade
brew install bazelisk
brew install llvm
```

Added `/home/linuxbrew/.linuxbrew/bin/brew update --auto-update` to autostart to keep brew up to date.

### Carbon-Lang

Install Carbon-Lang

```bash
cd ~/src
git clone https://github.com/carbon-language/carbon-lang
cd carbon-lang

# Build and run the explorer.
bazel run //explorer -- ./explorer/testdata/print/format_only.carbon
```

### Trouble?

Eventually one needs to add `export PATH="$(brew --prefix llvm)/bin:${PATH}"` to whatever shell RC there is. Then `bazel clean`.

Should return

```bash
0> bazel run //explorer -- ./explorer/testdata/print/format_only.carbon
INFO: Invocation ID: 4c7a1d45-233c-4156-bd59-204bc56b2efc
INFO: Analyzed target //explorer:explorer (67 packages loaded, 1560 targets configured).
INFO: Found 1 target...
Target //explorer:explorer up-to-date:
  bazel-bin/explorer/explorer
INFO: Elapsed time: 56.876s, Critical Path: 42.13s
INFO: 542 processes: 1 disk cache hit, 161 internal, 380 linux-sandbox.
INFO: Build completed successfully, 542 total actions
INFO: Build completed successfully, 542 total actions
Hello world!
result: 0
```

### Get Tooling straight

This project comes with a very good `pre-commit` setup:

```bash
sudo dnf install pre-commit
pre-commit run # in the folder
```

There is a bunch of stuff [recommended in the repo](https://github.com/carbon-language/carbon-lang/blob/trunk/docs/project/contribution_tools.md#optional-tools).

```bash
brew install black
brew install prettier
brew install codesmell
```

There exist also some recommended [Visual Studio Code extensions](https://github.com/carbon-language/carbon-lang/blob/trunk/.vscode/extensions.json).

## First steps

From the current README:

```carbon
// Carbon:
package Geometry api;
import Math

class Circle {
    var r: f32;
}

fn PrintTotalArea(circles: Slice(Circle)) {
    var area: f32 = 0
    for (c: Circle in circles) {
        area += Math.Pi * c.r  * c.r;
    }
    Print("Total area: {0}", area)
}

fn Main() -> 32 {
    var circles: Array(Circle) = ({.r = 1.0},
                                  {.r = 2.0});
    PrintTotalArea(circles);
    return 0;
}
```

It's also interesting to explore the [Compiler Explorer](https://carbon.compiler-explorer.com/).

### The mood?

The similarities to C# are quite interesting... but then, I am certain Google will not patch every god damn paradigm and language feature there is on this planet into the language such that a so-called "[Pocket Reference](https://www.oreilly.com/library/view/c-10-pocket/9781098122034/)" comes with >250 pages in a A5 format which doesn't fit any pocket... 

![image](https://user-images.githubusercontent.com/1167114/180646925-530a24e1-4812-4e7e-bc3c-c0ccbfbe8fdc.png)


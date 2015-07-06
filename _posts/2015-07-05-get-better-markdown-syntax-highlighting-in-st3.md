---
layout: post
title: Get Better Markdown Syntax Highlighting in Sublime Text 3
---

<!-- links -->
[markdown extended]: https://github.com/jonschlinkert/sublime-markdown-extended
[afterglow]: http://yabatadesign.github.io/afterglow-theme/

[before]: /images/beforeMDL.png
[after]: /images/afterMDL.png

[woLang]: /images/fenceWithoutLang.png
[wiLang]: /images/fenceWithLang.png
<!-- post -->

## Problem ##
Many color themes for Sublime Text 3 do not have very good support for highlighting Markdown syntax.

Here's a solution for those that want to use a theme with Markdown-support only for Markdown files and keep their preferred theme for everything else.

<!--excerpt-->

**Before**: No syntax highlighting whatsoever for the heading, numbered list, code, or code block using my preferred theme
![Without Markdown Light][before]

## Solution ##
1. Install MarkdownLight
  - In Sublime, press <kbd>cmd</kbd> + <kbd>shift</kbd> + <kbd>p</kbd> to open package control
  - Install Package: MarkdownLight

2. Associate all .md files with Markdown Light syntax
  - While in a file with .md extension, go to _View > Syntax > Open all with current extension as... > Markdown Light_

3. Keep your preferred syntax theme for all non-Markdown files
  - While in a file with .md extension, go to _Preferences > Settings - More > Syntax Specific - User_
  - Add the property `"color_scheme": "Packages/User/SublimeLinter/MarkdownLight (SL).tmTheme"`
  - Choose whatever color scheme you usually like under preferences. ST3 will only use MarkdownLight for files with the .md extension.

**After**: Heading, numbered list, code, and code block easily identified
![With Markdown Light][after]

To get proper syntax highlighting within code blocks, specify the langauge after the first set of backticks to have MarkdownLight color your code for you in ST3:

**Before**:
![Fenced code block without Markdown Light][woLang]

**After**:
![Fenced code block with Markdown Light][wiLang]

## Alternatives ##

If Markdown Light does not suit your tastes, you can follow the same procedure above to associate any Markdown color theme with .md files. One possible alternative is [Markdown Extended][markdown extended], which uses the Monokai color theme with Markdown support. Another is [Afterglow][afterglow].


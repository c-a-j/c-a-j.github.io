+++
title = "Code blocks examples Kaushal Modi"
author = ["Clint Jordan"]
description = "Collection of examples for this website."
date = 2018-05-02
lastmod = 2019-09-06T12:32:13-04:00
tags = ["ramblings"]
categories = ["unix"]
draft = true
+++

<a id="code-snippet--delta-gitconfig"></a>
```bash
[core]
    pager = delta

[interactive]
    diffFilter = delta --color-only
[add.interactive]
    useBuiltin = false # required for git 2.37.0

[diff]
    colorMoved = default

[delta]
    # https://github.com/dandavison/magit-delta/issues/13
    # line-numbers = true    # Don't do this.. messes up diffs in magit
    #
    side-by-side = true      # Display a side-by-side diff view instead of the traditional view
    # navigate = true          # Activate diff navigation: use n to jump forwards and N to jump backwards
    relative-paths = true    # Output all file paths relative to the current directory
    file-style = yellow
    hunk-header-style = line-number syntax
```
<div class="src-block-caption">
  <span class="src-block-number"><a href="#code-snippet--delta-gitconfig">Code Snippet 2</a>:</span>
  My configuration for <code>delta</code> in <code>.gitconfig</code>
</div>


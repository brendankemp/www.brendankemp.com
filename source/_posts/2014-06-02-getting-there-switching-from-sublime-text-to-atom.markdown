---
published: false
layout: post
title: "Getting There: Switching from Sublime Text to Atom"
date: 2014-06-02 07:41:47 -0400
comments: true
categories: work-flow atom
published: false
---

Atom has been open-sourced, so now is a good time to declare it the winner. It is a little slow (especially to boot), but now that it has been open-sourced, and has a full-time team supporting it at GitHub, I think it's a good bet that it will be the editor to use for everyone who does ruby or python work in the near future. The question is: it's currently a beta product. Is it ready to start using for day-to-day development? Here are some basic things that I am looking for to make the switch:

* launch from shell
* incremental search
* emacs-style word jumps
* universal search and replace
* cmd-t style file opening
* jump to method definition
* Run tests for the current file

A couple nice-to-haves:  
* project-wide auto-complete
* cursor duplication
* easy access to the command line
* Delete trailing spaces on save
* Add a closing newline on save
* Line and selection duplication
* Package manager
* Git integration

### Launch from shell

This Atom has built in with the `atom` command. Just like Sublime Text, you will always want to launch Atom from the command line by `cd`-ing into your project directory and using `atom .` so Atom will use that as your project directory.

### Incremental search

This is how I navigate files without the mouse: incremental search to jump to a nearby match, then emacs-style character and word jumps to refine the selection as necessary. The problem is: there is no incremental search available! We'll see if that is a deal-breaker, or maybe I can use jump-to-symbol in conjunction with the with the emacs navigation to get most of the way there until incremental search is implemented.

### Emacs-style word jumps

This is available by default, which is a relief, because in most OSX apps, `alt-b` and `alt-f` are bound to insert some special characters (`∫ƒ`). `ctrl-`, `n`, `p`, `a`, and `e` all work, as they usually do.

### Universal search and replace

`cmd-shift-f` brings this up, and it is awesome because the results list it keyboard navigable. Also, the filter field filters by paths (`app/`) or by file patterns (`.rb`). It seems really fast too. Then! If you enter something in the `replace` field, the results will *live preview* in the find results.

### Cmd-t file opening

The `cmd-t` file opener that was created first in TextMate has become the gold standard for most programmers. That functionality was copied in Sublime, ported to emacs and vim, and now copied to Atom. It works well. As a bonus, `cmd-b` does the same thing, but only for the buffers (tabs ;) that are open.

### Jump to method definition

First you have to install ctags `brew up && brew install ctags`, then generate them for your project: `ctags -R app lib`. Then `cmd-opt-down` will jump to the method definition. The problem is, you have to keep your ctags updated. If you add a new method, Atom won't know about it until you regenerate ctags. Right now, there doesn't seem to be package or a way to automate regenerating ctags. It looks like an opportunity for someone to write [autotags](http://www.vim.org/scripts/script.php?script_id=1343) for Atom.

### Run tests for the current file (Ruby)

This is available via the `atom-ruby-test` package. In your shell: `apm install ruby-test`, then re-open your project so the package can load. I had to change the package settings by opening preferences (`cmd-,`) searching for Ruby Test in the installed packages (left sidebar), and then configuring it to use Zeus.

![Configure Ruby Test package to use Zeus](/images/posts/configure-ruby-test-for-zeus.jpg)

```
# Test all command
zeus rake

# Test file command
zeus test {relative_path}

# Test single command
zeus test {relative_path}:{line_number}
```

^ That is just about the easiest integration I've done with a test-running package and my project's test setup, and it works perfectly.

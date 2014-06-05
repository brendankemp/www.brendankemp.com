---
layout: post
title: "Atom is ready to be your editor for the next 10+ years"
date: 2014-06-05 10:09:00 -0400
comments: true
categories: workflow atom
published: true
---

Atom is already good enough to become my primary editor. Its combination of ease of extensibility and powerful editing could make it the editor of choice for serious programmers. The fact that it's open source and already has a strong community means it has a very low chance of abandonment and stagnation.

What the core team needs to do now is improve the (already quite good) responsiveness to remove all doubt about the viability of JS/HTML technology stack, and ship binaries for Windows and Linux. And in fact, those are the goals that the core team has identified for the 1.0 release. I believe that they can do it, and then there will be nothing preventing Atom from being your #1 editor for the next 10+ years.

Don't take my word for it, though. Let's see how far along Atom is already. I made a list of features I thought I needed to use an editor in my day-to-day work:

* launch from shell
* incremental search
* emacs-style word jumps
* universal search and replace
* cmd-t style file opening
* jump to method definition
* run tests for the current file

A few nice-to-haves:  

* project-wide auto-complete
* cursor duplication
* delete trailing spaces on save
* add a closing newline on save
* line and selection duplication
* package manager
* Git integration
* Markdown support

### Launch from shell

This Atom has built in with the `atom` command. Just like Sublime Text, you will always want to launch Atom from the command line by `cd`-ing into your project directory and using `atom .` so Atom will use that as your project directory.

### Incremental search

This is how I navigate files without the mouse: incremental search to jump to a nearby match, then emacs-style character and word jumps to refine the selection as necessary. The problem is: theres no incremental search available! I'll have to try to get by without it.

### Emacs-style word jumps

These are available by default. That's a relief because in most OS X apps `alt-b` and `alt-f` are bound to insert special characters (`∫ƒ`).

### Universal search and replace

`cmd-shift-f` brings this up, and it is awesome because the results list it keyboard navigable. Also, the filter field filters by paths (`app/`) or by file patterns (`.rb`). It seems really fast too. Then! If you enter something in the `replace` field, the replacements will *live preview* in the find results. Atom's search and replace is already the nicest out of any that I've used.

![Atom search and replace](/images/posts/search-and-replace.gif)

### Cmd-t file opening

The `cmd-t` file opener that was created first in TextMate has become the gold standard for file navigation. That functionality was copied in Sublime, ported to emacs and vim, and now copied to Atom. It works well. As a bonus, `cmd-b` does the same thing, but only for the buffers (tabs ;) that are open. `cmd-shift-b` searches only the files have been modified since last commit (the git status list).

### Jump to method definition

Ctags is the default recommendation for symbol indexing, but it has to be installed and maintained by the user. You are going to want a package to maintain ctags, append and delete them when you save a file. There doesn't seem to be a package for this at the momement. It also doesn't support Coffeescript or CSS.

[Goto](https://github.com/mkleehammer/goto) takes a promising approach to this problem. By using the Atom syntaxes to find symbols, it (theoretically) supports all the languages that Atom does. It also updates the symbol index for you, so there is no manual ctag maintenance. The problem is that it is an early stage of development, and needs optimizing and debugging.

Neither option is perfect. Goto seems like it will be the best solution, but right now it is too slow. I think I'll be manually managing ctags for my projects until Goto gets to a point where it can be used.

With either system you use, (`cmd-r`) is for jumping to a method definition within a file, (`cmd-shift-r`) is for jumping to methods across a project, and to jump from a method call to its definition, place cursor over method call, and press `cmd-opt-down`.

### Run tests for the current file (Ruby)

This is available via the `ruby-test` package. In your shell: `apm install ruby-test`, then re-open your project so the package can load. I had to change the package settings by opening preferences (`cmd-,`) searching for Ruby Test in the installed packages (left sidebar), and then configuring it to use Zeus.

![Configure Ruby Test package to use Zeus](/images/posts/configure-ruby-test-for-zeus.jpg)

```bash
# Test all command
zeus rake

# Test file command
zeus test {relative_path}

# Test single command
zeus test {relative_path}:{line_number}
```

^ That is just about the easiest integration I've done with a test-running package and my project's test setup, and it works perfectly. <3

### Project-wide autocomplete

Something that I had when I used emacs, and have missed since, is project-wide autocomplete. That is, not just autocompleting with words from the same file, but something that autocompletes on method definitions across the whole project you are working on. Ideally, it would know what is a method call in this syntax across the whole project and make those completions available, while preferring matches from the current file.

Nothing exists like that right now. There is the built-in autocomplete function, that will autocomplete when you press `ctrl-space`. That will only autocomplete matches in the current file. Then there is the [autcomplete-plus](https://github.com/saschagehlich/autocomplete-plus) package, which will match across all open buffers, which is marginally better. By default, it pops up the autocomplete menu every time you start typing. There is an option to disable auto-activation, and change the keyboard shortcut to `ctrl-space`, from the default `ctrl-shift-space`. I'm going to try it out for a bit, but I'm not sure how useful it will be.


### Cursor duplication

Cursor duplication is now an essential part of how I work. It's cognitively lighter-weight than a find-and-replace, but it allows you to do quick batch updates. Atom has decent support, with both `cmd-d` and `ctrl-shift-down`. The details that I miss from Sublime are the ability to skip a match with `cmd-k` and to undo a match with `cmd-u`.

### Delete trailing whitespace & Add trailing newline on save

Both are supported by the [Whitespace](https://github.com/atom/whitespace) package that ships with Atom.

### Package manager

This is one of the nicest features of Atom, even in its beta stage. Not only is the in-app interface for managing packages pretty stellar, there is also a command line utility, `apm`. That allows you to search for, install, and uninstall packages. The one big shortcoming I found was not being able to easily search for packages, either by their popularity or by their purpose. `apm` could use a more complete rubygems.org-type site, and probably a rubytoolbox.org type categorizer as well.

### Git integration  

Atom (unsurprisingly, from Github) ships with amazing git integration: color bars in the gutter to indicate changes, branch name in the lower right and a lines changed summary. [Git plus](https://github.com/akonwi/git-plus) adds some vim-fugitive-like functionality, including adding, commiting, pushing, pulling and branching.

![Atom Git status](/images/posts/atom-git-status.jpg)

### Markdown support

One of the nicest surprises has been the Markdown support of the Atom Monokai theme. It does highlighting/styling on most Markdown constructs **and syntax-aware highlighting inside code fences!!** The snippets that come with the Markdown package are also handy, and I find myself using them to add links, images and bold styling as I write this. One other note: the Whitespace package is configured to allow trailing whitespace for markdown files. One more thing that just works that would have taken some fiddling in Sublime or other editors.

![Markdown syntax highlighting in code fences](/images/posts/atom-markdown-code-highlighting.jpg)

### Sweet, sweet flat icon

I stumbled upon an icon that looks a little more modern than the official icon. Download the full-size version [here](https://dribbble.com/shots/1454329-Atom-Icon).

![Devin Ruppert Atom Icon](/images/posts/devin-ruppert-atom-icon.png)

### Overall

Atom already exceeds my expectations. My main concern was the speed and responsiveness, because in earlier versions that was bad enough to make it unusable. Not so anymore. In fact, in the current version, lag is rarely an issue. And the capabilities of the editor and related packages are already impressive. The lack of incremental search was the biggest dissapointment, but that was balanced by best-ever implementations of search-and-replace and test integration. I'm confident enough now in the editor, its chosen technologies, and its community to make the switch to using Atom as my main editor. To the next 10 years of programming with Atom!

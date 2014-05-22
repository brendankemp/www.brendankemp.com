---
published: false
layout: post
title: "How to add a shortcut for any command in Sublime Text"
date: 2014-05-22 08:30
comments: true
categories:
---

Two of my favorite commands in Sublime Text are `upper_case` and `lower_case`. I've remapped `<Caps Lock>` to `<Ctrl>`, so if I have to type `SOME_LONG_CONSTANT_IN_SCREAMING_SNAKE_CASE`, I can type it all out in lower case and press `super+k, super+u`.

Now I'd like the same thing for title case. I'd like to highlight some text, and type `super+k, super+t` and have it all titleized.

## Find the Sublime command

I don't know if Sublime has this function, so I open up the help menu and search 'title case'. A command is available under `Edit > Convert Case > Title Case`.

![Search Help for Title Case](/images/posts/search-help-for-title-case.png)

To find the internal command name for that function, open up the console (ctrl+\`) and enter this to tell Sublime to log every command it receives:

```python
sublime.log_commands(True)
```

Now select `Title Case` in the `Edit` menu, and it will show up in the Sublime console.

![Title case command logged in Sublime console](/images/posts/title-case-command-in-sublime-console.png)

## Create a keyboard shortcut

Now we know that the command name is `title_case` (who knew?), we can create a keyboard shortcut for it. Open up `Sublime Text > Preferences > Key Bindings — User` and add an entry:

```json
{ "keys": ["super+k", "super+t"], "command": "title_case" },
```

This key combo is a little different. If you pass an array to the `"keys"` key like that, it means to release the keys between presses. So you hold `super`, press and release `k`, then press and release `t`. It's common in Sublime to prefix keyboard shortcuts with `super+k` for commands that are higher level or less commonly used.

If you want to use the more common shortcut pattern of pressing all keys at once, you can see many examples in `Sublime Text > Preferences > Key Bindings — Default`. Here is one example:

```json
{ "keys": ["super+shift+s"], "command": "prompt_save_as" },
```

## Pass arguments to a command

I immediately added a second shortcut for toggling word wrap:

```json
{ "keys": ["super+k", "super+w"], "command": "toggle_setting", "args": { "setting": "word_wrap" } },
```

If you need to pass arguments to a command, that's how it's done.

## That's it

Now you know how to discover the command name for any function available from the menus. Use it to automate your workflow. Any time you feel the pain of doing something with the mouse, consider creating a keyboard shortcut.

Any tips I'm missing? Anything else you want to know about? Send me an email at the address in the footer.

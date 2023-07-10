---
title: 'Trying out Helix as my text editor'
date: Sun, 09 Jul 2023 15:00:00 -0700
tags: []
---


[Helix](https://helix-editor.com) is a modal editor in the same vein as Vim which means that there are different "modes" where
the same key strokes mean different things. The main benefit for a modal editor is that many complex
actions can be handled with just the keyboard rather than using the mouse or menus.
I've been a Vim user for over a decade so I was interested to see what a modal editor that doesn't
come from that lineage would look like. My motivation was the complexity of getting my Neovim
config set up just right when working with language servers and tree-sitter. Helix had these
features built in so I wanted to see if Helix could improve my productivity without having to
mess around with a lot of plugins.

I installed helix using homebrew as recommended (<kbd>brew install helix</kbd>). Initally I was confused
about the command I needed to run. Turns out that it isn't Helix but `hx`. The documentation
directed me to use the tutorial with `hx --tutor` and I'm glad I did as it presented the editing
commands in a logical and easy way. I did want to skip ahead at some points as I was familiar with
and there are many similarities but as you get further through the differences show up. You'll want to
keep a link to the [keymap](https://docs.helix-editor.com/keymap.html) handy.
The are basic similarities in the key bindings but in helix the grammar is often reversed.
For example, instead of typing <kbd>dw</kbd> you would type <kbd>wd</kbd>. This small change is really interesting
as you can see what you are selecting (and potentially modify it) before doing anything
destructive. It's sort of like having Vim visual mode always being on. 
Comands that normally move the cursor in vim will change the selection in helix. So, <kbd>w</kbd>
won't just move to the end of the word but instead select the whole word. For basic incantations
like <kbd>dw</kbd> (in Vim) or <kbd>wd</kbd> (in Helix) this will have the same effect - the word will be deleted.

The muscle memory from Vim to Helix can be tough. In particular, I'm fond of typing <kbd>x</kbd> to delete
a single character and <kbd>gg</kbd> (or <kbd>G</kbd>) to move to parts of the current file. Having these be different
things in Helix took some getting used to. But not in a bad way. I enjoy Helix take on `g` command
to let me move to all parts of the line or file. The mnemonic for `g` being "go" in Helix versus "global"
in Vim seems more natural to me.
I do miss marks though. A very common workflow for me was setting locations in Vim using <kbd>ma</kbd> or <kbd>mb</kbd>
to jump around a buffer. In Helix, `m` is for entering "match mode" that lets you select different parts
of the buffer. It's very useful just hard to break habits. There currently isn't an equivelent to vim's
marks in helix. Instead you can save locations in a jump list using <kbd>Crtl</kbd> + <kbd>s</kbd> and then returning
to those locations using <kbd>Space</kbd> + <kbd>j</kbd> and choosing them from a pop up dialog. It's certainly not
as fast so I hope the helix developers consider having named marks in the future.  

The other issue I ran into was the default python configuration requires `pylsp` which doesn't
work well with my setup that uses `pyenv`.
Because I use per-project python environments but `pyenv` creates a shim in my path which Helix
picks up and uses. The language server command needs to be in your path and I couldn't figure out
how to get it to work without installing `pylsp` into all of my environments. I ended up using
`pywright` which I installed globally through `npm`. It's not the cleanest solution but it works.  

## Installation
```sh
brew install helix npm
npm install --location=global pyright
```

## Config
### `~/.config/helix/config.toml`
```toml
theme = "noctis"

[editor]
line-number = "absolute"

[editor.cursor-shape]
insert = "bar"
normal = "block"
select = "underline"

[editor.file-picker]
hidden = false
```

### `~/.config/helix/languages.toml`
```toml
[[language]]
name = "python"
scope = "source.python"
injection-regex = "python"
file-types = ["py","pyi","py3","pyw","ptl",".pythonstartup",".pythonrc","SConstruct"]
shebangs = ["python"]
roots = ["setup.py", "setup.cfg", "pyproject.toml"]
comment-token = "#"
language-server = { command = "pyright-langserver", args = ["--stdio"] }
indent = { tab-width = 4, unit = "    " }
# will get "Async jobs timed out" errors if this empty config is not added
config = {}
```

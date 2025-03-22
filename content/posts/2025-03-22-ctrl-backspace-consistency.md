+++
date = '2025-03-22T19:07:01+01:00'
title = 'Annoying Linux: Ctrl-Backspace'
summary = "Why doesn't it behave the same everywhere?!"
tags = ["#annoyingLinux"]
+++

In the majority of systems that I interact with, <kbd>Ctrl</kbd> - <kbd>←</kbd>/<kbd>→</kbd>, means
*"move one word left or right"*. And naturally then, <kbd>Ctrl</kbd> - <kbd>⌫</kbd>/<kbd>Delete</kbd>,
means *"delete previous word* or *"delete next word"*.


On the graphical side of Linux systems, this seems to be the default behavior, but not when
interacting with the terminal.

In `bash` and `zsh`, <kbd>Ctrl</kbd> - <kbd>Delete</kbd> behaves as expected, but
adding <kbd>Ctrl</kbd> to <kbd>⌫</kbd> doesn't change it's behavior at all. Instead, you'll notice
that <kbd>Alt</kbd> - <kbd>⌫</kbd> does what you expect. This probably stems from the ancient
history of emacs.

And in vim, nothing works (surprised?).

## How do we fix it?

For `bash`, we can fix this by adding: `"\C-H":"\C-W"` to `~/.inputrc`.

*What does that string even mean? And how does it work?*

That's a tangent I don't want to get into right now. Checkout the
[Further reading](#further-reading) section if you're interested.

For `zsh`, we have to use `bindkey`. Add this line to your `~/.zshrc` file:

```bash
bindkey '^H' backward-kill-word
```

And finally, vim. To fix <kbd>Ctrl</kbd> - <kbd>Delete</kbd>, you can simply add this to `~/.vimrc`

```text
" Make Ctrl-Delete delete the next word in "normal mode"
nnoremap <C-Del> dw
" ditto "insert mode"
inoremap <C-Del> <C-o>dw
```

But to fix <kbd>Ctrl</kbd> - <kbd>⌫</kbd>, I can't just ask you to copy paste some text, because
the `^H` in the snippet below is just a visual representation of the actual value.

**Open `~/.vimrc` with vim**. Then, instead of writing `^H` in the file, **in insert mode**, press
<kbd>Ctrl</kbd> - <kbd>V</kbd> and then <kbd>Ctrl</kbd> - <kbd>⌫</kbd>. You should then see a `^H`
appear.

In the end, you should have this:

```text
" Equivalent for ctrl-backspace
nnoremap ^H db
inoremap ^H <C-o>db
```

See you in the next one!

## Further reading

* `man 1 stty`
* `man 3 readline`
* <https://en.wikipedia.org/wiki/Control_character#In_ASCII>
* <https://en.wikipedia.org/wiki/Caret_notation>

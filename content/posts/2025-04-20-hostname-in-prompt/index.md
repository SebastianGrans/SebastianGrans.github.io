+++
date = '2025-04-20T20:07:49+02:00'
draft = false
title = 'Custom Prompt When Connected Over SSH'
summary = 'To prevent me from deleting the wrong files...'
+++

I use `zsh` with a theme called `robbyrussell` that makes my prompt very minimalistic. I already
know who I am, and what machine I'm on, so the standard `user@host` prompt is just noisy.

```bash
➜  ~ echo "Hello"
```

I quite like it, except when I SSH to my main machine at work, then it suddenly becomes difficult
to remember which terminal is which.

Let's prefix my prompt with the hostname, but only if I'm connected over SSH.

## Customizing the prompt

**TL;DR:** Add this snippet to your `~/.zshrc` or `~/.bashrc` (on the remote machine)

```bash
if [[ -n $SSH_CONNECTION ]] ; then
    PS1="[$(hostname)] $PS1"
fi
```

And your prompt will look like this when you connect to it

```bash
[some-pc-name] ➜  ~ echo "Hello"
```

But why stop there? It's always nice with a bit of flair! So I came up with this:

```bash
if [[ -n $SSH_CONNECTION ]]; then
    hostname_string="%{$fg_bold[blue]%}[%{$fg_bold[yellow]%}%U$(hostname)%u%{$fg_bold[blue]%}]%{$reset_color%}"
    PS1="$hostname_string $PS1"
fi
```

And how we end up with this when I SSH into my laptop.

![alt text](img.png)

Ant that's good enough for me.

## How does it work?

`SSH_CONNECTION` is an environment variable that is set when you connect via SSH. We're just using
it as a flag, but it contains the IP of the remote and local machine (see `man ssh`).

`PS1` is another environment variable, the primary prompt string. This string is what defines your
prompt.

In the first example, I'm simply prepending the existing `PS1` variable with `[$(hostname)]`. In
the second example, it's a bit more complicated. There `hostname_string` is formatted according to
the section `EXPANSION OF PROMPT SEQUENCES` in `man zshmisc`.

But we also make use of the variables `fg_bold` and `reset_color`. Which you can read more about
in `man zshcontrib` under `OTHER FUNCTIONS`.

## Further reading

* `man zshall` - All `zsh` manuals in one big `man` page.
* `man bash`

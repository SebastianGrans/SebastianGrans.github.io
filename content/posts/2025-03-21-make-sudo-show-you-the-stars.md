+++
date = '2025-03-21T23:39:58+01:00'
draft = false
title = 'Annoying Linux: Make Sudo Show You the Stars'
summary = 'My issue with the `sudo` password prompt, and how to fix it.'
tags = ["#annoyingLinux"]
+++

I've recently switched back to Linux as my primary OS at work, after running windows for ~2 years.
While setting up my system, I've noticed quite a few "default settings" that really annoy me.

This will probably be the first blog post in a series of my annoyances. Today's topic:

## The `sudo` password prompt

If you run Linux, you'll have to type in your password quite a lot. That's another thing that annoys
me, but that's for a later day. This post is about how the default settings of the `sudo` password
prompt gives you *zero* feedback.

And straight away I can hear someone shouting - *"It's more secure!"*

If it's that important, then why the hell does all the display managers
show your password as a string of asterisks when you log on to your machine?

I've been using *nix flavoured OSes for almost 20 years now, and I can't believe it took me this
long to google how to fix this. It turns out that it is very easy.

## How do we fix it?

Just add this to `/etc/sudoers`:

```text
Defaults    pwfeedback
```

Next time you enter your password you'll be greeted by lovely stars:

```text
[sudo] password for you: ******
```

See you in the next one!

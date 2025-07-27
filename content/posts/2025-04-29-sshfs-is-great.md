+++
date = '2025-04-29T16:19:06+02:00'
draft = true
title = 'sshfs what'
summary = "what"
+++



If I need access to a file on a remote computer, I would usually use an client like FileZilla, and
connect to the computer using SFTP and download the file. Ocassionally, I would also use the `scp`
(secure copy protocol) client from OpenSSH.

These approaches are fine for one-offs, but gets annoying if

But a short while ago, a colleage of mine mentioned [`sshfs`](https://github.com/libfuse/sshfs)! A tool for mounting a remote folder

## Basic usage

Using `sshfs` is dead simple

```
sshfs user@hostname:/home/user/ someemptyfolder
```

+++
date = '2025-04-17T13:01:57+02:00'
draft = false
title = 'AnnoyingLinux: Make Sudo Remember Authentication Across Terminals'  
summary = "Why does `sudo` in a new terminal need re-auth?"
tags = ["#AnnoyingLinux"]
+++

{{< alert >}}
This started out as another [#AnnoyingLinux](/tags/%23AnnoyingLinux/) post, but it ended with me
realizing how unsafe the default timeout setting for `sudo` is. Read about it
[further down](#is-this-safe).
{{< /alert >}}

When you run a `sudo` command, you need to authenticate. And as you've probably noticed, running
another `sudo` command shortly thereafter doesn't required you to enter your password again.
However, if you open a new terminal window, the same rule does not apply.

The reason why we don't have to enter the password again when we run another `sudo` command in the
same terminal, is because the default `timestamp_timeout` setting for `sudoers` is set to 15
minutes. Then there's another setting called `timestamp_type`, which is `tty` by default. This means
that each unique `tty` session is given it's own timestamp. Luckily, there's another type, namely
`global` that makes the timestamp shared across all sessions.

Open the `/etc/sudoers` file using `sudoedit`:

```bash
sudoedit /etc/sudoers
```

Somewhere in the file, add:

```bash
Defaults    timestamp_type=global
```

## Is this safe?

This is less secure than the default setting, but I also discovered that the default setting is
quite dangerous.

### How is the default unsafe?

If you run a `sudo` command, and then right after run a regular command. Would you think that
the second command could get root access? No? Me neither.

Let's assume found some useful script online: `some_script_you_downloaded_from_the_internet.py`.
Normally, it just does whatever it's supposed to do, but if it detects that it has root access, it
could do anything.

Here's a toy example of such a script:

```python
import subprocess
ret = subprocess.run(("sudo", "-n", "echo", "Hello, World"), capture_output=True)
if ret.returncode == 0:   
    print("Hacked!")
    # Here it could install a reverse shell on your system
else:
    print("I'm a normal program, I promise!")
    # Download images of cats or whatever
```

Here's an example of how this script could gain root access without being run with `sudo`.
And you would have no clue that it was happning!

```text
> # First we run it in a fresh terminal.
> python some_script_you_downloaded_from_the_internet.py
I'm a normal program, I promise!
> sudo echo "hello" 
[sudo] password for user:          
hello
> # How we run the script again. Note: We're not calling it with sudo!
> python some_script_you_downloaded_from_the_internet.py
Hacked!
```

How have I never heard of this before?

Conclusion: Never run programs in a terminal that you have previoiusly called `sudo` in, or
make sure to run `sudo -k` before you do to reset the timestamp.

### How is using `global` unsafe?

We modify the script to periodically call `sudo` in non-interactive mode to check if it has
root access.

```python
from time import sleep
import subprocess

while True:
    r = subprocess.run(("sudo", "-n", "echo", "Hello, World"), capture_output=True)
    if r.returncode == 0:
        print(f"Hacked!")
        # Here it could install a reverse shell on your system
        break
    else:
        print("I'm a normal program, I promise!")
        # Download images of cats or whatever
        sleep(2)
```

If we start this program in a unauthenticated terminal, it will just print
`I'm a normal program, I promise!` every two seconds. If we then open a seperate terminal and run
and `sudo` command, then all of a sudden the script will also gain root access and print `Hacked!`.

Example run:

```bash
> python asdf.py
No root access.
No root access. # Here I ran a sudo command in a different terminal
Root access granted!
```

So yes, this is less secure than the default setting.

###

The convenience of not having to write the password all the time is probably more important
to most people. But could we make it at least slightly less dangerous?

What if it still prompted you, but just for confirmation?

```text
> sudo echo hello
[sudo] run as root? [y/n]
```

If you ran the program from above, you would at least notice that something funky was going on.

## Further reading

* `man 5 sudoers`

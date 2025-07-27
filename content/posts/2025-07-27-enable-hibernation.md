+++
date = '2025-07-27T11:51:54+02:00'
draft = false
title = 'Never return to a drained laptop again!'
summary = 'Enable hibernation in Ubuntu 24.10'
+++

There have been several occasions where I haven't used my laptop in a few days, only to return to a
completely drained battery. This is not only annoying, but also not very good for the battery.

Maybe it's a false memory, but I want to recall that most OS's used to have a "Suspend, and then hibernate" feature. But now hibernation isn't even a default option in Ubuntu?

This is a short guide on how to enable hibernation on Ubuntu 24.10.

{{< alert >}}
**Note:** I'm writing this in retrospect, and the instructions are probably not complete.
This is mostly as a reminder to myself if I ever have to set it up again.
{{</alert >}}

## Setting it up

### Resizing the swap file

My system had a swap file (as opposed to a swap partition), but it was only 4GB in size (iirc). For
hibernation purposes, it's recommended that your swap should be the size of your RAM, although
there are ways to get
[away with less](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#About_swap_partition/file_size).

This command told me that my swap file was located at `/swap.img`.

```bash
swapon --show
```

To resize it, I did the following commands:

```bash
# Disable the swap file
sudo swapoff -a
# Remove the old swap file
sudo rm /swap.img
# Use `dd` to create a new 16GB file with zeroes
sudo dd if=/dev/zero of=/swap.img count=16384 bs=1MiB
# Set the correct permissions
sudo chmod 600 /swap.img
# Enable this file to be used as swap
sudo swapon /swap.img
```

Instead of creating the file manually, it's also possible to use the command `mkswap`, which will
set up the file with the correct permissions.

Since I already had a swap file, it was already listed in `/etc/fstab` as:

```bash
/swap.img       none    swap    sw      0       0
```

### Add kernel parameter to `grub`

We need to pass `resume` and `resume_offset` as kernel parameters at boot.

{{< alert >}}
I'm not certain this is necessary. Newer `systemd` and `initramfs` will
try to figure out where the resume data is. See `man initramfs-tools` and
`man systemd-hibernate-resume`. But maybe when we are using a swap **file** rather than a
partition, we need to where on a specific partition that file is?
{{</alert>}}

In `/etc/default/grub`, we add the `resume` parameter as the `UUID` of partition where the swap file lives. And `resume_offset` is where on that partition the file begins. (I think?)

How do we find these values?

The `UUID` we can find with the `findmnt` command:

```bash
sudo findmnt -no UUID -T /swap.img
```

And it should print the UUID. E.g.:

```plaintext
abcd1234-1234-1234-bdbd-asdfaaaabbbb
```

We find the offset with the `filefrag` command.

```bash
sudo filefrag -v /swap.img
```

```plaintext
File size of /swap.img is 17179869184 (4194304 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..  425983:   [22118400..]  22544383: 425984:            
...
```

I've "highlighted" where the swap file starts, our offset, with square brackets, i.e. `22118400`.

```bash
GRUB_CMDLINE_LINUX_DEFAULT="<...> resume=UUID=<your uuid> resume_offset=<some offset>"
```

Example:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="<...> resume=UUID=abcd1234-1234-1234-bdbd-asdfaaaabbbb resume_offset=1234"
```

And finally, to add this to your `grub` menu, run:

```bash
sudo update-grub
```

Then reboot to make it go into effect. After reboot, you can test it out by running:

```bash
systemctl hibernate
```

## Suspend, then hibernate

I don't want my system to hibernate all the time, I just want it to hibernate if I don't use it for a while. Thankfully, [Chris Arderne](https://rdrn.me/ubuntu-hibernate-luks/#sleep-then-hibernate) has us covered.

We can get this behavior by changing the `HandleLidSwitch` parameter in `/etc/systemd/logind.conf`:

```plaintext
[Login]
HandleLidSwitch=suspend-then-hibernate
```

If the battery goes below 5%, the computer will hibernate. But we can also set it to hibernate based on time!

In `/etc/systemd/sleep.conf` we can set the `HibernateDelaySec` paramter. I've set it to go into hibernation after 30 minutes.

```plaintext
[sleep]
...
HibernateDelaySec=1800
```

And then restart the logind service:

```bash
sudo systemctl restart systemd-logind.service
```

## Sources

* [Arch Linux Wiki](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation)
* [Chris Arderne blog](https://rdrn.me/ubuntu-hibernate-luks/)
* `man systemd-sleep.conf`
* `man logind.conf`

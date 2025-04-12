+++
date = '2025-04-12T17:24:03+02:00'
title = 'Barcode Scanner Password Manager'
summary = 'Typing long passwords by hand? Hell no!'
+++

There's a Windows PC at work that, for some reason, requires its bitlocker recovery key after every
reboot. It doesn't reboot that often, but entering a 48 digit long key by hand gets really old fast.
Especially after you enter in the third time, because of course you mistyped something in the first
two tries.

I work in the Production Test team at Zivid, and we're the team that is responsible for all the
tests and infrastructure that is required for production of our cameras. In the production line,
there are many instances where a barcode scanner is used for tracking of various things.

So natuarlly, someone in my team came up with the brilliant idea of making the bitlocker key into a
QR code, and then simply scan it with a barcode scanner. No typing required and zero errors!

{{< alert >}}
**Note!** If you decide to replicate this, do *not* use a web service to generate your QR code.
They are most likely logging everything! Instead, follow the [guide](#generate-a-qr-password-safely)
below.
{{< /alert >}}

After this, we started joking that we should print all the password that we have on 1password, and
instead walk around with a book of QR codes and a barcode scanner.

I hope it goes without saying that this is *not* recommended.

## Generate a QR password safely

Like I mentioned above, if you decide to replicate this, you should *never* use an online service
to generate your QR code!

Instead, you can generate a QR code locally using the [`qrcode`](https://pypi.org/project/qrcode/)
python library. It will install a script entrypoint allowing you to simply call:

```bash
qr "hello"
```

And it will print out a QR directly in your terminal using unicode
[Block Elements](https://en.wikipedia.org/wiki/Block_Elements).

<pre style="font-family: monospace; line-height: 1.15;">
█████████████████████████████
█████████████████████████████
████ ▄▄▄▄▄ █▀ ▄███ ▄▄▄▄▄ ████
████ █   █ ██▄▀ ▄█ █   █ ████
████ █▄▄▄█ █▄▄██ █ █▄▄▄█ ████
████▄▄▄▄▄▄▄█▄█ ▀ █▄▄▄▄▄▄▄████
████▄█ █ ▀▄██▄▀▄██▀█▄██ ▀████
████▀ █▄▀█▄█▄▄▄█▄█▀█▄▄▄ ▄████
█████▄▄█▄█▄▄▀▀ ▀▄▀▄▀▄█▀▀▀████
████ ▄▄▄▄▄ ███  ▀ ▄ ▀█▄▄▄████
████ █   █ █▄▀▄▄█▄  ██▀ ▄████
████ █▄▄▄█ █▄ ▄█▄█▀█ █▄▀▄████
████▄▄▄▄▄▄▄█▄▄▄█▄█▄▄███▄▄████
█████████████████████████████
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
</pre>

+++
date = '2025-03-09T18:03:56+01:00'
draft = false
title = 'Make Keychron Bluetooth Keyboard Connect Faster On Linux'
+++

Every time I return to my computer after a longer break, it would take forever (read: ~5 seconds)
for my keyboard to connect. On Windows, this was never an issue.

Thankfully, [@andrebrait](https://gist.github.com/andrebrait) had kindly posted a [gist](https://gist.github.com/andrebrait/961cefe730f4a2c41f57911e6195e444#enable-bluetooth-fast-connect-config) covering
this and other Keychron-on-linux related issues.

As his gist instructs, we edit the configuration file for `bluetoothd` - the bluetooth daemon.

```bash
sudoedit /etc/bluetooth/main.conf
```

Uncomment the following settings. (They will be on different lines in the config file.)

```bash
FastConnectable = true
ReconnectAttempts = 7
ReconnectIntervals = 1,2,3,4,8,16,32,64
```

Then restart the bluetooth service

```bash
systemctl daemon-reload && systemctl restart bluetooth
```

And now I don't have to wait for my keyboard to connect anymore!

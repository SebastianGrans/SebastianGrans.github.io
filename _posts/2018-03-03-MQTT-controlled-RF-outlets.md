---
layout: post
title:  MQTT Controlled RF Outlets
---

<style>
.mono {
  font-family: monospace;
}

</style>

In the last [post](http://sebastiangrans.github.io/Sniff-and-control-433MHz-remote-controlled-outlets/), I briefly went through how I sniffed and was able to control RF controlled outlets. Now I have ported the code to the Raspberry Pi and also written a short BASH MQTT daemon that enables me to controll these switches using MQTT. 

All the code is available here on my Github, [here](https://github.com/SebastianGrans/RF-MQTT). 

## Let's begin! 

In the last post, I used the [RCSwitch](https://github.com/sui77/rc-switch) library, which is already compatible with the Raspberry Pi! However, it relies on [wiringPi](http://wiringpi.com), which is a library to utilize the GPIO pins of the RPi. The pin numbering varies a lot between libraries and the actual physical layout on the Pi, but luckily someone made this nifty website: [pinout.xyz](https://pinout.xyz). In my project, I used the physical pin 13, which maps to pin 2 in wiringPi or pin 27 on the Broadcom chip itself. 

Below is a short program for transmitting data. E.g. 
`./send 1234`

<details markdown="1">
<summary><h3 style="display: inline">Program for transmitting</h3></summary>

```c++
/* 
 * Send a RF data through a transmitter module connected to 
 * TRANS_PIN-pin on the Raspberry Pi 
 *  
 *	Example: 
 * 		./send 1234
 */
#include "rc-switch-master/RCSwitch.h"
#include <stdio.h> // printf, fgets
#include <stdlib.h> // exit

RCSwitch trans;
        
int main(int argc, char *argv[]) {
  if(argc < 2) {
    printf("No command specified.\n");
    exit(EXIT_FAILURE);
  }
  
  /* Change these to fit your RF outlets */ 
  int TRANS_PIN = 2;
  int PROTOCOL = 4;
  int PULSE_LENGTH = 380;
  int DATA_LENGTH = 24

  /* Setup wiringPi */
  if(wiringPiSetup() == -1) {
    printf("wiringPi failed. Exiting...\n");
    exit(EXIT_FAILURE);
  }
 
  trans = RCSwitch();
  trans.enableTransmit(TRANS_PIN);
  trans.setProtocol(PROTOCOL);
  trans.setPulseLength(PULSE_LENGTH);
 
  trans.send(atoi(argv[1]), DATA_LENGTH);
  
  exit(EXIT_SUCCESS);
}
```

</details>

But that only takes us half way! I wanted to control the outlets using the MQTT protocol. 

I'm running the [Mosquitto](https://mosquitto.org) broker, which also comes with a publish and subscribe client. So it was relatively easy to construct a small BASH MQTT daemon. 

<details markdown="1">
<summary><h3 style="display: inline">BASH MQTT Daemon</h3></summary>

```bash
#!/bin/bash

# Set the topic to subscribe to. 
SUB_TOPIC="cmnd/switches/+"

# MQTT credentials 
USER="<replace me>"
PW="<replace me>"

# Declare the commands here. 
# If we recieve 'cmnd/switches/A ON' the script executes './send 11840172' 
# The code was sniffed from the remote using 'ProtocolAnalyzeDemo' from this
# fork of rc-switch: https://github.com/Martin-Laclaustra/rc-switch/tree/protocollessreceiver

# This is an associative array. 
declare -A COMMANDS
COMMANDS["A ON"]=11840172
COMMANDS["A OFF"]=12552332
COMMANDS["B ON"]=11949797
COMMANDS["B OFF"]=11639189
COMMANDS["C ON"]=12284094
COMMANDS["C OFF"]=12502382
COMMANDS["D ON"]=11639191
COMMANDS["D OFF"]=12379911

# Just some pretty colorized output. 
RED='\e[31m'
GREEN='\e[32m'
YELLOW='\e[93m'
DEFAULT='\e[39m'

while read MSG;
do
    # SWITCH=$(sed -e 's/.*\/\([A-B]\).*/\1/' <<< $MSG)
    # COMMAND=$(sed -e 's/.* \(\w*$\).*/\1/' <<< $MSG)
    
    # Extract everything after the last slash in the MQTT command. 
    COMMAND=$(sed -e 's/.*\/\(.*\)/\1/' <<< $MSG) 
    echo -en $YELLOW$(date)':' $DEFAULT$MSG;
    
    # If the received command was valid, execute ./send with the approriate 
    # decimal command. 
    if [[ ${COMMANDS[$COMMAND]} ]]; then     
        echo -e $GREEN' ✔︎' $DEFAULT
        ./send ${COMMANDS[$COMMAND]}
    else 
        echo -e $RED' ✘ No such command' $DEFAULT
    fi

# Setup the subscription using mosquitto_sub
done < <(mosquitto_sub -u ${USER} -P ${PW} -t ${SUB_TOPIC} -q 1 -v)
```

</details>

Then, in order to make it start automatically on boot I had to make a service file for `systemd` to read. It should be placed in `/etc/systemd/system`. 
In order for `systemd` to see the new service, we need to reload it. <br>
`sudo systemctl daemon-reload`<br>
and then enable <br>
`sudo systemctl enable mqttexec.service` <br>
and start <br>
`sudo systemctl start mqttexec.service`. <br>

<details markdown="1">
<summary><h3 style="display: inline">MQTTExec.service</h3></summary>

```
#!/bin/bash

# Set the topic to subscribe to. 
SUB_TOPIC="cmnd/switches/+"

# MQTT credentials 
USER="<replace me>"
PW="<replace me>"

# Declare the commands here. 
# If we recieve 'cmnd/switches/A ON' the script executes './send 11840172' 
# The code was sniffed from the remote using 'ProtocolAnalyzeDemo' from this
# fork of rc-switch: https://github.com/Martin-Laclaustra/rc-switch/tree/protocollessreceiver

# This is an associative array. 
declare -A COMMANDS
COMMANDS["A ON"]=11840172
COMMANDS["A OFF"]=12552332
COMMANDS["B ON"]=11949797
COMMANDS["B OFF"]=11639189
COMMANDS["C ON"]=12284094
COMMANDS["C OFF"]=12502382
COMMANDS["D ON"]=11639191
COMMANDS["D OFF"]=12379911

# Just some pretty colorized output. 
RED='\e[31m'
GREEN='\e[32m'
YELLOW='\e[93m'
DEFAULT='\e[39m'

while read MSG;
do
    # SWITCH=$(sed -e 's/.*\/\([A-B]\).*/\1/' <<< $MSG)
    # COMMAND=$(sed -e 's/.* \(\w*$\).*/\1/' <<< $MSG)
    
    # Extract everything after the last slash in the MQTT command. 
    COMMAND=$(sed -e 's/.*\/\(.*\)/\1/' <<< $MSG) 
    echo -en $YELLOW$(date)':' $DEFAULT$MSG;
    
    # If the received command was valid, execute ./send with the approriate 
    # decimal command. 
    if [[ ${COMMANDS[$COMMAND]} ]]; then     
        echo -e $GREEN' ✔︎' $DEFAULT
        ./send ${COMMANDS[$COMMAND]}
    else 
        echo -e $RED' ✘ No such command' $DEFAULT
    fi

# Setup the subscription using mosquitto_sub
done < <(mosquitto_sub -u ${USER} -P ${PW} -t ${SUB_TOPIC} -q 1 -v)
```

</details>


I hope this helps someone 🙂

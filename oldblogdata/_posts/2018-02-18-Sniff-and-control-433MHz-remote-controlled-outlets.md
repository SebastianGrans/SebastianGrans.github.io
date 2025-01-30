---
layout: post
title: Sniffing and controlling 433MHz remote controlled outlets
---

<style>
.mono {
  font-family: monospace;
}

</style>
I'm interested in home automation and previously I've fiddled with a Sonoff Basic switch. I flashed it with [Tasmota](https://github.com/arendst/Sonoff-Tasmota) and controlled it using the MQTT protocol and Mosquitto running on a Raspberry Pi. It works great, but the fact that the Sonoff switch is not CE rated makes me nervous (eventhough the build quality of the PCB looks fine in my layman eyes).

Then I spotted some remote controlled outlets (about the same price as a Sonoff) and though that I could probably control them using a microcontroller and an RF transmitter. After searching the web I found tonnes of people that had done the same thing. 

I bought a set of Luxorparts Mini remote sockets kit (Item: 50987) from the Swedish electronics store Kjell & Co. I also bought a set of 433MHz receiver and transmitter modules from the same store. 

<span style="margin-bottom: 0;"><b>**Note!**</b>
<span style="color: darkred; margin-top: 0;">
I struggled like hell to get this to work, but I finally got it working thanks to [this](https://github.com/sui77/rc-switch/issues/194) issue on Github. So if it doesn't work the first time, just keep on going and try different approaches. 
</span>
</span>

## Materials
* Arduino 
* 433MHz receiver and transmitter modules
* RF Controlled outlets 

## Steps
I assume that you have some experience with the Arduino IDE. 

1. Download [this](https://github.com/Martin-Laclaustra/rc-switch/tree/protocollessreceiver) repository and install it. <br>(Sketch ➔ Include Library ➔ Add .ZIP Library...)
2. Connect the transmitter and receiver module.
3. Open the `ProtocolAnalyzeDemo.ino` sketch from the examples menu. <br>(File ➔ Examples ➔ rc-switch ➔ ProtocolAnalyzeDemo)
4. Make sure that the receiver module is connected to the correct pin. (The pin number on the board is not the same as the number of the "interrupt pin". It depends on the Arduino model.)
5. Compile and upload it to the Arduino. 
6. Open the Serial monitor and press a button on the remote. You should then see something like this:

	~~~
	Received 12414590 / 24bit Protocol: 4
	data bits of pulse train duration: 37116
	proposed protocol: { 387, { 1, 6 }, { 1, 3 }, { 3, 1 }, false }
	====
	first level down
	2408,
	1052,492,280,1252,1068,484,1060,484,1064,480,1068,500,276,1252,1064,492,
	284,1260,1056,488,1056,488,288,1260,1056,484,1064,496,1052,496,280,1260,
	284,1296,1020,484,1064,492,1056,508,1040,492,1056,504,1044,496,280,1264,
	284
	====
	~~~

	My remote had a cycle of 4 different signals it sent out for each button. The outlet seems to react to any of the signals. 

7. Now you just have to make a table for each button. Below is a table of what I collected. The raw data is shown in the spoiler below the table. 

	<style>
	table {
	margin-right: auto;
	margin-left: auto;
	}
	th {
	border-bottom: 2px solid #000000;
	font-weight: bold;
	}
	td {
	padding-right: 1em;
	padding-left: 1em;
	}
	</style> 


	| Button |                        On                     |                   Off                                        |
	|--------|:---------------------------------------------:|:------------------------------------------------------------:|
	| A      | <span class="mono">101101001010101010101100</span>  | <span class="mono">101111111000100010001100</span>     |
	|        | <span class="mono">101111101100010101101100</span>  | <span class="mono">101110110111000010111100</span>     | 
	|        | <span class="mono">101110010010010000011100</span>  | <span class="mono">101111010110111001111100</span>     | 
	|        | <span class="mono">101101111011110001011100</span>  | <span class="mono">101100110100000100101100</span>     | 
	|		 |													   |                                                        |
	| B      | <span class="mono">101101100101011011100101</span>  | <span class="mono">101100011001100110010101</span>     | 
	|        | <span class="mono">101111001110011100000101</span>  | <span class="mono">101101011101110111010101</span>     | 
	|        | <span class="mono">101110000011111100110101</span>  | <span class="mono">101110100001001001000101</span>     | 
	|        | <span class="mono">101100101111001111110101</span>  | <span class="mono">101100000000101111000101</span>     |
	|		 |													   |                                                        |  
	| C      | <span class="mono">101110110111000010111110</span>  | <span class="mono">101111101100010101101110</span>     | 
	|        | <span class="mono">101111010110111001111110</span>  | <span class="mono">101110010010010000011110</span>     | 
	|        | <span class="mono">101100110100000100101110</span>  | <span class="mono">101101111011110001011110</span>     | 
	|        | <span class="mono">101111111000100010001110</span>  | <span class="mono">101101001010101010101110</span>     |
	|		 |													   |                                                        | 
	| D      | <span class="mono">101100011001100110010111</span>  | <span class="mono">101111001110011100000111</span>     | 
	|        | <span class="mono">101101011101110111010111</span>  | <span class="mono">101110000011111100110111</span>     | 
	|        | <span class="mono">101110100001001001000111</span>  | <span class="mono">101100101111001111110111</span>     | 
	|        | <span class="mono">101100000000101111000111</span>  | <span class="mono">101101100101011011100111</span>     | 


8. Now you can start sending your own signals, without using the remote! Place the two files below (send.ino and output.ino) in a folder. Compile and upload! 

9. Now you can try to send commands (the binary strings!) from the serial monitor and your RF controlled device should respond as if it was the original remote! 😀


<details markdown="1">
<summary><h3 style="display: inline">Intercepted remote signals</h3></summary>

### A On
```
Decimal: 11840172 (24Bit) Binary: 101101001010101010101100 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2440,1048,508,264,1268,1048,504,1040,500,272,1276,1028,496,280,1272,272,1268,1044,512,264,1264,1032,500,276,1276,1036,504,272,1276,1036,496,268,1272,1040,496,280,1264,1048,504,272,1268,1044,500,1044,504,268,1272,272,1304,

Decimal: 12502380 (24Bit) Binary: 101111101100010101101100 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2420,1040,500,272,1280,1036,508,1036,500,1044,516,1048,508,1040,512,264,1276,1040,512,1036,508,268,1272,276,1272,272,1272,1044,504,276,1276,1032,504,268,1272,1044,508,1036,520,256,1272,1044,500,1044,500,276,1268,276,1292,

Decimal: 12133404 (24Bit) Binary: 101110010010010000011100 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1048,496,280,1272,1040,500,1044,524,1020,508,276,1272,272,1276,1040,508,264,1268,276,1272,1044,496,276,1268,276,1268,1048,512,264,1396,172,396,160,380,172,1400,172,1404,368,184,1384,188,1372,192,372,184,380,172,

Decimal: 12041308 (24Bit) Binary: 101101111011110001011100 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2424,1044,504,272,1272,1044,504,1044,500,276,1272,1052,500,1048,500,1052,508,1040,492,284,1280,1036,496,1052,496,1048,504,1040,500,276,1276,268,1272,272,1272,1040,508,268,1276,1040,504,1040,508,1036,516,256,1292,252,1296,
```

### A Off

```
Decimal: 12552332 (24Bit) Binary: 101111111000100010001100 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2404,1056,492,284,1264,1060,488,1060,504,1052,488,1060,512,1036,500,1052,496,1056,492,284,1272,284,1268,280,1264,1056,528,252,1260,284,1268,276,1272,1048,512,264,1268,276,1284,264,396,160,380,172,1396,172,384,168,1400,

Decimal: 12284092 (24Bit) Binary: 101110110111000010111100 Tri-State: not applicable PulseLength: 388 microseconds Protocol: 4
Raw data: 2420,1040,508,272,1268,1056,500,1052,500,1052,500,280,1272,1048,496,1056,524,256,1272,1052,496,1056,504,1048,496,284,1268,280,1260,288,1264,284,1292,1032,500,280,1272,1048,508,1048,496,1056,516,1036,496,280,1276,272,1296,

Decimal: 12414588 (24Bit) Binary: 101111010110111001111100 Tri-State: not applicable PulseLength: 388 microseconds Protocol: 4
Raw data: 2412,1052,504,276,1268,1056,500,1052,504,1048,508,1044,512,264,1276,1048,508,268,1280,1044,504,1052,504,272,1272,1052,500,1052,508,1044,1404,164,388,168,388,164,1400,176,1396,396,156,1388,184,376,176,1400,176,372,180,

Decimal: 11747628 (24Bit) Binary: 101100110100000100101100 Tri-State: not applicable PulseLength: 388 microseconds Protocol: 4
Raw data: 2400,1064,492,288,1260,1064,500,1052,496,284,1272,288,1264,1056,488,1064,500,276,1264,1056,496,288,1268,280,1260,288,1260,288,1256,292,1380,184,376,184,376,176,1392,180,1388,368,184,1380,192,368,184,1396,176,360,192,
```

### B On
```
Decimal: 11949797 (24Bit) Binary: 101101100101011011100101 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2420,1052,492,280,1268,1048,492,1052,492,284,1260,1044,492,1052,496,280,1264,280,1264,1048,492,272,1272,1044,492,280,1260,1052,496,1052,496,280,1264,1052,500,1044,488,1056,504,184,376,184,388,160,1384,188,376,172,1388,

Decimal: 12379909 (24Bit) Binary: 101111001110011100000101 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2412,1056,496,280,1260,1052,484,1064,488,1056,488,1060,488,284,1264,280,1256,1056,492,1056,488,1056,492,284,1264,280,1260,1056,492,1052,492,1052,488,284,1260,284,1256,288,1264,280,1264,280,1260,1052,500,276,1260,1052,508,

Decimal: 12074805 (24Bit) Binary: 101110000011111100110101 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2420,1048,492,284,1264,1052,484,1060,488,1056,496,288,1260,284,1268,276,1264,280,1256,288,1260,1060,484,1060,488,1056,508,1040,520,1024,504,1040,500,272,1260,284,1256,1064,488,1052,492,284,1260,1056,488,288,1256,1056,516,

Decimal: 11727861 (24Bit) Binary: 101100101111001111110101 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2412,1052,500,276,1256,1060,488,1056,492,284,1260,280,1260,1056,488,284,1276,1040,492,1052,496,1056,492,1052,492,284,1260,284,1260,1052,500,1048,504,1044,492,1052,492,1052,500,1048,500,272,1264,1052,492,284,1264,1052,540,

```


### B off
```
Decimal: 11639189 (24Bit) Binary: 101100011001100110010101 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2412,1056,496,280,1264,1056,492,1060,492,284,1264,280,1260,284,1276,1044,504,1048,496,280,1268,284,1268,1052,488,1060,496,280,1260,288,1264,1052,492,1060,488,288,1276,268,1264,1396,168,384,176,380,172,1392,180,380,360,

Decimal: 11918805 (24Bit) Binary: 101101011101110111010101 Tri-State: not applicable PulseLength: 388 microseconds Protocol: 4
Raw data: 2412,1036,504,272,1272,1052,504,1052,504,272,1276,1048,492,288,1268,1056,496,1056,500,1052,504,284,1276,1048,512,1040,496,1064,496,276,1272,1048,496,1056,516,1040,496,280,1292,1032,496,284,1268,1056,496,284,1272,1052,520,

Decimal: 12194373 (24Bit) Binary: 101110100001001001000101 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2404,1056,488,288,1264,1056,488,1060,488,1060,504,280,1268,1052,488,288,1260,284,1272,276,1260,292,1264,1056,500,276,1256,292,1260,1056,496,280,1272,276,1260,1060,492,284,1252,292,1252,296,1272,1048,488,288,1260,1060,504,

Decimal: 11537349 (24Bit) Binary: 101100000000101111000101 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2440,1060,492,284,1260,1060,492,1056,500,276,1260,284,1260,288,1256,288,1264,284,1256,288,1260,288,1268,280,1256,1064,484,292,1260,1060,496,1052,492,1060,492,1056,496,280,1260,288,1260,288,1260,1064,500,276,1256,1068,512,

```

### C On
```
Decimal: 12284094 (24Bit) Binary: 101110110111000010111110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2404,1048,488,284,1260,1052,488,1056,492,1052,496,280,1260,1056,496,1048,488,284,1260,1052,504,1044,496,1044,500,276,1264,280,1264,280,1256,284,1252,1060,484,288,1264,1048,488,1056,496,1048,512,1028,488,1056,496,280,1288,

Decimal: 12414590 (24Bit) Binary: 101111010110111001111110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1052,492,284,1256,1056,496,1048,496,1048,512,1052,496,280,1264,1048,508,268,1272,1040,488,1048,500,272,1264,1052,492,1052,500,1044,492,280,1268,276,1268,1048,488,1056,492,1052,496,1044,496,1048,496,1048,496,280,1300,

Decimal: 11747630 (24Bit) Binary: 101100110100000100101110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2424,1052,484,288,1252,1064,484,1056,488,288,1252,296,1244,1068,496,1048,480,292,1260,1056,488,288,1256,292,1260,276,1256,292,1264,284,1248,1060,480,296,1264,280,1248,1064,480,292,1260,1052,496,1048,484,1060,492,280,1280,

Decimal: 12552334 (24Bit) Binary: 101111111000100010001110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1060,492,280,1256,1060,484,1056,496,1048,512,1052,484,1056,492,1052,488,1056,488,288,1268,288,1256,284,1268,1048,488,284,1268,280,1256,276,1256,1060,492,280,1252,292,1260,284,1260,1056,508,1036,500,1044,492,284,1284,


```

### C Off
```
Decimal: 12502382 (24Bit) Binary: 101111101100010101101110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2408,1048,496,280,1264,1048,496,1048,496,1048,500,1052,512,1032,496,276,1268,1048,492,1052,508,280,1264,280,1272,272,1268,1044,496,280,1268,1044,516,260,1264,1048,508,1040,492,168,376,184,376,172,1392,180,376,176,1396,

Decimal: 12133406 (24Bit) Binary: 101110010010010000011110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1052,496,280,1264,1052,492,1056,484,1060,496,268,1268,280,1264,1052,488,284,1268,280,1256,1064,492,284,1256,288,1256,1060,484,292,1260,276,1264,284,1260,284,1260,284,1264,1052,496,1048,492,1056,496,1048,488,288,1280,

Decimal: 12041310 (24Bit) Binary: 101101111011110001011110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2412,1044,496,280,1264,1052,492,1060,500,268,1264,1052,496,1052,504,1040,504,1044,500,272,1280,1052,492,1052,508,1040,496,1048,496,280,1284,284,1264,280,1272,1040,508,268,1268,1048,508,1036,504,1040,500,1048,500,272,1284,

Decimal: 11840174 (24Bit) Binary: 101101001010101010101110 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2428,1048,492,284,1264,1052,488,1056,492,280,1264,1048,492,284,1268,276,1260,1052,496,280,1264,1056,488,284,1260,1056,492,284,1264,1048,496,280,1264,1052,508,264,1268,1048,492,284,1268,1044,500,1044,508,1036,500,276,1292,

```

### D On
```
Decimal: 11639191 (24Bit) Binary: 101100011001100110010111 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2424,1048,500,276,1268,1048,500,1044,500,276,1272,272,1268,276,1268,1048,504,1040,496,280,1312,276,1268,1048,512,1036,496,280,1284,260,1264,1052,500,1048,496,280,1264,280,1272,164,380,180,380,168,1400,172,392,164,1396,

Decimal: 11918807 (24Bit) Binary: 101101011101110111010111 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2424,1048,504,272,1296,1020,512,1036,496,280,1280,1040,504,268,1276,1044,504,1040,500,1048,504,268,1272,1044,500,1044,500,1048,500,276,1272,1044,516,1032,508,1040,508,264,1280,1040,508,264,1276,1040,504,1044,508,1040,524,

Decimal: 12194375 (24Bit) Binary: 101110100001001001000111 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1056,496,276,1272,1044,500,1044,500,1048,508,272,1268,1048,492,284,1264,280,1260,284,1276,280,1260,1056,496,276,1264,284,1260,1052,500,284,1276,268,1260,1056,496,276,1280,268,1260,284,1256,1060,500,1044,500,1048,512,

Decimal: 11537351 (24Bit) Binary: 101100000000101111000111 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2412,1052,492,284,1268,1044,500,1048,496,280,1268,280,1264,280,1260,284,1268,276,1260,284,1272,284,1264,280,1276,1040,496,280,1268,1048,488,1056,500,1048,496,1048,496,280,1268,276,1268,276,1268,1048,496,1048,496,1048,516,

```

### D Off
```
Decimal: 12379911 (24Bit) Binary: 101111001110011100000111 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2424,1036,504,268,1268,1048,504,1040,516,1028,500,1024,516,260,1300,244,1276,1036,512,1032,512,1032,508,264,1284,260,1276,1040,524,1020,508,1036,500,276,1272,272,1268,276,1268,176,372,184,380,172,1392,180,364,184,1400,

Decimal: 12074807 (24Bit) Binary: 101110000011111100110111 Tri-State: not applicable PulseLength: 386 microseconds Protocol: 4
Raw data: 2416,1052,496,276,1268,1048,500,1044,496,1048,504,280,1284,260,1268,276,1264,280,1272,272,1264,1048,492,1056,500,1044,500,1044,508,1036,496,1052,496,276,1268,276,1264,1052,496,1048,492,284,1264,1052,492,1052,500,1048,520,

Decimal: 11727863 (24Bit) Binary: 101100101111001111110111 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2416,1052,492,280,1280,1036,504,1044,496,280,1268,260,1276,1040,500,276,1272,1044,500,1048,516,1052,520,1024,500,276,1268,276,1264,1052,500,1048,500,1048,500,1048,496,1048,504,1044,504,272,1268,1048,500,1048,512,1036,520,

Decimal: 11949799 (24Bit) Binary: 101101100101011011100111 Tri-State: not applicable PulseLength: 387 microseconds Protocol: 4
Raw data: 2416,1048,500,276,1276,1040,508,1040,488,284,1280,1044,508,1040,492,280,1268,280,1264,1052,500,284,1268,1048,496,280,1268,1048,492,1056,496,284,1272,1044,504,1044,496,1052,492,284,1264,280,1276,1040,496,1052,496,1052,512,

```

</details>


<details markdown="1">
<summary><h3 style="display: inline">send.ino</h3></summary>
	
```c

	#include <RCSwitch.h>
	
	// Setup the RCSwitch instances for the receiever and transmitter
	RCSwitch trans = RCSwitch();
	RCSwitch recv = RCSwitch();
	
	// Input buffer
	char data[25];
	void setup() {
	  Serial.begin(9600);
	  
	  // Transmitter is connected to pin 10 
	  trans.enableTransmit(10);
	  // Receiver is connected to pin 2 (Interrupt pin 0 for Arduino Uno)
	  recv.enableReceive(digitalPinToInterrupt(2));
	  
	  // When sniffing, the protocol was found to be '4'. 
	  // I'll have to dig into what that means. 
	  trans.setProtocol(4);
	  
	  // This must also be set to the value you got.  
	  trans.setPulseLength(380);
	  
	  // I arbitrarily set it to repeat a transmission 4 times. 
	  trans.setRepeatTransmit(4);
	
	}
	
	void loop() {
	  // I listen for signals at the same time as I can send them.
	  if (recv.available()) {
	    output(recv.getReceivedValue(), recv.getReceivedBitlength(), recv.getReceivedDelay(), recv.getReceivedRawdata(),recv.getReceivedProtocol());
	    recv.resetAvailable();    
	  }
	  // Type in the bit-string into the serial monitor to send it.
	  if (Serial.available() > 0) {
	    Serial.readString().toCharArray(data, 25);
	    Serial.print("I received: ");
	    Serial.println(data);
	    trans.send(data);
	    }
	}
```

</details>

<details markdown="1"><summary><h3 style="display: inline">output.ino</h3></summary>

```c

/* Code from the rc-switch repository. 
	Check it out here:
	https://github.com/sui77/rc-switch/blob/master/examples/ReceiveDemo_Advanced/output.ino
	*/
	static const char* bin2tristate(const char* bin);
static char * dec2binWzerofill(unsigned long Dec, unsigned int bitLength);

void output(unsigned long decimal, unsigned int length, unsigned int delay, unsigned int* raw, unsigned int protocol) {

  const char* b = dec2binWzerofill(decimal, length);
  Serial.print("Decimal: ");
  Serial.print(decimal);
  Serial.print(" (");
  Serial.print( length );
  Serial.print("Bit) Binary: ");
  Serial.print( b );
  Serial.print(" Tri-State: ");
  Serial.print( bin2tristate( b) );
  Serial.print(" PulseLength: ");
  Serial.print(delay);
  Serial.print(" microseconds");
  Serial.print(" Protocol: ");
  Serial.println(protocol);
  
  Serial.print("Raw data: ");
  for (unsigned int i=0; i<= length*2; i++) {
    Serial.print(raw[i]);
    Serial.print(",");
  }
  Serial.println();
  Serial.println();
}

static const char* bin2tristate(const char* bin) {
  static char returnValue[50];
  int pos = 0;
  int pos2 = 0;
  while (bin[pos]!='\0' && bin[pos+1]!='\0') {
    if (bin[pos]=='0' && bin[pos+1]=='0') {
      returnValue[pos2] = '0';
    } else if (bin[pos]=='1' && bin[pos+1]=='1') {
      returnValue[pos2] = '1';
    } else if (bin[pos]=='0' && bin[pos+1]=='1') {
      returnValue[pos2] = 'F';
    } else {
      return "not applicable";
    }
    pos = pos+2;
    pos2++;
  }
  returnValue[pos2] = '\0';
  return returnValue;
}

static char * dec2binWzerofill(unsigned long Dec, unsigned int bitLength) {
  static char bin[64]; 
  unsigned int i=0;

  while (Dec > 0) {
    bin[32+i++] = ((Dec & 1) > 0) ? '1' : '0';
    Dec = Dec >> 1;
  }

  for (unsigned int j = 0; j< bitLength; j++) {
    if (j >= bitLength - i) {
      bin[j] = bin[ 31 + i - (j - (bitLength - i)) ];
    } else {
      bin[j] = '0';
    }
  }
  bin[bitLength] = '\0';
  
  return bin;
}
```

</details>




## Todo
* Port the code so that I can run it directly on the Raspberry Pi
* Create an interface for it so that I can control it with Mosquitto (MQTT Broker)


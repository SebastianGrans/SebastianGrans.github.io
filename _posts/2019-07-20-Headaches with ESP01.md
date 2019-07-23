---
layout: post
title: Headaches with the ESP01
---
<center>
<img src="{{ site.baseurl }}/images/ESP01/ESP-01.png" alt="ESP01 and a USB-to-Serial Adapter" style="display: block;"/>
</center>

In a recent splurge on an unnamed chinese cheap-things online store, I bought an ESP01 and a ESP01 UART adapter. I have played around with ESP-12 development boards before which has a built in USB-to-Serial interface, so I thought this too would be as easy as plug-and-play. Boy, was I wrong!

I began by trying to upload the 'Hello World' of hardware, the Blink sketch, but instantly hit a wall. 
	
<details markdown="1"><summary>ESPTool Output</summary>
~~~
esptool.py v2.6
2.6
esptool.py v2.6
Serial port /dev/cu.wchusbserial1420
Connecting........_____....._
Chip is ESP8266EX
Features: WiFi
MAC: 2c:f4:32:53:8d:c7
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Auto-detected Flash size: 1MB
Flash params set to 0x0320
Compressed 289728 bytes to 204022...

Writing at 0x00000000... (7 %)
Writing at 0x00004000... (15 %)
Writing at 0x00008000... (23 %)
Writing at 0x0000c000... (30 %)
Writing at 0x00010000... (38 %)
Writing at 0x00014000... (46 %)
Writing at 0x00018000... (53 %)
Writing at 0x0001c000... (61 %)
Writing at 0x00020000... (69 %)
Writing at 0x00024000... (76 %)
Writing at 0x00028000... (84 %)
Writing at 0x0002c000... (92 %)
Writing at 0x00030000... (100 %)
Wrote 289728 bytes (204022 compressed) at 0x00000000 in 18.3 seconds (effective 126.7 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
~~~

</details> 

Some quick [Googling](https://www.youtube.com/watch?v=6uaIWZCRSz8) told me that, in order to program the ESP01, it must be put into programming mode. Which is achieved by pulling pin GPIO0 low when it is powered on. Fine... I cut Male-Female jumper wire in half and soldered one to `GND` and the other to `GPIO0` on the UART Adapter. 
    
I then tried programming the ESP by first connect the jumper cable, and then plugging it in the adapter. Sure enough, the upload works! To my dismay, the success was all but long lived...

After disconnecting the jumper wires and power cycling the ESP nothing happened. No blinking. My initial thought was that maybe the predefined pin `LED_BUILTIN` was wrong and, to no avail, tried different pin numbers. After lots of Googling and trying various settings, such as DOUT flash mode, changing upload speeds, etc, I accepted defeat and called it quits for the night.
    
The day after I continued were I left of and that's when I struck gold! It was all in the details, as a _comment_ on an _answer_ to a _question_ on [electronics.stackexchange](https://electronics.stackexchange.com/questions/413918/esp8266-not-running-uploaded-arduino-sketch/414069#414069?newreg=53fc4d5ff0fb485d8e6c6dd1d3f709df)[^2] gave me the solution:

>   No! It's the 2.4 that fixed your problem. I had exactly the same issue now :( DOUT and 2.4 
>   It's not working with 2.5 
>   – Adrian Andrei Apr 23 at 19:01

What Adrian is talking about, is the version of ["ESP8266 Core for Arduino"](https://github.com/esp8266/Arduino). When I went into Tools▸Board▸Board Manager... I saw that, indeed, I had been using version 2.5.2. After downgrading to 2.4.0 (arbitrarily. I haven't tried the other 2.4.x versions), it worked! IT BLINKS! 

<center>
<video autoplay loop>
  <source src="https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.mp4" type="video/mp4">
</video>
</center>

So after spending a couple of hours on this, I can finally do what I actually wanted to do... 
I hope this helps someone.


[^1]: This is with "ESP8266 core for Arduino 2.5.2". It will be important later.
[^2]: The way the original poster added jumper wires is similar to how I solved it.
    
<details markdown="1"><summary>Some tags for the Google spider</summary>
    ESP01 upload works but doesn't run  
    ESP01 doesn't blink  
    Sketch doesn't run  
    ESP8266  
    ESP01 programming  
</details> 



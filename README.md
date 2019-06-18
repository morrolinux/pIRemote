# pIRemote
IR universal remote using a Raspberry Pi - connect that to a Google Home or Amazon Alexa and you're good to go!

It's more of a gist as of now, but I'll automate the process where possible.

# Hardware Requirements 
IR emitter and receiver(s): https://amzn.to/2InADWe

# Hardware Setup
Just plug the IR transmitter and receiver diods properly, here's my configuration:
![img](https://lh3.googleusercontent.com/69TPVSIhKsSwTYMDX7Y6-yJQP7zPPv2NPYlQk8bxoMbfoXmK4wn77LYohyIwlwWnkDssQjpLOmROd85sI_t1fWr3YUFhh76AqjnEpBJzLb2333wOFQKdjCXUVv9yjN7X1xbF0PoMNUj8rGhAQkcazLW8fD7AUwP6dusyUkGklpJjfyxSYoNyZYKYCK62912s2pMzJc_xI6j6E_trN-m-tR-ExT4cSQdbkVluQFBOmco_Tdrwl3yQ0v1UohmndnErVFZCiugo-X_PRzOBfbFsCAnCyIryks5XaCCW54YImmUDcJCMp8flyCCK9uLuyB8FlkPvBz7bfbmquOqUpqmpL9QcFv1Cx6mOZVsx-WE1qN3xJ9cYNpgs0ylFqsduhjiCWeodkTTkHM2i8cJwhBH2mtrlIgeXWXS7jN7mjmAaZ98UT1KdRideUoSVdLllrVcZHA3_jza6SQhcO6DChcgs9FeaS_JaXIYh0iftvqlGn2HjQ6SGTmz-Y6XWfQoRb4Lp6PYjFa_Dh_RgbD4VQmidVxboci0NWNUT80rdPxY5uSTYe0soOemruPmURJ9NZzDLIRtalbUEzLnHb2mWLH_7oylWwu0RXKHsGxGm3gEq3gRwTsYc5kmw9aG1ie3H1LN1_FmlVFC5YdOYXifFjtOC7kGWu3H03Czm=w1056-h385-no)
GPIO Pinout reference: https://pinout.xyz/

# Software Setup
Install lirc: `sudo apt-get install lirc`
Add the following to your `/etc/modules`:
```
lirc_dev
lirc_rpi gpio_in_pin=27 gpio_out_pin=14
```
Remember to change `gpio_in_pin` and `gpio_out_pin` according to your hardware setup

Put the following in `/etc/lirc/hardware.conf`:
```
LIRCD_ARGS="--uinput"
# Try to load appropriate kernel modules
LOAD_MODULES=true
# Run "lircd --driver=help" for a list of supported drivers.
DRIVER="default"
# usually /dev/lirc0 is the correct setting for systems using udev
DEVICE="/dev/lirc0"
MODULES="lirc_rpi"
# Default configuration files for your hardware if any
LIRCD_CONF=""
LIRCMD_CONF=""
```
Add the following to your `/boot/config.txt`:
```
dtoverlay=gpio-ir,gpio_pin=27
dtoverlay=gpio-ir-tx,gpio_pin=14
```

create your `/etc/systemd/system/piremote.service` like so:
```
[Unit]
Description=Pi remote service
After=network.target 

[Service]
Type=idle
User=pi
Group=pi
EnvironmentFile=/home/pi/.profile
ExecStart=/bin/bash -l -c '/home/pi/piremote.py'
ExecReload=/bin/kill -HUP $MAINPID
KillMode=control-group

[Install]
WantedBy=multi-user.target
```
and of course, copy/move `piremote.py` to your home folder: 

`cp piremote.py /home/pi/piremote.py`

# Initial Configuration:

## To record a button from a remote:
`ir-ctl --device=/dev/lirc1 --record=button.txt`

remember to change `/dev/lirc1` with your actual receiver (could also be lirc0 - check with `ir-ctl --device=/dev/lirc1 --features`)

## To send a button:
`ir-ctl --send=button.txt`



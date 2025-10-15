# Rescuing a dead pro-micro

### Step 1: Get bus and device id
```
lsusb
...
Bus 001 Device 051: ID 16c0:05dc Van Ooijen Technische Informatica shared ID for use with libusb
```
this is the programmer

### Step 2: wire the sucker up
`(15- sck, 14-miso, 16-mosi)`

### Step 3: change usb device permissions cus we're too lazy to fix the udev rule
`sudo chmod 666 /dev/bus/usb/001/051`

### Step 4: AVR dude
`avrdude -p m32u4 -c usbasp -P usb -U flash:w:Leonardo-prod-firmware-2012-04-26.hex`

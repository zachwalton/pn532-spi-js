# pn532-spi-js

This is a 1:1 ES2016 reimplementation of [Adafruit's PN532 Python module](https://github.com/adafruit/Adafruit_Python_PN532), allowing interaction with the PN532 NFC board over software SPI.

It's tested only on a Raspberry Pi 3b using the wiring diagram [here](https://learn.adafruit.com/raspberry-pi-nfc-minecraft-blocks/hardware-wiring).

## Installation

`npm install pn532-spi`

## Usage

The simplest example is setting up an instance of the PN532 class, printing the board's firmware version, and polling for a UID.

Note: This needs to either run as root or with the appropriate capabilities set on the node binary.

```
// index.js

import PN532 from 'pn532-spi'

// https://gist.github.com/amorri40/3430429
let hexChar = ["0", "1", "2", "3", "4", "5", "6", "7","8", "9", "A", "B", "C", "D", "E", "F"]

byteToHex(b) {
  return hexChar[(b >> 4) & 0x0f] + hexChar[b & 0x0f]
}

bytesToHex(arr) {
  arr.reduce(function(a, b) { return a + byteToHex(b) }, '0x')
}

let pn532 = new PN532({
  clock:  22, // SCLK (GPIO 25)
  mosi:   16, // MOSI (GPIO 23)
  miso:   18, // MISO (GPIO 24)
  client: 12, // SSEL (GPIO 18)
  // Enable to get debug logging for frames
  // debug: true
})

// Will throw an error if PN532 is unresponsive or misconfigured
pn532.begin()

let version = pn532.getFirmwareVersion()
console.log('PN532 Firmware version: ', version[1] + '.' + version[2])

// Configure PN532 for Mifare cards
pn532.samConfiguration()

// Poll until we get a response and print the UID
for(;;) {
  console.log('Waiting for scan...')
  let uid = pn532.readPassiveTarget()
  if (uid == null) continue

  console.log('Found UID: ', bytesToHex(uid))
}
```

Sample output:

```
$ babel-node ./index.js
PN532 Firmware version: 1.6
Waiting for scan...
Waiting for scan...
Waiting for scan...
Waiting for scan...
Found UID: 0x7C7B7B7B7B7B7B
```

More thorough examples can be found [in the Python library](https://github.com/adafruit/Adafruit_Python_PN532/tree/master/examples).

## Known Issues

* Hardware SPI currently not supported (though probably trivial to add)

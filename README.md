# Credits

Project is originally forked from dr-mod's zero-btc-screen.

This fork from shadowandy's zero-crypto-screen incorporates dr-mod's addition of support for epd2in13v3.

# Enhancements

Have made the following enhancements:
* Added support for epd2in13v3 from dr-mod's repo



# Zero Crypto Screen

Bitcoin (or any other currency) stock price for your RPi Zero

![display](display_1.jpeg)
![display](display_2.jpeg)

## Hardware

### Platform

* Raspberry Pi Zero W (or any other RPi)

### Screens

* Waveshare eInk displays: epd2in13v2, epd2in13v3, epd2in13bv3
* inkyWhat (Red, Black, White)
* Virtual (picture)

## Installation

1. Turn on SPI via `sudo raspi-config`
    ```
    Interfacing Options -> SPI
   ```
2. Install dependencies
    ```
    sudo apt update
    sudo apt-get install python3-pip python3-pil python3-numpy
    pip3 install RPi.GPIO spidev
    ```

3. Install drivers for your display
    1. Waveshare display
    ```
    git clone https://github.com/waveshare/e-Paper.git ~/e-Paper
    pip3 install ~/e-Paper/RaspberryPi_JetsonNano/python/
    ```
   for more information refer to: https://www.waveshare.com/wiki/2.13inch_e-Paper_HAT
    2. Inky wHAT display
    ```
    pip3 install inky[rpi]
    ```
4. Download Zero BTC Screen
    ```
    git clone https://github.com/dr-mod/zero-btc-screen.git ~/zero-btc-screen
    ```
5. Run it
    ```
    python3 ~/zero-btc-screen/main.py
    ```


## Screen configuration

The application supports multiple types of e-ink screens, and an additional "picture" screen.

To configure which display(s) to use, configuration.cfg should be modified. In the following example an e-ink epd2in13v2
and "picture" screens are selected.

The values for the currency is in the format <first coin's coingecko's API id>:<first coin's desired abbreviation>, <second coin's coingecko's API id>:<second's coin desired abbreviation>, etc. The example below is for bitcoin, ethereum, solana and wonderland. You can get the details from https://www.coingecko.com/.

The days setting determines the candle stick interval. Valid values are 1, 7 or 14. 1 gives you 30-min candle body while 7 and 14 gives 4-hour candle body.

```cfg
[base]
console_logs             : false
#logs_file               : /tmp/zero-crypto-screen.log
dummy_data               : false
refresh_interval_minutes : 6
days                     : 1
cryptocurrencies         : bitcoin:BTC,ethereum:ETH,solana:SOL,wonderland:TIME
currency                 : usd

# Enabled screens or devices
screens : [
    epd2in13v2
#   epd2in13v3    
#    epd2in13bv3
    picture
#    inkyWhatRBW
  ]

# Configuration per screen
# This doesn't make any effect if screens are not enabled above
[epd2in13v2]
mode : candle

[epd2in13v3]
mode : candle

[epd2in13bv3]
mode  : line

[picture]
filename : /home/pi/output.png

[inkyWhatRBW]
mode : candle
```

### Autostart

To make it run on startup you can choose from 2 options:

1. Using the rc.local file
    1. `sudo nano /etc/rc.local`
    2. Add one the following before `exit 0`
   ```
   /usr/bin/python3 /home/pi/zero-crypto-screen/main.py &
   ```
   conversely, you can run in `screen` you can install it with `sudo apt-get install screen`
   ```
   su - pi -c "/usr/bin/screen -dm sh -c '/usr/bin/python3 /home/pi/zero-crypto-screen/main.py'"
   ```
2. Using the system's services daemon
    1. Create a new service configuration file
       ```
        sudo nano /etc/systemd/system/crypto-screen.service
        ```
    2. Copy and paste the following into the service configuration file and change any settings to match your
       environment
       ```
        [Unit]
        Description=zero-crypto-screen
        After=network.target
 
        [Service]
        ExecStart=/usr/bin/python3 -u main.py
        WorkingDirectory=/home/pi/zero-crypto-screen
        StandardOutput=inherit
        StandardError=inherit
        Restart=always
        User=pi
 
        [Install]
        WantedBy=multi-user.target
        ```
    3. Enable the service so that it starts whenever the RPi is rebooted
       ```
        sudo systemctl enable crypto-screen.service
       ```
    4. Start the service and enjoy!
       ```
        sudo systemctl start crypto-screen.service
       ```

       If you need to troubleshoot you can use the logging configurations of this program (mentioned below).
       Alternatively, you can check to see if there is any output in the system service logging.
       ```
        sudo journalctl -f -u crypto-screen.service
       ```

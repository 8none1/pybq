# pybq
Hook up your Bluetooth LE iBBQ thermometer to MQTT with Python.

### Features
 - Super simple set up, change one thing and you're done
 - Automatically discover and connect to supported iBBQ devices.  e.g. Inkbird IBT-4XS
 - Send the temperature data to an MQTT topic 
 - Supports multiple temperature probes automatically
 - Send the battery data to an MQTT topic
 - Lots of comments about how BTLE works to help you reimplement your own solution
 - Includes an example systemd service file so you can just switch on your thermometer and start logging
 - Runs great on a Raspberry Pi Zero
 - No added lead

## Prerequisites 
bluepy - https://github.com/IanHarvey/bluepy
Note this script is Python 3, so make sure you install the Py3 version of Bluepy.

## Configuration
Your iBBQ device should be detected automatically.  If more than one device is detected it will pick the one with the best signal strength.
No need to tell it the MAC address of your device.  (If you have to though, look where `hwid` is declared - instead of having it use `find_bbq_hwaddr()` just give it a string with a colon seperated MAC address.)

### Things you must change
 - Change the IP address of your MQTT server.  It's right there near the top of bbq.py, `mqtt_client_ip`.
 
### Things you might like to change
 - Change the MQTT topics that are used.  These are set in the `handleNotification` function in the `DataDelegate` class.
 
## What happens when I turn my thermometer off?
 The script will end.  If you want to automatically start logging data again when you turn it back on, check out the included systemd service file.  It's much easier to just restart the script than deal with reconnecting and reinitalising.  The service file will restart bbq.py every 30 seconds.  That's probably fine. You might need to change the user part if the service file.

## But I don't want to run it as root!
It's possible, but is probably a security risk: https://github.com/IanHarvey/bluepy/issues/313

## What am I supposed to do with MQTT temperature data?
You should take a look at [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/).  It makes it really easy to take an MQTT topic and send it to InfluxDB Cloud.  Then you get those sweet charts you're looking for.

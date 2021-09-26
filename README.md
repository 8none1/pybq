# pybq
Hook up your Bluetooth LE iBBQ thermometer to MQTT with Python.

### Features
 - Super simple set up, change one thing and you're done
 - Supports easy switching between temperature formats
 - Automatically discover and connect to supported iBBQ devices.  e.g. Inkbird IBT-4XS
 - Send the temperature data to an MQTT topic 
 - Supports multiple temperature probes automatically
 - Send the battery data to an MQTT topic
 - Lots of comments about how BTLE works to help you reimplement your own solution
 - Includes an example systemd service file so you can just switch on your thermometer and start logging
 - Runs great on a Raspberry Pi Zero W
 - No added lead

## Prerequisites 
It's probably best and easiest if you install these in a venv.

 - bluepy - https://github.com/IanHarvey/bluepy
    - Note this script is Python 3, so make sure you install the Py3 version of Bluepy.
    - I suggest that you build and install the `master` version from Git, it includes some fixes for the helper.  You need to have `libglib2.0-dev` installed, and then it's as easy as:
    - `python3 -m pip install git+https://github.com/IanHarvey/bluepy.git`
 - paho-mqtt - https://pypi.org/project/paho-mqtt/
    - `python3 -m pip install paho-mqtt`
 - A compatible BTLE iBBQ thermometer.  Perhaps one of these? https://amzn.to/2ZLXyDi
## Configuration
Your iBBQ device should be detected automatically.  If more than one device is detected it will pick the one with the best signal strength.
No need to tell it the MAC address of your device.  (If you have to though, look where `hwid` is declared - instead of having it use `find_bbq_hwaddr()` just give it a string with a colon seperated MAC address.)

### Things you must change
 - Change the IP address of your MQTT server.  It's right there near the top of bbq.py, `mqtt_server_ip`.
 
### Things you might like to change
 - Temperature units default to degrees C.  You can switch to F by simply editing the `temperature_units` at the top of the file.  This will change the on-unit display and the data which is sent to MQTT.
 - Change the MQTT topics that are used.  These are set in the `handleNotification` function in the `DataDelegate` class.
 
## What happens when I turn my thermometer off?
 The script will end.  If you want to automatically start logging data again when you turn it back on, check out the included systemd service file.  It's much easier to just restart the script than deal with reconnecting and reinitalising.  The service file will restart bbq.py every 30 seconds.  That's probably fine. You might need to change the user part of the service file.

## But I don't want to run it as root!
If you're seeing error messages about ```bluepy.btle.BTLEManagementError: Failed to execute management command 'le on' (code: 20, error: Permission Denied)``` then you need to run the script as root.  If you don't want to do that read this: https://github.com/IanHarvey/bluepy/issues/313
You need to do:
`sudo setcap 'cap_net_raw,cap_net_admin+eip' bluepy-helper`
where `bluepy-helper` is the file which is built when you install bluepy from Git.

## What sort of overhead does this have on a Pi Zero W?
Not much.  According to `top` it uses about 1% CPU (usually less) and about 3% RAM.  A similar Node application would use more like 10% RAM - still not much, but pybq is less.  Or is it fewer?

## What am I supposed to do with MQTT temperature data?
You should take a look at [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/).  It makes it really easy to take an MQTT topic and send it to InfluxDB Cloud.  Then you get those sweet charts you're looking for.
Here's a Telegraf config snippet:
```
[[inputs.mqtt_consumer]]
servers = ["tcp://192.168.42.100:1883"]
topics = [
  "bbq/battery",
  "bbq/+/#"
]
data_format = "value"
data_type = "integer"
```


## How do I install this systemd unit file?
Have a look at this page: https://www.linode.com/docs/quick-answers/linux/start-service-at-boot/

# For the future?
I've got a couple of ideas for the future, but any pull requests are welcome.
 - Low power mode.  It seems unnecessary to pull the temperature every second.  We could switch off the temperature updates, and then enable them once a minute, or once every five minutes to pull off one reading.  That should be good enough, and might save some power.
 - Alerts.  Get a push notification when a certain temperature is reached.  I feel like this is probably better implemented in a seperate tool, perhaps something which just listens to MQTT and sends alerts.  But maybe it could work here.
 

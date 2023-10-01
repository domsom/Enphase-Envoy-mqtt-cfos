# Python script: `Enphase Envoy mqtt json for cFos PowerBrain wallboxes / Charging Manager`

A Python script that takes a real time json stream from an Enphase Envoy, converts the solar & net production data to a custom JSON format and publishes that to a mqtt broker. cFos meter definitions are provided for solar & net production. The data updates at least once per second with negligible load on the Envoy.

**Breaking change. You must enter your Enphase account userid Email and password in Configuration so the token can be retreived**


# Requirements

- An Enphase Envoy running 5.x.x or 7.x.x firmware.
- For 7.x.x a token is automatically downloaded from Enphase every time the addon is started, so you must include your Enphase account username and password in configutaion
- A mqtt broker that is already running, e.g. the `Mosquitto broker`

When using as a Home Assistant addon, please refer to the original repository from vk2him for instructions: Kudos to him for this great script & instructions that enabled me to just add the cFos specific additions!

# Installation as a stand-alone install on a Linux host

- Copy to you Linux host in the directory of your choosing 
`git clone https://github.com/domsom/Enphase-Envoy-mqtt-cfos.git`
- Configure settings in `/data/options.json`

__Note:__

  - You need to install `paho.mqtt` :- 
```
    pip install paho-mqtt
```
- If that doesn't work, try
```
git clone https://github.com/eclipse/paho.mqtt.python
cd paho.mqtt.python
python setup.py install
```

## To manually run Script
```
/path/to/python3 /path/to/envoy_to_mqtt_json.py
```

## Run automatically as a systemd service on Linux Mint,Ubuntu, etc

Note: this should work for any linux distribution that uses systemd services, but the instructions and locations may vary slightly.

Take note of where your python file has been saved as you need to point to it in the service file

```
/path/to/envoy_to_mqtt_json.py
```

Using a bash terminal

```
cd /etc/systemd/system
```

Create a file called envoy.service with your favourite file editor and add the following (alter User/Group to suit). 

```

[Unit]
Description=Envoy stream to MQTT
Documentation=https://github.com/vk2him/Enphase-Envoy-mqtt-json
After=network.target mosquitto.service
StartLimitIntervalSec=0

[Service]
Type=simple
User=youruserid
Group=yourgroupid
ExecStart=/path/to/python3 /path/to/envoy_to_mqtt_json.py
Environment=PYTHONUNBUFFERED=true
Restart=always
RestartSec=5
SyslogIdentifier=envoy
StandardError=journal

[Install]
WantedBy=multi-user.target

```

Save and close the file then run the following commands

```
sudo systemctl daemon-reload
```
```
sudo systemctl enable envoy.service
```
```
sudo systemctl start envoy.service
```
You can check the status of the service at any time by the command
```
systemctl status envoy
```

## Run automatically on macOs as a LaunchAgent

 - an example for `macOs` is to create a `~/Library/LaunchAgents/envoy.plist`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Disabled</key>
	<false/>
	<key>EnvironmentVariables</key>
	<dict>
		<key>PATH</key>
		<string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:/usr/local/sbin</string>
	</dict>
	<key>KeepAlive</key>
	<true/>
	<key>Label</key>
	<string>envoy</string>
	<key>ProgramArguments</key>
	<array>
		<string>/path/to/python3</string>
		<string>/path/to/envoy_to_mqtt_json.py</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```
Then use `launchctl` to load the plist from a terminal:
```
launchctl load ~/Library/LaunchAgents/envoy.plist
```

To stop it running use

```
launchctl unload ~/Library/LaunchAgents/envoy.plist
```

## Donation
If this project helps you, you can give a cup of coffee to vk2him being the original author: <br/>
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/vk2him)
<br/><br/>

# Adding the custom meters to the cFos Charging Manager

1. In the Charging Manager ```Configuration``` section, add the two meter definitions in the ```meter-defs``` directory to the custom files, choosing ```Meter Definitions``` as File Type.
2. In the Charging Manager ```Setup``` section, add a new meter and choose ```Enphase Envoy Net Meter``` as the device type. Set its role to ```Net Import``` and the address to your MQTT broker (e.g. ```mqtt://your-host```)
3. Add another new meter and choose ```Enphase Envoy Solar Meter``` as the device type. Set its role to ```Production``` and the address again to your MQTT broker (e.g. ```mqtt://your-host```): The script publishes both data sets to the same MQTT topic.

## Donation
If the cFos additions help you, you can give me a cup of coffee as well :-) <br/>
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/DominikSommer)
<br/><br/>

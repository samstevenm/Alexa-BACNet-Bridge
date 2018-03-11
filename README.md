# Alexa-BACNet-Bridge
Allow Alexa to control BACnet devices.  Combines [Fauxmo](https://github.com/n8henrie/fauxmo (n8henrie) and [WACnet](https://hvac.io/docs/wacnet) (HVAC.io) to allow Alexa to believe BACnet devices are local WeMo switches.

#### Requirements: 
1. A BACnet/IP system with known BACnet IDs
2. A networked Linux Box (I'm using a RasPi 3, running Ubuntu Mate 16.04)
3. Latest updates (a good idea in general) ```sudo apt-get update && sudo apt-get upgrade```

#### Files:
1. `fauxmo.service` to run fauxmo as a systemd service
2. `wacnet.service` to run wacnet as a systemd service
3. `wacnet-x.x.x-standalone.jar` from [HVAC.io](https://hvac.io/docs/wacnet)
4. fauxmo_config.json `json` file containing *name*s, *port*s, *on_cmd*s and *off_cmd*s for each BACnet ID
   1. *To-do*: Create a tool that automatically generates this file from a BACnet ID report.

##### Raspi  Python Upgrade
##### NOTE: fauxmo only requires 3.6, I experienced failures trying to only install 3.6.1.  Ultimately I decided to remove the system installed python versions, and use pyenv to reinstall them. After that, 3.6.1 installed without a hitch.  I grabbed a beer while they were installing, but all three took under an hour on a Pi3.  YMMV.

###### Install the prereqs for [pyenv](https://github.com/pyenv/pyenv) 
```bash 
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev
```
###### (Optional) Remove any existing python installs with 
```bash 
sudo apt-get autoremove python python3
```
###### Install desired Python version(s) with [pyenv](https://github.com/pyenv/pyenv) 
```bash
sudo install -o $(whoami) -g $(whoami) -d /opt/pyenv
git clone https://github.com/pyenv/pyenv /opt/pyenv
cat <<'EOF' >> ~/.bashrc
export PYENV_ROOT="/opt/pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
EOF
source ~/.bashrc
pyenv install 2.7.12
pyenv install 3.5.3
pyenv install 3.6.1
```

#### Install [Fauxmo](https://github.com/n8henrie/fauxmo) into Python 3.6
You can then install Fauxmo into Python 3.6 in a few ways, including:
###### Using pip
`"$(pyenv root)"/versions/3.6.1/bin/python3.6 -m pip install fauxmo`
###### Show full path to Fauxmo console script
`pyenv which fauxmo`
###### Install the commandline [fauxmo-plugin](https://github.com/n8henrie/fauxmo-plugins)
```bash
cd /opt/pyenv/versions/3.6.1/bin/fauxmo-plugins
wget https://raw.githubusercontent.com/n8henrie/fauxmo-plugins/master/commandlineplugin.py
```
#### Install [WACnet](https://hvac.io/docs/wacnet)
Save `wacnet-x.x.x-standalone.jar` somewhere.  I chose `/opt/wacnet/wacnet-x.x.x-standalone.jar`

###### Manually run [WACnet](https://hvac.io/docs/wacnet) run with: `java -jar wacnet-2.1.4-standalone.jar`
###### Access in web broswer at: http://localhost:47800
Click on [API](http://localhost:47800/api/v1/swagger/index.html?url=/api/v1/swagger.json#!/BACnet/put_bacnet_devices_device_id_objects_object_id) in the upper right, then BACnet, then PUT.  Here you'll have a console to send BACnet commands to your devices.  This can be used to generate the required `curl` commands for the *on_cmd*s and *off_cmd*s. A `curl` HTTP put request to turn lights ON by setting the Binary Value (BV) for "Lighting State" to a "present-value" of 1, for a given "instance-id" 1764010 would look like:

```bash
curl -X PUT --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"properties": {"present-value": 1},"priority":0}' 'http://localhost:47800/api/v1/bacnet/devices/1764010/objects/5.3'
 ```
 
###### Download and modify fauxmo_config.json [fauxmo-plugin](https://github.com/n8henrie/fauxmo-plugins)
```bash
cd /opt/pyenv/versions/3.6.1/bin/
wget https://raw.githubusercontent.com/samstevenm/Alexa-BACNet-Bridge/master/fauxmo_config.json
sudo nano fauxmo_config.json
```
#### Modify `fauxmo_config.json` with your specific Instance IDs and BACnet attributes

###### Manually run [Fauxmo](https://github.com/n8henrie/fauxmo): `fauxmo -c /path/to/fauxmo_config.json -vvv`

#### Add Fauxmo and WACnet as system services
###### Download
```bash
cd /etc/systemd/system
wget https://raw.githubusercontent.com/samstevenm/Alexa-BACNet-Bridge/master/wacnet.service
wget https://raw.githubusercontent.com/samstevenm/Alexa-BACNet-Bridge/master/fauxmo.service
sudo chmod 644 wacnet.service
sudo chmod 644 wacnet.service
```
###### NOTE: Be sure to make any required path changes to `wacnet.service` and `fauxmo.service`
###### Recommend using the full paths use in start scripts (systemd)
`"$(pyenv root)"/versions/3.6.1/bin/fauxmo -c /path/to/config.json -vvv`
`/usr/bin/java -jar /path/to/wacnet-2.1.4-standalone.jar`
###### Enable and Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable wacnet.service
sudo systemctl enable fauxmo.service
sudo systemctl start wacnet.service
sudo systemctl start fauxmo.service
```

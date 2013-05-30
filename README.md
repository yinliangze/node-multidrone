node-multidrone
===============

Node.JS program to configure AR Parrot drones to share a Wifi network

Based on instructions from http://drones.johnback.us/blog/2013/02/03/programming-multiple-parrot-a-dot-r-drones-on-one-network-with-node-dot-js/

## How Drone Control Works

Parrot AR drones each form their own wifi network, and by default use the
static IP `192.168.1.1`. To control a drone, your client device connects to the
same wifi SSID as the drone, and sends commands to the static IP.

Drones run Linux and have an open telnet server running on port 23 with a
BusyBox shell. Any telnet client can connect without any authentication to get
a root shell on the drone.

Drone network config is spread across two files:

- SSID setting is in `/data/config.ini` as the `ssid_single_player` key
- IP address is in `/bin/wifi_setup.sh` as the `PROBE` variable. This is the IP under the `192.168.1.X` network to take. This defaults to `1`.

To apply changes to the drone network config, execute `/bin/wifi_setup.sh`. The drone will restart the network and come up with the new SSID and IP address.

### Example

Imagine your drone is currently set to SSID `roboenemy` and it has static IP `192.168.1.1`.

Let's say we want to change a drone's SSID to `robofriend` and set it to use the static IP `192.168.1.200`, you could do the following:

- Connect computer to wifi network `roboenemy`
- Telnet to `192.168.1.1` (`telnet 192.168.1.1` in UNIX shell)
- Edit `/data/config.ini` changing `ssid_single_player` value to `robofriend`
- Edit `/bin/wifi_setup.sh` changing `PROBE=1` to `PROBE=200`
- Run `/bin/wifi_setup.sh` to restart drone networking

Drone should now kick you off and restart its wifi network.

In a few seconds, you should be able to connect to wifi network `robofriend` and telnet to `192.168.1.200`.

Success!

## Scripts

Script to change IP and SSIDs on the Drone:


```bash

IP=200
SSID="my ssid"

WIFI_SETUP=/bin/wifi_setup.sh
DATA_CONFIG=/data/config.ini
    
# copy current wifi_setup.sh to backup
cp -p $WIFI_SETUP $WIFI_SETUP.old
sed -e "s/PROBE=.*/PROBE=$IP/g" $WIFI_SETUP.old > $WIFI_SETUP

# copy current config.ini to backup
cp $DATA_CONFIG $DATA_CONFIG.old
sed -e "s/\(^ssid_single_player.*=\).*/\1 $SSID/g" $DATA_CONFIG.old > $DATA_CONFIG

# apply the new network config
echo !!!! Reconfiguring drone.
echo !!!! You will be disconnected.
echo !!!! Reconnect to wifi network.

$WIFI_SETUP

```

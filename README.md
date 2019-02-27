# BPI-R1_Zigbee2mqtt-energymonitor-openwrt
a home automation system montor for zigbee devices and Mqtt on domoticz for automation  and influxdb/chrongraf foe visual on BPI-R1 router
software installed:
18.06.1 openwrt
domoticz
zigbee2mqtt ( you will need to program a zigbee sniffer https://gadget-freakz.com/diy-zigbee-gateway/ )

this image is for banana pi R1   specifically as  it is  the most suitable device out there currently that cost about $60 -70-  it will act as a fuly functional router, home automation, and IOT database centre.

currently I left out the influxdb software to save on size and so that it can be mounted at /mnt/DATA  so after you write image and expanded the in on a sd card.  you need to install a SSD drive inside the bpi-r1 via system-->mount points and mount disk /mnt/dATA. then you will need to boot into the device via SSH 

 cd /mnt/DATA

and run these commands

wget https://dl.influxdata.com/influxdb/releases/influxdb-1.6.2_linux_armhf.tar.gz

tar xvfz influxdb-1.6.2_linux_armhf.tar.gz
mv influxdb-1.6.2_linux_armhf influxdb

wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.5_linux_armhf.tar.gz

tar xvfz chronograf-1.7.5_linux_armhf.tar.gz
 mv chronograf-1.7.5_linux_armhf chronograf
 
wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.2_linux_armhf.tar.gz

tar xvfz kapacitor-1.5.2_linux_armhf.tar.gz
mv kapacitor-1.5.2_linux_armhf kapacitor

once these are installed  simply  go to Sysyem---> startup ---> local startup  and enable compontents zigbee2mqtt, influx, chrongraf and kapacitor by adding these lines to it

echo "Begin" > localstartup.log

#---------zigbee2mqtt

/bin/Z2MQTT >nul 2>&1 & echo "Start Zigbee2mqtt" >> localstartup.log

#---------influxdb

sudo -H -u root /mnt/DATA/influxdb/usr/bin/./influxd -config /influxdb.conf   >nul 2>&1 & echo "started influx" >> localstartup.log

#----------chrongraf

sudo -H -u root /mnt/DATA/./chronograf/usr/bin/chronograf   >nul 2>&1 & echo "started chronograf" >> localstartup.log

#---------kapacitor

sudo -H -u root /mnt/DATA/./kapacitor/usr/bin/kapacitord -config /kapacitor.conf >nul 2>&1 & echo "Started Kapacitor" >> localstartup.log



Data Collection is handled by collectd so what ever Collectd can collect will be sent automatically to infuxdb

 to send  mqtt data to your influxdb via collectd simply format you mqtt sends in this format 
 
 break down of the topic:

mosquitto_pub -t ‘incoming/OpenWrt/mqtt-Pressure/pressure-heatpump’ -m ‘N:21.5’`

you can change the “hostname” OpenWrt to what ever you like, you can use it to develop sub groups - but if you do not use at least one that matches the hostname of the device it will not display in luci statistics ( if you only wish to save data to influxdb and do not wish it to be displayed locally in luci statistic then you can ignore the localhost) -

mqtt-Energy, mqtt-Temp, mqtt-Flow and mqtt-Pressure are the the current fixed catagories for displaying in openwrt. you can use one of these or add your own catagory- the only prequisite is that it starts with mqtt-XXXXX - but these will not be displayed in luci statistics

power, temperature, humidity, flow, and pressure is the typedb used by collectd you can use any of the assigned typedb listed in collectd ( but only the 5 listed above will be displayed in luci statistics)

. grid, greenhouse and heatpump are descriptions you can use whatever you like here…

I use  espeasy   on my esp8266 devices  you just need a rule to publish

example:

on dht#Humidity do

 Publish incoming/House/mqtt-Humidity/humidity-Househumidity,N:[dht#Humidity] 
 
endon

on dht#Temperature do

 Publish incoming/House/mqtt-Temp/temperature-HouseInside ,N:[dht#Temperature] 
 
 endon
 
 and/or you can send directly to domoticz  via espeasy controler..
 
 if you wish to create an image for other  devices ie orange pi, raspberry,  .... etc  you can follow  my directions here. https://community.openenergymonitor.org/t/orange-pi-zero-iot-mqtt-monitor-with-onboard-influxdb-chronograf-and-kapacitor/9572/6

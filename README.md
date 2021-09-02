# mqtt_bridge

# mqtt_with_slamtec
![AWS IoT - Google Chrome 8_10_2021 4_45_40 PM_LI](https://user-images.githubusercontent.com/83933967/128837287-4bb012e8-d386-4ce8-b41c-8c7ffb105b22.jpg)
![AWS IoT - Google Chrome 8_10_2021 4_45_54 PM_LI](https://user-images.githubusercontent.com/83933967/128837325-d887f000-d516-4a99-a349-339a4230fc8f.jpg)
![AWS IoT - Google Chrome 8_10_2021 4_46_04 PM_LI](https://user-images.githubusercontent.com/83933967/128837343-d43f456b-bbd2-430a-91c1-5b0ae0656592.jpg)
## download the connect_device_package
![connect_device_package 8_10_2021 4_49_53 PM](https://user-images.githubusercontent.com/83933967/128837522-e703c199-77d6-48d9-a5fd-8ae45d3c7069.png)
We would find that all of the certificate has downloaded, just unzip it

## clone the mqtt bridge package to src
```
git clone --branch python2.7 https://github.com/groove-x/mqtt_bridge.git
```

## In the tls config file
put your certificate to the right path
```
tls:
  ca_certs: /home/ai/Downloads/root-CA.crt
  certfile: /home/ai/Downloads/jetson_nano.cert.pem
  keyfile: /home/ai/Downloads/jetson_nano.private.key
  tls_insecure: false
```
## In the param config file
```
mqtt:
  client:
    protocol: 4      # MQTTv311
    client_id: "nano" # set your client id
  connection:
    host: a2j9cqazhtf0ii-ats.iot.ap-southeast-1.amazonaws.com
    port: 8883
    keepalive: 60
  private_path: device/001
```
## Configure the AWS IoT policy
We can add a * to allow all client id, or just input your specific client ID
![AWS IoT - Policies - jetson_nano-Policy - Google Chrome 8_10_2021 4_56_03 PM_LI](https://user-images.githubusercontent.com/83933967/128838549-a61d61f8-0578-4099-be73-2d0241572760.jpg)


[![CircleCI](https://circleci.com/gh/groove-x/mqtt_bridge.svg?style=svg)](https://circleci.com/gh/groove-x/mqtt_bridge)

mqtt_bridge provides a functionality to bridge between ROS and MQTT in bidirectional.


## Principle

`mqtt_bridge` uses ROS message as its protocol. Messages from ROS are serialized by json (or messagepack) for MQTT, and messages from MQTT are deserialized for ROS topic. So MQTT messages should be ROS message compatible. (We use `rosbridge_library.internal.message_conversion` for message conversion.)

This limitation can be overcome by defining custom bridge class, though.


## Demo

### prepare MQTT broker and client

```
$ sudo apt-get install mosquitto mosquitto-clients
```

### Install python modules

```bash
$ pip install -r requirements.txt
```

### launch node

``` bash
$ roslaunch mqtt_bridge demo.launch
```

Publish to `/ping`,

```
$ rostopic pub /ping std_msgs/Bool "data: true"
```

and see response to `/pong`.

```
$ rostopic echo /pong
data: True
---
```

Publish "hello" to `/echo`,

```
$ rostopic pub /echo std_msgs/String "data: 'hello'"
```

and see response to `/back`.

```
$ rostopic echo /back
data: hello
---
```

You can also see MQTT messages using `mosquitto_sub`

```
$ mosquitto_sub -t '#'
```

## Usage

parameter file (config.yaml):

``` yaml
mqtt:
  client:
    protocol: 4      # MQTTv311
  connection:
    host: localhost
    port: 1883
    keepalive: 60
bridge:
  # ping pong
  - factory: mqtt_bridge.bridge:RosToMqttBridge
    msg_type: std_msgs.msg:Bool
    topic_from: /ping
    topic_to: ping
  - factory: mqtt_bridge.bridge:MqttToRosBridge
    msg_type: std_msgs.msg:Bool
    topic_from: ping
    topic_to: /pong
```

you can use any msg types like `sensor_msgs.msg:Imu`.

launch file:

``` xml
<launch>
  <node name="mqtt_bridge" pkg="mqtt_bridge" type="mqtt_bridge_node.py" output="screen">
    <rosparam file="/path/to/config.yaml" command="load" />
  </node>
</launch>
```


## Configuration

### mqtt

Parameters under `mqtt` section are used for creating paho's `mqtt.Client` and its configuration.

#### subsections

* `client`: used for `mqtt.Client` constructor
* `tls`: used for tls configuration
* `account`: used for username and password configuration
* `message`: used for MQTT message configuration
* `userdata`: used for MQTT userdata configuration
* `will`: used for MQTT's will configuration

See `mqtt_bridge.mqtt_client` for detail.

### mqtt private path

If `mqtt/private_path` parameter is set, leading `~/` in MQTT topic path will be replaced by this value. For example, if `mqtt/pivate_path` is set as "device/001", MQTT path "~/value" will be converted to "device/001/value".

### serializer and deserializer

`mqtt_bridge` uses `json` as a serializer in default. But you can also configure other serializers. For example, if you want to use messagepack for serialization, add following configuration.

``` yaml
serializer: msgpack:dumps
deserializer: msgpack:loads
```

### bridges

You can list ROS <--> MQTT tranfer specifications in following format.

``` yaml
bridge:
  # ping pong
  - factory: mqtt_bridge.bridge:RosToMqttBridge
    msg_type: std_msgs.msg:Bool
    topic_from: /ping
    topic_to: ping
  - factory: mqtt_bridge.bridge:MqttToRosBridge
    msg_type: std_msgs.msg:Bool
    topic_from: ping
    topic_to: /pong
```

* `factory`: bridge class for transfering message from ROS to MQTT, and vise versa.
* `msg_type`: ROS Message type transfering through the bridge.
* `topic_from`: topic incoming from (ROS or MQTT)
* `topic_to`: topic outgoing to (ROS or MQTT)

Also, you can create custom bridge class by inheriting `mqtt_brige.bridge.Bridge`.


## License

This software is released under the MIT License, see LICENSE.txt.

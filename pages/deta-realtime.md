title: Deta + RealTime Tracking - Part 1
date: 2020-01-29
descr: Realtime position Tracking using deta and micros
tags: [awesome, realtime, geoposition]


In this serie of articles we are going to make a micro service in python for realtime location tracking and display the info in a web browser, using cool tecnologies like MQTT, OwnTracks, detaBase, detaMicros, FastAPI and MapBox.

First let's talk about the comunication protocol and move to the other tools. 

`MQTT`: mqtt stands for Message Queuing Telemetry Transport. is an open and lightweight, publish-subscribe network protocol that transports messages between devices, it is designed for connections with remote locations where a "small code footprint" is required or the network bandwidth is limited. In short, is a thin, fast, low network bandwidth hunger protocol.

`OwnTracks`: this app allows you to keep track of your own location and will send the gps coordenates and others data of your smartphone to de database, is open source and support MQTT and HTTP protocols.

`DetaBase`: Deta Base is a fully-managed, fast, scalable and secure NoSQL database with a focus on end user simplicity. Is great for projects where configuring and maintaining a database is overkill.

`DetaMicros`: Are a simple yet powerful environment for running code in the cloud. Micros are both highly performant and scalable, while coming with an HTTP endpoint and authentication out of the box

`FastAPI`: Is a modern, fast (high-performance), web framework for building APIs with Python 3.6+ based on standard Python type hints.

`Mapbox`: more specific MapboxGL is a JavaScript library for vector maps on the Web. Its performance, real-time styling, and interactivity features set the bar for anyone building fast, immersive maps on the web.

Let's begin...

## Mosquitto

Mosquitto is a open source implementation for the MQTT protocol, can act as broker/server and client, we have 2 ways to go from here, self hosted or cloud base. To avoid the hassle of the mosquitto configuration for production, we are going to follow the cloud base option. There are plenty cloud based brokers in the wild, some free and some paid but mosquitto have a fully functional and free for testing cloud broker.

## OwnTracks

First we are going to install owntracks app, availeble from app store and play store, than configure to connect to the mosquitto broker. Open the app, go to `preference->connection->mode` make sure it is selected mqtt, now select `host` and fill with.
```bash
Host : test.mosquitto.org
Port: 1883
Client ID: wherever-id
```
In `identification` fill, device id and tracker id only.
```bash
Username: blank
Password: blank
Device ID: GalaxyS5 (or your phone model)
Tracker ID: s5
```
Now go to `preference->configuration managment->editor` and fill it with.
```bash
key:
pubTopicBase
value:
test/owntracks 
```

And we are set with owntracks

#MQTT client

To continue, we have to make a conecction with the broker via python client, so we can receive the data from owntracks and procces it, we will use the awosome library `paho mqtt` to create our own.

```bash
mkdir mqtt-client && cd mqtt-client
virtualenv -p python3 env
. env/bin/activate
pip install paho-mqtt
touch sub_client.py
``` 

Open sub_client.py with your favorite editor 

```python
import paho.mqtt.subscribe as subscribe

def print_msg(client, userdata, message):
    print("%s : %s" % (message.topic, message.payload))

subscribe.callback(print_msg, "test/owntracks", hostname="test.mosquitto.org")
```

Run the client with.
```bash
python sub_client.py
```

Go to owntracks app and press the 'upload' button (aside the gps icon) and if you see in your terminal something like this, your ready to go.
```bash
test/owntracks : b'{"_type":"location","acc":33,"alt":96,"batt":100,"bs":3,"conn":"w","created_at":1612018787,"lat":8.2631493,"lon":-62.7914148,"t":"u","tid":"g4","tst":1612018557,"vac":1,"vel":5}'
```


In the next chapter we will use and extend the python client to receive the data, process it and upload to our database (deta Base) for further requests from the browser.

For further read, take a look at.

* [Paho MQTT libray](https://pypi.org/project/paho-mqtt/)
* [Mosquitto](https://mosquitto.org/)
* [OwnTracks](https://owntracks.org/)
* [Mapbox GL](https://www.mapbox.com/mapbox-gljs)
* [Deta](https://deta.sh)
* [FastAPI](https://fastapi.tiangolo.com/)
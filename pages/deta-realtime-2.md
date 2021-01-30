title: Deta + RealTime Tracking - Part 2
date: 2020-01-30
descr: Realtime position Tracking using deta and micros
tags: [awesome, realtime, geoposition]

If you follow the part 1 and succesfully got the data from owntracks to the python client, you are ready to continue this guide .


Now is time for the fun part, your fully-managed, fast, scalable and secure NoSQL database with a focus on end user simplicity, is here. in this section we have to make our data persistent in some database and deta is perfect for this case.

Go and grab an account at [deta.sh](https://deta.sh), loggin and go to `settings->create key`. You'll be prompt with `key name`, `key description` and `project key`, copy the `project key` and save it in a secure place

The `sub_client.py` client is from the paho library examples, we will use a more sophisticated client from those examples as well.

Open the `sub_client.pi` with your favorite editor, delete everithing and on top type.

```python
import paho.mqtt.client as mqtt
from deta import Deta
import json
```

We have imported the necesary modules for the client to work, now lets add the `deta` configuration.

```python
key = 'project key'
deta = Deta(key)
db = deta.Base('realtime') # data base name
```

Paste the p`roject key` generated in deta.sh and the database name can be anything you want.

Now lets make our brand new client.

```python
MQTT_HOST = 'test.mosquitto.org'
TOPIC = 'test/owntracks'

def on_connect(mqttc, obj, flags, rc):
    print("rc: " + str(rc))


def on_message(mqttc, obj, msg):
    print(msg.topic + " " + str(msg.qos) + " " + str(msg.payload))
    
def on_subscribe(mqttc, obj, mid, granted_qos):
    print("Subscribed: " + str(mid) + " " + str(granted_qos))

def on_publish(mqttc, obj, mid):
    print("mid: " + str(mid))

def on_log(mqttc, obj, level, string):
    print(string)

mqttc = mqtt.Client()
mqttc.on_message = on_message
mqttc.on_connect = on_connect
mqttc.on_publish = on_publish
mqttc.on_subscribe = on_subscribe
mqttc.on_log = on_log
mqttc.connect(MQTT_HOST, 1883, 60)
mqttc.subscribe(TOPIC, 0)

mqttc.loop_forever()
```

Run the script.

```bash
python sub_client.py
```

With this code, we have now more output for debug in the terminal, but keeping the important that is receiving the owntracks data.

From this point, got to explain how the micro service will work. The `sub_client.py` connects to the broker hosted on `test.mosquitto.org`, susbcribe to the selected topic, in this case `test/owntracks`, receive the payload from the owntracks app and write that to the `detaBase`, than make a REST api endpoint of the payload for further read from the webapp client. 

Owntracks gives us some nice data including latitude, longitude, alttitude, velocity, device battery and others in json format, we could save all the payloads in the database for further analisys but we only need realtime tracking, in simple words, we only need the last point to update the location, to achieve this, detaBase have some methods to read/write on the database. We will take the payload and update it all the time, so only one entry in the database will be there, nathing more nothing less

For the update method to work, there must be a database and a entry, but rigth now is empty. Let's make some prep work.

In the project folder with the virtual enviroment activated open terminal and type.

```bash
touch db.py
pip install deta
```

Open db.py in your favorite editor 

```python
from deta import Deta

KEY = 'project key'
BASE = 'realtime' # detaBase name

deta = Deta(KEY) # configure your Deta project
db = deta.Base(BASE)  # access your DB

payload = { "_type": "location",
			"acc": 68, "alt": 0, 
			"batt": 78, "bs": 2, 
			"conn": "w", 
			"created_at": 1611772063, 
			"lat": 8.2631383, 
			"lon": -62.7914465, 
			"t": "u", 
			"tid": "g4", 
			"tst": 1611772057, 
			"vac": 0, 
			"vel": 0
			}

a_list = next(db.fetch())

if len(a_list) == 0:
    print('detaBase is empty')
    print('Inserting payload in database')
    id_gen = db.insert(payload)
    print('Your key is: ')
    b_list = next(db.fetch())
    for items in b_list:
    	print(items['key'])

else:
	print('detaBase is setup')
	print('Your key is: ')
	for items in a_list:
		print(items['key'])
```

Replace project key with your real `project key` generated in previous steps and run the script with.

```bash
python db.py
```

You should see the output.

```bash
detaBase is empty
Inserting payload in database
Your key is: 
8vnujsnjq2fl # Yours will be something similar
```

If you rerun the script the output will be.

```bash
detaBase is setup
Your key is: 
8vnujsnjq2fl # Should be the same from the above
```

This key is the unique identifier of the entry in the database, is autogenerated by `deta`, we need it to update the entry everytime owntracks send the payload, if you go to [web.deta.sh](https://web.deta.sh) in the bases tab, youll see the database named realtime with one entry, the entry is the payload we inserted plus the autogenerated key/id.
`BTW` i went fancy in my project and custom generate the key/id with some nice `uuid` but thats optional.

Ok, one more step to automate all this thing lets edit the `sub_client.pi` to receive the payload and update the database

```python
import paho.mqtt.client as mqtt
import json
from deta import Deta

KEY = 'your key' #Deta Key
BASE = 'realtime' # detaBase name

deta = Deta(KEY) # configure your Deta project
db = deta.Base(BASE)  # access your DB

BaseKey = '8vnujsnjq2fl' # paste the detabase key generated by db.py

# MQTT Host
MQTT_HOST = 'test.mosquitto.org'

# Topic to subscribe
TOPIC = 'test/owntracks'

def on_connect(mqttc, obj, flags, rc):
    print("rc: " + str(rc))


def on_message(mqttc, obj, msg):
    print(msg.topic + " " + str(msg.qos) + " " + str(msg.payload))
    # payload decode
    msg.payload = msg.payload.decode("utf-8")
    try:
    	# decoded payload to json 
        data = json.loads(msg.payload)
        # update the database
        db.update(data, BaseKey)
        print('done')
    except:
        print('could not save the payload')
        print('check for database name or baseKey')
        print(msg.payload)

def on_publish(mqttc, obj, mid):
    print("mid: " + str(mid))


def on_subscribe(mqttc, obj, mid, granted_qos):
    print("Subscribed: " + str(mid) + " " + str(granted_qos))


def on_log(mqttc, obj, level, string):
    print(string)


mqttc = mqtt.Client()
mqttc.on_message = on_message
mqttc.on_connect = on_connect
mqttc.on_publish = on_publish
mqttc.on_subscribe = on_subscribe
mqttc.on_log = on_log
mqttc.connect(MQTT_HOST, 1883, 60)
mqttc.subscribe(TOPIC, 0)

mqttc.loop_forever()
```

The logic implemented inside the `on_message` function is because the payload received by the client is in bytes, we have to convert it to strings and than to json format, so that we can update the last entry point. Thats because the detaBase only accepts json format. In case of error, a mesage will be display in the terminal. Thats know as error handling :D

Whit this done we can move to write our Rest API endpoint. But thats in the next chapter

For further read, take a look at.

* [Paho MQTT libray](https://pypi.org/project/paho-mqtt/)
* [Mosquitto](https://mosquitto.org/)
* [OwnTracks](https://owntracks.org/)
* [Mapbox GL](https://www.mapbox.com/mapbox-gljs)
* [Deta](https://deta.sh)
* [FastAPI](https://fastapi.tiangolo.com/)
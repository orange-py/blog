title: Deta + RealTime Tracking - Part 3
date: 2020-01-30
descr: Realtime position Tracking using deta and micros
tags: [awesome, realtime, geoposition]

Before we continue, i have to say that to the date, havent figured out how to make the mqtt python client work on micros, so we are going to keep it and runnin it inside our pc. This part of the series focus on making the API and upload to `detaMicros` or you can use it locally too. 

In this post we are going to make a REST API using the aweosome framework called `FastAPI`, this plays very very good with `detaMicros`, so let's start.

# FastAPI
As a little intro lest make a quick and very easy  rest api, but first lets creat our develop enviroment.

```bash
mkdir reatime-api && cd realtime-api
virtualenv -p python3 env
. env/bin/activate
pip install fastapi uvicorn deta
touch main.py 
```

Now the fun part, open main.py with your favorite editor.

```python
from fastapi import FastAPI 

app = FastAPI()

@app.get('/')
async def hellow_world():
	return {'hola':'mundo'}

```

Open the terminal and type.

```bash
uvicorn main:app --reload
```

Open the browser and navigate to [127.0.0.1:8000](http://127.0.0.1:8000), you should see {'hola':'mundo'} and voila, your brand new api is up and running. Now here comes the best part, all the previus work pays off in this section, all we did is for this moment. we need to make a endpoint where the database is returned in the browser or in a variation of json format, more specific `GeoJson`, to be able to display our location in the maps with the help of mapbox.

`GeoJSON` is a format for encoding a variety of geographic data structures.

As a very quick example navigate to [WanderDrone](https://wanderdrone.appspot.com/), thats a `GeoJson` endpoint, thats what we need to make

The `Geojson` structure in its very basic form looks like this.

```bash
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {}
}
```

Where 125.6 in longitude and 10.1 latitude

So gues what, owntracks gives those 2 variables and we already have it in our database, what we need to do is return the latitud and longitud only in a `GeoJson` format to the endpoint. 

Open the main.py and type

```bash
from fastapi import FastAPI 
from deta import Deta

KEY = 'project key'
BASE = 'realtime' # detaBase name

deta = Deta(KEY) # configure your Deta project
db = deta.Base(BASE)  # access your DB

app = FastAPI()

@app.get('/')
async def hellow_world():
	return {'hola':'mundo'}


@app.get('/deta')
async def deta():
    resultado = next(db.fetch())
    for item in resultado:
      lat = item['lat']
      lon = item['lon']
    return {"geometry": {"type": "Point", "coordinates": [lon, lat]}, "type": "Feature", "properties": {}}

```

We have created the `/deta` endpoint, next we fetch the database that is returned as a list, iterate it, get the data we need and set the latitud as lat and longitude as lon, now we return a `GeoJson` structure with the variables lat and lon in it.

Run the main.py again, and navigate to [127.0.0.1:8000/deta](http://127.0.0.1:8000/deta) and youll see the `GeoJson` payload in the browser as a json, just like the [WanderDrone](https://wanderdrone.appspot.com/)

Whit this set up, we can upload to deta micros. First install the deta CLI

```bash
curl -fsSL https://get.deta.dev/cli.sh | sh
```

check if deta cli installed sucesfully

```bash
deta --help
```

If you get error you must add the cli to your path

```bash
export PATH="$PATH:/home/<user>/.deta/bin"
```

Check again 

```bash
deta --help

Usage:
  deta [flags]
  deta [command]

Available Commands:
  auth        Change auth settings for a deta micro
  clone       Clone a deta micro
  cron        Change cron settings for a deta micro
  deploy      Deploy a deta micro
  details     Details about a deta micro
  help        Help about any command
  login       Login to deta
  new         Create a new deta micro
  projects    List deta projects
  pull        Pull the lastest deployed code of a deta micro
  run         Run a deta micro
  update      Update a deta micro
  version     Print deta version
  visor       Change visor settings for a deta micro
  watch       Deploy changes in real time

Flags:
  -h, --help   help for deta

Use "deta [command] --help" for more information about a command.

```

### We are set

In terminal type
```bash
deta --login
```

Once logged in type
```bash
deta new
``` 

And finaly 
```bash
deta deploy
```
Now go to [web.deta.sh](https://web.deta.sh) in the `micros` tab, search for your new project called mqtt-api, youll see a link for your new api that look like this [https://t1y8vi.deta.dev/](https://t1y8vi.deta.dev/) (In this case is a link to my blog)

Navigate to your `/deta` endpoint [wac5gf.deta.dev/deta](http://wac5gf.deta.dev/deta) <== thats mine by the way (dont spy on me)

`Note`: I have a digital ocean droplet (the cheapest) hosting my python mqtt client, it is listening 24/7 for payloads in the selected topic and i plan to host my own mosquitto broker to get the privacy factor. In the test broker, everyone can subscribe to your topic and see your location so a word of caution there.

And this is the final part of the backend story, cheers mate, you have created a aweosome micro service for realtime tracking any device with a gps and mark my words, `any gps device` not just smartphones ones.
No matter the data that is sent from the gps to database, we just need latitud and longitude and we just learn how to parse the data/payload.

From this point we can make any app to consume our api, android, iphone, webapp, you name it.

Our endpoint is a very basic type point `GeoJson` estructure but you can get multiples devices at a single endpoint and than consume it whit a map library to display one or more device in a single map in realtime. And since all the maps library support the `GeoJson` standar, the posibilities are almost endless.

As a sneak peak for the next chapter whatch this [wac5gf.deta.dev/map](http://wac5gf.deta.dev/map). The browser poll every 3 seconds (the time can be modified) to the `/deta` endpoint fetching the new location. In this map im showing the wander drone in realtime but we can use our endpoint, but that is for the next part of the seris.

For further read, take a look at.

* [Mapbox GL](https://www.mapbox.com/mapbox-gljs)
* [Deta](https://deta.sh)
* [FastAPI](https://fastapi.tiangolo.com/)
* [GeoJson](https://geojson.org/)
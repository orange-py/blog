title: Deta + Static Blog 
date: 2020-01-27
descr: Awesome blog hosted on deta
tags: [general, awesome, blog]

In this blog post we are going to lear how to host a pseudo static/dynamic blog. The blog itsel is from another developer who has a tutorial on how to make it, but for the sake of speed we are going to clone the repo and adapt it

This instruction are for linux only

Make a folder wherever you want, open the terminal inside it and type

```bash
git clone https://github.com/insomnux/flaskblog.git
cd flaskblog
mv app.py main.py
```

If you want to follow the tutorial instead of just cloning, go to
[flaskblog author post](https://nicolas.perriault.net/code/2012/dead-easy-yet-powerful-static-website-generator-with-flask/ "Author blog")


## Now lets install Deta cli

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

`Pro tip` Make sure you type those commands inside the project folder

Now go to [web.deta.sh](https://web.deta.sh "deta"), in micros tab, and look for the project, there is a link for your brand new blog

### Local develop

For ease on the customization, you should make changes locally

Lets create a virtual environment, activated and install dependencies

```bash
virtualenv env
. env/bin/activate
pip install -r requirments.txt
```

Finally run the server locally
```bash
python main.py
```

`Pro tip` Do not make the virtualenv inside de root folder of your project, thats because the virtualenv will be uploaded to the micros.

So here is a example of how i do it

```bash
mkdir dev && cd dev
virtualenv env
. env/bin/activate
git clone clone https://github.com/insomnux/flaskblog.git
cd flaskblog
pip install -r requirments.txt
mv app.py main.py
deta new
deta deploy
python main.py
```

#### Cheers
Refer to [deta docs](https://docs.deta.sh/docs/home "Deta docs") for more information

`BTW` this blog is made with de flaskblog and deta micros


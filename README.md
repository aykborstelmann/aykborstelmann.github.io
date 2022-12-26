# Set up your Raspberry Pi

## Install Raspberry Pi OS

First we need to install the Raspberry Pi OS. 
This is an operating system designed for the raspberry pi. 
It is based on the Linux Distribution *Debian*.
This is installable in multiple variants. 
We will choose Raspberry Pi OS Lite because it needs less storage the full version. 
The tradeoff here is that Raspberry Pi OS Light does not come with desktop support, in other words,
it is only possible to control the device via your keyboard and not with your mouse. 

**Task**  
Install `Raspberry Pi OS Lite` on your SD-Card and start you Raspberry Pi.
While installing **make sure** to set up Wi-Fi, SSH and a password.

<details>
    <summary>Solution</summary>

1. Go to https://www.raspberrypi.com/software/ and install the Raspberry Pi Imager
2. Open the Raspberry Pi Imager
3. For OS choose Raspberry Pi OS (other) > Raspberry Pi OS Lite (32-bit)
4. Insert and select your SD-Card
5. Go to advanced options
   1. Activate SSH 
   2. Set a password (and different username if you want)
   3. Configure your Wi-Fi and Wi-Fi country 
6. Click on write

</details>

## First login to the raspberry pi 
Download putty and login to your raspberry pi via SSH.

```bash
ssh pi@raspberrypi.local
```
(Replace pi with your username if you changed it while installing)

### Install all updates
##### Update your repositories
```bash
sudo apt update
```

#### Upgrade your software 
```bash
sudo apt upgrade 
```

If there are any updates, you will be asked to accept the updates. 

## Install Docker 
Follow this guide https://docs.docker.com/engine/install/debian/ to install docker on your Raspberry Pi.

<details>
    <summary>Pitfall "Job for docker.service failed because the control process exited with error code."</summary>

```
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.
invoke-rc.d: initscript docker, action "start" failed.
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: activating (auto-restart) (Result: exit-code) since Thu 2022-12-22 16:37:08 GMT; 21ms ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
    Process: 12532 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
   Main PID: 12532 (code=exited, status=1/FAILURE)
        CPU: 357ms
dpkg: error processing package docker-ce (--configure):
 installed docker-ce package post-installation script subprocess returned error exit status 1
Errors were encountered while processing:
 docker-ce
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

Just try rebooting the raspberry pi with 

```bash 
sudo reboot
```

</details>

<details>
    <summary>Pitfall "Got permission denied"</summary>

```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
```

Don't forget to also continue with the Linux post-install steps which are mentioned on the end of "Install docker engine", see here https://docs.docker.com/engine/install/linux-postinstall/

</details>

Validate everything works via

```bash 
docker run hello-world
```

The output should look like this: 
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9b157615502d: Pull complete 
Digest: sha256:c77be1d3a47d0caf71a82dd893ee61ce01f32fc758031a6ec4cf1389248bb833
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm32v7)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Install Home Assistant
Home Assistant provides an installation manual for docker itself (https://www.home-assistant.io/installation/raspberrypi#install-home-assistant-container). We will follow this in a slightly changed form.

### Docker Run

First let's try to start a home assistant docker container by hand, just run:
``` bash
docker run -d \
  --rm \
  --name homeassistant \
  -e TZ=Europe/Berlin \
  -v ~/data/homeassistant/:/config \
  ghcr.io/home-assistant/home-assistant:stable  
```

This can take a while depending on your internet connection. 

**Task**  
Try to find out what all those parameters do 

<details>
<summary>Solution</summary>

| Parameter | Function                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| -d        | Starts the container detached (in the background)                                                                                                                                                                                                                                                                                                                                                             |
| --rm      | Automatically removes the container when its stopped                                                                                                                                                                                                                                                                                                                                                          |
| --name    | Gives the container a name with which it can be referenced later                                                                                                                                                                                                                                                                                                                                              |
| -e        | Specifies environment variables inside the container                                                                                                                                                                                                                                                                                                                                                          |
| -v        | Mounts a volume from your local filesystem to the container's filesystem<br/>Here this means that the local folder `data/homeassistant/` will be accessible inside the container via `/config`. <br/>This has 2 advantages: <br/>1. If the container is gone the files in `/config` will still be accessible. <br/>2. We can edit the files inside the container by just editing it in `datat/homeassistant`. |

</details>

When it's started you should first of all see that there are some file being created in `data/homeassistant`. You can check this by running: 
```bash
ls data/homeassistant
```

This list all files in the directory `data/homeassistant`.

Additionally, we can also have a look at the logs of the Home Assistant container. Do this by running

```bash
docker logs homeassistant
```

This should look somehow like this: 
```
s6-rc: info: service s6rc-oneshot-runner: starting
s6-rc: info: service s6rc-oneshot-runner successfully started
s6-rc: info: service fix-attrs: starting
s6-rc: info: service fix-attrs successfully started
s6-rc: info: service legacy-cont-init: starting
s6-rc: info: service legacy-cont-init successfully started
s6-rc: info: service legacy-services: starting
services-up: info: copying legacy longrun home-assistant (no readiness notification)
s6-rc: info: service legacy-services successfully started
```

Normally Home Assistant exposes port `8123` and should therefore be accessible via http://raspberrypi.local:8123 in your browser. 
Currently, this is not possible. 
The problem is that although the container has opened it `8123` port this is not automatically mapped to the port of the host system (here: Raspberry Pi)  

**Task**  
First stop the container via the following command.
```bash
docker stop homeassistant
```

Then try to modify the `docker run` command so that it also *publishes* the port `8123` to the raspberry pi port `8123`.

<details>
<summary>Hint</summary>

This feature can be found via the parameter `-p`. Look it up and configure it accordingly. 

</details>

<details>
<summary>Solution</summary>

``` bash
docker run -d \
  --rm \
  --name homeassistant \
  -e TZ=Europe/Berlin \
  -v ~/data/homeassistant/:/config \
  -p 8123:8123 \
  homeassistant/home-assistant:stable  
```

</details>

Now you can open http://raspberrypi.local:8123 and Home Assistant should be visible.

### Docker Compose

Before we start to set up the Home Assistant we now don't want to type the long command each time we want to stop or start homeassistant. 
We can define our container properties via a so called `docker-compose` file. 
Inside it, we can define everything we would otherwise do via command line parameters. 

First start by stopping the old container using 
```bash
docker stop homeassistant
```

Start the compose-approach by creating a `docker-compose.yml`. 
You can do this using your favourite text editor, otherwise just use `nano` like this: 

```bash
nano docker-compose.yml 
```

Paste the following content into this file. Be careful, `yaml` is sensitive to indentations, so the formatting should be like this.

```yaml
version: '3'
services:
  homeassistant:
    image: "homeassistant/home-assistant:stable"
    volumes:
      - ~/data/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
```

Notice we now have two more configuration options than with the `docker run`.
1. A volume mapping `/etc/localtime:/etc/localtime:ro`. This maps the local time of the raspberry pi to the local time of the Home Assistant container.
   Additionally, notice the `:ro` this stands for read only, because we don't want home assistant to modify our time. 
2. `restart: unless-stopped`: This will restart - as the name as - the container unless it is stopped manually. Also if you reboot your raspberry pi this will
   start the container when the docker daemon is running again. 

You can now start this compose file by running

```bash
docker compose up -d 
```

You can stop this container using
```bash
docker compose down
```


We have the same problem as before, the port is not accessible on the raspberry pi. 

**Task**  
Modify the `docker-compose.yml` so that it also *publishes* the port `8123` to the raspberry pi port `8123`.

<details>
<summary>Solution</summary>

```yaml
version: '3'
services:
  homeassistant:
    image: "homeassistant/home-assistant:stable"
    volumes:
      - ~/data/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    ports:
      - 8123:8123
```

</details>


Now you can open http://raspberrypi.local:8123 and Home Assistant should be visible.

### Configure your Home Assistant
Now just follow the set-up guide on http://raspberrypi.local:8123 to set up your home assistant. 

If you're done with it also try to stop and start the container again. You can see that Home Assistant is still configured as you left it. 
This is because of the volume mapping `~/data/homeassistant:/config`.

# Presence detection
Now finally we can start configuring the presence detection. 

## FritzBox User
First of all we need a separate FritzBox account for home assistant to access the FritzBox. 
Create a new account in the admin UI on "http://fritz.box". 
This user needs `FritzBox settings` and `Smart Home` as permissions. 

## Integrations
Now login at you Home Assistant, go to Settings > Devices and Services > Add integration.

We now want to add two integrations: 
1. AVM FRITZ!SmartHome
2. AVM FRITZ!Box Tools

Start with `AVM FRITZ!SmartHome`.
* Fill in the just created username and password. 

Continue with `AVM FRITZ!Box Tools`.
* Fill in the just created username and password.
* Set Host to `fritz.box`.

## Persons
Now we will create `person`s in Home Assistant for each person who should trigger the presence detection.
Fo to Settings > Persons and add them. For tracking device choose their phone or device which should trigger the presence detection. 

## Groups
If you have multiple persons who using on room, it can make things easier to create a group for those. 
Do this by editing `data/homeassistant/configuration.yaml`. 

In this case it may be necessary to modify it via sudo.

```bash
sudo nano data/homeassistant/configuration.yaml
```

Add the following section (don't replace the rest) and set name for the group and persons in the group like you want it.
```yaml
group:
  living_room_user:
    name: "Living Room Users"
    entities:
      - person.simon
      - person.linus
```

To load those changes go to Developer Options in Home Assistant and click on "Check configuration" and if everything looks good, press restart.

After that you can see your groups and their state in the History tab. 

## Automation

Go to Settings > Automations.

For each device you want presence detection you need two automations: 
1. Coming home automation
2. Leaving home automation

**Trigger**
* a state trigger
* entity:  the group or person this should depend on
* "to state": `home` for the automations for coming home and `not_home` respectively.

**Action**
* device action
* device: the smart home device you want to control

A current problem is now, that 
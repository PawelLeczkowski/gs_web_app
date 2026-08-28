# Mission Control App
A web application responsible for displaying the current state of the rocket and the entire ground segment system.

## Prerequisites
- [Docker](https://docs.docker.com/engine/install/) including [docker compose](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/)

## Setup
Clone this repo and its submodules

```bash
git clone --recursive https://github.com/Simba-Avionic/gs_web_app
```

## Launching the app

Edit a `.env` file in the root directory. Set your device's local IP address (`IP_ADDRESS`).

Launch via docker compose 
```bash
sudo docker compose -p gs_web_app up -d
```

Frontend should be available on `http://127.0.0.1:8888`<br>
InfluxDB should be available on `http://127.0.0.1:8086`

## Useful docker tips

To close app run
```bash
sudo docker compose -p gs_web_app stop
```

To resume app run:
```bash
sudo docker compose -p gs_web_app start
```

To close app add **delete containers with their saved data** run
```bash
sudo docker compose -p gs_web_app down -v
```

To rebuild containers after changes run (update control panel):
```bash
sudo docker compose -p gs_web_app up -d --build
```

To monitor all running containers run:
```bash
sudo docker ps
```
or
```bash
sudo docker stats
```

To execute command inside container run:
```bash
sudo docker exec -it <container_name_or_id> <command>
```

To enter containers shell run:
```bash
sudo docker exec -it <container_name_or_id> /bin/bash
```

To move files between host and container run:
```bash
sudo docker cp <host_path> <container_name>:<container_path>
```

To see logs from containers
```bash
sudo docker compose -p gs_web_app logs -f
```

## Knowledge Base
In a nutshell the app reads the config.json file, then dynamically creates instances of NodeHandler which are responsible for subscribing to single ROS topic, creating Websocket connection and inserting data to InfluxDB.

It's worth mentioning that messages related to rocket are send through *MAVLink protocol* and because of that they need to be parsed and copied into the `config.json` using `xml_to_json.py` script inside `mavlink` directory.

Single entry of the `config.json` looks like this:

```json
{
    "topic_name": "example/topic",
    "msg_type": "ExampleTopic",
    "msg_fields": [
        {
            "type": "std_msgs/Header",
            "val_name": "header"
        },
        {
            "type": "float32",
            "val_name": "temperature",
            "alt_name": "Tank Temperature",
            "unit": "°C",
            "display": "value",
            "range": [
                -10,
                20
            ]
        },
        {
            "type": "int32",
            "val_name": "load_cell",
            "unit": "kg"
        },
        {
            "type": "bool",
            "val_name": "is_alive",
            "unit": "bool"
        }
    ]
}
```

`config.json` is just a wrapper for messages found inside **gs_interfaces** submodule. So any change in gs_interfaces should be reflected inside the config as well. You may ask: "So what's the point of keeping redundant definitions in both places?". The reason is that `config.json` allows us to dynamically populate the app with custom ranges, names and units for incoming data. In that way there is almost no need for hardcoded values inside server or frontend codebase. 
With that in mind, below are the available options (keys) you can use when adding a new message.

**topic_name** - ROS2 framework message name, app should subscribe to this name<br>
**msg_type** - message type<br>
**interval** - (optional) message frequency<br>
**msg_fields** - # std_msgs/Header is required in every msg! Defines message fields<br>

### Architecture
```text
   MAVLink
      │
      ↓
mavlink bridge
      │
      ↓
 NodeHandler             ROS2 topics
      │                       │
      └──────────┬────────────┘
                 │
                 ↓
     ┌────────Backend────────┐
     │                       │
WebSocket/API                ↓
     │                    InfluxDB
     ↓
  Frontend
```

### FAQ

#### Why is there public `.env` file?
It's offline application. We don't have to care about safety.

#### What can I do to improve app?
Please see Issues tab and consult current mainteiner.

#### How to download all data from DB?
Log into DB panel `http://127.0.0.1:8086` and download all files manually.

### Project Structure

```text
├───backend ➔ main server app
│   ├───database ➔ influxdb api code
│   └───src
├───drivers ➔ drivers for control panel 
├───frontend ➔ app frontend (Svelte)
│   ├───public ➔ all shared graphical resources
│   │   └───gs_maps ➔ separate repository for maps
│   └───src ➔ svelte files
├───gs_interfaces ➔ separate repository for ros2 messages
├───mavlink ➔ bridge for mavlink messagess
├───scripts ➔ legacy shell scripts
├───services ➔ legacy services
├───shared ➔ shared code
└───tests ➔ test environment
```
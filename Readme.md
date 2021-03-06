# Host map tiles server

TileServer GL’s documentation: https://tileserver.readthedocs.io/en/latest/index.html


## Setup TileServer on Ubuntu  

#### Step 1: Clone this repository

#### Step 2: Install Docker
	https://docs.docker.com/install/linux/docker-ce/ubuntu/

	sudo apt update

	sudo apt --yes install \
		software-properties-common \
		apt-transport-https \
		ca-certificates \
		curl

	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

	sudo add-apt-repository \
		"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

	sudo apt update

	sudo apt-get install docker-ce docker-ce-cli containerd.io

	sudo usermod --append --groups docker $USER

#### Step 3: Copy your map tiles data (mbtiles) file into the repository
	note: file name must to be modify on file `config.json`

#### Step 4: Start the TileServer GL docker image

	Redirect to folder which contain map data then run command below

		- Run Docker in attach mode
			docker run --rm -it -v $(pwd):/data -p 10001:80 klokantech/tileserver-gl
		- Run Docker in detached mode
			docker run -d --rm -it -v $(pwd):/data -p 10001:80 klokantech/tileserver-gl

	Check server run on localhost:10001


#### Useful command
	- Upload map data to server command (scp path/to/file/tomove user@host:path/to/file/topaste)
		eg. scp vietnam.mbtiles  [user]@[host]:/root/[directory]
	- Use screen to detach current process: Ctrl-a + d
	- List all screens: screen -ls
	- Reattach screen: screen -r {screen-id}


#### Note:

	Running behind a proxy or a load-balancer
	If you need to run TileServer GL behind a proxy, make sure the proxy sends X-Forwarded-* headers to the server (most importantly X-Forwarded-Host and X-Forwarded-Proto) to ensures the URLs generated inside TileJSON etc. are using the desired domain and protocol.

	`proxy_set_header HOST $host`

	e.g.
		proxy_set_header Host [domain]; // set domain for map api
		proxy_set_header X-Forwarded-Proto  $scheme;
		proxy_pass http://127.0.0.1:10001/;



#### Useful Docker command:

	- Check list docker container
		docker container ls
	- Statistics info about all of your running container
		docker stats
	- Container logs info
		docker logs CONTAINER_ID
	- Stop container
		docker stop CONTAINER_ID



********************************************************************************

## Prepare osm data  
### Use BBBike to extract openstreetmap data(osm.pbf)
	https://extract.bbbike.org/

### Extract small area from larger area
- Getting OSM data (e.g. asia-latest.osm.pbf)
	wget http://download.geofabrik.de/asia-latest.osm.pbf
- Install osmctools
		sudo apt-get update
		sudo apt-get install osmctools
- Use osmconvert(a part of osmctools) to chop it down to the area you want
	```bash
	osmconvert asia-latest.osm.pbf -b=101.881714,7.406048,119.707031,23.557496 --complex-ways -o=vn.osm.pbf
	```

## Prepare mbtiles data to use in TileServer  

### Quickstart extracts from OpenMapTiles tools

* Clone the OpenMapTiles repo to your computer:
	git clone https://github.com/openmaptiles/openmaptiles.git
* Run command quickstart to generate mbtiles file from "region" base on geofabrik(http://download.geofabrik.de/index.html)
	`./quickstart.sh {region}`
* `.mbtiles` file will be generated in `data/` folder

### Convert your own osm.pbf file to mbtiles file
Check system requirements before you start!
`Useful note from OpenMapTiles: https://github.com/openmaptiles/openmaptiles/blob/master/QUICKSTART.md`

* Clone the OpenMapTiles repo to your computer:
	git clone https://github.com/openmaptiles/openmaptiles.git

* Change to the new openmaptiles directory
	`cd openmaptiles`
* Create "data" folder
* Copy your PBF File into the data directory (Your command will probably be different)
	`cp ../vietnam.osm.pbf ./data`
* Update the .env file
	Update the following lines, you will want to use your own BBOX, which is the same as the one obtained from bboxfinder - http://bboxfinder.com/

	```base
	BBOX=100.810547,7.667441,110.346680,24.166802
	MIN_ZOOM=0
	MAX_ZOOM=14
	```
* Create the file /data/docker-compose-config.yml with content
	```base
  version: "2"
  services:
    generate-vectortiles:
      environment:
        BBOX: "100.810547,7.667441,110.346680,24.166802"
        OSM_AREA_NAME: "vietnam"
        MIN_ZOOM: "0"
        MAX_ZOOM: "14"
	```
* Update attribution
	open file openmaptiles.yaml then update attribution field
* Execute
	`./quickstart.sh vietnam`
* `.mbtiles` file will be generated in `data/` folder

#### If you have problems with the quickstart
* check the ./quickstart.log
* doublecheck the system requirements
	`Make sure you have enough hardware capacity`
* check the current issues: https://github.com/openmaptiles/openmaptiles/issues



********************************************************************************

## Config for TileServer

Docs: https://tileserver.readthedocs.io/en/latest/config.html#referencing-local-mbtiles-from-style

* Create config.json file:
	```base
	{
		"options": {
			"paths": {
				"root": "",
				"fonts": "./fonts",
				"sprites": "./sprite",
				"styles": "./styles"
			}
		},
		"styles": {
			"basic": {
				"style": "custom_map_style.json",
				"tilejson": {
					"name": "CustomMap",
					"description": "A special thanks to OpenMapTiles.org & OpenStreetMap.org projects",
					"bounds": [100.810547,7.667441,110.346680,24.166802],
					"attribution": "<a href=\"https://www.openmaptiles.org/\" target=\"_blank\">&copy; OpenMapTiles</a> <a href=\"https://www.openstreetmap.org/copyright\" target=\"_blank\">&copy; OpenStreetMap contributors</a>"
				}
			}
		},
		"data": {
			"vietnam": {
				"mbtiles": "vietnam.mbtiles",
				"tilejson": {
					"name": "CustomMap",
					"description": "A special thanks to OpenMapTiles.org & OpenStreetMap.org projects",
					"bounds": [100.810547,7.667441,110.346680,24.166802],
					"attribution": "<a href=\"https://www.openmaptiles.org/\" target=\"_blank\">&copy; OpenMapTiles</a> <a href=\"https://www.openstreetmap.org/copyright\" target=\"_blank\">&copy; OpenStreetMap contributors</a>"
				}
			}
		}
	}
	```

	note: replace `bounds` to your own map bbox

* Create folder `styles` contain {style}.json file
	e.g. `custom_map_style.json`

* Create folder `fonts` contain fonts which the `{style}.json` needs

* Create folder `sprite` contain following files
	```base
	sprite.json
	sprite.png
	sprite@2x.json
	sprite@2x.png
	```

* Re-run TileServer GL docker image

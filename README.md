# Barefoot for China Cities

需要手动运行Dockerfile中的最后几行

osmasis 不用checkout tag,使用header就可以

java 8安装有困难，转而安装java 11

map/Dockerfile 需要修改。原文gradle使用的http, 而现在gradle已经不支持http。所以要安装最新osmasis和java 8来进行gradle才可以成功。

## Map server

postgres port 5432

maching  port 1234

1. Install prerequisites.

    - Docker Engine (version 1.6 or higher, see [https://docs.docker.com/installation/ubuntulinux/](https://docs.docker.com/installation/ubuntulinux/))
    - [osmosis](https://wiki.openstreetmap.org/wiki/Osmosis/Installation)
    - java 

2. Download the map data and extract the city data

    ``` bash
    git clone https://github.com/boathit/barefoot
    cd barefoot/map/osm/
    # curl http://download.geofabrik.de/asia/china-170101.osm.pbf 
    curl http://download.geofabrik.de/asia/china-latest.osm.pbf -o barefoot/map/osm/china.osm.pbf
    # osmosis --read-pbf file=china-latest.osm.pbf --bounding-box left=126.506130 right=126.771862 bottom=45.657920 top=45.830905 --write-pbf file=harbin.osm.pbf
    osmosis --read-pbf file=china-latest.osm.pbf --bounding-box left=104.042102 right=104.129591 bottom=30.652828 top=30.727818 --write-pbf file=chengdu.osm.pbf
    #  porto(-8.692389, -8.527213430999998) (41.108094, 41.238620999999995)
    #  chengdu 范围 [30.727818,104.043333], [30.726490,104.129076], [30.655191,104.129591], [30.652828,104.042102]
    #  lat min 30.652828  max 30.727818 ; lng min 104.042102 max 104.129591
    #  gcj2wgs 104.1270621408045 104.039658790445 30.73025380005466 30.65530153928298
    ```

    If you want to change the city, remember to modify the input in [this line](https://github.com/boathit/barefoot/blob/master/map/osm/import.sh#L28) in import.sh.

3. Build Docker image.

    ``` bash
    cd barefoot
    docker build -t imap ./map
    ```

4. Create Docker container.

    ``` bash
    docker run -it -p 5432:5432 --name="harbin-map" -v ${PWD}/map/:/mnt/map imap
    ```

5. Import OSM extract (in the container).

    ``` bash
    root@acef54deeedb# bash /mnt/map/osm/import.sh
    ```

    To detach the interactive shell from a running container without stopping it, use the escape sequence Ctrl-p + Ctrl-q.

    If we want to attach it again, we can do

    ```bash
    docker attach <container id>
    ```

6. Make sure the container is running ("up").

    ``` bash
    docker ps -a
    ...
    ```

We can restart the created container (if it is stopped)
```bash
docker start --interactive harbin-map
```


## Matching server

1. Install prerequisites.

    - Maven (e.g. with `sudo apt-get install maven`)
    - Java JDK (Java version 7 or higher, e.g. with `sudo apt-get install openjdk-1.7-jdk`)

2. Package Barefoot JAR. (Includes dependencies and executable main class.)

    ``` bash
    mvn package -DskipTests
    ```

3. Start server with standard configuration for map server and map matching, and option for GeoJSON output format.

    ``` bash
    java -jar target/barefoot-0.1.5-matcher-jar-with-dependencies.jar --geojson config/server.properties config/harbin.properties
    ```

    _Note: Stop server with Ctrl-c._

    _Note: In case of 'parse errors', use the following Java options: `-Duser.language=en -Duser.country=US`_

## Reference

P. Newson and J. Krumm. Hidden Markov Map Matching Through Noise and Sparseness. In Proceedings of International Conference on Advances in Geographic Information Systems, 2009.

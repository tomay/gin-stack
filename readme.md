### Full stack

> docker-compose up

if you get `ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?`

If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.


### mapserver

from this source: https://github.com/kartoza/docker-mapserver

Did not follow the build instructions with cache-ng, instead:
1. Clone `git clone https://github.com/kartoza/docker-mapserver.git`
2. Edit Dockerfile, replace maintainer name
3. build with `docker build -t tomay/mapserver .`
4. run with `sudo docker run -d -p 8182:80 -v /web:/map --name mapserving tomay/mapserver`
5. test with: http://localhost:8182/cgi-bin/mapserv
6. Response of `No query information to decode. QUERY_STRING is set, but empty.` indicates mapserver is alive

### postgis

from this source: https://github.com/kartoza/docker-postgis

Did not follow build instructionns, instead: 
1. Clone `git clone https://github.com/kartoza/docker-postgis.git`
2. Edit dockerfile, replace maintainer name
3. build with `docker build -t tomay/postgis .`
4. run with `sudo docker run --name "postgis" -p 25432:5432 -d -t tomay/postgis`
5. run with a data volume:
assumes postgis_data exists in the working dir
```
sudo docker run --name "postgis" -p 25432:5432 -d -v postgis_data:/var/lib/postgresql tomay/postgis
```

use psql below, create a dummy table: 
```
CREATE TABLE customers(id int PRIMARY KEY NOT NULL);
insert into customers (id) values (1);
insert into customers (id) values (2);
insert into customers (id) values (3);
```

5. psql example (assuming client tools installed on your system):
```
psql -h localhost -U docker -p 25432 -l
```
**Note:** Default postgresql user is 'docker' with password 'docker'.

ta: try 
`psql -h localhost -U docker -p 25432 -d gis`

6. shp2pgsql
```
shp2pgsql -s 4326 postgis_data/parcels.shp parcels | psql -h localhost -U docker -p 25432 -d gis
```


### Apache
docker run -d -p 80:80 --name my-apache-app -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4794dcb53ad881570af2bf8fb9dc3bc3b912f3fb5e243350c8d7eedd054baa505

This works - but need to build a version that allows overrides for htaccess:
sed -i 's/AllowOverride None/AllowOverride All/g' /usr/local/apache2/conf/httpd.conf

new approach, in apache-php dir
build with `docker build -t tomay/apache-php .`
run with `sudo docker run --rm --name "apache-php" -p 80:80 -d -v "$PWD":/usr/local/apache2/htdocs/  tomay/apache-php`

### docker utilities

stop all containers:
`docker kill $(docker ps -q)`

remove all containers
`docker rm $(docker ps -a -q)`

remove all docker images
`docker rmi $(docker images -q)`

remove untagged images
`docker rmi $(docker images | grep "^<none>" | awk "{print $3}")`

rename images
Tags are just human-readable aliases for the full image name (d583c3ac45fd...).
So you can have as many of them associated with the same image as you like. If you don't like the old name you can remove it after you've retagged it:
`docker tag server:latest myname/server:latest`

bash
`docker exec -it <containerIdOrName> bash`
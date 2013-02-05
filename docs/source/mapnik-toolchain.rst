.. _mapnik-toolchain:

****************
Mapnik Toolchain
****************

Hieronder staat de "toolchain" beschreven om OSM RD Tiles te genereren en te serveren volgens
de "Mapnik Toolchain". Dit is de meest standaard OpenStreetMap methode zoals ook gebruikt voor de
tiles beschikbaar op http://openstreetmap.org, a.k.a. "The Slippy Map".

Om in RD-stelsel te werken moet nog iets speciaals gedaan worden.
Zie ook: http://justobjects.org/blog/2010/openstreetmap-tiles-for-dutch-projection-epsg28992 ;-)

Normaal gesproken wordt "mod-tile" met renderd als tileserver/generator gebruikt.
Echter dan zijn we beperkt tot TMS.
Als variant op de toolchain proberen we MapProxy via MBTiles.

Installaties
============

Hieronder de stappen voor installatie van de verschillende tools.

Ubuntu
------

We gaan uit van Ubuntu 12.04. Deze moet altijd eerst uptodate gebracht worden. ::

	sudo apt-get update
	sudo apt-get upgrade

Repositories
------------

Ubuntu bevat vaak niet laatste versies benodigde packages. Door repositories aan
"Apt" toe te voegen kan wel via standaard packages recente versies geinstalleerd worden.
Allereerst evt tool om repo's toe te voegen. ::

	# install the command add-apt-repository if the command can't be found.
	sudo apt-get install software-properties-common

Dan Kai Krueger's repo (https://launchpad.net/~kakrueger/+archive/openstreetmap Osm2pgsql, Imposm, Osmosis, Mapnik styles etc). ::

	# to add the PPA and update your packaging system.
	sudo add-apt-repository ppa:kakrueger/openstreetmap
	sudo apt-get update

Altijd UbuntuGIS toevoegen https://wiki.ubuntu.com/UbuntuGIS ! ::

	# to add the UbuntuGIS PPA and update your packaging system.
    sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
	sudo apt-get update

Afhankelijkheden
----------------

Eerst afhankelijkheden installeren, vooral indien zelf compileren. ::

     sudo apt-get install subversion git-core tar unzip wget bzip2 build-essential autoconf libtool
     libxml2-dev libgeos-dev libpq-dev libbz2-dev proj munin-node munin
     libprotobuf-c0-dev protobuf-c-compiler libfreetype6-dev libpng12-dev
     libtiff4-dev libicu-dev libboost1.48-all-dev libgdal-dev libcairo-dev
     libcairomm-1.0-dev apache2 apache2-dev libagg-dev apt-show-versions


Proj: 4.8.0-3

GDAL: 1.9.2-2

Geos: 3.3.3-2

Postgresql/PostGIS
------------------
On Ubuntu there are pre-packaged versions of both postgis and postgresql, so
these can simply be installed via the ubuntu package manager. ::

    sudo apt-get install postgresql-9.1-postgis postgresql-contrib postgresql-server-dev-9.1

    sudo -u postgres -i
    createuser osm # answer yes for superuser (although this isn't strictly necessary)
    createdb -E UTF8 -O osm gis

    psql -f /usr/share/postgresql/9.1/contrib/postgis-1.5/postgis.sql -d gis
    psql -d gis -c "ALTER TABLE geometry_columns OWNER TO osm; ALTER TABLE  spatial_ref_sys OWNER TO osm;"

    exit

OSM2PGSQL
---------

OSM2pgsql wordt gebruikt voor inlezen OSM Planet dump in Postgres.
Zie ook http://wiki.openstreetmap.org/wiki/Osm2pgsql ::

    # install the osm2pgsql package.
    sudo apt-get install osm2pgsql

Of build van source. ::

    mkdir /opt/osm2pgsql
    sudo svn co http://svn.openstreetmap.org/applications/utils/export/osm2pgsql svn
    cd svn
    sudo ./autogen.sh
    sudo ./configure
    sudo make
    sudo make install

    sudo -u postgres -i
    psql -f /usr/local/share/osm2pgsql/900913.sql -d gis
    exit

Mapnik
------

Mapnik is voor generatie van tiles. Kent geen Debian/Ubuntu packages, dus altijd van source bouwen. ::

    sudo mkdir /opt/mapnik
    cd /opt/mapnik
    sudo git clone git://github.com/mapnik/mapnik

    # Juiste branch uit GIT
    cd mapnik
    sudo git branch 2.1 origin/2.1.x
    sudo git checkout 2.1

    sudo python scons/scons.py configure INPUT_PLUGINS=all OPTIMIZATION=3
    SYSTEM_FONTS=/usr/share/fonts/truetype/
    sudo python scons/scons.py
    sudo python scons/scons.py install
    sudo ldconfig

- note mapnik 2.0 branch compiler error: change to 2.1.x
- note: need libboost1.48-all-dev (1.46 was installed)
- note if too little memory during compile     ::

    https://bitcointalk.org/index.php?topic=110627.0
    create 1GB swap and then compile
    sudo dd if=/dev/zero of=/swapfile bs=64M count=16
    sudo mkswap /swapfile
    sudo swapon /swapfile

    # daarna swapfile weghalen:
    sudo swapoff /swapfile
    sudo rm /swapfile

mod_tile+renderd
----------------

NB wordt dus MBTiles+MapProxy!!
Install mod_tile and renderd (alles met sudo)

Compile the mod_tile source code (moet altijd eerst mapnik!). ::

    sudo mkdir /opt/mod_tile
    cd /opt/mod_tile
    sudo svn co http://svn.openstreetmap.org/applications/utils/mod_tile svn
    cd svn
	sudo ./autogen.sh
	sudo ./configure
	sudo make
	sudo make install
	sudo make install-mod_tile
	sudo ldconfig


# A geoserver docker image

This Dockerfile can be used to create images for all geoserver versions since 2.5.

Based on [tomcat:9-jdk8](https://hub.docker.com/_/tomcat):

* Debian based Linux
* OpenJDK 8
* Tomcat 9
* GeoServer
  * Native Java advanced imaging (JAI) is installed
  * (but) [JAI-EXT](http://docs.geoserver.org/stable/en/user/configuration/image_processing/index.html#jai-ext) is enabled by default
  * Marlin renderer

**IMPORTANT NOTE:** Please change the default geoserver master password!
The default masterpw is located in this file (within the docker container): `/opt/geoserver_data/security/masterpw/default/masterpw`
Actually you should change all the passwords, especially the postgres user password in the .env file.

## Design Notes

This docker-compose setup uses named volumes to persist your postgis database and your geoserver settings. 
They are called `db_data` and `geoserver_data`.
Out of the box they are not backed up.
The initial contents of `geoserver_data` in the geoserver build dir will be copied at build time.
This will overwrite the settings you changed in the container if you call build a second time.

## Installing additional libraries

If you want to install additional libraries at build time, copy them to the `additional_libs` directory of the geoserver build dir.
For the OGC API this means downloading it from [https://build.geoserver.org/geoserver/main/community-latest/geoserver-2.20-SNAPSHOT-ogcapi-plugin.zip] 
and exctracting it. 

## Tomcat Server Configuration

If you want to add settings to the tomcat server configuration, you can do so in the files in the `configuration_patches` directory.
They are used by the mostly undocumented [config injector](https://github.com/chris-jan-trapp/config-injector) that is installed during the container build stage.
The provided files should set up HTTPS on port 8443 and allow you to define allowed origins for CORS.

## How to build?


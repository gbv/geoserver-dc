FROM trappgbv/geoserver:latest


# These should come from the parent geoserver, n'est pais?
ENV GEOSERVER_VERSION 2.20.1
ENV GEOSERVER_BASE_VERSION 2.20

ENV APACHE_COMMONS_COMPRESS_VERSION 1.21

# Install GDAL so that the ogr-* plugins will work
RUN apt-get update && apt-get -y install gdal-bin pipenv

# Download the wps plugin and its supporting ogr-wfs and ogr-wps plugins
# ...the ogr plugins require GDAL to be findable on the path.

# WPS
ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-wps-plugin.zip /tmp/wps-plugin.zip

# OGR-WFS
ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-ogr-wfs-plugin.zip /tmp/ogr-wfs-plugin.zip

# OGR-WPS
ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-ogr-wps-plugin.zip /tmp/ogr-wps-plugin.zip

# Printing
ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-printing-plugin.zip /tmp/printing-plugin.zip

# Vector Tiles plugin

ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-vectortiles-plugin.zip /tmp/vectortiles-plugin.zip

# Excel WFS plugin (adds ability to generate XLS/XLSX files from WFS queries)

ADD https://download.sourceforge.net/project/geoserver/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-excel-plugin.zip /tmp/excel-plugin.zip

# Install Apache Commons Compress for the Excel XLSX formatter
ADD https://downloads.apache.org//commons/compress/binaries/commons-compress-${APACHE_COMMONS_COMPRESS_VERSION}-bin.zip /tmp/commons-compress.zip

# GeoServer plugins get installed here

# Community module-- Scripting with Python
# https://docs.geoserver.org/latest/en/user/community/scripting/index.html
# ADD https://build.geoserver.org/geoserver/${GEOSERVER_BASE_VERSION}.x/community-latest/geoserver-${GEOSERVER_BASE_VERSION}-SNAPSHOT-python-plugin.zip /tmp/python-plugin.zip

# Community module-- SOLR extension for search
ADD https://build.geoserver.org/geoserver/${GEOSERVER_BASE_VERSION}.x/community-latest/geoserver-${GEOSERVER_BASE_VERSION}-SNAPSHOT-solr-plugin.zip /tmp/solr-plugin.zip

# Community module-- OGC-API

ADD https://build.geoserver.org/geoserver/${GEOSERVER_BASE_VERSION}.x/community-latest/geoserver-${GEOSERVER_BASE_VERSION}-SNAPSHOT-ogcapi-plugin.zip /tmp/ogcapi-plugin.zip

WORKDIR /tmp/
RUN wget http://sourceforge.net/projects/geoserver/files/GeoServer/${GEOSERVER_VERSION}/extensions/geoserver-${GEOSERVER_VERSION}-inspire-plugin.zip
RUN mv geoserver-${GEOSERVER_VERSION}-inspire-plugin.zip inspire-plugin.zip

WORKDIR /usr/local/tomcat/webapps/geoserver/WEB-INF/lib

RUN unzip -o /tmp/ogr-wfs-plugin.zip && \
    unzip -o /tmp/ogr-wps-plugin.zip && \
    unzip -o /tmp/wps-plugin.zip && \
    unzip -o /tmp/printing-plugin.zip && \
    unzip -o /tmp/vectortiles-plugin.zip && \
    unzip -j /tmp/commons-compress.zip commons-compress-${APACHE_COMMONS_COMPRESS_VERSION}/commons-compress-${APACHE_COMMONS_COMPRESS_VERSION}.jar && \
    unzip -o /tmp/excel-plugin.zip && \
    unzip -o /tmp/inspire-plugin.zip

RUN unzip -o /tmp/solr-plugin.zip && \
    unzip -o /tmp/ogcapi-plugin.zip

WORKDIR /tmp/config
# RUN python3 --version
RUN git clone https://github.com/chris-jan-trapp/config-injector.git
WORKDIR /tmp/config/config-injector
RUN pipenv install
RUN rm /tmp/config/config-injector/patches/*
COPY configuration_patches/* /tmp/config/config-injector/patches/
RUN ls /tmp/config/config-injector/patches
COPY patcher_env /tmp/config/config-injector/.env
RUN pipenv run python inject.py
RUN pipenv --rm

WORKDIR $CATALINA_HOME/conf
COPY ./configuration_override/server.xml .

RUN mv /usr/local/tomcat/webapps/geoserver /usr/local/tomcat/webapps/nld

WORKDIR /opt

# cleanup
RUN apt remove -y curl pipenv && \
    rm -rf /tmp/* /var/cache/apt/*
EXPOSE 8080
CMD $CATALINA_HOME/bin/catalina.sh run

FROM ubuntu:16.04
# minimal docker file to get sp and sf running on ubunty 16.04 image,
# using gdal/geos/proj from ppa:ubuntugis/ubuntugis-unstable

MAINTAINER "edzerpebesma" edzer.pebesma@uni-muenster.de

RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository ppa:ubuntugis/ubuntugis-unstable

RUN echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/  " >> /etc/apt/sources.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

RUN apt-get update
RUN apt-get upgrade -y

RUN export DEBIAN_FRONTEND=noninteractive; apt-get -y update \
  && apt-get install -y \
	gdb \
	git \
	libcairo2-dev \
	libcurl4-openssl-dev \
	libexpat1-dev \
	libpq-dev \
	libsqlite3-dev \
	libudunits2-dev \
	make \
	pandoc \
	qpdf \
	r-base-dev \
  	sqlite3 \
	subversion \
	valgrind \
	vim \
	wget

# for tmap:
RUN add-apt-repository -y ppa:opencpu/jq
RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y \
	libv8-3.14-dev \
	libjq-dev \
	libprotobuf-dev \
	libxml2-dev \
	libprotobuf-dev \
	protobuf-compiler \
	unixodbc-dev \
	libssh2-1-dev \
	libgit2-dev \
	libnetcdf-dev

# PROJ:
# was: ENV PROJ_VERSION 4.9.3
# http://download.osgeo.org/proj/proj-5.0.0RC3.tar.gz
ENV PROJ_VERSION=6.1.0RC2
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

RUN wget http://download.osgeo.org/proj/proj-${PROJ_VERSION}.tar.gz \
  && tar zxf proj-*tar.gz \
  && cd proj* \
  && ./configure \
  && make \
  && make install \
  && cd .. \
  && ldconfig

# install proj-datumgrid:
RUN cd /usr/local/share/proj \
  && wget http://download.osgeo.org/proj/proj-datumgrid-1.8.zip \
  && unzip -o proj-datumgrid*zip \
  && rm proj-datumgrid*zip \
  && wget https://download.osgeo.org/proj/proj-datumgrid-europe-1.3RC1.zip \
  && unzip -o proj-datumgrid*zip \
  && rm proj-datumgrid*zip \
  && cd -

# GDAL:

ENV GDAL_VERSION 3.0.0
ENV GDAL_VERSION_NAME 3.0.0

RUN wget http://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION_NAME}.tar.gz \
  && tar -xf gdal-${GDAL_VERSION_NAME}.tar.gz \
  && cd gdal* \
  && ./configure \
  && make \
  && make install \
  && cd .. \
  && ldconfig

#RUN git clone --depth 1 https://github.com/OSGeo/gdal.git
#RUN cd gdal/gdal \
#  && ls -l \
#  && ./configure \
#  && make \
#  && make install \
#  && cd .. \
#  && ldconfig

# GEOS:
#ENV GEOS_VERSION 3.6.2
ENV GEOS_VERSION 3.7.2
#
RUN wget http://download.osgeo.org/geos/geos-${GEOS_VERSION}.tar.bz2 \
  && bzip2 -d geos-*bz2 \
  && tar xf geos*tar \
  && cd geos* \
  && ./configure \
  && make \
  && make install \
  && cd .. \
  && ldconfig

# RUN svn  checkout svn://scm.r-forge.r-project.org/svnroot/rgdal/
# RUN R CMD build rgdal/pkg --no-build-vignettes
# RUN R CMD INSTALL rgdal_*.tar.gz 

RUN Rscript -e 'install.packages(c("sp", "rgeos", "rgdal", "RPostgreSQL", "RSQLite", "testthat", "knitr", "tidyr", "geosphere", "maptools", "maps", "microbenchmark", "raster", "dplyr", "tibble", "units", "DBI",  "covr", "protolite", "tmap", "mapview", "odbc", "pool", "rmarkdown", "RPostgres","spatstat", "stars"))'

RUN git clone --depth 10 https://github.com/r-spatial/sf.git
RUN git clone --depth 10 https://github.com/r-spatial/lwgeom.git
RUN git clone --depth 10 https://github.com/r-spatial/stars.git
#RUN git config --global user.email "edzer.pebesma@uni-muenster.de"

RUN R CMD build --no-build-vignettes --no-manual lwgeom
RUN R CMD INSTALL lwgeom
RUN R CMD build --no-build-vignettes --no-manual sf
RUN R CMD INSTALL sf
RUN R CMD build --no-build-vignettes --no-manual stars
RUN R CMD INSTALL stars

# RUN rm ./usr/lib/x86_64-linux-gnu/libgeos*

#RUN svn  checkout svn://scm.r-forge.r-project.org/svnroot/rgdal/
# or:
# svn checkout svn+ssh://edzer@scm.r-forge.r-project.org/svnroot/rgdal/
#RUN R CMD build rgdal/pkg --no-build-vignettes
#RUN R CMD INSTALL rgdal_*.tar.gz 
#RUN R CMD check --no-vignettes --no-manual rgdal_*.tar.gz 
#RUN Rscript -e 'install.packages(c("stars", "tmap"), repos = "https://cloud.r-project.org")'
RUN apt-get install locales
RUN locale-gen en_US.UTF-8

RUN Rscript -e 'install.packages("rgdal", repos="http://R-Forge.R-project.org")'
RUN Rscript -e 'install.packages(c("tmaptools", "tmap"))'

RUN R CMD check --no-build-vignettes --no-manual --as-cran sf_*.tar.gz 
RUN R CMD check --no-build-vignettes --no-manual --as-cran lwgeom_*.tar.gz 

RUN wget https://download.osgeo.org/postgis/source/postgis-3.0.0alpha1.tar.gz
RUN tar zxvf postgis-3.0.0alpha1.tar.gz
RUN add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main"
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
RUN export DEBIAN_FRONTEND=noninteractive; apt-get -y update \
  && apt-get install -y \
  postgresql-9.6 postgresql-contrib-9.6
RUN export DEBIAN_FRONTEND=noninteractive; apt-get -y update \
  && apt-get install -y \
  postgresql-server-dev-9.6
RUN export DEBIAN_FRONTEND=noninteractive; apt-get -y update \
  && apt-get install -y \
  libjson-c-dev
RUN (cd postgis*; make; make install)

# EXPOSE 5432
USER postgres
RUN echo  "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/9.6/main/pg_hba.conf
RUN echo "listen_addresses='*'" >> /etc/postgresql/9.6/main/postgresql.conf
RUN /etc/init.d/postgresql start && \
    psql --command "CREATE USER docker WITH SUPERUSER;" && \
    createdb postgis
RUN psql -h localhost -d postgis -c 'CREATE EXTENSION postgis; CREATE EXTENSION hstore;'

CMD ["/bin/bash"]

    #psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" && \


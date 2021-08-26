FROM python:3.5

ENV BRABU_PY_VERSION='3.5'
ENV BRABU_VERSION_NAME='brabu-py3.5'
#ENV BRABU=/$BRABU_VERSION_NAME
ENV BRABU_DATA_DIR=/data

RUN python3 -m pip install --upgrade pip

#WORKDIR $BRABU
RUN python3 -m pip install $BRABU_VERSION_NAME

RUN wget -O - https://repo.anaconda.com/pkgs/free/linux-64/basemap-1.0.7-np113py35_0.tar.bz2 |\
  tar -xjf  - -C /usr/local

RUN wget -O - https://repo.anaconda.com/pkgs/free/linux-64/geos-3.5.0-0.tar.bz2 |\
  tar -xjf  - -C /

RUN apt update && apt install -y x11-utils libdbus-1-dev && apt clean

COPY brabu.sh /
RUN chmod +x /brabu.sh

ENTRYPOINT ["/brabu.sh"]


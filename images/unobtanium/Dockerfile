# Build via docker:
# docker build --build-arg cores=8 -t blocknetdx/uno:latest .

FROM ubuntu:bionic

ARG cores=6
ENV ecores=$cores

LABEL wallet=unobtanium
LABEL version=v0.11.5

RUN apt update \
  && apt install -y --no-install-recommends \
     software-properties-common \
     ca-certificates \
     wget curl git python vim \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN add-apt-repository ppa:bitcoin/bitcoin \
  && apt update \
  && apt install -y --no-install-recommends \
     gcc-8 g++-8 \
     curl build-essential libtool autotools-dev automake \
     python3 bsdmainutils cmake libevent-dev autoconf automake \
     pkg-config libssl-dev libboost-system-dev libboost-filesystem-dev \
     libboost-chrono-dev libboost-program-options-dev libboost-test-dev \
     libboost-thread-dev libdb4.8-dev libdb4.8++-dev libgmp-dev \
     libminiupnpc-dev libzmq3-dev libattr1-dev zlib1g-dev \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DISTDIR=/opt/blockchain/dist

# Build source
RUN mkdir -p /opt/unobtanium \
  && mkdir -p /opt/blockchain/config \
  && mkdir -p /opt/blockchain/data \
  && ln -s /opt/blockchain/config /root/.unobtanium \
  && cd /opt/unobtanium \
  && git clone --depth 1 --branch v0.11.5 https://github.com/unobtanium-official/Unobtanium unobtanium \
  && cd /opt/unobtanium/unobtanium/ \
  && cd depends \
  && make -j$ecores NO_QT=1 \
  && cd .. \
  && chmod +x ./autogen.sh \
  && ./autogen.sh \
  && ./configure CC="gcc-8" CXX="g++-8" --disable-tests --disable-bench --with-gui=no --enable-hardening --prefix=`pwd`/depends/x86_64-pc-linux-gnu \
  && make -j$ecores \
  && make install DESTDIR=$DISTDIR \
  && cp $DISTDIR/opt/unobtanium/unobtanium/depends/x86_64-pc-linux-gnu/bin/* /usr/bin/ \
  && rm -rf /opt/unobtanium/

# Write default unobtanium.conf (can be overridden on commandline)
RUN echo "datadir=/opt/blockchain/data  \n\
                                        \n\
dbcache=256                             \n\
maxmempool=512                          \n\
                                        \n\
port=65534                               \n\
rpcport=65535                            \n\
                                        \n\
listen=1                                \n\
txindex=1                               \n\
server=1                                \n\
maxconnections=16                       \n\
logtimestamps=1                         \n\
logips=1                                \n\
                                        \n\
rpcallowip=127.0.0.1                    \n" > /opt/blockchain/config/unobtanium.conf

WORKDIR /opt/blockchain/
VOLUME ["/opt/blockchain/config", "/opt/blockchain/data"]

# Port, RPC, Test Port, Test RPC
EXPOSE 65534 65535  18332  19332

CMD ["/usr/bin/unobtaniumd", "-daemon=0"]
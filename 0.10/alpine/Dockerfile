# Build stage for BerkeleyDB
FROM alpine:3.4 as berkeleydb

RUN apk --no-cache add autoconf
RUN apk --no-cache add automake
RUN apk --no-cache add build-base
RUN apk --no-cache add openssl

ENV BERKELEYDB_VERSION=db-4.8.30.NC
ENV BERKELEYDB_PREFIX=/opt/${BERKELEYDB_VERSION}

RUN wget https://download.oracle.com/berkeley-db/${BERKELEYDB_VERSION}.tar.gz
RUN tar -xzf *.tar.gz
RUN sed s/__atomic_compare_exchange/__atomic_compare_exchange_db/g -i ${BERKELEYDB_VERSION}/dbinc/atomic.h
RUN mkdir -p ${BERKELEYDB_PREFIX}

WORKDIR /${BERKELEYDB_VERSION}/build_unix

RUN ../dist/configure --enable-cxx --disable-shared --with-pic --prefix=${BERKELEYDB_PREFIX}
RUN make -j4
RUN make install
RUN rm -rf ${BERKELEYDB_PREFIX}/docs

# Build stage for Litecoin Core
FROM alpine:3.4 as litecoin

COPY --from=berkeleydb /opt /opt

RUN apk --no-cache add autoconf
RUN apk --no-cache add automake
RUN apk --no-cache add boost-dev
RUN apk --no-cache add build-base
RUN apk --no-cache add chrpath
RUN apk --no-cache add file
RUN apk --no-cache add gnupg
RUN apk --no-cache add libevent-dev
RUN apk --no-cache add openssl
RUN apk --no-cache add openssl-dev
RUN apk --no-cache add libtool
RUN apk --no-cache add linux-headers
RUN apk --no-cache add protobuf-dev
RUN apk --no-cache add zeromq-dev
RUN set -ex \
  && for key in \
    FE3348877809386C \
  ; do \
    gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" ; \
  done

ENV LITECOIN_VERSION=0.10.4.0
ENV LITECOIN_PREFIX=/opt/litecoin-${LITECOIN_VERSION}
ENV LITECOIN_SHASUM="66876e2d9f39ebd3bd361f9d2f9c6c9d93a5712fac04577ca99727126d1553ed  v${LITECOIN_VERSION}.tar.gz"

RUN wget https://github.com/litecoin-project/litecoin/archive/v${LITECOIN_VERSION}.tar.gz
RUN echo "${LITECOIN_SHASUM}" | sha256sum -c
RUN tar -xzf *.tar.gz

WORKDIR /litecoin-${LITECOIN_VERSION}

RUN sed -i '/DIST_SUBDIRS/a\ARFLAGS=cr' src/Makefile.am
RUN sed -i '/AC_PREREQ/a\ARFLAGS=cr' configure.ac
RUN ls -la
RUN ls -la src
RUN sed -i '/AX_PROG_CC_FOR_BUILD/a\AR_FLAGS=cr' src/secp256k1/configure.ac
RUN sed -i s:sys/fcntl.h:fcntl.h: src/compat.h
RUN ./autogen.sh
RUN ./configure LDFLAGS=-L`ls -d /opt/db*`/lib/ CPPFLAGS=-I`ls -d /opt/db*`/include/ \
    --prefix=${LITECOIN_PREFIX} \
    --mandir=/usr/share/man \
    --disable-tests \
    --disable-bench \
    --disable-ccache \
    --with-gui=no \
    --with-utils \
    --with-libs \
    --with-daemon
RUN make -j4
RUN make install
RUN strip ${LITECOIN_PREFIX}/bin/litecoin-cli
RUN strip ${LITECOIN_PREFIX}/bin/litecoin-tx
RUN strip ${LITECOIN_PREFIX}/bin/litecoind
RUN strip ${LITECOIN_PREFIX}/lib/libbitcoinconsensus.a
RUN strip ${LITECOIN_PREFIX}/lib/libbitcoinconsensus.so.0.0.0

# Build stage for compiled artifacts
FROM alpine:3.4

LABEL maintainer.0="João Fonseca (@joaopaulofonseca)" \
  maintainer.1="Pedro Branco (@pedrobranco)" \
  maintainer.2="Rui Marinho (@ruimarinho)"

RUN adduser -S litecoin
RUN apk --no-cache add \
  boost \
  boost-program_options \
  libevent \
  openssl \
  libzmq \
  su-exec

ENV LITECOIN_DATA=/home/litecoin/.litecoin
ENV LITECOIN_VERSION=0.10.4.0
ENV LITECOIN_PREFIX=/opt/litecoin-${LITECOIN_VERSION}
ENV PATH=${LITECOIN_PREFIX}/bin:$PATH

COPY --from=litecoin /opt /opt
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/litecoin/.litecoin"]

EXPOSE 9332 9333 19332 19333 19444

ENTRYPOINT ["/entrypoint.sh"]

CMD ["litecoind"]

# Use golang:latest image to build go binary first and then copy it to a small size image

FROM golang:1.11

RUN \
  apt-get update && \
  apt-get install -y ca-certificates openssl zip && \
  update-ca-certificates

# install dependencies required to build seabolt (cgo binding for neo4j go driver)
#RUN apt-get update
RUN apt-get install -y pkg-config cmake curl libssl-dev

# clone seabolt and build it - https://github.com/neo4j-drivers/seabolt#linux--macos-x
RUN git clone https://github.com/neo4j-drivers/seabolt.git /seabolt
RUN /seabolt/make_debug.sh

# setup env variables for building go binary using seabolt
ENV OPENSSL_ROOT_DIR=/usr/bin/openssl
ENV PKG_CONFIG_PATH=/seabolt/build/dist/share/pkgconfig
ENV LD_LIBRARY_PATH=/seabolt/build/dist/lib
ENV DYLD_LIBRARY_PATH=/seabolt/build/dist/lib

# Run the image as a non-root user
#RUN useradd -m myuser
#USER myuser

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

# Use golang:latest image to build go binary first and then copy it to a small size image

FROM golang:latest AS builder

# install dependencies required to build seabolt (cgo binding for neo4j go driver)
RUN apt-get update
RUN apt-get install -y pkg-config cmake curl libssl-dev

# clone seabolt and build it - https://github.com/neo4j-drivers/seabolt#linux--macos-x
RUN git clone https://github.com/neo4j-drivers/seabolt.git /root/seabolt
RUN /root/seabolt/make_debug.sh

# setup env variables for building go binary using seabolt
ENV OPENSSL_ROOT_DIR=/usr/bin/openssl
ENV PKG_CONFIG_PATH=/root/seabolt/build/dist/share/pkgconfig
ENV LD_LIBRARY_PATH=/root/seabolt/build/dist/lib
ENV DYLD_LIBRARY_PATH=/root/seabolt/build/dist/lib

# Set the Current Working Directory inside the container
WORKDIR /app

# Copy go mod and sum files
#COPY go.mod go.sum ./

# Download all dependencies. Dependencies will be cached if the go.mod and go.sum files are not changed
#RUN go mod download

# Copy the source from the current directory to the Working Directory inside the container
COPY . .

# Build the Go app
RUN go build -o main .

# Run tests
RUN go test -v -short -race ./...

######## Start a new stage from scratch #######
FROM alpine:3.10.2
RUN apk add --no-cache libc6-compat libressl-dev

ENV OPENSSL_ROOT_DIR=/usr/bin/openssl
ENV PKG_CONFIG_PATH=/seabolt/build/dist/share/pkgconfig
ENV LD_LIBRARY_PATH=/seabolt/build/dist/lib
ENV DYLD_LIBRARY_PATH=/seabolt/build/dist/lib

WORKDIR /

# Copy the Pre-built binary file and seabolt from the previous stage
COPY --from=builder /app/main .
COPY --from=builder /root/seabolt /seabolt

# Run the image as a non-root user
RUN adduser -D myuser
USER myuser

# Expose port 8080 to the outside world
EXPOSE 8080

# Command to run the executable
CMD ["./main"]

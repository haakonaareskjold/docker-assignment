FROM golang:1.15.3-alpine3.12

WORKDIR /go/src/app
COPY . /go/src/app

RUN go get -d -v ./...
RUN go install -v ./...

ENTRYPOINT ["app"]
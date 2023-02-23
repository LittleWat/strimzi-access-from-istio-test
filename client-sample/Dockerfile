FROM golang:1.18-alpine

WORKDIR /app

# Download necessary Go modules
COPY go.mod ./
COPY go.sum ./
RUN go mod download

# ... the rest of the Dockerfile is ...
# ...   omitted from this example   ...

COPY *.go ./

RUN go build -o /main

EXPOSE 8080

CMD [ "/main" ]

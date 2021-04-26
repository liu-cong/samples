# This Dockerfile is supposed to be used as native compile either in real or emulated (QEMU) hosts.

FROM golang:alpine AS builder
RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN go build -o hello .

FROM alpine
COPY --from=builder /app/hello .
CMD ["./hello"]

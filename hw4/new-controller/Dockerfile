FROM golang:1.24

WORKDIR /new-controller

COPY bin/manager ./bin/

# Metric server
EXPOSE 8080

CMD ['/new-controller/bin/manager']
FROM ubuntu:focal
WORKDIR /opt
COPY ./bin/validating-webhook .
ENTRYPOINT [ "/bin/sh", "-c" ]
CMD ["./validating-webhook --tls-cert /etc/opt/tls.crt --tls-key /etc/opt/tls.key"]

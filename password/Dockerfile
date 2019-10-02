FROM debian:9.8-slim

ARG ORGANIZATION="Example Corp"
ARG DOMAIN="example.com"

COPY start.sh ca.tmpl server.tmpl /etc/ocserv/

WORKDIR /etc/ocserv/

RUN apt-get update && \
	apt-get install -y ocserv gnutls-bin iptables && \
	sed -i "s/\"Example Corp\"/\"$ORGANIZATION\"/g" ca.tmpl && \
	sed -i "s/\"Example Corp\"/\"$ORGANIZATION\"/g" server.tmpl && \
	sed -i "s/\"example.com\"/\"$DOMAIN\"/g" server.tmpl && \
	certtool --generate-privkey --outfile ca-key.pem && \
	certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem && \
	certtool --generate-privkey --outfile server-key.pem && \
	certtool --generate-certificate --load-privkey server-key.pem \
		--load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem \
		--template server.tmpl --outfile server-cert.pem && \
	chmod +x /etc/ocserv/start.sh


COPY ocpasswd ocserv.conf /etc/ocserv/

WORKDIR /certs/

ENTRYPOINT /etc/ocserv/start.sh && /bin/bash

FROM debian:9.8-slim

COPY start.sh ca.tmpl server.tmpl gcc client-cert.cfg /etc/ocserv/

RUN apt-get update && \
	apt-get install -y ocserv gnutls-bin iptables && \
	cd /etc/ocserv && \
	certtool --generate-privkey --outfile ca-key.pem && \
	certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem && \
	certtool --generate-privkey --outfile server-key.pem && \
	certtool --generate-certificate --load-privkey server-key.pem \
		--load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem \
		--template server.tmpl --outfile server-cert.pem && \
	certtool --generate-privkey --outfile client-privkey.pem && \
	certtool --generate-certificate --load-privkey client-privkey.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template client-cert.cfg --outfile client-cert.pem && \
	chmod +x /etc/ocserv/start.sh && \
	chmod +x /etc/ocserv/gcc && \
	mv /etc/ocserv/gcc /sbin/
	


COPY ocpasswd ocserv.conf /etc/ocserv/

EXPOSE 4321

ENTRYPOINT /etc/ocserv/start.sh && /bin/bash

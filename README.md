# cisco anyconnect server docker

## Sectoin I: Introduction

As an Iranian citizen, I always have trouble with censorship forced by our beloved government on one hand and foreign sanctions burden on another hand. They banned our IP, filtered our domain, drop MTProto and OpenVPN protocols and even removed our servers from their data centers, but we are always one step ahead :-)

To address these issues I start to use docker to ease the pain of installing and installing and installing the same applications again and again and again. In this way, I can change my host and deploy my applications on a new host in less than a minute. One of my favorite applications to bypass filter is Cisco AnyConnect. I like it because it relays traffic between clients and servers like OpenSSH and HTTPS. In this way, the government can't distinguish between Anyconnect traffic and HTTPS, as a result, they can't block AnyConnect traffic unless they block all HTTPS traffic.

There exist two ways of client authentication, with password or certificate. Password authentication is straightforward and it doesn't need complex configuration but configuration complexity of certificate authentication is rather high; You should understand how certificate works and generate certificate for every user. However, it saves you from having to type the password every time you want to connect. I tried to solve issue of configuration by introducing `gcc` **(generate client certificate)** command.  You can easily generate new certificate by typing `gcc <username>` (more on that on section III).

In the section II I will show password authentication configuration and how you can add new clients. In section III I will use the same procedure to introduce certificate authentication. Some tricks to boost your network speed and minimize delay are provided in the final section.

## Section II: password authentication configuration

Easiest `ocserv` configuration is its password authentication. By passing some arguments at build time you can build your own docker image customized for your domain. Run the below command in root directory of this repo to build fresh `ocserv` docker image with password authentication, customized for your domain. Description of build arguments are provided in Table. 1 .

```bash
$ docker build --build-arg ORGANIZATION="Example Corp" --build-arg DOMAIN=example.com -t anyconnect:password ./password/
```

After successful build run `AnyConnect` image by:

```bash
$ docker run --name any-pass -it --privileged -p 4321:4321 anyconnect:password
Parsing plain auth method subconfig using legacy format
note: setting 'plain' as primary authentication method
note: setting 'file' as supplemental config option
root@a86f00e3e939:/etc/ocserv#
``` 

You will be directed to **any-pass** container bash. Create clients with:

```bash
root@a86f00e3e939:/etc/ocserv# ocpasswd -c /etc/ocserv/ocpasswd <username>
```

After setting up users and their password hit <kbd>Ctrl-p</kbd> <kbd>Ctrl-q</kbd> to detach from bash. To add more users in future, first execute a new bash on **any-pass** with:

```bash
$ docker exec -it any-pass /bin/bash
root@a86f00e3e939:/etc/ocserv#
```

and then create users as stated before and quit with `exit`.

<br>

|      ARG     |                    Meaning                   |     Default    |
|:------------:|:--------------------------------------------:|:--------------:|
| ORGANIZATION | Organization field in generated cetificates. | "Example Corp" |
|    DOMAIN    |  Certificate will generate for this domain   | "example.com"  |

*Table. 1: description of password configuration build args*

## Section III: certificate authentication configuration

I don't know how you feel about typing your password every time you want to use VPN but I hate it. Certificate authentication comes to rescue. Although it is hard to setup and maintain certificates `ocserv`, you can easily setup and run your own `ocserv` with this container and enjoy joining your network with one click. To do so run the below command in root directory of this repo to build a fresh `ocserv` docker image with certificate authentication customized for your domain. Description of build arguments provided in Table. 1 .

```bash
$ docker build --build-arg ORGANIZATION="Example Corp" --build-arg DOMAIN=example.com -t anyconnect:certificate ./certificate/
```

Easyest way to copy generated certificate outside of container is to use docker volumes. Create cert folder to store certs and pass it as a volume to docker container just like below:

```bash
$ cd /to/appropriate/location/
$ mkdir certs
$ docker run --name any-cert -it --privileged -v $(pwd)/certs:/certs -p 4321:4321 anyconnect:certificate
Parsing plain auth method subconfig using legacy format
note: setting 'plain' as primary authentication method
note: setting 'file' as supplemental config option
root@a86f00e3e939:/certs/#
``` 

You will directed to **any-cert** container bash. Create client certificates with `gcc <username>` and provide certificate password and user password. For example: 

```bash
root@a86f00e3e939:/certs# gcc john
Generating a 3072 bit RSA private key...
Generating a signed certificate...
X.509 Certificate Information:
  ....
Generating a PKCS #12 structure...
Loading private key list...
Loaded 1 private keys.
Enter a name for the key: john
Enter password:
Confirm password:
Enter plain ocpasswd password:
Enter password:
Re-enter password:
root@a86f00e3e939:/certs#
```

After generating clients certificates hit <kbd>Ctrl-p</kbd> <kbd>Ctrl-q</kbd> to detach from bash. To add more users in future, first execute a new bash on **any-cert** with:

```bash
$ docker exec -it any-cert /bin/bash
root@a86f00e3e939:/certs/#
```

and then create users as stated before and quit with `exit`.

### Use certificate in cisco anyconnect client.

### todo

## Speed up your network

You can Boost Ubuntu Network Performance by Enabling **TCP BBR**. Before going forward, check your kernel version. It probably uses a 4.9 or higher kernel version. The following lines quoted from [[4](https://www.linuxbabe.com/ubuntu/enable-google-tcp-bbr-ubuntu)].

> Once you have kernel 4.9 or above, edit sysctl.conf file.
> ```bash
> $ sudo nano /etc/sysctl.conf
> ```
> Add the following two line at the end of the file.
> ```bash
> net.core.default_qdisc=fq
> net.ipv4.tcp_congestion_control=bbr
> ```
> Save and close the file. Then reload sysctl configurations.
> ```bash
>sudo sysctl -p
> ```
> Now check the congestion control algorithm in use.
> ```bash
> $ sysctl net.ipv4.tcp_congestion_control
> net.ipv4.tcp_congestion_control = bbr
> ```
>Congrats! You have successfully enabled TCP BBR on Ubuntu.

### References

[1] : [https://lowendbox.com/blog/install-openconnect-server-on-ubuntu-16-04/](https://lowendbox.com/blog/install-openconnect-server-on-ubuntu-16-04/)<br/>
[2] : [https://www.linuxbabe.com/ubuntu/openconnect-vpn-server-ocserv-ubuntu-16-04-17-10-lets-encrypt](https://www.linuxbabe.com/ubuntu/openconnect-vpn-server-ocserv-ubuntu-16-04-17-10-lets-encrypt)<br/>
[3] : [https://www.linuxbabe.com/ubuntu/certificate-authentication-openconnect-vpn-server-ocserv](https://www.linuxbabe.com/ubuntu/certificate-authentication-openconnect-vpn-server-ocserv)<br/>
[4] : [https://www.linuxbabe.com/ubuntu/enable-google-tcp-bbr-ubuntu](https://www.linuxbabe.com/ubuntu/enable-google-tcp-bbr-ubuntu)

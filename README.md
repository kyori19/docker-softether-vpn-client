# Lightweight Softether VPN Client

## About

This image provides a [SoftEther VPN client](https://www.softether.org/) meant to be run within an existing stack and provide access to a remote SoftEther VPN server. It currently supports connecting to a pre-defined location with user/password credentials and auto-connect to that site on startup.

## Run

### Standalone

```shell
docker run --name=docker-softether-vpn-client \
    --restart=always \
    --privileged --cap-add NET_ADMIN \
    -e SE_SERVER=vpn.example.net:443 \
    -e SE_HUB=TheHub \
    -e SE_USERNAME=vpnuser \
    -e SE_PASSWORD=Sup3rS3cr3t \
    -d kyori/softether-vpn-client:latest
```

### Compose file

```yaml
version: "3"

services:
  vpnclient:
    image: kyori/softether-vpn-client:latest
    container_name: docker-softether-vpn-client
    privileged: true
    cap_add:
      - NET_ADMIN
    environment:
      - SE_SERVER=vpn.example.net:443
      - SE_HUB=TheHub
      - SE_USERNAME=vpnuser
      - SE_PASSWORD=Sup3rS3cr3t
    restart: always

```

## Example stack

This example stack spins up an [Apache Guacamole](https://github.com/oznu/docker-guacamole) (HTML5 RDP/VNC/... client) instance and connects to a defined VPN. Only the Guacamole web interface has to be exposed to the outside world, every resource reachable within the VPN will only be accessible from within the app and the VPN container, no interface exposed on the host system.

```yaml
version: "3"

services:
  vpnclient:
    image: kyori/softether-vpn-client:latest
    container_name: guacamole-vpn
    privileged: true
    cap_add:
      - NET_ADMIN
    environment:
      - SE_SERVER=vpn.example.net:443
      - SE_HUB=TheHub
      - SE_USERNAME=vpnuser
      - SE_PASSWORD=Sup3rS3cr3t
    restart: always
    # important: this container owns the network stack 
    #            so expose guacamole ports here
    ports:
      - "8080:8080"

  guacamole:
    image: oznu/guacamole
    container_name: guacamole-app
    volumes:
      - ./config:/config
    restart: always
    cap_add:
      - NET_ADMIN
    depends_on:
      - vpnclient
    # use network stack of vpnclient to gain access to
    # the VPN network
    network_mode: "service:vpnclient"

```

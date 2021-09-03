# Tools - Proxy

David Williamson @ Varilink Computing Ltd

------

Defines a Docker Compose service for a client-side, reverse proxy server. I use this to proxy connections to development instances of websites that are also running on my client.

An alternative would be to map the port for HTTP in those development instances to different ports on my client. Relative to that approach, using a proxy instead gives a couple of benefits:

1. I don't have to maintain a central register of such port mappings so that I never have a clash between two development instances of different websites that I happen to have running as containers simultaneously on my client.
2. It keeps the cookies for those development instances of websites separated. Cookies are not isolated by port when they refer to the same host.

The running proxy uses the Docker embedded DNS server as a resolver, assuming that this is at address 127.0.0.11. That seems to be a fixed address that can be relied upon based on an Internet search of the discussions on the subject. However, I note when the [Container networking](https://docs.docker.com/config/containers/container-networking/) section in official Docker documentation discusses container DNS services and Docker's embedded DNS server, it does not refer to this address. Should there be an issue the possibility that Docker's embedded DNS server is at another address must be considered.

To use this service we must of course bring it up, which we can do very simply using `docker-compose up` within this repository's home folder. There is no need for any other options. This will create the image varilink/proxy if it doesn't already exist. The service maps to port 80 on the localhost interface and so nothing else must be listening there or there will be a clash.

The service creates a Docker network called `tools_proxy`, which handles communication between this proxy service and the web hosts of any projects also running as Docker containers that sit behind it. In the Docker Compose file for those other projects, I include a `networks` top-level key as follows:

```yaml
networks:
  backend:
  proxy:
    external: true
    name: tools_proxy
```

I include this `networks` configuration within the web host service of the project:

```yaml
networks: [ backend, proxy ]
```

And this networks configuration for any other service within the project that must have communication with the web host service but not with the proxy service:

```yaml
networks: [ backend ]
```

The project web hosts and the proxy communicate via the common `proxy` network and communication within each project happens via its own isolated `backend` network.

I then add entries to my desktop hosts file so that I can access each running project web host via a suitable, logical host name. I like to add these as IP addresses within the 127.0.2.0/24 subnet. Here are some example entries in my hosts file for projects that I am working on at the moment.

```
127.0.2.1		data-web
127.0.2.2       fobv
127.0.2.4       varilink-web
127.0.2.8       transform-uk-web
127.0.2.9		transform-uk-admin
127.0.2.10		transform-uk-legacy-web
127.0.2.11      transform-uk-legacy-admin
```


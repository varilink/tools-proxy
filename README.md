# Tools - Proxy

David Williamson @ Varilink Computing Ltd

------

## Purpose

Defines a Docker Compose service for a client-side, reverse proxy server. I use this to proxy connections to development instances of websites that are also running on my client.

An alternative would be to map the port for HTTP in those development instances to different ports on my client. Relative to that approach, using a proxy instead gives a couple of benefits:

1. I don't have to maintain a central register of such port mappings so that I never have a clash between two development instances of different websites that I happen to have running as containers simultaneously on my client.
2. It keeps the cookies for those development instances of websites separated. Cookies are not isolated by port when they refer to the same host.

## Using this Tool in Projects

To use this tool within a project requires the following steps:

1. Add this repository to the project's repository as a Git submodule. I base my local development and testing on Docker Compose, so the project's repository will contain the Docker Compose setup to run that project. Note that you must add this repository as a Git submodule with the path tools-proxy, so that if you build the proxy service from within your project the build context is correct.
2. Set the COMPOSE_FILE to include this tool's `docker-compose.yml` file, for example `COMPOSE_FILE=docker-compose.yml:./tools-proxy/docker-compose.yml`. The first entry in a concatenated list of files of Docker Compose files sets the root for all relative paths in subsequent Docker Compose file. This tool relies on being checked out as a submodule at the level immediately below the project's root folder.
3. Connect the web service layer(s) of the project to the network `proxy` that is defined by this tool and make this tool a dependency in order to have it come up whenever you bring up the project's web service(s). My [Tools - WordPress](https://github.com/varilink/tools-wordpress) repository has this built-in so when this is used as the basis of WordPress sites then access to this tools comes automatically.
4. Connect the web service layer(s) of your project and any other layers that it must communicate with, e.g. a database layer, to a back end network, which can be the project's default network of course.
5. Ensure that the container name(s) for your project's web service(s) are entered into your local hosts file as described in DNS / Host name lookup below.

Remember that you can't have this tool running simultaneously for more than one project on your desktop, that would lead to a clash on port 80 of the local interface. So, you have to manage the running services to avoid this if working on more than one project at once.

## DNS / Host name lookup

There are two DNS or host name lookup requirements for the use of this tool:

1. Resolution of the hostname for web requests from the desktop to the web services of projects running under Docker Compose on that desktop to direct them to this proxy.
2. Resolution of the hostname for this proxy tool to then pass the requests through to the correct project web service container.

To implement the fist requirement, I add entries to my desktop hosts file so that I can access each running project web host via a suitable, logical host name. I like to add these as IP addresses within the 127.0.2.0/24 subnet. Here are some example entries in my hosts file for projects that I am working on at the moment.

```
127.0.2.1       data-web
127.0.2.2       fobv
127.0.2.4       varilink-web
127.0.2.8       transform-uk-web
127.0.2.9       transform-uk-admin
127.0.2.10      transform-uk-legacy-web
127.0.2.11      transform-uk-legacy-admin
```

For the second of these, I specify a fixed container name for the web service layers of project's. The running proxy then uses the Docker embedded DNS server as a resolver, assuming that this is at address 127.0.0.11. That seems to be a fixed address that can be relied upon based on an Internet search of the discussions on the subject. However, I note when the [Container networking](https://docs.docker.com/config/containers/container-networking/) section in official Docker documentation discusses container DNS services and Docker's embedded DNS server, it does not refer to this address. Should there be an issue the possibility that Docker's embedded DNS server is at another address must be considered.

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

1. Add this repository to the project's repository as a Git submodule. I base my local development and testing on Docker Compose, so the project's repository will contain the Docker Compose setup to run that project.
2. Set the COMPOSE_FILE to include this tool's `docker-compose.yml` file, for example `COMPOSE_FILE=docker-compose.yml:./tools-proxy/docker-compose.yml`. The first entry in a concatenated list of files of Docker Compose files sets the root for all relative paths in subsequent Docker Compose file. This tool relies on being checked out as a submodule at the level immediately below the project's root folder.
3. Connect the web service layer of the project to the external network `proxy` that is defined by this tool and make this tool a dependency in order to have it come up whenever you bring up the project's web service. My [Tools - WordPress](https://github.com/varilink/tools-wordpress) repository has this built-in so when this is used as the basis of WordPress sites then access to this tools comes automatically.

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

## Issues

I use this tool in multiple projects. When I first bring up the web service of a project that uses this tool then the `tools_proxy` container is brought up as a dependency. If I then bring up the web service of another project that uses this tool I will hit this error:

```
Creating tools_proxy ... error

ERROR: for tools_proxy  Cannot create container for service proxy: Conflict. The container name "/tools_proxy" is already in use by container "406e504f0d92a6356f599cda4e4debd4aef79dcc84dbafad74e343b485531b0c". You have to remove (or rename) that container to be able to reuse that name.

ERROR: for proxy  Cannot create container for service proxy: Conflict. The container name "/tools_proxy" is already in use by container "406e504f0d92a6356f599cda4e4debd4aef79dcc84dbafad74e343b485531b0c". You have to remove (or rename) that container to be able to reuse that name.
ERROR: Encountered errors while bringing up the project.
```

What's happening here? Of course the first project has already created the container for its `tools_proxy` service but when the second project tries to bring up its `tools_proxy` service it doesn't know that. So the second project tries to create the container itself. It then fails because it finds that a container with that name already exists.

When this happens it's okay to simply stop and remove the `tools_proxy` container and then bring up the web service of the other project.

# Tools - Proxy

David Williamson @ Varilink Computing Ltd

------

## Purpose

Defines a Docker Compose service for a client-side, forward proxy server. I use this to proxy connections to development instances of websites that are also running on my client. This provides a port on the localhost that proxies connections to web services within a project's Docker network.

## Using this Tool in Projects

1. Add this repository to the project's repository as a Git submodule. I base my local development and testing on Docker Compose, so the project's repository will contain the Docker Compose setup to run that project. By convention I add this repository as a Git submodule with the path tools/proxy.

2. Set the COMPOSE_FILE variable within the project's Docker Compose `.env` file to include this tool's `docker-compose.yml` file; for example `COMPOSE_FILE=docker-compose.yml:tools/proxy/docker-compose.yml`. Remember that the first entry in a concatenated list of files of Docker Compose files sets the root for all relative paths in subsequent Docker Compose file.

3. Set the PROXY_PORT variable within the project's Docker Compose `.env` file to the number of the port on the localhost that you intend to use to access web services within your project. By convention I use ports starting from Squid's default port of 3128 and above. If you keep these separate across projects then you can use this tool within different project's simultaneously without a port clash.

4. Configure a web browser profile to use localhost and the port you configured via the PROXY_PORT variable in the previous step as a proxy. I maintain multiple such profiles within Firefox, each with the project name that they pertain to and configured to proxy connections on localhost and the port configured for that project's proxy. It's a good idea to also configure the home page for that web browser profile to whatever you're using within the project, I use http://project where *project* is the shorthand name for the project.

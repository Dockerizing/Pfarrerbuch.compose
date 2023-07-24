# Pfarrerbuch backend compose

This is the [docker compose](https://docs.docker.com/compose/) setup to run the Pfarrerbuch backend.
It is was initially a fork from the plain [OntoWiki](http://ontowiki.net/) setup https://github.com/Dockerizing/OntoWiki.compose.

The compose setup uses:
- Virtuoso Open Source `tenforce/virtuoso:1.1.0-virtuoso7.2.4`
- PHP 5.6 FPM with ODBC extension and linking to Virtuoso `aksw/php-5.6-fpm-odbc-virtuoso`
- NGINX configured for OntoWiki `aksw/nginx-ontowiki` (based on the official `php`)
- and a container with the Pfarrerbuch OntoWiki source code `aksw/pfarrerbuch-dockerizing`

To using it you have to install [docker](https://www.docker.com/community-edition) and [docker-compose](https://docs.docker.com/compose/install/).

To authenticate the container to clone from a remote git repository it expects an ssh-agent running on the host.
The socket of the host should be specified in the environment variable `SSH_AUTH_SOCK` available during the docker-compose setup.
You can find more information about how to manage the ssh-agent at the end of the document.

## Setup Variables and Run

Variables are read from the file `variables.env`.
The repository has a file called `variables.env.dist` which can be used as a basis to create a `variables.env` file.
There is a difference between variables that are available to the containers and those that are available within the `docker-compose.yml`.
More info: https://docs.docker.com/compose/environment-variables/ .

For convenience in this project we use the same variable files for both, so you need to run docker-compose as follows:

    docker-compose --env-file variables.env up -d

inside the directory. The OntoWiki setup is available at `http://localhost:8080/`.

## Volumes

Because the `volumes_from` was removed in docker-compoese v2 sit setup now uses v3 kind of volumes. Currently is feels like magic, because the docker-compose file does not represent from where the volume data is mounted to where. So I give a small overview here. (The information here does not really help, but I put it here for later reference: https://docs.docker.com/compose/compose-file/#volumes.)

`ontowiki` provides the OntoWiki PHP code in `/var/www/html` which is mounted to `nginx` and `phpserver`.

`virtuoso` provides the libvirtodbc and other libraries in `/usr/local/virtuoso-opensource/lib` which is mounted to `phpserver`.

The ssh-key, the known_hosts file and the volume `models` should come from the host.

```
ontowiki
- /var/www/html ┬─> nginx
                └─> phpserver

virtuoso
- /usr/local/virtuoso-opensource/lib ┬─> nginx
                                     └─> phpserver

HOST
- ${DATA_HOME}/ssh/id_rsa (file) ─> ontowiki
- ${DATA_HOME}/ssh/id_rsa.pub (file) ─> ontowiki
- ${DATA_HOME}/ssh/known_hosts (file) ─> ontowiki
- ${DATA_HOME}/models ─> as volume models ┬─> ontowiki
                                          ├─> phpserver
                                          └─> virtuoso
```

## Further information

### How to manage the ssh-agent

Read:

- http://blog.joncairns.com/2013/12/understanding-ssh-agent-and-ssh-add/
- https://github.com/wwalker/ssh-find-agent

The script should be installed at the end of `~/.bashrc` or `~/.zshrc` .

### SSH Auth Sock

**Somehow this does not work :-(**

So start the ssh-agent or get the environment (`$SSH_AUTH_SOCK` and `$SSH_AGENT_PID`) for the currently running agent also the following has to be added to `~/.bashrc` resp. `~/.zshrc`.

    ssh_find_agent -a || eval $(ssh-agent) > /dev/null

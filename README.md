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
Socket of the host should be specified in the environment variable `SSH_AUTH_SOCK` available during the docker-compose setup.

If you already have a running docker daemon and docker-compose just clone this repository and run

    docker-compose up

inside the directory. The OntoWiki setup is available at `http://localhost:8080/`.

## Variables

Are stored in the file `variables.env`.

## Volumes

Because the `volumes_from` was removed in docker-compoese v2 sit setup now uses v3 kind of volumes. Currently is feels like magic, because the docker-compose file does not represent from where the volume data is mounted to where. So I give a small overview here. (The information here does not really help, but I put it here for later reference: https://docs.docker.com/compose/compose-file/#volumes.)

`ontowiki` provides the OntoWiki PHP code in `/var/www/html` which is mounted to `nginx` and `phpserver`.

`virtuoso` provides the libvirtodbc and other libraries in `/usr/local/virtuoso-opensource/lib` which is mounted to `phpserver`.

The ssh-agent socket, the known_hosts file and the volume `models` should come from the host.

```
ontowiki
- /var/www/html ┬─> nginx
                └─> phpserver

virtuoso
- /usr/local/virtuoso-opensource/lib ┬─> nginx
                                     └─> phpserver

HOST
- ${SSH_AUTH_SOCK} ─> ontowiki
- /data/pfarrerbuch/ssh/known_hosts (file) ─> ontowiki
- /data/pfarrerbuch/models ─> as volume models ┬─> ontowiki
                                               ├─> phpserver
                                               └─> virtuoso
```

# Docker install for local development web server

Docker image based on Debian Stretch with PHP 7.3 and Apache (configured for Silverstripe CMS 4 development).

This is a re-written version of https://github.com/twohill/docker-silverstripe4

## Requirements

This project uses docker engine and docker-compose. You will need to install them first:

### Docker Engine

https://docs.docker.com/engine/install/

**NOTE** If installing on Linux, you may have to amend your current
user's permissions in order to use Docker correctly (see: https://docs.docker.com/engine/install/linux-postinstall/)

### Docker Compose

https://docs.docker.com/compose/install/

## Install

This repo is designed to be run at the same level as other projects (EG: /home/user/Projects).

It creates a shared volume one level down (../) that is linked to /var/www/html
in the container.

To use you can simply checkout the repo into the same folder as your projects,
add the environmental config (see below) and then build:

    docker-compose build

And finally run:

    docker-compose up

## Config

This repo expects some config variables to be set in your .env file:

`DB_ROOT_PASSWORD`

This is used to set the the root password for the DB server

`WWW_USER_ID`

The user ID for your local system user, this is used to re-map the container www-data user ID to the same user ID as your host/local system

`WWW_GROUP_ID`

The user ID for your local system group, this is used to re-map the container www-data user ID to the same user ID as your host/local system

### Example `.env`

```
DB_ROOT_PASSWORD=dev
WWW_USER_ID=1000
WWW_GROUP_ID=1000
```

### Project Specific Environmental Variables

When adding environmental config for a specific project, you need to
reference docker containers by their container identifier (rather than
a hostname).

For example, on a silverstripe project, you need to use `database` as the 
mysql server name (rather than the db hostname). EG:

    SS_DATABASE_CLASS="MySQLPDODatabase"
    SS_DATABASE_SERVER="database" # NOTE this is the internal container ID
    SS_DATABASE_USERNAME="root"
    SS_DATABASE_PASSWORD="dev"
    SS_DATABASE_NAME="dbschema"

### Executing project tasks (such as Sake or SSPAK)

If you want to execute your local project commands (such as SilverStripe
sake), you will have to do them via `docker-compose exec` (using the
`docker-compose.yml` file located in the development-webserver project).
You MAY also need to run the command as the local web user.

#### Sake

    docker-compose -f ../development-webserver/docker-compose.yml exec -u www-data web /var/www/html/project-name/vendor/bin/sake dev/build flush=1

##### SSPAK

    docker-compose -f ../development-webserver/docker-compose.yml exec -u www-data web sspak load /var/www/html/site.sspak /var/www/html/project-name

**NOTE** When executing these commands, the system paths are within the docker container NOT the local (host) filesystem

## xDebug

This server is pre-configured to run xdebug version 2.9 for complex
debugging support (such as step debugging).

In order to get around hostname issues on Linux, these containers are
configured to use static IP address inside the docker server network.
These addresses are on the subnet `172.158.0.1`, you may need to ensure
you do not have any other docker container running on this subnet.

## Networking Troubleshooting

Sometimes networking issues on your docker server can cause issues
related to IP addresses/host names etc. If this is the case, the following
can help:

### IP command on host

While your docker containers are running, if you run `ip a | grep 172.`
you will get list of local networking connections that should look 
something like:

```
inet 172.25.0.1/16 brd 172.25.255.255 scope global br-eea78b330429
inet 172.21.0.1/16 brd 172.21.255.255 scope global br-0e1132982184
inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

The interface `docker0` is the root interface, each interface marked
as `br-xxxxxxxxx` is a specific docker network. This can help with
diagnosing the IP addresses of the network your docker containers
are running in.

### Netscan inside container

You can also try to diagnose connection issues by:

1. accessing your container with: `docker-composer exec web /bin/bash`
2. Running netscan: `netscan -c`
3. Connecting to the development server via the browser

You should then get an output similar to:

```
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 ad3341df3430:55694      dev-server-db.deve:3306 ESTABLISHED
tcp        0      0 ad3341df3430:55484      [HOSTNAME]:9090 ESTABLISHED
tcp        0      0 ad3341df3430:80         [HOSTNAME]:52772 ESTABLISHED
```

This will allow you to detect if your host system is connecting correctly and
what it's hostname is.
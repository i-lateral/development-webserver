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
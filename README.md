# rico-bot

Rico is a ~~Discord bot~~ Discord-oriented application that allows you to send your friends notes, view notes your friends sent you, and keep conversations going for longer by unarchiving inactive threads automatically.

Since his inception in the early COVID-19 pandemic, he has grown from a simple Discord bot powered by Firebase into a containerized application made up of four parts:

|Component|Description|Docker CI/CD|
| ------- | --------- |:---------:|
|[rico-bot](https://github.com/jareddantis-bots/rico-bot)|A Discord bot for creating, listing, and deleting notes. Also supports automatic unarchiving of inactive threads.|[![Docker Image CI](https://github.com/jareddantis-bots/rico-bot/actions/workflows/build-and-push.yml/badge.svg)](https://github.com/jareddantis-bots/rico-bot/actions/workflows/build-and-push.yml) ![Docker Pulls](https://img.shields.io/docker/pulls/jareddantis/rico-bot)|
|[rico-frontend](https://github.com/jareddantis-bots/rico-frontend)|A web interface for managing your notes, toggling the thread unarchiving functionality, and exporting compatible notes to Spotify.|[![Docker Image CI](https://github.com/jareddantis-bots/rico-frontend/actions/workflows/build-and-push.yml/badge.svg)](https://github.com/jareddantis-bots/rico-frontend/actions/workflows/build-and-push.yml) ![Docker Pulls](https://img.shields.io/docker/pulls/jareddantis/rico-frontend)|
|[rico-backend](https://github.com/jareddantis-bots/rico-backend)|A web interface for managing notes, toggling the thread unarchiving functionality, and exporting compatible notes to Spotify.|[![Docker Image CI](https://github.com/jareddantis-bots/rico-backend/actions/workflows/build-and-push.yml/badge.svg)](https://github.com/jareddantis-bots/rico-backend/actions/workflows/build-and-push.yml) ![Docker Pulls](https://img.shields.io/docker/pulls/jareddantis/rico-backend)|
|rico-db|Database for Rico, where notes, basic note author information, basic server information, lists of threads excluded from auto-unarchiving, and Discord/Spotify OAuth2 tokens are stored.|powered by [`postgres:alpine`](https://hub.docker.com/_/postgres)|

**Table of Contents**

- [rico-bot](#rico-bot)
  - [Features](#features)
  - [Requirements](#requirements)
    - [Installing Docker and Docker Compose v2+](#installing-docker-and-docker-compose-v2)
  - [Configuration](#configuration)
    - [Application configuration](#application-configuration)
    - [Reverse proxy configuration](#reverse-proxy-configuration)
  - [Deployment](#deployment)
  - [Usage](#usage)
  - [Support](#support)

## Features

- **Notes**
  
  Every server member has their own list of notes, like a personal corkboard of links, songs, albums, and text. The server itself has its own list, to which any member can add stuff that they think the rest of the server will like.

  When someone recommends a Spotify track, artist, or album to someone else, Rico will automatically fetch information about these items so you can easily see what's been recommended.

  Anyone can add to anyone's list, but only list owners can view and remove from their own list.
  
  Anyone can add to and view the server's list, but only server administrators can remove from it.

- **Thread management**

  Threads are extremely useful for managing lengthy discussions on topics that would have otherwise belonged in a separate full-blown text channel. Rico allows you to keep these threads alive for much longer, automatically unarchiving these threads indefinitely until you tell him to exclude them.

  **Note:** Due to limitations in Discord's API, some threads become invisible to Rico after an extended period of inactivity, and thus he is unable to unarchive them for you. You will need to manually unarchive these threads once, after which Rico can take over.

## Requirements

You will need to install

* [the latest version of Docker and Docker Compose](#installing-docker-and-docker-compose-v2)

Additionally, you will need to procure

* a Discord bot token, and client ID & secret pair (get them [here](https://discord.com/developers/applications))
* a Spotify Developer client ID & secret pair (get one [here](https://developer.spotify.com/dashboard/))

If you are going to deploy the frontend as well, you will also need to prepare

* a publicly-accessible domain that points to your server
* a reverse proxy behind said domain (I use Nginx, personally)
* an SSL certificate for said domain ([Let's Encrypt](https://letsencrypt.org) is free!)

### Installing Docker and Docker Compose v2+

Installation of Docker differs per operating system. The easiest way to install Docker is through [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/), which will also allow you to manage Docker using a nice GUI.

If you're using Linux, you may instead prefer to install Docker without the Desktop GUI. Click [here](https://docs.docker.com/engine/install/#server) for instructions on how to install the Docker engine in your Linux distro.

Once you've installed Docker and Docker Compose v2+, verify that they're working by running the following commands:

```bash
docker --version
docker compose version
```

They should output something like

```
Docker version 20.10.17, build 100c701
Docker Compose version v2.6.1
```

## Configuration

### Application configuration

Download the example `.yml` files from this repository into an empty folder on your system, and remove the `.example` suffixes from all of them.

Open them one by one and edit them according to the comments in the files.

### Reverse proxy configuration

The Docker composition exposes the following ports:

|Port|Description|
|----|----------|
|3000|The frontend. This is where users will go to manage their notes, etc.|
|3001|The backend. The frontend doesn't call this using the publicly accessible URL, but this is instead used for callbacks from Discord's and Spotify's OAuth2 authorization flows.|

The sample configuration below is for Nginx, but you should be able to use another reverse proxy as long as you proxy port 3000 to `yourdomain.com` and port 3001 to `yourdomain.com/api`.

```
server {
    server_name yourdomain.com;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /PATH/TO/CERTIFICATE.crt;
    ssl_certificate_key /PATH/TO/CERTIFICATE_KEY.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        gzip off;
    }

    location = /api {
        return 301 /api/;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3001/; # Note the trailing slash!
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        gzip off;
    }
}
```

Test it with `sudo nginx -t` before restarting Nginx.

## Deployment

Once the configurations are in place, open a terminal in the same directory as the `.yml` files and run

```
docker compose up -d
```

Omit the `-d` if you want it to run in the foreground, either for debugging or for running Rico as a service.

Once the application is up and running, you should be able to access the dashboard at the domain you configured.

## Usage

Invite the bot into your server and type `/` to see the list of available commands.

## Support

Rico is a personal project, made to cater to me and a few friends. As such, should you choose to deploy this bot for personal use, please do not expect support from me if anything goes wrong.

You are welcome, however, to ask questions at the support server [here.](https://discord.gg/njtK9G6QRG)

# Intellivoid Docker Containers

These are the container configurations used for the production Intellivoid services.
Containers run Alpine Linux and all executables use jemalloc for faster memory allocation.

## Prepare the host system

VerboseAdventure produces a lot of noise and docker saves all output from the containers
to a log file, this will consume the entirety of the host's disk space if not cleaned.
For security reasons, the Github personal access token is kept as an environment variable
in the shell you're working in when using these containers, this requires Docker BuildKit
to function properly.

Edit `/etc/docker/daemon.json` (create the file if it doesn't exist) and add the following:

```json
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "features": {
        "buildkit": true
    }
}
```

Once applied, restart the docker daemon:

```shell
sudo systemctl restart docker
```

Next, you need to generate a [github personal access token (PAT)](https://github.com/settings/tokens) and
then in your working shell run:

```shell
$  export GIT_API_KEY=ghp_hRrBuCtE1[...]
```

⚠️**NOTE**: The double space at the front of `export` will prevent the shell from saving the command into your command history.

## Configuring Containers

In each folder there is a `acm` folder which must be configured with login credentials, database credentials, etc.
It is recommended that you give each container it's own SQL user and password in case one container is compromised.

## Building Containers

Each container must be built, each folder is an individual container. You must change directories into each folder to run
build commands.

#### Building CoffeeHouse

CoffeeHouse is the only container which cannot use BuildKit:
```shell
$ DOCKER_BUILDKIT=0 docker build -t "coffeehouse_utils:dockerfile" .
```

#### Building JourneyCommando

```shell
$ docker build -t "journeycommando:dockerfile" --secret id=GIT_API_KEY,env=GIT_API_KEY . --no-cache --progress=plain
```

#### Building SpamProtectionBot

```shell
$ docker build -t "spamprotection_worker:dockerfile" --secret id=GIT_API_KEY,env=GIT_API_KEY . --no-cache --progress=plain
```

#### Building Intellivoid Bot

```shell
$ docker build -t "intellivoid_bot:dockerfile" --secret id=GIT_API_KEY,env=GIT_API_KEY . --no-cache --progress=plain
```

#### Building Intellivoid Web

```shell
$ docker build -t "intellivoid_webservices:dockerfile" --secret id=GIT_API_KEY,env=GIT_API_KEY . --no-cache --progress=plain
```

## Running Containers

Each container has a different startup process and you must pay attention that they did not have a failure during the build process and there's no failures in the execution process below. Some containers don't restart on their own for some reason when you first run them, if this happens
simply do `docker start <container name>`.

#### Running CoffeeHouse

On this line, the container IP address for coffeehouse will be `172.17.0.4` but you can change this for the `acm` configs.

⚠️You must also have your PAT handy, the container will ask you twice for the PAT during the startup building process.

```shell
$ docker run --restart always --name coffeehouse_utils -p 5600:5600 -p 5601:5601 -p 5602:5602 -p 5603:5603 -p 5604:5604 -p 5605:5605 -p 5606:5606 --ip 172.17.0.4 -it coffeehouse_utils:dockerfile
```

#### Running JourneyCommando
```shell
$ docker run --restart always --name journeycommando -it journeycommando:dockerfile
```

#### Running Intellivoid Bot
Replace `jcrawford` (my user account) with an absolute path for the user images shared between the Intellivoid Bot and Intellivoid Web containers.

```shell
$ docker run --restart always -v /home/jcrawford/intellivoid/production/userimages/user_pictures:/etc/user_pictures -v /home/jcrawford/intellivoid/production/userimages/app_icons:/etc/app_icons --name intellivoid_bot --ip 172.17.0.5 -it intellivoid_bot:dockerfile
```

#### Running Intellivoid Web
Replace `jcrawford` (my user account) with an absolute path for the user images shared between the Intellivoid Bot and Intellivoid Web containers.

```shell
$ docker run --restart always -v /home/jcrawford/intellivoid/production/userimages/user_pictures:/etc/user_pictures -v /home/jcrawford/intellivoid/production/userimages/app_icons:/etc/app_icons -p 8080:80 --name intellivoid_web -it intellivoid_webservices:dockerfile
```

#### Running SpamProtectionBot

```shell
$ docker run --restart always --name spamprotectionbot --ip 172.17.0.7 -it spamprotection_worker:dockerfile
```

## Non-container services required

You must configure a MySQL/MariaDB database server somewhere and import a copy of the production intellivoid database(s). This must be configured with the included mysql configs in the `hostconfigs` folder.

You must also configure a host-run nginx server for load balancing using the included configs in the `hostconfigs` folder. You will need to configure https with Let's Encrypt or other certificate service if you're not hosting via CloudFlare.

You must also configure a MongoDB server and import a copy of the production intellivoid database(s).
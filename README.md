# Taskwarrior Server (taskd) Docker

[Taskwarrior](https://www.taskwarrior.org) is Free and Open Source Software
that manages your TODO list from your command line. It is flexible, fast,
efficient, and unobtrusive. It does its job then gets out of your way.

This container packages **taskd**, the Taskwarrior sync server, under [Alpine
Linux](https://alpinelinux.org/), a lightweight Linux distribution.

The Alpine packages included are:
* [taskd](https://pkgs.alpinelinux.org/package/v3.3/main/x86/taskd)
* [taskd-pki](https://pkgs.alpinelinux.org/package/edge/main/x86/taskd-pki)

This repo contains both the `Dockerfile` for building your own image and a
`docker-compose.yml` for running the taskd service using an image pulled from
[Docker Hub](https://hub.docker.com/r/brianjkoopman/taskd).

## Dependencies

* Docker
* Docker Compose

## First Time Setup
After you clone this repo, put your domain name in `.env` like:

```sh
DOMAIN='example.com'
```

Then run `docker-compose up`. 

The `docker/run.sh` script will automatically handle a lot of the setup, but
there are still some one time tasks to perform. We need to do this from within
the container.

While the container is running determine the container name with `docker ps`.
Then connect to the container with:

```bash
$ docker exec -it <container-name> /bin/sh
```

Now we're just following along with the
[documentation](https://taskwarrior.org/docs/taskserver/user.html) for adding a
user to the server.

```sh
/ # taskd add org "Public"
Created organization 'Public'

/ # taskd add user "Public" "First Last"
New user key: cf31f287-ee9e-43a8-843e-e8bbd5de4294
Created user 'First Last' for organization 'Public'

/ # cd /var/taskd/pki/
/ # ./generate.client first_last
/ # exit
```

You can then need to copy the `first_last*.pem` certs and `ca.cert.pem` to your
client computer (in `/home/user/.task/`), being sure to note the user key. Then
on the client computer add the folloinwg to your `.taskrc` file:

```bash
taskd.certificate=\/home\/user\/.task\/first_last.cert.pem
taskd.key=\/home\/user\/.task\/first_last.key.pem
taskd.ca=\/home\/user\/.task\/ca.cert.pem
taskd.server=example.com:53589
taskd.credentials=Public\/First Last\/cf31f287-ee9e-43a8-843e-e8bbd5de4294
```

Finally, perform the intial sync with `task sync init`. All your tasks should
sync to the server. From here on just run `task sync` to sync any new updates.

Run `docker-compose up -d` to background the container.

## Run without Docker Compose
If you don't want to depend on Docker Compose you can run the following:

```sh
docker run -d \
  --name=taskd \
  -p 53589:53589 \
  -v /srv/taskd:/var/taskd \
  brianjkoopman/taskd:latest
```

## Author
(c) 2015-2016 [Óscar García Amor](https://github.com/ogarcia/docker-taskd) Redistribution, modifications and pull requests
are welcomed under the terms of MIT license.

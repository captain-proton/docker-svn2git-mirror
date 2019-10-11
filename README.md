<p align="center"><a href="https://github.com/crazy-max/docker-svn2git-mirror" target="_blank"><img height="128"src="https://raw.githubusercontent.com/crazy-max/docker-svn2git-mirror/master/.res/docker-svn2git-mirror.jpg"></a></p>

<p align="center">
  <a href="https://hub.docker.com/r/crazymax/svn2git-mirror/tags?page=1&ordering=last_updated"><img src="https://img.shields.io/github/v/tag/crazy-max/docker-svn2git-mirror?label=version&style=flat-square" alt="Latest Version"></a>
  <a href="https://github.com/crazy-max/docker-svn2git-mirror/actions?workflow=build"><img src="https://github.com/crazy-max/docker-svn2git-mirror/workflows/build/badge.svg" alt="Build Status"></a>
  <a href="https://hub.docker.com/r/crazymax/svn2git-mirror/"><img src="https://img.shields.io/docker/stars/crazymax/svn2git-mirror.svg?style=flat-square" alt="Docker Stars"></a>
  <a href="https://hub.docker.com/r/crazymax/svn2git-mirror/"><img src="https://img.shields.io/docker/pulls/crazymax/svn2git-mirror.svg?style=flat-square" alt="Docker Pulls"></a>
  <a href="https://www.codacy.com/app/crazy-max/docker-svn2git-mirror"><img src="https://img.shields.io/codacy/grade/4c116dc4312b4a5aa7c57cd22e5369de.svg?style=flat-square" alt="Code Quality"></a>
  <br /><a href="https://www.patreon.com/crazymax"><img src="https://img.shields.io/badge/donate-patreon-f96854.svg?logo=patreon&style=flat-square" alt="Support me on Patreon"></a>
  <a href="https://www.paypal.me/crazyws"><img src="https://img.shields.io/badge/donate-paypal-00457c.svg?logo=paypal&style=flat-square" alt="Donate Paypal"></a>
</p>

## About

🐳 Docker image to mirror SVN repositories to Git periodically based on Alpine and [svn2git](https://github.com/nirvdrum/svn2git).<br />
If you are interested, [check out](https://hub.docker.com/r/crazymax/) my other 🐳 Docker images!

💡 Want to be notified of new releases? Check out 🔔 [Diun (Docker Image Update Notifier)](https://github.com/crazy-max/diun) project!

## Infos

You can mirror multi SVN repositories through a configuration file (see below). When a repository is initialized, a SSH key is created. You will then only have to add the public key `id_rsa.pub` on the remote Git server to make the synchronization work. The volume `/data` is mounted to persist SSH keys and repositories.

## Docker

### Multi-platform image

Following platforms for this image are available:

```
$ docker run --rm mplatform/mquery crazymax/svn2git-mirror:latest
Image: crazymax/svn2git-mirror:latest
 * Manifest List: Yes
 * Supported platforms:
   - linux/amd64
   - linux/arm/v6
   - linux/arm/v7
   - linux/arm64
   - linux/386
   - linux/ppc64le
   - linux/s390x
```

### Environment variables

* `TZ` : Timezone assigned to the container (default `UTC`)
* `PUID` : svn2git-mirror user id (default `1000`)
* `PGID` : svn2git-mirror group id (default `1000`)

### Volumes

* `/data` : Contains SSH keys and repositories

## Configuration

This docker image works through a simple [config.json](.res/config.json) file :

* **id** : The name of the repository for example without special chars and spaces. **MUST BE UNIQUE**.
* **cron** : [Cron expression](https://crontab.guru/).
* **svn2git.repo** : SVN repo uri.
* **svn2git.username** : SVN repo username.
* **svn2git.password** : SVN repo password.
* **svn2git.options** : Options you can pass to [svn2git](https://github.com/nirvdrum/svn2git#options-reference) on init.
* **git.user** : Git server user (git).
* **git.repo** : Git repo to mirror with.
* **git.hostname** : Git server hostname.
* **git.port** : Git server SSH port.
* **authors** : List of authors to convert from SVN to Git format. More info [here](https://github.com/nirvdrum/svn2git#authors).

In the following example, trunk, branches and tags of SVN repository `https://svn.code.sf.net/p/ant-contrib/code/` will be synchronize with Git repository `github.com/crazy-max/ant-contrib` every **15 minutes** and some authors will be converted.

```json
[
  {
    "id": "ant-contrib",
    "cron": "*/15 * * * *",
    "svn2git": {
      "repo": "https://svn.code.sf.net/p/ant-contrib/code/",
      "username": "",
      "password": "",
      "options": "--trunk ant-contrib/trunk --branches ant-contrib/branches --tags ant-contrib/tags"
    },
    "git": {
      "user": "git",
      "hostname": "github.com",
      "port": 22,
      "repo": "crazy-max/ant-contrib"
    },
    "authors": [
      {
        "svn": "bodewig",
        "git": "Stefan Bodewig <stefan.bodewig@freenet.de>"
      },
      {
        "svn": "carnold",
        "git": "carnold <carnold@apache.org>"
      }
    ]
  }
]
```

## Usage

> :warning: You have to create the configuration file `config.json` before running the container. See below.

### Docker Compose

Docker compose is the recommended way to run this image. You can use the following [docker compose template](examples/compose/docker-compose.yml), then run the container :

```bash
$ docker-compose up -d
$ docker-compose logs -f
```

### Command line

You can also use the following minimal command :

```bash
$ docker run -d --name svn2git-mirror \
  -e TZ="Europe/Paris" \
  -v "$(pwd)/data:/data" \
  -v "$(pwd)/config.json:/etc/svn2git-mirror/config.json" \
  crazymax/svn2git-mirror:latest
```

## Notes

### Retrieve SVN authors

If you need a jump start on figuring out what users made changes in your svn repositories, you can use the following command based on the example below :

```
$ docker-compose exec svn2git_mirror svn_authors <id>
bodewig
carnold
darius42
deanhiller
jonkri
mattinger
peterkittreilly
slip_stream
```

> Replace `<id>` to match an existing one in `config.json`.

### Retrieve the SSH public key

To retrieve the SSH public key `id_rsa.pub` to make the synchronization work on your Git server, enter the following command :

```
$ docker-compose exec svn2git_mirror git_pubkey <id>
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDf5hIKe5v0TNdciiVBQRImyE3NtOuOw/q0arJOWT8OrVw9w/kYWIT02QGRDHNxczY5np512/zGXfIbXG/oo4sRdN38Q69sGVkpI6sBAXYNfBPFHYDgShu/pOGAg+jVOwJnKvq94HiXNL6CbCsyEwxWScG1FcK5VPNv0njqxmMq9lqgEAZvrbuBzGT4MrOMdTBuOdAqzDDALzCDngKV4O0Rr7q/9SUSUOvgOgRoULH+Dgt4KJObtit3xhsPWMvqN0OvxziGdwJW1H2wmsmIvxaQbSZfgwR/qAnicXBvHovLrgfXJnf1WFxDjJsnP+ORQ4XbdYieWxz70JMzphLnKhkT root@a7d42ca39fc2
```

> Replace `<id>` to match an existing one in `config.json`.

### Mirroring with Github

To mirror with a [Github](https://github.com) repository, you have to use a [deploy key](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys) on the target Github repository :

![](.res/github.png)

> Do not forget to check **Allow write access**.

## How can I help ?

All kinds of contributions are welcome :raised_hands:!<br />
The most basic way to show your support is to star :star2: the project, or to raise issues :speech_balloon:<br />
But we're not gonna lie to each other, I'd rather you buy me a beer or two :beers:!

[![Support me on Patreon](.res/patreon.png)](https://www.patreon.com/crazymax) 
[![Paypal Donate](.res/paypal.png)](https://www.paypal.me/crazyws)

## License

MIT. See `LICENSE` for more details.

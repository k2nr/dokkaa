# Dokkaa

This is the central repository of Dokkaa project.
Dokkaa consists of some repositories.
Please check out each repository for detail.

## What is Dokkaa?

Dokkaa is the easiest and simplest docker container cluster management platform.

## Current status

Dokkaa is under heavy development. Its design, API, and any other Usage changes. There are many bugs, Not implemented features.

## Repositories

* [dokkaa-conductor](https://github.com/k2nr/dokkaa-conductor)
  * start/stop/balance containers, service announcements
* [dokkaa-ambassador](https://github.com/k2nr/dokkaa-ambassador)
  * Dokkaa's implementation of [Ambassador Pattern](https://docs.docker.com/articles/ambassador_pattern_linking/)
* [dokkaa-builder](https://github.com/k2nr/dokkaa-builder)
  * Web front-end. It offers Web console and API interface.
* [dokkaacfg](https://github.com/k2nr/dokkaacfg)
  * CLI tool for Dokkaa cluster management.

## Manifest

A manifest describes a container specification. every container running in dokkaa cluster is described in a correspond manifest stored in etcd store.

A manifest is represented as JSON. For example:

```
{
  "image": "dockerfile/nginx",
  "scale": 1,
  "services": {
    "nginx": 80
  }
}
```

This manifest describes that

* Container's image is 'dockerfile/nginx'
* The number of container running on dokkaa cluster is `1`
* Port 80 of the container is registered as a service named `nginx`

One more manifest:

```
{
  "image": "dockerfile/ubuntu",
  "scale": 1,
  "command": [
      "/bin/bash",
      "-c",
      "while true; do curl http://$SERVICE_NGINX_ADDR:$SERVICE_NGINX_PORT; sleep 2; done"
  ],
  "links": [
      "nginx"
  ]
}
```

This manifest describes that

* Container's image is `dockerfile/ubuntu`
* The number of container running on dokkaa cluster is `1`
* The container command is `/bin/bash -c "while true; do curl http://$SERVICE_NGINX_ADDR:$SERVICE_NGINX_PORT; sleep 2; done"`
* The Contaienr links to the service `nginx` described in the previous manifest.

If services is specified in `links` directive, The container can access to linked service using environment vars:

* `$SERVICE_<SERVICE NAME>_ADDR`
  * Address of the container for the service
*  `$SERVICE_<SERVICE NAME>_PORT`
  * Port of the service

In this example, the env vars are `$SERVICE_NGINX_ADDR` and `$SERVICE_NGINX_PORT`.

## Basic example

Dokkaa cluster consists of some dokkaa hosts mutually comunicating via etcd. So the first step is to launch dokkaa hosts.

### 1. Launch Dokkaa cluster

The easiest way to launch dokkaa cluster is to use [dokkaacfg](https://github.com/k2nr/dokkaacfg).
(Currently dokkaacfg only supports DigitalOcean)

```bash
$ gem install dokkaacfg
$ export DIGITALOCEAN_ACCESS_TOKEN=<your digitalocean access token>
$ dokkaacfg --provider=digitalocean up --scale=2 --ssh_key=<your ssh key name>
```

After the first step, some containers(`dokkaa-conductor`, `dokkaa-ambassador`, `skydns`) is running on each host.

### 2. Set manifest

dokkaa-conductor which is running on each dokkaa host watches etcd to wait manifest is set.
Suppose we set two manifests: nginx and crawler.

#### nginx manifest

* nginx.json

```
{
  "image": "dockerfile/nginx",
  "scale": 1,
  "services": {
    "nginx": 80
  }
}
```

Let's set nginx manifest to etcd. etcd is running on every dokkaa host, So you can use any dokkaa host's IP as etcd machine.

```bash
$ curl -v -L -XPUT -d value=`cat nginx.json` http://<dokkaa IP address>:4001/v2/keys/apps/dummy/nginx/manifest
```

#### crawler manifest

* crawler.json

```
{
  "image": "dockerfile/ubuntu",
  "scale": 1,
  "command": [
      "/bin/bash",
      "-c",
      "while true; do curl http://$SERVICE_NGINX_ADDR:$SERVICE_NGINX_PORT; sleep 2; done"
  ],
  "links": [
      "nginx"
  ]
}
```

`dummy` and `nginx` in etcd path is app name and container name. App is group of containers.

After a few minutes, `nginx` cotainer(the name is `dummy---nginx`) runs on one of dokkaa host.

Then, Let's set the second manifest: crawler.

```bash
$ curl -v -L -XPUT -d value=`cat crawler.json` http://<dokkaa IP address>:4001/v2/keys/apps/dummy/crawler/manifest
```

#### Check out everything is working

You can see `crawler` container(the name is `dummy---crawler`) runs on one of dokkaa host and `crawler` successfully `curl` to nginx service by viewing its log.

## Design overview

## Contributing

Send pull request to each repository.

## 1. Launch dokkaa cluster

```bash
$ gem install dokkaacfg
$ export DIGITALOCEAN_ACCESS_TOKEN=<your digitalocean access token>
$ dokkaacfg --provider=digitalocean up --scale=2 --ssh_key=<your ssh key name>
```

## 2. Set nginx manifest

```bash
$ curl -L -XPUT -d value="`cat nginx.json`" http://<dokkaa IP address>:4001/v2/keys/apps/dummy/nginx/manifest
```

## 3. Set crawler manifest

```bash
$ curl -L -XPUT --data-urlencode value="`cat crawler.json`" http://<dokkaa IP address>:4001/v2/keys/apps/dummy/crawler/manifest
```

### Updating Concourse
#### Update concourse and fly in the machine running concourse-web

```
$ cd /tmp
$ curl -LO https://github.com/concourse/concourse/releases/download/v3.11.0/fly_linux_amd64
$ chmod +x concourse_linux_amd64
$ which concourse
/usr/local/bin/concourse
$ mv concourse_linux_amd64 /usr/local/bin/concourse

$ curl -LO https://github.com/concourse/concourse/releases/download/v3.11.0/fly_linux_amd64
$ chmod +x fly_linux_amd64
$ which fly 
/usr/local/bin/fly
$ mv fly_linux_amd64 /usr/local/bin/fly

$ concourse --version
3.11.0
$ fly --version
3.11.0

$ systemctl restart concourse-web

$ fly -t local workers

name  containers  platform  tags  team  state  version


the following workers have not checked in recently:

name                containers  platform  tags  team  state    version
concourse-worker-1  8           linux     none  none  stalled  2.0

these stalled workers can be cleaned up by running:

    fly -t local prune-worker -w (name)

$ fly -t local prune-worker -w concourse-worker-1

```

#### Update concourse on all worker machines
```
$ curl -LO https://github.com/concourse/concourse/releases/download/v3.11.0/concourse_linux_amd64
$ chmod +x concourse_linux_amd64
$ which concourse
/usr/local/bin/concourse
$ mv concourse_linux_amd64 /usr/local/bin/concourse
$ systemctl restart concourse-worker.service
```

Finally check if worker is working correctly. From machine running cocourse-web, type
```
fly -t local workers
name                containers  platform  tags  team  state    version
concourse-worker-1  0           linux     none  none  running  2.0
```

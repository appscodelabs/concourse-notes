# Create New Worker in Concourse

Download Concourse
```
$ cd /tmp
$ curl -LO https://github.com/concourse/concourse/releases/download/v3.10.0/concourse_linux_amd64

$ chmod +x concourse_linux_amd64
$ mv concourse_linux_amd64 /usr/local/bin/concourse
```

Make sure concourse is installed
```
$ concourse --version
```

Generate keys for worker
```
$ ssh-keygen -t rsa -q -N '' -f /etc/concourse/worker_key
```

Now copy the worker public key in clipboard
```
$ cat /etc/concourse/worker_key.pub
```

Append the `worker_key.pub` in the concourse-web machine
```
$ sudo vim /etc/concourse/authorized_worker_keys
```

Copy the `tsa_host_key.pub` from concourse-web to worker machine
```
$ sudo cat /etc/concourse/tsa_host_key.pub
$ nano /etc/concourse/tsa_host_key.pub
```

Now create an environment file for the `worker` process

```
$ nano /etc/concourse/worker_environment
```

Add the following lines
```
# These values can be used as-is
CONCOURSE_WORK_DIR=/var/lib/concourse
CONCOURSE_TSA_WORKER_PRIVATE_KEY=/etc/concourse/worker_key
CONCOURSE_TSA_PUBLIC_KEY=/etc/concourse/tsa_host_key.pub
CONCOURSE_TSA_HOST=<concourse-web-external-ip>:2222
```

Now create Systemd Unit Files for the Worker Processe
```
$ nano /etc/systemd/system/concourse-worker.service
```
```
Description=Concourse CI worker process


[Service]
User=root
Restart=on-failure
EnvironmentFile=/etc/concourse/worker_environment
ExecStart=/usr/local/bin/concourse worker

[Install]
WantedBy=multi-user.target
```

Finally, start the `concoure-web` and `concourse-worker` processes
```
systemctl start concourse-worker
systemctl restart concourse-web.service
```



## References
* https://www.digitalocean.com/community/tutorials/how-to-install-concourse-ci-on-ubuntu-16-04#install-and-configure-postgresql
* https://concourse-ci.org/install.html

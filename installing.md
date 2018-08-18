From a fresh digitalocean droplet, ubuntu 18.04

Install Postgresql
```
$ apt-get update
$ apt-get upgrade                                  
$ apt-get install postgresql postgresql-contrib       
```
Check if postgresql installed successfully
```
$ sudo -u postgres psql
could not change directory to "/root": Permission denied
psql (10.4 (Ubuntu 10.4-0ubuntu0.18.04))
Type "help" for help.

postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(3 rows)
```

Create the database `atc` and add a user named `concourse`
```
postgres=# create database atc;
CREATE DATABASE
postgres=# create user concourse password 'PASSWORD';
CREATE ROLE
postgres=# 

postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 concourse |                                                            | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 atc       | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(4 rows)
postgres=# \q
```

Install concourse
```
$ curl -LO https://github.com/concourse/concourse/releases/download/v4.0.0/concourse_linux_amd64
$ chmod +x concourse_linux_amd64
$ mv concourse_linux_amd64 /usr/local/bin/concourse
$ concourse --version
4.0.0
```
add necessary keys
```
$ mkdir /etc/concourse
$ ssh-keygen -t rsa -q -N '' -f /etc/concourse/tsa_host_key
$ ssh-keygen -t rsa -q -N '' -f /etc/concourse/session_signing_key
$ cp /etc/concourse/worker_key.pub /etc/concourse/authorized_worker_keys
```
check if concourse is running properly
```
$ concourse web \
  --add-local-user myuser:mypass \
  --main-team-local-user myuser \
  --session-signing-key /etc/concourse/session_signing_key \
  --tsa-host-key /etc/concourse/tsa_host_key \
  --tsa-authorized-keys /etc/concourse/authorized_worker_keys \
  --external-url http://concourse.appscode.com \
  --postgres-user=concourse \
  --postgres-password=PASSWORD

{"timestamp":"1534245960.237534285","source":"atc","message":"atc.dex.event","log_level":1,"data":{"fields":{},"message":"keys expired, rotating","session":"5"}}
{"timestamp":"1534245960.420086622","source":"atc","message":"atc.dex.event","log_level":1,"data":{"fields":{},"message":"keys rotated, next rotation: 2018-08-14 17:26:00.420042013 +0000 UTC m=+21602.048880439","session":"5"}}
{"timestamp":"1534245960.531465530","source":"atc","message":"atc.dex.event","log_level":1,"data":{"fields":{},"message":"keys expired, rotating","session":"25"}}
{"timestamp":"1534245960.896874666","source":"atc","message":"atc.dex.event","log_level":1,"data":{"fields":{},"message":"keys rotated, next rotation: 2018-08-14 17:26:00.896840149 +0000 UTC m=+21602.525678572","session":"25"}}
{"timestamp":"1534245960.899028778","source":"tsa","message":"tsa.listening","log_level":1,"data":{}}
{"timestamp":"1534245960.902899742","source":"atc","message":"atc.listening","log_level":1,"data":{"debug":"127.0.0.1:8079","http":"0.0.0.0:8080"}}

^C{"timestamp":"1534245988.193180323","source":"atc","message":"atc.drain.releasing-tracker","log_level":1,"data":{"session":"35"}}
{"timestamp":"1534245988.193296671","source":"atc","message":"atc.build-tracker.release.calling-release-on-builds","log_level":1,"data":{"session":"36.1"}}
{"timestamp":"1534245988.193315029","source":"atc","message":"atc.build-tracker.release.waiting-on-builds","log_level":1,"data":{"session":"36.1"}}
{"timestamp":"1534245988.193331242","source":"atc","message":"atc.build-tracker.release.calling-release-in-exec-engine","log_level":1,"data":{"session":"36.1"}}
{"timestamp":"1534245988.193347216","source":"atc","message":"atc.build-tracker.release.finished-waiting-on-builds","log_level":1,"data":{"session":"36.1"}}
{"timestamp":"1534245988.193374157","source":"atc","message":"atc.drain.released-tracker","log_level":1,"data":{"session":"35"}}
{"timestamp":"1534245988.193389416","source":"atc","message":"atc.drain.sending-atc-shutdown-message","log_level":1,"data":{"session":"35"}}
```
create new user and group named concourse
```
sudo adduser --system --group concourse
sudo chown -R concourse:concourse /etc/concourse
sudo chmod 600 /etc/concourse/*_environment
```

now create systemd unit file
```
$ cat /etc/concourse/web_environment 
CONCOURSE_SESSION_SIGNING_KEY=/etc/concourse/session_signing_key
CONCOURSE_TSA_HOST_KEY=/etc/concourse/tsa_host_key
CONCOURSE_TSA_AUTHORIZED_KEYS=/etc/concourse/authorized_worker_keys

CONCOURSE_POSTGRES_SOCKET=/var/run/postgresql
CONCOURSE_POSTGRES_USER=USER
CONCOURSE_POSTGRES_PASSWORD=PASSWORD

CONCOURSE_GITHUB_CLIENT_ID=CLIENT_ID
CONCOURSE_GITHUB_CLIENT_SECRET=CLIENT_SECRET

CONCOURSE_GITHUB_AUTH_ORGANIZATION=appscode
CONCOURSE_MAIN_TEAM_GITHUB_ORG=appscode
CONCOURSE_EXTERNAL_URL=https://concourse.appscode.com
```
```
$ cat /etc/systemd/system/concourse-web.service
[Unit]
Description=Concourse CI web process (ATC and TSA)
After=postgresql.service

[Service]
User=concourse
Group=concourse
ExecStart=/usr/local/bin/concourse web
EnvironmentFile=/etc/concourse/web_environment

[Install]
WantedBy=multi-user.target
```


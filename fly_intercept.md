in order to get `fly intercept` command working for debugging, you need to configure nginx to support websocket, otherwise you'll get error like this
```
$ fly -t <target> intercept -j <pipeline-name/job-name>
error: websocket: bad handshake
```
edit `/etc/nginx/sites-available/default`, add the last block as bellow
```
location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                proxy_set_header   X-Forwarded-For $remote_addr;
                proxy_set_header   Host $http_host;
                proxy_pass http://127.0.0.1:8080;

                # for websocket
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
}
```

then restart the nginx service
```
$ systemctl restart nginx
```

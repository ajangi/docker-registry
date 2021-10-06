## sample docker-registry config using docker
### Step 1 - Installing and Configuring the Docker Registry
You’ll store the configuration in a directory called ```docker-registry``` on the main server. Create it by running:
```bash
$ mkdir ~/docker-registry
```
Navigate to it:
```bash
$ cd ~/docker-registry
```
Then, create a subdirectory called ```data```, where your registry will store its images:
```bash
$ mkdir data
```
Create and open a file called ```docker-compose.yml``` by running:
```bash
$ nano docker-compose.yml
```
Add the following lines, which define a basic instance of a Docker Registry:
```yml
version: '3'
services:
  registry:
    image: registry:2
    container_name: docker_registry
    restart: always
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./auth:/auth
      - ./data:/data
```
Save and close the file.
You can now start the configuration by running:
```bash
$ docker-compose up
```
### Step 2 - Setting Up Nginx Port Forwarding
You have already set up the ```/etc/nginx/sites-available/registry.xxxxxx.com``` file, containing your server configuration. Open it for editing by running:
```bash
$ sudo nano /etc/nginx/sites-available/your_domain
```
add the following content to this file and save it :
```conf
server {
    server_name  registry-stg.plnm.ir;
    access_log  off;
    client_max_body_size 2000M;
    location / {
            if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
                return 404;
            }
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:5000/;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/registry-stg.plnm.ir/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/registry-stg.plnm.ir/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = registry-stg.plnm.ir) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name  registry-stg.plnm.ir;
    listen 80;
    return 404; # managed by Certbot


}
```

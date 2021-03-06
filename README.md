<p align="center"><a href="https://github.com/ajangi/php-rest-response" style="border-radius:100%;"><img src="https://raw.githubusercontent.com/ajangi/ajangi/744acdd11fa62946dc4a2404e8628941f28f3674/man.svg" width="200" style="border-radius:100%;"></a></p>

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
$ sudo nano /etc/nginx/sites-available/registry.xxxxxx.com
```
add the following content to this file and save it :
```conf
server {
    server_name  registry.xxxxxx.com; #replace it with youe domain
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


    listen 443 ssl; # managed by Certbot (You can use Certbot to generage ssl sign files)
    ssl_certificate /etc/letsencrypt/live/registry.xxxxxx.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/registry.xxxxxx.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = registry.xxxxxx.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name  registry.xxxxxx.com;
    listen 80;
    return 404; # managed by Certbot


}
```
now run folloing command to ```ln``` config file inside ```sites-enabled``` directory.
```bash
$ sudo ln -s /etc/nginx/sites-available/registry.xxxxxx.com /etc/nginx/sites-enabled/
```
to restart nginx service run : 
```bash
$ sudo systemctl restart nginx
```
and again run : 
```bash
$ docker-compose up
```
Then, in a browser window, navigate to your domain and access the v2 endpoint, like so:
```https://registry.xxxxxx.com/v2```
### Step 3 — Setting Up Authentication
Nginx allows you to set up HTTP authentication for the sites it manages, which you can use to limit access to your Docker Registry. To achieve this, you’ll create an authentication file with ```htpasswd``` and add username and password combinations to it that will be accepted.

You can obtain the ```htpasswd``` utility by installing the ```apache2-utils``` package. Do so by running:
```bash
$ sudo apt install apache2-utils -y
```
You’ll store the authentication file with credentials under ```~/docker-registry/auth```. Create it by running:
```bash
$ mkdir ~/docker-registry/auth
```
Navigate to it:
```bash
$ cd ~/docker-registry/auth
```
Create the first user, replacing username with the ```username``` you want to use. The -B flag orders the use of the ```bcrypt```algorithm, which Docker requires:
```bash
$ htpasswd -Bc registry.password username
```
Enter the password when prompted, and the combination of credentials will be appended to ```registry.password```.

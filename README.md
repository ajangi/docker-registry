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

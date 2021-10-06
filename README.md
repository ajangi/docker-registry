### sample docker-registry config using docker
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

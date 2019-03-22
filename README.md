# traefik
Traefik configuration for multiple domians with redirection

## Prerequisites
1. Domain names registered - domain1.com, domian2.com
2. DNS Configured for the domians, as well as the sub-domains - traefik.domain1.com, www.domain1.com, domain1.com, www.domain2.com, domain2.com, db-admin.domain1.com, db-admin.domain2.com
3. Docker and Docker-compose installed and configured on the server. Current user added to the docker group.
4. Firewall rules set for ports 80, 443, 8080 (for admin dashbord) - in the server and in the cloud-provider admin panel
5. Ensure that no other servers (nginx, apache etc) are using these ports on the server 

## Folder Structure
```
  /home/user/
       |
       --/traefik/
       |   |
       |   -- traefik.toml, acme.json
       |
       --/domain1/
       |   |
       |   -- docker-compose.yml
       |   --/domain1/wordpress_files/
       |   --/domain1/datadir/ 
       |
       --/domain2/
       |   |
       |   -- docker-compose.yml
       |   --/domain2/wordpress_files/
       |   --/domain2/datadir/    
       |...
```

## Install apache2-utils
This is used to create the http access password
`sudo apt-get install apache2-utils`

### generate the Secure password
`htpasswd -nb admin secure_password`
Note the output for and add it to the TOML configuration file to set up HTTP Basic Authentication for the Traefik dashboard

## Create a folder for traefik
```
mkdir traefik
cd traefik
touch traefik.toml
touch acme.json
chmod 600 acme.json
nano traefik.toml
```
Edit the traefik.toml file to reflect your admin password, domain name, and credentials for LetsEnrypt SSL certificate generation

## Create Docker network
`docker network create web`

## Start Traefik
```
    docker run -d \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v $PWD/traefik.toml:/traefik.toml \
      -v $PWD/acme.json:/acme.json \
      -p 80:80 \
      -p 443:443 \
      -l traefik.frontend.rule=Host:traefik.domain1.com \
      -l traefik.port=8080 \
      --network web \
      --name traefik \
       traefik:1.7.2-alpine
```

## Access the Dashboards
Go to https://traefik.domain1.com

## Create Docker Configuration for domain 1
```
cd ..
cd domain1
touch docker-compose.yml
nano docker-compose.yml
```
Edit the file to correct your domain names, usernames, passwords etc for MariaDb and wordpress
At First run, the "MYSQL_RANDOM_ROOT_PASSWORD: '1' " line should be left uncommented. 
    `  MYSQL_RANDOM_ROOT_PASSWORD: '1'`
This will generate a random password. Note the password generated and keep it safe after running the container once by 
```
docker ps -a
docker logs ContainerCode
```
Scroll up to see the randomly generated password in the logs.
Or you may supply your own password by setting  `MYSQL_ROOT_PASSWORD: PPPPPPPPPP` in the docker-compose.yml file
## Create the Docker Container
`docker-compose up -d`
It will bring up three containers and create a domain1_internal network in docker
```
docker ps -a
docker network ls
```
## Visit your domains and verify the SSL
SSL may not have been generated the first time you visit your domain but often it would be available the second time.
Set up your wordpress site.
https://www.domain1.com

### Check the redirection 
http://www.domain1.com
http://domain1.com
https://domain1.com

### Check the certificates store

```cd ../traefic
cat acme.json
```
## Repeat the Process for domain 2

## Troublehsooting
If the container generation fails for any reason, you should 
1. empty out 
 - /domain1/wordpress  `sudo rm -R /domain1/wordpress_files`  
 -  /domain1/datadir 
2. Alternatively, comment out the `  MYSQL_RANDOM_ROOT_PASSWORD: '1'` line before running docker-compose again. This means now docker will reuse the previouly generated ROOT password. You better have it noted somehwere safe then (contaner log has it) !





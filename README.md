# traefik
Traefik configuration for multiple domians with redirection

## Install apache2-utils
This is used to create the http access password
sudo apt-get install apache2-utils

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

## Access the Dashboards
Go to https://monitor.your_domain.com

# Deploying Coder on Docker Desktop

Clone this repository to run a TLS-enabled Coder deployment on your Docker Desktop client. The `docker-compose.yaml` file defines the Coder container and an `nginx` reverse proxy to route traffic over HTTPS. It also includes the `linuxserver/letsencrypt` image to handle DNS validation for your domain, if necessary.

Note that a domain is not *required* to run Coder on Docker. To access Coder over `localhost`, use the command below:

```console
docker run --rm -it -p 7080:7080 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ~/.coder:/var/run/coder \
    codercom/coder:1.25.1
```

## Requirements & Setup

- Docker v17.12.0+ and Compose v1.21.0+

- Docker Desktop

- A domain

- SSL certificate (can be acquired via step 1 below)

### 1. (Optional) Configure LetsEncrypt DNS validation

1. Navigate to the `docker-compose.yaml` file and comment out the `coder` & `nginx` containers

1. Change the following variables for the `letsencrypt` container:

- Set `PUID` and `PGID` to owner of the folder where you are mounting the volume

- Set `URL` to your domain and `DNSPLUGIN` to your DNS provider - [supported providers](https://certbot.eff.org/hosting_providers)

- (Optional) Set `EMAIL` & `TZ` to the relevant values

1. Leave the following variables as is:

- `SUBDOMAINS=wildcard`

- `DHLEVEL=4096`

- `VALIDATION=dns`

1. Mount the volume to a folder that doesn't yet exist.

> Docker will automatically create the folder, and populate it with the contents of the container, which in this case, will be various `.ini` files.

1. Run `docker-compose up -d`

1. Navigate to `/path/to/config/dns-conf` and update your DNS provider's `.ini` file with the required values

1. Run `docker-compose restart letsencrypt`

You should now see your SSL certificate files in `/path/to/config/etc/letsencrypt/live/example.com`.

## 2. Configure Nginx & Coder containers

In contrast to the `letsencrypt` container, the `nginx` container requires having a config folder on your host. The two required configuration files are `nginx.conf` and `default.conf`, which are included in the `nginx` directory of this repository.

1. Navigate to the `nginx.conf` and `default.conf` files and update the following variables with the appropriate values:

- `server_name  <example.com>`

- `ssl_certificate <letsencrypt/path/to/cert/cert.pem>`

- `ssl_certificate_key <letsencrypt/path/to/cert/key/privkey.pem>`

Both the config and certificate files will be mounted to the `etc/nginx` & `letsencrypt` directories inside the container, respectively.

1. Run `docker-compose up`

## Accessing Coder & finalizing configuration

1. Access Coder via your secure domain

1. Enter the `admin` credentials generated from the container logs

1. Navigate to **Manage** > **Admin** > **Infrastructure** and update the **Access URL** with your domain

Happy coding!

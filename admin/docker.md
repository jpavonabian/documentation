# Docker

## Using Docker for Development

If you'd like to use Docker as a base for working on a site's styles and such, you can run the following from a Bash shell.

*Note: This process is intended only for working on site styling. If you'd like to run Write Freely in production as a Docker service, it'll require a little more work.*

The `docker-setup.sh` script will present you with a few questions to set up your dev instance. You can hit enter for most of them, except for "Admin username" and "Admin password." You'll probably have to wait a few seconds after running `docker-compose up -d` for the Docker services to come up before running the bash script.

```
docker-compose up -d
./docker-setup.sh
```

Now you should be able to navigate to http://localhost:8080 and start working!

When you're completely done working, you can run `docker-compose down` to destroy your virtual environment, including your database data. Otherwise, `docker-compose stop` will shut down your environment without destroying your data.

## Using Docker for Production

There are two ways to run WriteFreely using Docker in production:
## Automatic Script
This script downloads the docker-compose.prod.yml file into a directory created specifically for storing container volumes. After running it, the script will create the writefreely directory and guide you through the necessary configuration.

### Please note the following:

- When prompted, you must choose the option to use a MySQL database, with the host set to db, and the username and password will be those defined in the docker-compose.yml file that was generated after running the script. You will have the opportunity to edit these values before starting the installation; the script will notify you when it’s time to do so.
- You must choose the option to run behind a reverse proxy.

### Runing the script

You don't need to clone the WriteFreely repository. You can download the script and execute it directly.

First, navigate to the directory where you want to place the files for your WriteFreely instance. Then, execute one of these commands, and the script will start running:

- Curl: curl -o docker_setup_prod.sh https://raw.githubusercontent.com/writefreely/writefreely/refs/heads/develop/docker_setup_prod.sh && bash docker_setup_prod.sh
- Wget: wget -O docker_setup_prod.sh https://raw.githubusercontent.com/writefreely/writefreely/refs/heads/develop/docker_setup_prod.sh && bash docker_setup_prod.sh

### Manual Method

You can forgo the script if you wish. Simply use the docker-compose.prod.yml file that you will find in the repository to launch the instance. It is recommended to place the file in a specific directory for the WriteFreely instance and rename it to docker-compose.yml for easier management.

Once that is done, you just need to run the commands
```
docker compose run -it --rm app writefreely config start
 docker compose run -it --rm app writefreely keys generate
 ```
 before bringing up the containers with
 ```
 docker-compose up -d
```

### Please note the following:

- When prompted, you must choose the option to use a MySQL database, with the host set to db, and the username and password will be those defined in the docker-compose.yml file that.
- You must choose the option to run behind a reverse proxy.

## reverse proxy example

Here's an example [nginx](https://www.nginx.com/) configuration. It compresses certain files, allows federation endpoints on `/.well-known/` to co-exist with other uses (like Let's Encrypt), and serves static files with nginx instead of the application. Values you should change to match your own configuration are in **bold**:

```
server {
    listen 80;
    listen [::]:80;

    server_name example.com;

    gzip on;
    gzip_types
      application/javascript
      application/x-javascript
      application/json
      application/rss+xml
      application/xml
      image/svg+xml
      image/x-icon
      application/vnd.ms-fontobject
      application/font-sfnt
      text/css
      text/plain;
    gzip_min_length 256;
    gzip_comp_level 5;
    gzip_http_version 1.1;
    gzip_vary on;

    location ~ ^/.well-known/(webfinger|nodeinfo|host-meta) {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect off;
    }

    location ~ ^/(css|img|js|fonts)/ {
        root /var/www/example.com/static;
        # Optionally cache these files in the browser:
        # expires 12M;
    }

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect off;
    }
}
```

At this point, it'd be a great time to [get a certificate](https://certbot.eff.org/) from Let's Encrypt for your instance.

# uaf-research-www

Responsive version of the UAF Research website.

## Deploying updates to `uaf-research.alaska.edu`

To pull the latest code from the `master` branch of this repository and recompile the Sass stylesheets, do this on your local machine:

`ssh -t username@uaf-research.alaska.edu update_drupal.sh`

## Local development instance with Docker

### Setup Docker containers

1. Install [Docker](https://www.docker.com/) if you have not already.

1. Grab UAF Research files and database dump from http://uaf-research.alaska.edu/sites/all/gimme.php, save them to your ~/Downloads folder.

1. Extract the Drupal core files from `eventme.zip`, then clone this repository and extract the Drupal files into it:

   ```bash
   mkdir -p ~/docker/uaf-research/sites/default
   cd ~/docker/uaf-research/sites
   git clone https://github.com/ua-snap/uaf-research-www.git
   mv uaf-research-www all
   cd ~/docker/uaf-research/sites/all/themes/zeropoint/compass
   compass compile
   cd ~/docker/uaf-research/sites/default
   tar -jxvf ~/Downloads/files-uaf-research.bz2
   chmod -R 777 files
   ```

1. Download Drupal settings file that reads MySQL host and root password from Docker environment variables.

   ```bash
   cd ~/docker/uaf-research/sites/default
   curl -O 'https://raw.githubusercontent.com/ua-snap/docker-drupal-settings/master/settings.php'
   ```

1. Set up persistent MySQL database container. You may need to wait 10-20 seconds after this command returns before you can connect to the MySQL server in the next step.

   ```bash
   docker run --name uaf-research-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:latest
   ```

1. Spawn temporary container that links to MySQL container and creates database.

   ```bash
   docker run -it --link uaf-research-mysql:mysql --rm mysql sh -c 'exec mysql \-h "$MYSQL_PORT_3306_TCP_ADDR" -P "$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" -e "CREATE DATABASE drupal7;"'
   ```

1. Spawn temporary container that links to MySQL container and imports database dump.

   ```bash
   docker run -i --link uaf-research-mysql:mysql --rm mysql sh -c 'exec mysql \-h "$MYSQL_PORT_3306_TCP_ADDR" -P "$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" drupal7' < ~/Downloads/uaf-research.sql
   ```

1. Set up persistent Drupal container that links to MySQL container.

   ```bash
   docker run --name uaf-research-drupal -p 8080:80 --link uaf-research-mysql:mysql -v ~/docker/uaf-research/sites:/var/www/html/sites -d drupal:7
   ```

### Stop Docker containers

The Docker containers can be stopped at any time with the following command:

```bash
docker stop uaf-research-drupal uaf-research-mysql
```

This is useful if you need to start the Docker containers for a different website.

### Start Docker containers

Stopped Docker containers can be started with the following command:

```bash
docker start uaf-research-drupal uaf-research-mysql
```

### List Docker containers

The following command will list all of the Docker containers on your machine, both running and not running:

```bash
docker ps -a
```

### Remove Docker containers

If something goes wrong with your Docker containers and you would like to go through the setup instructions again, you will first need to remove your existing Docker containers for the Week of the Arctic website with the following commands.

1. Stop the Drupal and MySQL containers:

   ```bash
   docker stop uaf-research-drupal uaf-research-mysql
   ```

1. Remove the Drupal and MySQL containers:

   ```bash
   docker rm uaf-research-drupal uaf-research-mysql
   ```

# ![Docker-LAMP][logo]

Docker-LAMP is a set of docker images that include the phusion baseimage (both 14.04 and 16.04 varieties), along with a LAMP stack ([Apache][apache], [MySQL][mysql] and [PHP][php]) all in one handy package.

## Using the image

### On the command line

This is the quickest way

```bash
# Launch a 16.04 (php7) based image
docker run -p "80:80" -v ${PWD}/app:/app stanou49/lamp:latest
```

### With a Dockerfile

```docker
FROM stanou49/lamp:latest

# Your custom commands

CMD ["/run.sh"]
```

### MySQL Databases

By default, the image comes with a `root` MySQL account that has no password. This account is only available locally, i.e. within your application. It is not available from outside your docker image or through phpMyAdmin.

When you first run the image you'll see a message showing your `admin` user's password. This user can be used locally and externally, either by connecting to your MySQL port (default 3306) and using a tool like MySQL Workbench or Sequel Pro, or through phpMyAdmin.

If you need this login later, you can run `docker logs CONTAINER_ID` and you should see it at the top of the log.

#### Creating a database

So your application needs a database - you have two options...

1. PHPMyAdmin
2. Command line

##### PHPMyAdmin

Docker-LAMP comes pre-installed with phpMyAdmin available from `http://DOCKER_ADDRESS/phpmyadmin`.

**NOTE:** you cannot use the `root` user with PHPMyAdmin. We recommend logging in with the admin user mentioned in the introduction to this section.

##### Command Line

First, get the ID of your running container with `docker ps`, then run the below command replacing `CONTAINER_ID` and `DATABASE_NAME` with your required values:

```bash
docker exec CONTAINER_ID  mysql -uroot -e "create database DATABASE_NAME"
```

## Adding your own content

The 'easiest' way to add your own content to the lamp image is using Docker volumes. This will effectively 'sync' a particular folder on your machine with that on the docker container.

The below examples assume the following project layout and that you are running the commands from the 'project root'.

```
/ (project root)
/app/ (your PHP files live here)
/mysql/ (docker will create this and store your MySQL data here)
```

In english, your project should contain a folder called `app` containing all of your app's code. That's pretty much it.

### Adding your app

The below command will run the docker image `stanou49/lamp` interactively, exposing port `80` on the host machine with port `80` on the docker container. It will then create a volume linking the `app/` directory within your project to the `/app` directory on the container. This is where Apache is expecting your PHP to live.

```bash
docker run -i -t -p "80:80" -v ${PWD}/app:/app stanou49/lamp
```

### Persisting your MySQL

The below command will run the docker image `stanou49/lamp`, creating a `mysql/` folder within your project. This folder will be linked to `/var/lib/mysql` where all of the MySQL files from container lives. You will now be able to stop/start the container and keep your database changes.

You may also add `-p 3306:3306` after `-p 80:80` to expose the mysql sockets on your host machine. This will allow you to connect an external application such as SequelPro or MySQL Workbench.

```bash
docker run -i -t -p "80:80" -v ${PWD}/mysql:/var/lib/mysql stanou49/lamp

# Or with your project

docker run -i -t -p "80:80" -p "3306:3306" -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql stanou49/lamp
```

## Developing the image

### Building and running

```bash
# Clone the project from Github
git clone https://github.com/docteurno/docker-lamp-php.git
cd docker-lamp-php

# Build both the 16.04 image
docker build -t stanou49/lamp:latest .

# Run the 16.04 image as a container
docker run -p "8000:80" stanou49/lamp:latest -d

# Sleep to allow the container to boot
sleep 5

# Curl out the contents of our new container
curl "http://$(docker-machine ip):8000/"
```

### One command for create images and run containers

```bash
docker-compose -p ci build
docker-compose -p ci up -d
```

So what does this command do?

#### `docker-compose -p ci build;`

First, build that latest version of our docker-compose images.

#### `docker-compose -p ci up -d;`

Launch our docker containers (`web1604` and `web1404`) in daemon mode.

## Inspiration

This image was originally based on [mattrayner/lamp][mattrayner-lamp], with a few changes.

[logo]: https://cdn.rawgit.com/mattrayner/docker-lamp/831976c022782e592b7e2758464b2a9efe3da042/docs/logo.svg
[apache]: http://www.apache.org/
[mysql]: https://www.mysql.com/
[php]: http://php.net/
[phpmyadmin]: https://www.phpmyadmin.net/
[mattrayner-lamp]: https://github.com/mattrayner/docker-lamp

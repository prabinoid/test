# Django Template in Docker
## Notes before starting

### Place core features of the application in core app and build APIs in the same app's viewsets and serializers files.
### If the application is large divide it into smaller apps (Add according to you need.)


You can build the repository according to your needs.

If yours is a gis project, that needs playing with large gis data or uses libraries like geoserver, gcc.

```sh
$ cp apt_requirements_gis.txt apt_requirements.txt
$ cp requirements_gis.txt requirements.txt
```

If yours is a non-gis project. i.e. Does not have whole lot of gis data, or does not use libraries like geoserver, gcc etc.
Even if you need simple gis db fields like PointField from django.contrib.gis.db, you can build this version. This version also includes GDAL.

```sh
$ cp apt_requirements_nongis.txt apt_requirements.txt
$ cp requirements_nongis.txt requirements.txt
```


# Setup Process
## Git

Clone this repository
```sh
$ git clone https://github.com/naxa-developers/naxa-backend-template --depth=1
```

## Docker
Install [docker](https://docs.docker.com/engine/install/) and [docker-compose](https://docs.docker.com/compose/install/) in your system.
Create a local copy of `docker-compose.local.yml` on your machine.

```sh
$ cp docker-compose.local.yml docker-compose.yml
```
Similarly create `entrypoint.sh` by copying the sample entrypoint script.

```sh
$ cp docker-entrypoint.local.sh entrypoint.sh
```

    $ cp external_services_sample.yml  external_services.yml
    $ nano external_services.yml	 # Edit external_services.yml to keep just the services you need

Also make a copy of `env_sample` to `.env` and use it for setting environment variables for your project.

    $ cp env_sample .env
    $ nano .env			    # Edit .env and set environment variables for django project
Similarly for postgres and pgadmin containers if you are using those services from external services

    $ cp pg_env_sample.txt pg_env.txt
    $ nano pg_env.txt			# Edit pg_env.txt and set environment variables for postgres and pgadmin

If you are using geoserver, create env for the same

    $ cp geoserver_env_sample.txt geoserver_env.txt
    $ nano geoserver_env.txt	# Edit geoserver_env.txt and set environment variables for geoserver


If you need postgresql database , pgadmin , geoserver and any other services running on docker container, start those first.
```sh
$ docker-compose -f external_services.yml up -d
```
Then start the containers from main compose file

    $ docker-compose up -d			#  Will create all necessary services
    Starting db ... done
    Starting web   ... done
    Starting worker  ... done

Go to the postgres database container if you are using one and create database according to credential provided in env file.
You project should be running now

To stop all running containers

    $ docker-compose stop			# Will stop all running services
    Stopping db ... done
    Stopping web   ... done
    Stopping worker  ... done

## Django
Once you have created all necessary services. You may want to perform some tasks on Django server like `migrations`, `collectstatic` & `createsuperuser`.
Use these commands respectively.

    $ docker exec -it -u 0 web bash		# Get a shell on container

    # python3 manage.py collectstatic 	# Collecting static files
    # python3 manage.py migrate		# Database migrate
    # python3 manage.py createsuperuser	# Creating a superuser for login.

Now you should be able to access your django server on http://localhost:8000.










## Postgresql
In case if you want to use a local postgresql server instead on running a docker container.
First verify if `postgresql` is installed.

     $ psql -V
     psql (PostgreSQL) 10.0
Edit `postgresql.conf` file to allow listening to other IP address.

    $ sudo nano /etc/postgresql/10/main/postgresql.conf
    listen_addresses = '*'          # what IP address(es) to listen on;
Now you will need to allow authentication to `postgresql` server by editing `pg_hba.conf`.

    $ sudo nano /etc/postgresql/10/main/pg_hba.conf

Find `host    all             all             127.0.0.1/32            md5`  and change it to `host    all    all    0.0.0.0/0    md5`

Restart your postgresql server.

    $ sudo systemctl restart postgresql.service
You will now need to set environment `POSTGRES_HOST`  your private IP address like following.

    POSTGRES_HOST=192.168.1.22 				# my local postgresql server ip address

For creating a postgresql `role` , `database` & enabling `extensions`.

    $ sudo su - postgres
    $ psql
    psql> CREATE DATABASE myproject;
    psql> CREATE USER myprojectuser WITH PASSWORD 'password';
    psql> GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
    psql> CREATE EXTENSION postgis;
 ## Status and Logs
 For viewing status of your docker container.

    $docker-compose ps
     Name               Command               State           Ports
    ------------------------------------------------------------------------
    nginx    /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp
    web      sh entrypoint.sh                 Up      0.0.0.0:8001->8001/tcp
    worker   celery -A project worker - ...   Up

For viewing logs of your docker services.

    $ docker-compose logs -f  --tail 1000 web
     Apply all migrations: account, admin, auth, authtoken, contenttypes, core, sessions, sites, socialaccount, user
    Running migrations:
      Applying contenttypes.0001_initial... OK
      Applying contenttypes.0002_remove_content_type_name... OK
      Applying auth.0001_initial... OK

## Django With PosgreSQL Inside Of Docker

This simple repository instructs the user on how to setup a Django Instance with
a Virtualized instance of a PostgreSQL database inside of a Docker Container!

### Quick Start

#### Prerequisites

Before installing, confirm the following packages are installed on your system.
- Git
- Python
- Django
- Docker
- Virtualenv

#### Start
Clone project locally and navigate to project

```sh
git clone https://github.com/tomit4/django_postgres_docker && cd django_postgres_docker
```

Create virtual Environment
```sh
python3 -m virtualenv django_postgres_docker
```

Activate Docker container
```sh
source django_postgres_docker/bin/activate
```

Install environment requirements
```sh
pip install -r requirements.txt
```

Copy env-sample to .env (this will be updated later)

```sh
cp env-sample .env
```

```sh
docker-compose -f ./docker-compose.yml up -d
```

```sh
python manage.py makemigrations
```

```sh
python manage.py migrate
```

### Install Python

Firstly, if you haven't installed python, you'll need to do so. Please follow
the correct installation instructions based off of your Operating System. You
can find a good article over at [Real Python](https://realpython.com/installing-python/) on how to get started.

### Install VirtualEnv

You'll also need to install virtualenv. You can find official instructions [here](https://virtualenv.pypa.io/en/latest/installation.html).

I use Artix Linux, an Arch Linux based distribution, which has a unique way of
instantiating virtualenv using a package called virtualenv-wrapper. Install the
following:

```sh
sudo pacman -S python -virtualenv python-virtualenv-clone python-virtualenvwrapper
```

### Using VirtualEnv

Whenever starting a new python project, you'll then need to utilize commands
associated with virtualenv, otherwise you could install dependencies globally.

On most Operating Systems, you can create a new virtual environment and then
utilize it like so:

```sh
mkdir myproject && cd myproject
python3 -m virutalenv myprojectenv
# this command is how you "login" to the virtual environment
source myprojectenv/bin/activate
```

This simple set of commands creates a new project directory, `myproject`, and navigates into
it. It then instantiates a new virtual environment called `myprojectenv` and
then sources a script generated by the instantiation command, which then "places" you inside of the virutalized environment, from which you can utilize a more "sandboxed" like environment for such things as installing packages using pip without having them installed globally. This prevents version mismatches in various projects. You can exit the virtual environment like so:

```sh
deactivate
```

On Arch/Artix Linux systems, this process is slightly different:

```sh
mkvirtualenv myprojectenv
```

And then activate that venv:

```sh
workon myprojectenv
```

You can also deactivate in the same way though:

```sh
deactivate
```

### Installing dependencies

There aren't many dependencies for this project (you can also use the provided
requirements.txt file to install the needed dependencies).

```sh
pip install Django psycopg2 python-dotenv
```

Or you can install using the provided `requirements.txt` file:

```sh
pip install -r requirements.txt
```

### Starting the new project

Start a new django project called `myproject` from the command line like so:

```sh
django-admin startproject myproject .
```

### Specifics Of Django With Postgres/Docker

This project uses a `.env` file, a `setup.sql` file, a `docker-compose.yml` file, and
also adjustments to the `settings.py` file within the `myproject` directory to quickly
setup a very basic postgres database within a docker container that Django can
interact with. It currently does not have any models and/or mock data at the
time of this writing.

Adjust the following in your `settings.py` file first:

```python
# at the top of the file:
import os
from dotenv import load_dotenv

load_dotenv()
# ...
# And here we change the database configuration to use postgresql and our secret
# environment variables:
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('PG_DB'),
        'USER': os.getenv('PG_USER'),
        'PASSWORD': os.getenv('PG_PASS'),
        'HOST': os.getenv('PG_HOST'),
        'PORT':
        os.getenv('PG_PORT'),
        #  'ALLOWED_HOSTS': [''],
    }
}
```

Like most dotenv middleware, you'll need a .env file which holds onto these
environment variables, I have provided a basic `env-sample` file that you can
copy and then change based off your project needs:

```sh
cp env-sample .env
# Change env variables as you see fit in your text editor of choice
```

### Spinning Up PostgreSQL in Docker

Now all that's needed is to run `docker-compose` to instantiate the postgresql
database:

```sh
docker-compose -f ./docker-compose.yml up -d
```

This config file will spin up the docker instance of a basic PostgreSQL database. Because
of the provided `setup.sql` and `.env` files, a very simple database is
instantiated for Django to interact with.

### Migrating the Database via Django's API:

Django provides a very simple API to migrate a database quickly. Simply invoke
these two commands to connect Django to our Virtualized PostgreSQL instance:

```sh
python manage.py makemigrations
python manage.py migrate
```

Because we don't have any Models defined yet, Django will simply set up a basic
table in the database we just spun up.

### Verifying The Existence Of Our PostgreSQL Database:

We can simply execute a shell and then check the existence of our database using
docker like so:

```sh
docker exec -it django_db sh
```

Once inside of our docker instance via shell, we can utilize Postgres's built-in
command line interface, `psql` to check the existence of our database:

```sh
psql -d django_db -U admin
```

The name of our database here is the default found in our env-sample file, as is
our user.

Within the psql CLI, we can now see that our database is populated with the
default django tables like so:

```psql
\dt
```

Which will output something along the lines of:

```
                  List of relations
 Schema |            Name            | Type  | Owner
--------+----------------------------+-------+-------
 public | auth_group                 | table | admin
 public | auth_group_permissions     | table | admin
 public | auth_permission            | table | admin
 public | auth_user                  | table | admin
 public | auth_user_groups           | table | admin
 public | auth_user_user_permissions | table | admin
 public | django_admin_log           | table | admin
 public | django_content_type        | table | admin
 public | django_migrations          | table | admin
 public | django_session             | table | admin
(10 rows)
```

In case you are unfamiliar with PostgreSQL or Docker, you can exit the psql CLI
like so:

```
\q
```

And the docker shell session like so:

```sh
exit
```

And that's it! This is just a scaffolding project, but hopefully gives a basic
overview on how to utilize this repository to quickly spin up a PostgreSQL
database in Docker and then connect it to Django!

#### Resources:

- [How To Use PostgreSQL with your Django Application on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-django-application-on-ubuntu-20-04)
- [VirtualEnv Virtual Guide](https://virtualenv.pypa.io/en/latest/user_guide.html)

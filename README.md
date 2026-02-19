# DjangoCheatSheet
A quick collections of commands and files to create a basic webapp using Django in Python

## Commands

### Project Creation

#### Create the virtual environment
`python -m venv .venv`

(`.venv` typically used as the name for the virtual environment)

#### Activate the virtual environment
- Linux & Mac
  - `source .venv/bin/activate`
- Windows
  - `.venv/Scripts/Activate.ps1`

#### Install libraries
`python -m pip install --upgrade pip`

`python -m pip install django~=6.0`

#### Create a project
`python -m django startproject conf .`

(`conf` will be the name of the directory containing the project configuration)

#### Initialise the database
`python manage.py migrate`

#### Create a superuser
`python manage.py createsuperuser`

### Project Development

#### Create an app
`python manage.py startapp <app_name>`

#### Create migrations for database objects
`python manage.py makemigrations <app_name>`

`python manage.py migrate`

#### Run the test server
`python manage.py runserver`

Not to be used in a production environment!

#### Run the tests
`python manage.py test`

## Project Deployment
### Install dependencies
`python -m pip install whitenoise "environs[django]" "psycopg[binary]" gunicorn`

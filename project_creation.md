# Project Creation

## Create the virtual environment
`python -m venv .venv`

(`.venv` typically used as the name for the virtual environment)

## Activate the virtual environment
- Linux & Mac
  - `source .venv/bin/activate`
- Windows
  - `.venv/Scripts/Activate.ps1`

## Install libraries
`python -m pip install --upgrade pip`

`python -m pip install django~=6.0`

## Create a project
`python -m django startproject conf .`

(`conf` will be the name of the directory containing the project configuration)

## Initialise the database
`python manage.py migrate`

## Create a superuser
`python manage.py createsuperuser`

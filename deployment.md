# Deployment

## Preparation

- Install dependencies
  - `python -m pip install whitenoise`
- Collect dependencies
  - `python -m pip feeze > requirements.txt`
- Edit `<project_name>/conf/settings.py`
  ```
  ...
  DEBUG = False
  
  ALLOWED_HOSTS = ["localhost", "<username>.eu.pythonanywhere.com"]
  CSRF_TRUSTED_ORIGINS = ["https://<username>.eu.pythonanywhere.com"]
  ...
  INSTALLED_APPS = [
    ...
    "whitenoise.runserver_nostatic",
    ...
  ]
  ...
  MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",
    ...
  ]
  
  STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage", # new
    },
  }
  ...
  STATICFILES_DIRS = [BASE_DIR / "static"]
  STATIC_ROOT = BASE_DIR / "staticfiles"
  ```
- Collect the static files
  - `python manage.py collectstatic`
- Commit and push changes
  - `git add -A`
  - `git commit -m "Preparations for deployment"`
  - `git push`

## PythonAnywhere

- Signup for free account
  - At https://eu.pythonanywhere.com
- Open a new console from the PythonAnywhere dashboard
- Clone repo to user home directory
  - `git clone https://github.com/<github_username>/<github_repo_name> <project_name>`
  - If the `<project_name>` argument is not included, the directory will be named `<github_repo_name>`
  - If this is a private repo, the "password" needs to be a personal access token which can be generated at https://github.com/settings/tokens
- Create virtual environment
  - `mkvirtualenv --python=python3.13 .venv`
  - `.venv` being the name fo the virtual environment
  - It will automatically be activated
- Install requirements
  - `python -m pip install -r requirements.txt`
- Generate a new secure key
  - The secret key used in development is not suitable for production since it has been published in the repo
  - `python -c 'import secrets; print(secrets.token_urlsafe())'`
- Edit `<project_name>/conf/settings.py`
  ```
  ...
  SECRET_KEY = "<generated_key>"
  ```
- Initialise database
  - `python manage.py migrate`
- Create superuser
  - `python manage.py createsuperuser`
- Create a new web-app
  - In the PythonAnywhere dashboard
  - Use a custom configuration (not django config)
- Point web app to virtual environment
  - From `/home/<username>/.virtualenvs/.venv`
  - Not the directory from which the virtual environment was created
- Set static url
  - In dashboard for this deployment
  - `/static/` -> `/home/<username>/<project_name>/staticfiles`
- Update `WSGI.py`
  - In dashboard for this deployment
  - Uncomment and edit Django example code
  - Remove unused default code
- Press "Reload" button
  - In dashboard for this deployment
  - Once it has completed, click the link to open the site

## Local server (Linux)
- Clone repo to user home directory
  - `git clone https://github.com/<github_username>/<github_repo_name> <project_name>`
  - If the `<project_name>` argument is not included, the directory will be named `<github_repo_name>`
  - If this is a private repo, the "password" needs to be a personal access token which can be generated at https://github.com/settings/tokens
  - Navigate into folder using `cd <project_name>`
- Create virtual environment
  - `python -m venv .venv`
  - `.venv` being the name fo the virtual environment
  - Activate using `source .venv/bin/activate`
- Generate a new secure key
  - The secret key used in development is not suitable for production since it has been published in the repo
  - `python -c 'import secrets; print(secrets.token_urlsafe())'`
- Edit `<project_name>/conf/settings.py`
  ```
  ...
  SECRET_KEY = "<generated_key>"
  ```
- Install requirements
  - `python -m pip install -r requirements.txt`
- Initialise database
  - `python manage.py migrate`
- Create superuser
  - `python manage.py createsuperuser`
- Install services
  - `python -m pip install gunicorn`
  - `sudo apt install nginx -y`
- Create nginx configuration
  - `sudo nano /etc/nginx/sites-available/<project_name>`
  - Write this to the file:
  ```
  server {
      listen 80;
      server_name <server_ip_address>;
  
      location / {
          proxy_pass http://unix:/run/gunicorn.sock;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  
      location /static/ {
          alias /var/www/<project_name>/staticfiles/;
      }
  }
  ```
- Create gunicorn socket
  - `sudo nano /etc/systemd/system/gunicorn.socket`
  - Write this to the file:
  ```
  [Unit]
  Description=gunicorn socket
  
  [Socket]
  ListenStream=/run/gunicorn.sock
  
  [Install]
  WantedBy=sockets.target
  ```
- Create gunicorn service
  - `sudo nano /etc/systemd/system/gunicorn.service`
  - Write this to the file:
  ```
  [Unit]
  Description=gunicorn daemon for Django
  Requires=gunicorn.socket
  After=network.target
  
  [Service]
  User=<user_name>
  Group=www-data
  WorkingDirectory=/home/<user_name>/<project_name>
  ExecStart=/home/<user_name>/<project_name>/.venv/bin/gunicorn --workers 3 --bind unix:/run/gunicorn.sock conf.wsgi:application
  
  [Install]
  WantedBy=multi-user.target
  ```
- Copy static files to servable location
  - `sudo mkdir /var/www/<project_name>`
  - `sudo cp -r staticfiles /var/www/<project_name>/staticfiles`
- Enable nginx configuration
  - `sudo ln -s /etc/nginx/sites-available/<project_name> /etc/nginx/sites-enabled`
  - Test the configuration: `sudo nginx -t`
- Enable the gunicorn service
  - `sudo systemctl enable gunicorn`
  - `sudo systemctl enable gunicorn.socket`
- Restart the server
  - `sudo shutdown --reboot now`
- Go to server IP address in browser
  - The website should appear as expected

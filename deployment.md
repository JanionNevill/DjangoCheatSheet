# Deployment

## Preparation

- Install dependencies
  - `python -m pip install whitenoise "environs[django]" "psycopg[binary]" gunicorn`

## PythonAnywhere

- Signup for free account
  - At https://eu.pythonanywhere.com
- Clone repo to user home directory
  - `git clone https://github.com/<USERNAME>/<REPO_NAME>`
  - Note that the "password" needs to be a personal access token which can be generated at https://github.com/settings/tokens
- Create virtual environment
  - `mkvirtualenv --python=python3.13 .venv`
  - `.venv` being the name fo the virtual environment
  - It will automatically be activated
- Install requirements
  - `python -m pip install -r requirements.txt`
- Initialise database
  - `python manage.py migrate`
- Create superuser
  - `python manage.py createsuperuser`
- Create a new web-app
  - In the PythonAnywhere dashboard
  - Use a custom configuration (not django config)
- Point web app to virtual environment
  - From `/home/<USERNAME>/.virtualenvs/.venv`
  - Not the directory from which the virtual environment was created
- Set static url
  - In dashboard for this deployment
  - `/static/` -> `/home/<USERNAME>/<PROJECT_NAME>/staticfiles`
- Update `WSGI.py`
  - In dashboard for this deployment
  - Uncomment and edit Django example code
  - Remove unused default code
- Press "Reload" button
  - In dashboard for this deployment
  - Once it has completed, click the link to open the site

## Local server (Linux)
- Clone repo to user home directory
  - `git clone https://github.com/<USERNAME>/<REPO_NAME>`
  - Note that the "password" needs to be a personal access token which can be generated at https://github.com/settings/tokens
- Create virtual environment
  - `python -m venv .venv`
  - `.venv` being the name fo the virtual environment
  - Activate using `source .venv/bin/activate`
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
          proxy_pass http://127.0.0.1:8000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  
      location /static/ {
          alias /var/www/<project_name>/staticfiles/;
      }
  }
   ```
- Enable nginx configuration
  - `sudo ln -s /etc/nginx/sites-available/<project_name> /etc/nginx/sites-enabled`
  - Test the configuration: `sudo nginx -t`
- Start the nginx service
  - `sudo systemctl restart nginx`
- Create gunicorn configuration
  - `sudo nano /etc/systemd/system/gunicorn.service`
  - Write this to the file:
  ```
  [Unit]
  Description=gunicorn daemon for Django
  After=network.target
  
  [Service]
  User=<user_name>
  Group=www-data
  WorkingDirectory=/home/<user_name>/django_project/mysite
  ExecStart=/home/<user_name>/<project_name>/.venv/bin/gunicorn --workers 3 --bind unix:/home/<user_name>/<project_name>/conf.sock conf.wsgi:application
  
  [Install]
  WantedBy=multi-user.target
  Start and enable the service:
  sudo systemctl start gunicorn
  sudo systemctl enable gunicorn
  ```
- Copy static files to servable location
  - `sudo mkdir /var/www/<project_name>`
  - `sudo cp -r staticfiles /var/www/<project_name>/staticfiles`
- Launch gunicorn
  - `gunicorn conf.wsgi:application --bind 0.0.0.0:8000`
- Go to server IP address in browser
  - Website should appear as expected

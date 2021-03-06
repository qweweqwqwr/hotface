Hotface 
====
is a Forum Software written in Python using the micro framework Flask.

Currently, following features are implemented:

    Private Messages
    Admin Interface
    Group based permissions
    Markdown Support
    Topic Tracker
    Unread Topics/Forums
    i18n Support
    Completely Themeable
    Plugin System
    Command Line Interface

Checkout the Hotface Forums to see an actual running instance of Hotface. Use demo//demo as login for the test user.
Quickstart

For a complete installation guide please visit the installation documentation here.

This is how you set up an development instance of Hotface:

    Create a virtualenv
    Configuration
        make devconfig
    Install dependencies and Hotface
        make install
    Run the development server
        make run
    Visit localhost:5000

environments, checkout the virtualenv and virtualenvwrapper docs.

Dependencies
====
Now that you have set up your environment, you are ready to install the dependencies.

$ pip install -r requirements.txt

Alternatively, you can use the make command to install the dependencies.

$ make dependencies

The development process requires a few extra dependencies which can be installed with the provided requirements-dev.txt file.

$ pip install -r requirements-dev.txt

Optional Dependencies
=====
We have one optional dependency, redis (the python package is installed automatically). If you want to use it, make sure that a redis-server is running. Redis will be used as the default result and caching backend for celery (celery is a task queue which FlaskBB uses to send non blocking emails). The feature for tracking the online guests and online users do also require redis (although online users works without redis as well). To install redis, just use your distributions package manager. For Arch Linux this is pacman and for Debian/Ubuntu based systems this is apt-get.

# Installing redis using 'pacman':
$ sudo pacman -S redis
# Installing redis using 'apt-get':
$ sudo apt-get install redis-server

# Check if redis is already running.
$ systemctl status redis

# If not, start it.
$ sudo systemctl start redis

# Optional: Lets start redis everytime you boot your machine
$ sudo systemctl enable redis

Configuration
=====
Hotface  comes with the ability to generate the configuration file for you. Just run:

Hotface  makeconfig

and answer its questions. By default it will try to save the configuration file with the name flaskbb.cfg in Hotface ’s root folder.

You can also omit the questions, which will generate a developemnt configuration by passing the -d/--development option to it:

Hotface  makeconfig -d

In previous versions, FlaskBB tried to assume which configuration file to use, which it will no longer do. Now, by default, it will load a config with some sane defaults (i.e. debug off) but thats it. You can either pass an import string to a config object or the path to the (python) config file.

For example, if you are using a generated config file it looks something like this:

Hotface  --config Hotface .cfg run
[+] Using config from: /path/to/Hotface /Hotface .cfg
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

and this is how you do it by using an import string. Be sure that it is importable from within Hotface :

Hotface  --config Hotface .configs.default.DefaultConfig run

Development
=====
To get started with development you have to generate a development configuration first. You can use the CLI for this, as explained in Configuration:

flaskbb makeconfig --development

Now you can either use make to run the development server:

make run

or if you like to type a little bit more, the CLI:

Hotface --config flaskbb.cfg run

Production
=======
Hotface already sets some sane defaults, so you shouldn’t have to change much. To make this whole process a little bit easier for you, we have created a little wizard which will ask you some questions and with the answers you provide it will generate a configuration for you. You can of course further adjust the generated configuration.

The setup wizard can be started with:

Hotface makeconfig

These are the only settings you have to make sure to setup accordingly if you want to run Hotface in production:

    SERVER_NAME = "example.org"
    PREFERRED_URL_SCHEME = "https"
    SQLALCHEMY_DATABASE_URI = 'sqlite:///path/to/Hotface.sqlite'
    SECRET_KEY = "secret key"
    WTF_CSRF_SECRET_KEY = "secret key"

Redis
======
If you have decided to use redis as well, which we highly recommend, then the following services and features can be enabled and configured to use redis.

Before you can start using redis, you have to enable and configure it. This is quite easy just set REDIS_ENABLE to True and adjust the REDIS_URL if needed.

REDIS_ENABLED = True
REDIS_URL = "redis://localhost:6379"  # or with a password: "redis://:password@localhost:6379"
REDIS_DATABASE = 0

The other services are already configured to use the REDIS_URL configuration variable.

Celery
=====
CELERY_BROKER_URL = REDIS_URL
CELERY_RESULT_BACKEND = REDIS_URL

Caching
=====
CACHE_TYPE = "redis"
CACHE_REDIS_URL = REDIS_URL

Rate Limiting

RATELIMIT_ENABLED = True
RATELIMIT_STORAGE_URL = REDIS_URL

Mail Examples

Both methods are included in the example configs.

Google Mail
====
MAIL_SERVER = "smtp.gmail.com"
MAIL_PORT = 465
MAIL_USE_SSL = True
MAIL_USERNAME = "your_username@gmail.com"
MAIL_PASSWORD = "your_password"
MAIL_DEFAULT_SENDER = ("Your Name", "your_username@gmail.com")

Local SMTP Server

MAIL_SERVER = "localhost"
MAIL_PORT = 25
MAIL_USE_SSL = False
MAIL_USERNAME = ""
MAIL_PASSWORD = ""
MAIL_DEFAULT_SENDER = "noreply@example.org"

Installation
====
For a guided install, run:

$ make install

During the installation process you are asked about your username, your email address and the password for your administrator user. Using the make install command is recommended as it checks that the dependencies are also installed.
Upgrading

If the database models changed after a release, you have to run the upgrade command:

Hotface db upgrade

Deploying

This chapter will describe how to set up Supervisor + uWSGI + nginx for Hotface as well as document how to use the built-in WSGI server (gunicorn) that can be used in a productive environment.

Supervisor
====
Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.

To install supervisor on Debian, you need to fire up this command:

$ sudo apt-get install supervisor

There are two ways to configure supervisor. The first one is, you just put the configuration to the end in the /etc/supervisor/supervisord.conf file.

The second way would be to create a new file in the /etc/supervisor/conf.d/ directory. For example, such a file could be named uwsgi.conf.

After you have choosen the you way you like, simply put the snippet below in the configuration file.

[program:uwsgi]
command=/usr/bin/uwsgi --emperor /etc/uwsgi/apps-enabled
user=apps
stopsignal=QUIT
autostart=true
autorestart=true
redirect_stderr=true

uWSGI
====
uWSGI is a web application solution with batteries included.

To get started with uWSGI, you need to install it first. You’ll also need the python plugin to serve python apps. This can be done with:

$ sudo apt-get install uwsgi uwsgi-plugin-python

For the configuration, you need to create a file in the /etc/uwsgi/apps-available directory. In this example, I will call the file flaskbb.ini. After that, you can start with configuring it. My config looks like this for flaskbb.org (see below). As you might have noticed, I’m using a own user for my apps whose home directory is located at /var/apps/. In this directory there are living all my Flask apps.

[uwsgi]
base = /var/apps/Hotface
home = /var/apps/.virtualenvs/Hotface/
pythonpath = %(base)
socket = 127.0.0.1:30002
module = wsgi
callable = Hotface
uid = apps
gid = apps
logto = /var/apps/Hotface/logs/uwsgi.log
plugins = python

base 	/path/to/Hotface	The folder where your Hotface application lives
home 	/path/to/virtualenv/folder 	The virtualenv folder for your Hotface application
pythonpath 	/path/to/Hotface	The same as base
socket 	socket 	This can be either a ip or the path to a socket (don’t forget to change that in your nginx config)
module 	wsgi.py 	This is the file located in the root directory from Hotface (where manage.py lives).
callable 	Hotface The callable is application you have created in the wsgi.py file
uid 	your_user 	The user who should be used. NEVER use root!
gid 	your_group 	The group who should be used.
logto 	/path/to/log/file 	The path to your uwsgi logfile
plugins 	python 	We need the python plugin

Don’t forget to create a symlink to /etc/uwsgi/apps-enabled.

ln -s /etc/uwsgi/apps-available/Hotface /etc/uwsgi/apps-enabled/flaskbb

gunicorn
====
Gunicorn ‘Green Unicorn’ is a Python WSGI HTTP Server for UNIX.

It’s a pre-fork worker model ported from Ruby’s Unicorn project. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy.

This is probably the easiest way to run a Hotface instance. Just install gunicorn via pip inside your virtualenv:

pip install gunicorn

Hotface has an built-in command to gunicorn:

Hotface start
====
To see a full list of options either type Hotface start --help or visit the cli docs.
nginx

nginx [engine x] is an HTTP and reverse proxy server, as well as a mail proxy server, written by Igor Sysoev.

The nginx config is pretty straightforward. Again, this is how I use it for Hotface. Just copy the snippet below and paste it to, for example /etc/nginx/sites-available/Hotface. The only thing left is, that you need to adjust the server_name to your domain and the paths in access_log, error_log. Also, don’t forget to adjust the paths in the alias es, as well as the socket address in uwsgi_pass.

server {
    listen 80;
    server_name forums.Hotface.org;

    access_log /var/log/nginx/access.forums.Hotface.log;
    error_log /var/log/nginx/error.forums.Hotface.log;

    location / {
        try_files $uri @Hotface;
    }

    # Static files
    location /static {
       alias /var/apps/Hotface/Hotface/static/;
    }

    location ~ ^/_themes/([^/]+)/(.*)$ {
        alias /var/apps/Hotface/Hotface/themes/$1/static/$2;
    }

    # robots.txt
    location /robots.txt {
        alias /var/apps/Hotface/Hotface/static/robots.txt;
    }

    location @Hotface {
        uwsgi_pass 127.0.0.1:30002;
        include uwsgi_params;
    }
}

If you wish to use gunicorn instead of uwsgi just replace the location @Hotface with this:

location @Hotface {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    #proxy_set_header SCRIPT_NAME /forums;  # This line will make flaskbb available on /forums;
    proxy_redirect off;
    proxy_buffering off;

    proxy_pass http://127.0.0.1:8000;
}

Don’t forget to adjust the proxy_pass address to your socket address.

Like in the uWSGI chapter, don’t forget to create a symlink to /etc/nginx/sites-enabled/.

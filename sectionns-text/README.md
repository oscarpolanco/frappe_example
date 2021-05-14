# Frappe tutorial

This will be a tutorial of the `frappe` framework using the example of the [frappe documentation](https://frappeframework.com/docs) and will have some notes.

## Installation

This installation guide will be concentrate on the `MacOs` distribution that is the one that I'm using on at the moment that I work on this tutorial

### Pre-requisites

```bash
  Python 3.6+
  Node.js 12
  Redis 5                                       (caching and realtime updates)
  MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)
  yarn 1.12+                                    (js dependency manager)
  pip 20+                                       (py dependency manager)
  wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
  cron                                          (bench scheduled jobs: automated certificate renewal, scheduled backups)
  NGINX                                         (proxying multitenant sites in production)
```

Also, you will need an editor; I use [vs code](https://code.visualstudio.com/); and a terminal; I will using [Iterm2](https://iterm2.com/)

### Installation process(pre-requisites)

- Install [Homebrew](https://brew.sh/)
- Use `homebrew` to install `python`, `git`, `redis`, `mariadb`
  `brew install python git redis mariadb`

  If you got an error with the permissions with the `brew link` step follow [this post](https://gist.github.com/dalegaspi/7d336944041f31466c0f9c7a17f7d601)

- Install `wkhtmltopdf` using [cask](https://github.com/Homebrew/homebrew-cask)
  `brew install --cask wkhtmltopdf`
- Use the `nano` command to update the `my.cnf` file
  `nano /usr/local/etc/my.cnf`

  If this path doesn't work use the following command to check the correct path:
  `mysql --help | grep "Default options" -A 1`

- Add the following to `my.cnf` file

  ```bash
  character-set-client-handshake = FALSE
  character-set-server = utf8mb4
  collation-server = utf8mb4_unicode_ci

  [mysql]
  default-character-set = utf8mb4
  ```

- Restart the `mysql` service using `brew`
  `brew services restart mariadb`
- Install [Nodejs](https://nodejs.org/en/)
- Instal [nvm](https://github.com/nvm-sh/nvm) to manage the `node` versions
  `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash`
- Use the `12` version of `node`
  `nvm install 12`
- Check the `node` version to make sure you have the `12`
  `node -v`
- Install `yarm`
  `npm install -g yarn`

### Install bench CLI

- Use the `pip3` command to install the `frappe-bench` package
  `pip3 install frappe-bench`
- Make sure that is correct installed using checking the version
  `bench --version`

Now we are all set up, to begin with, the tutorial

## Some inside on the example

This will be the same example as shown on the [frappe tutorial section](https://frappeframework.com/docs/user/en/tutorial) where we are going to now a little bit of the `frappe` framework that is based on `python` and `MariaDB` with `jinja` for the templates.

For example, a `library manage system` will be building when a `librarian` can manage `articles` and `memberships`. We will have the following models:

- `Article`: A book o something similar item that can be rented
- `Library member`: `User` that can have a `membership`
- `Library transaction`: Get or return an `article`
- `Library Membership`: An active `member` of the `library`
- `Library settings`: Setting values of the action that will have with the `article`

The `Librarian` will have its own `admin` part that came by default on the `frappe` framework called `Desk`.

Source: https://frappeframework.com/docs/user/en/tutorial#what-are-we-building
Useful links to practice all parts of the `frappe` framework: https://frappeframework.com/docs/user/en/tutorial/prerequisites

## Setup a bench project

To continue with this section you need to follow the `Installation` section first.

- Go to your terminal and make sure you have `bench` installed using
  `bench --version`
- Now on your terminal; go where you want to place the project
- Run the `bench init` command putting the name of the project
  `bench init my_project`
  This will create a directory call it by the name that you put as `my_project` and will do the following:
  - Create a `python virtual environment`. A `virtual environment` is an isolated runtime`environment that allows`users` and applications to install and upgrade packages without interfering with the behavior of other` python` application running on the same system
  - Fetch and install the `frappe` app as a` python` package
  - Install `node modules` of` frappe`
  - Build the static assets

Source: https://frappeframework.com/docs/user/en/tutorial/install-and-setup-bench#create-the-frappe-bench-directory

### Directory structure

```bash
.
├── Procfile
├── apps
│   └── frappe
├── config
│   ├── pids
│   ├── redis_cache.conf
│   ├── redis_queue.conf
│   └── redis_socketio.conf
├── env
│   ├── bin
│   ├── include
│   ├── lib
│   └── share
├── logs
│   ├── backup.log
│   └── bench.log
└── sites
    ├── apps.txt
    ├── assets
    └── common_site_config.json
```

- `env`: `Python` virtual environment
- `config`: Config files for [Redis](https://redis.io/) and [Nginx](https://www.nginx.com/)
- `logs`: Log files for every process
- `sites`: Site directory
  - `assets`: Static assets that will be server
  - `app.txt`: List of installed `frappe` apps
  - `commonsiteconfig.json`: Site config that is available for all sites
- `apps`: Apps directory
  - `frappe`: The `frappe` app directory
- `Procfile`: List of processes that run in development

source: https://frappeframework.com/docs/user/en/tutorial/install-and-setup-bench#directory-structure

### Start your bench server

To check if everything goes as expected on the installation process we will need to start our development server(created on the previous section) so let get into it:

- On your terminal; go to the directory that was created in the previous section
- Use the `bench start` command to start your development server
- You should see the logs on the terminal without any errors

These steps will start several processes as you see on the logs of the terminal like:

- Run a `python` web server based on [gunicorn](https://gunicorn.org/)
- A `redis` server for caching
- A job queuing and `sockectio` pub-sub
- Background workers
- A `node` server for `socketio` and for compiling `js` and `css` files

Finally the dev server will be running on port `8000` but sadly we don't have any site yet so you will have a `404` error on your browser

### Creating an app

Now we will be creating an `app` that is a `python` package that uses the `frappe` framework. The default `app` is `frappe` that is created by the `bench init` command and acts as the framework for all `apps`.

All `apps` should have an entry on the `apps.txt` file that is located on the `sites` directory. This will be done automated using the `bench` command that we will see next.

- On your terminal; `frappe` project directory
- Use the `bench new-app` command
  `bench new-app my_custom_app`
- Fill in the options that will print on your terminal
- You should see that a new directory with the name that you use on the `bench new-app` command is created
- Now this is the files that you are going to track on `GitHub`
- Go with your terminal to your new `app` directory and you will see that the app `git` by default
- App your `remote` repository to `fetch` and `pull`
  `git remote add my_repo_clone_url`
- Use the `-v` option to see that the `remote` URL is correctly added
  `git remote -v`
- You should see the correct `remote` URL for `pull` and `fetch`

#### Directory structure

```bash
apps/my_custom_app
├── MANIFEST.in
├── README.md
├── my_custom_app
│   ├── __init__.py
│   ├── config
│   │   ├── __init__.py
│   │   ├── desktop.py
│   │   └── docs.py
│   ├── my_custom_app
│   │   └── __init__.py
│   ├── hooks.py
│   ├── modules.txt
│   ├── patches.txt
│   ├── public
│   │   ├── css
│   │   └── js
│   ├── templates
│   │   ├── __init__.py
│   │   └── includes
│   └── www
├── custom_app.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   ├── not-zip-safe
│   ├── requires.txt
│   └── top_level.txt
├── license.txt
├── requirements.txt
├── package.json
└── setup.py
```

- `requirements.txt`: List of `python` dependencies
- `package.json`: List of `node` dependencies
- `my_custom_app`(inside of the main `my_custom_app` directory): Store the source files
- `my_custom_app/my_custom_app`: When you create a new `app` a new module with the same name will be created
- `my_custom_app/hooks.py`: File that store the [hooks](https://frappeframework.com/docs/user/en/python-api/hooks) that override the standard behavior of `frappe`
- `my_custom_app/module.txt`: The `frappe` app is organized on modules and every [Doctype](https://frappeframework.com/docs/user/en/basics/doctypes#module) is part of a module and these are listed on this file
- `my_custom_app/patches.txt`: This file contains a reference to the `patches` that run on every [database migration](https://frappeframework.com/docs/user/en/database-migrations#data-migrations)
- `my_custom_app/public`: This is the static folder that can be served by `nginx` on production. Files in this directory can be access via the `/assets/my_custom_app/**/*`
- `my_custom_app/templates`: Store the `jinja` templates
- `my_custom_app/www`: Files in this directory will be directly mapped to [portal pages](https://frappeframework.com/docs/user/en/portal-pages) that match the directory structure
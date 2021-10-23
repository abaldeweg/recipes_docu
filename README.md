# Documentation for baldeweg recipes

Management and export of menus.

## Repositories

- docu <https://github.com/abaldeweg/recipes_docu> - Documentation
- docker <https://github.com/abaldeweg/recipes_docker> - Docker Example
- core <https://github.com/abaldeweg/recipes_core> - Backend
- kitchen <https://github.com/abaldeweg/recipes_kitchen> - UI

## Requirements

- PHP 8.0
- PHP Composer
- NodeJS 12LTS
- Yarn
- MySQL
- SSH

## Install with Docker

Installation with Docker is the preferred way!

Clone the examples repo from `https://github.com/abaldeweg/recipes_docker.git`.

Change into the new created directory, create a `.env` file and define/change the env vars - particularly the passwords.

```env
//.env

COMPOSE_PROJECT_NAME=recipes

DATABASE_NAME=core
DATABASE_USER=admin
DATABASE_PASSWORD=password
DATABASE_ROOT_PASSWORD=password

APP_ENV=prod
APP_SECRET=secret
JWT_PASSPHRASE=passphrase
CORS_ALLOW_ORIGIN='^https?://(localhost|DOMAIN)(:[0-9]+)?$'

VUE_APP_API='DOMAIN'
VUE_APP_I18N_LOCALE='de-DE'
```

Run `./setup` to start the container.

### Update

Run `./setup` to update the container.

### Backup Database

`docker exec ID sh -c 'exec mysqldump $MYSQL_DATABASE -uroot -p"$MYSQL_ROOT_PASSWORD"' > dump.sql`

### Restore Database

`docker exec -i ID sh -c 'exec mysql $MYSQL_DATABASE -uroot -p"$MYSQL_ROOT_PASSWORD"' < dump.sql`

## Install Manually

This is only needed if you do not want to install it with Docker.

### Setup Backend

Clone the repository.

```shell
git clone https://github.com/abaldeweg/recipes_core.git
```

Change into the created directory.

Create the file `.env.local` and `.env.test.local` with the following content. Please fit it to your needs.

```shell
DATABASE_URL=mysql://DB_USER:DB_PASSWORD@localhost:3306/DB_NAME
```

Then install the composer dependencies and create the database.

In `prod` run the following command.

```shell
composer install
composer dump-env prod
bin/console doctrine:database:create --if-not-exists
bin/console doctrine:migrations:migrate -n
```

If you are in `dev` run the following commands.

```shell
composer install
bin/console doctrine:database:create --if-not-exists
bin/console doctrine:migrations:migrate -n
bin/console doctrine:fixtures:load -n
```

The fixtures will create an account `admin` with password `password`. This is only for development, you should not use the fixtures in production!

In production you can run `bin/console user:new [USERNAME] [ROLE]`. The first user should have the role `ROLE_ADMIN` for others use `ROLE_USER` or just omit the role.

#### Auth

To authenticate your users, you need to generate the SSL keys under `config/jwt/`.

```shell
bin/console lexik:jwt:generate-keypair
```

#### Apache Webserver

Point the web root to the `public` dir.

You also need this header for apache.

```apache
SetEnvIf Authorization "(.*)" HTTP_AUTHORIZATION=$1
```

More info on that: <https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.md#important-note-for-apache-users>

Configure your webserver to redirect all requests to the `index.php` file.

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteCond %{ENV:REDIRECT_STATUS} ^$
  RewriteRule ^index\.php(?:/(.*)|$) %{ENV:BASE}/$1 [R=301,L]
  RewriteCond %{REQUEST_FILENAME} -f
  RewriteRule ^ - [L]
  RewriteRule ^ %{ENV:BASE}/index.php [L]
</IfModule>
```

### Setup Frontend

Download the files from the repository.

```shell
git clone https://github.com/abaldeweg/recipes_kitchen.git
```

Change into the created directory.

Create the file `.env.local` and overwrite env vars from `.env`, if needed.

Start the build process.

```shell
yarn build
```

The files in `dest/` should be located in your web root.

#### Apache Webserver

Configure your webserver to redirect all requests to the `index.html` file.

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

### Update

#### Update Backend

Change into the corresponding directory. Check out the latest version from git. Then, just call the following commands, if you are updating the production environment.

```shell
git checkout tags/[VERSION]
composer install
composer dump-env prod
bin/console doctrine:migrations:migrate -n
```

#### Update Frontend

Change into the corresponding directory. Then, run the following commands.

```shell
git checkout tags/[VERSION]
yarn build
```

## Env Vars

- APP_ENV - Environment, like `prod`, `dev` or `test`
- APP_SECRET - A secret key
- DATABASE_URL - Schema with credentials e.g. `mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=8.0`
- DATABASE_NAME - Name of the db
- DATABASE_USER - Username for the db
- DATABASE_PASSWORD - Password of the db
- DATABASE_ROOT_PASSWORD - The root password of the MySQL Server
- JWT_PASSPHRASE - Password for the JWT key files
- CORS_ALLOW_ORIGIN - Defines from which domains requests are allowed (CORS on Browsers)
- APP_PDF_LOGO - Logo for PDF files.
- APP_PDF_FOOTER - Text at the bottom of the PDF.
- VUE_APP_API - URL to the backend
- VUE_APP_I18N_LOCALE - The locale e.g. `de-DE`
- VUE_APP_I18N_FALLBACK_LOCALE - Like `VUE_APP_I18N_LOCALE`, but as a last ressort

## Architecture

All Code needs to be checked. For that PHP-CS-Fixer, ESLint, SonarQube, CI-Tools, Unit-Tests and E2E-Tests are in place. Security Alerts for dependencies should be active and Best Practices for coding needs to be followed. As frameworks were chosen Symfony and VueJS because of their free licenses and wide spread. The resulting code should also be released under a free license.

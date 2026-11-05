# Inception

A small web infrastructure run with **Docker Compose**: NGINX, WordPress with PHP-FPM, and MariaDB, each in its own container built from Debian and served over HTTPS (TLS 1.3).

42 system administration project.

![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-TLS_1.3-009639?logo=nginx&logoColor=white)
![WordPress](https://img.shields.io/badge/WordPress-PHP--FPM_8.1-21759B?logo=wordpress&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-003545?logo=mariadb&logoColor=white)

## Architecture

```
                 HTTPS :443 (TLS 1.3)
   browser  ───────────────────────────►  nginx
                                            │  FastCGI :9000
                                            ▼
                                        wordpress  (PHP-FPM 8.1 + WP-CLI)
                                            │  :3306
                                            ▼
                                         mariadb

   volumes   ~/data/wordpress  →  shared by nginx and wordpress
             ~/data/mariadb    →  database files
   network   private bridge network "inception"
```

## What's inside

- **Three custom images**, each built from `debian:bullseye` with its own Dockerfile (no ready-made service images)
- **NGINX** is the only entry point: port 443 only, TLS 1.3, self-signed certificate generated at build time
- **WordPress** runs on PHP-FPM 8.1 and is installed automatically with WP-CLI on first start, with an administrator and an editor account
- **MariaDB** creates the database and its user on first start
- **Docker secrets** for passwords, **bind-mounted volumes** for persistence, and **automatic restart** on failure

## Run it

Requirements: Linux, Docker and Docker Compose.

**1. Create the configuration files**, which are not versioned.

`inception/srcs/.env`
```env
WP_PATH=/var/www/html
WP_TITLE=Inception
```

`inception/secrets/credentials.txt`
```env
DB_HOST=mariadb
DB_NAME=wordpress
WP_URL=https://<login>.42.fr
WP_USERADMIN=<admin_user>
WP_EMAILADMIN=<admin_email>
WP_USERDUMMY=<editor_user>
WP_EMAILDUMMY=<editor_email>
```

`inception/secrets/db_password.txt`
```env
DB_USER=<db_user>
DB_PASSWORD=<db_password>
```

`inception/secrets/db_root_password.txt`
```env
DB_USER=<db_user>
DB_PASSWORD=<db_password>
WP_PASSWORDADMIN=<admin_password>
WP_PASSWORDDUMMY=<editor_password>
```

**2. Point the domain to your machine.**

```bash
echo "127.0.0.1 tpicoule.42.fr" | sudo tee -a /etc/hosts
```

**3. Start the stack.**

```bash
cd inception
make
```

Then open **https://tpicoule.42.fr** and accept the self-signed certificate.

## Makefile targets

| Target | Effect |
|---|---|
| `make` | Creates `~/data/*`, builds the images and starts the containers |
| `make stop` | Stops the containers |
| `make clean` | Stops everything, removes the Docker volumes and prunes images |
| `make fclean` | `clean`, then deletes `~/data` |

> [!WARNING]
> `make clean` runs `docker system prune -a` and removes **every** Docker volume on the machine, not only this project's.

## Structure

```
inception/
├── Makefile
├── secrets/                    (not versioned, see above)
└── srcs/
    ├── docker-compose.yml
    ├── .env                    (not versioned, see above)
    └── requirements/
        ├── nginx/      Dockerfile, conf/nginx.conf
        ├── wordpress/  Dockerfile, conf/www.conf, tools/script_w.sh
        └── mariadb/    Dockerfile, conf/maria.conf, tools/script.sh
```

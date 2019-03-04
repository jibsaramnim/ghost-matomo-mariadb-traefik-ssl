# Ghost, Matomo, MariaDB and Traefik with SSL stack for personal blogs

Please check out [my blog post for more information on this stack and how to use it](https://davejansen.com/ghost-matomo-mariadb-and-traefik-with-ssl-stack/).

I have created a basic docker stack for using [Ghost](https://ghost.org), [Matomo](https://matomo.org), [MariaDB](https://mariadb.org) and [Traefik](https://traefik.io) for easyy deployment. The stack assumes you'll want everything behind HTTPS using Traefik's built-in Let's Encrypt functionality.

I created this stack as I couldn't find one already out there that was fully ready to go. I found ones that had part of this setup, but nothing that had all four of the key features I wanted; Ghost, Matomo, SSL and one shared DB.

## What you should know

There's definitely room for improvement with this project, but as it is right now it works as I want it to, and I want to share it now in hopes that it might help others as-well. I'll probably improve this a bit over time, and definitely welcome improvements from you, too.

## In a nut-shell

This stack launches all the parts you need. Preparation-wise you only need to set the right environment variables. Please refer to `example.env`, copy it as `.env` and set it up for your own domain(s). When done, simply launch everything and you should be good to go. I highly recommend you read through the `docker-compose.yml` file to get an idea for what's happening.

Both Ghost as-well as Matomo will use one database as specified in the `DB_NAME` environment variable. While you could set up two databases, I choose to do one to make backups easier for me, and because both Ghost and Matomo support table name prefixes there will be no conflicts fortunately. If you prefer though, you can split out both into two separate databases by modifying the appropriate parts.

## First-time launch

The first time you launch the stack, Ghost will likely have trouble connecting to the DB. This is because the first time MariaDB (or MySQL) launches, it needs a bit more time to prepare itself. Therefore I recommend you launch the entire stack without the `-d` flag the first time, then stop it, and re-launch it proper. It should launch fine then.

To launch the entire stack, run the following command on your server that has Docker and docker-compose already installed, and has the domains you plan to use already pointing at it:

`docker-compose up -d`

If you made changes and need to relaunch your stack, run:

`docker-compose down && docker-compose up -d`

Generally I recommend you first run you stack without the `-d` flag so you can easily read the boot logs and spot potential issues, then when all looks good just bring down the stack and bring it right back up, this time with the `-d` flag.

## Configuring Ghost

All the DB connection settings have been prepared in the `docker-compose.yml` file, so you can simply visit your blog at the domain you have set up and its admin panel at `/ghost` to set up a user and get writing. Neato!

## Configuring Matomo

Matomo requires a bit of setting up, as its DB connection settings are stored in a php file. It is pretty straight-forward though, so just head on over to the (sub-)domain you set up for Matomo and follow its setup guide. When you get to the DB connection settings, be sure to choose `mysql` as the database type, write `db` as the database host name, and the `username`, `password`, and `dbname` should be what you have specified in the `DB_USER`, `DB_PASSWORD` and `DB_NAME` environment variables (in `.env`).

Next, I recommend you set up Matomo to forcibly use SSL. In the same `config.ini.php` file you can do this by adding `force_ssl = 1` under the `[General]` header. If this was not already set up after going through the Matomo setup, be sure to also add the domain names you're using for Matomo here by creating an array called `trusted_hosts`. For example:

```ini
[General]
force_ssl = 1
trusted_hosts[] = "matomo.mydomain.com"
; everything else
```

## Directories & configuration

Here's a brief summary of files and directories to be aware of:

### `config/php.ini`

This repository comes with a `php.ini` file with some settings already prepared to make Matomo work well. You can add your own settings here if needed, but note that this file is only used for Matomo.

### `config/acme/`

Traefik will, upon successfully fetching and setting up SSL certificates, automatically create a file called `acme.json` that contains this information. While you should probably never edit this file directly yourself, it is stored here to preserve it even if you destroy your docker images or move to another server.

### `config/matomo/`

Matomo's configuration files can be found here after you have successfully launched the stack at least once. You can place your existing `config.ini.php` file here, or modify the one generated by Matomo after following its setup process.

### `content/`

Ghost's `content` folder is linked to here, and will be where you can find your blog's actual content, images uploaded, themes, et cetera. This is the folder you'll want to backup, for example.

## Closing notes

I hope this stack might help someone else in setting up their (personal) blog with self-hosted analytics. This way you can quickly get started, have SSL out of the box, and use Matomo's great analytics features to better understand who or what visits your site, without shipping off boatloads of personal data to some other company. I would love to hear your thoughts on this stack and if you have improvements, I very much appreciate enhancements. Sharing is caring!

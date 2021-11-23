# Staging Environment

We have one staging sever currently hosted with Google Cloud Platform which is controled by ServerPilot.

Login credentials are stored in 1Password as "**DotDev Stage (GCP)**"

## Setup App (example)

* [ ] Login to ServerPilot: [https://manage.serverpilot.io/#servers](https://manage.serverpilot.io/#servers)
* [ ] Select **dotdev-site** server and **Create App**
* [ ] Name: **mothersdayclassic**
* [ ] Domain: **mothersdayclassic.dotdev.site **(use viewsite.io if project another agency)
* [ ] Runtime: **PHP 7.1**
* [ ] Server: **dotdev-site**
* [ ] System User: **serverpilot**
* [ ] Create Database: Name/User: **mothersdayclassic**
* [ ] Database Pass: **\[use generated]**
* [ ] SSL > **Enable AutoSSL** + **Redirect HTTP to HTTPS** (will need to wait for domain to authorise - 2-5 minutes)

## Setup Auto Deploy

* [ ] Login to **Buddy.works**: [https://app.buddy.works/dotdev/create-project](https://app.buddy.works/dotdev/create-project)
* [ ] Select repository: **mothers-day-classic**
* [ ] Add a new pipeline: **mothersdayclassic**
* [ ] Trigger: **on-push**
* [ ] Branch: **dev** (or select current working branch)
* [ ] Add site URL: [https://mothersdayclassic.dotdev.site](https://mothersdayclassic.dotdev.site)
* [ ] Setting: **clear cache **
* [ ] Continue next and select: **SFTP**
* [ ] Enter **host**, **username** and **password** for stage server (1password)
* [ ] Browse remote path and select: **apps/mothersdayclassic/public**
* [ ] **Test and add this action > Run pipeline **(first deployment)

## Import Database

* [ ] Login to : [http://control.dotdev.site/database/](http://control.dotdev.site/database/)
* [ ] Select database > **Import**
* [ ] **Update site config** file with staging database details.

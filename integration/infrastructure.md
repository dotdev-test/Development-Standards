# Infrastructure

Create new Server under Clients vault in 1Password - Use Minimax Integration as an example.

## AWS

Create new m4medium server on AWS

* 50gb drive
* Security group for SSH, HTTP, HTTPS, MySQL
* Generate new key as the client name, save to \_Client Shared and attach to 1Password

Setup Elastic IP and attach to instance

## ServerPilot / SSH

Login to ServerPilot under DotDev Clients > Servers > Connect Server

* \[x] I don't have a root password or public IP address.
* Hostname: \[client]-integration
* SFTP password (generate new in 1Password)
* Enable SSH password auth
* Connect to Server

Copy install script, ssh into the server and run to complete setup.

Setup new apps in ServerPilot called shopify-\[platform], db and media.

[Get mysql root password and save to 1Password](https://serverpilot.io/community/articles/how-to-access-mysql-with-the-mysql-root-user.html)

[Update timezone on the server](https://serverpilot.io/community/articles/how-to-change-the-timezone-of-your-server.html)

[Setup default php version for serverpilot](https://serverpilot.io/docs/how-to-change-the-version-of-the-php-command)

## CloudFlare

Setup temp URLs in DotDev CloudFlare account.

## FTP

[Generate new htpasswd](http://www.htaccesstools.com/htpasswd-generator/)

Upload [Adminer](https://www.adminer.org) to db app, add htpasswd and htaccess.

## BuddyWorks (deployments)

Setup auto deploy for integration through [buddy.works](https://app.buddy.works)

## Client

Request DNS records to be setup from client.

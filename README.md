# autocert-docs
Automated installation of TLS certificates for WordPress multisites

[Start reading the docs »](https://dekodeinteraktiv.github.io/autocert-docs/)

## Audience
You should have a grasp of what DevOps is and be comfortable running web servers for WordPress.

## Goal
We are running a WordPress multisite network with tens of subsites with domain mapping, and are both removing and adding sites on a regular basis. We want to have TLS certificates for those domains installed automatically. Extending the current project to support hundreds or thousands of sites should be possible without too much effort.

## Project Outline
The project consists of several sub-projects. Some of these may be used alone, with the other subprojects or with 3rd party projects. All sub-projects are maintained individually in their own repositories on GitHub.

### Current sub-projects
* [WP ACME](https://github.com/dss-web/wp-acme) – Answers ACME challenges in WordPress.
* [acme-tiny-wp](https://github.com/DekodeInteraktiv/acme-tiny-wp) – Use WordPress for ACME challenges.
* [WP Autocert](https://github.com/dss-web/wp-autocert) – Provides us with a list of domains and updates them to use https.
* [autocert-server](https://github.com/dss-web/autocert-server) – Fetches domain names from WP and installs certificates.

### Discontinued sub-projects
* [WP auth plugin for Certbot](https://github.com/dss-web/certbot-wordpress) – Use WordPress for ACME challenges with Certbot (no longer maintained).

### ACME challenge solving with WordPress
The default challenge solving (domain verification) by ACME clients is to put files in a predetermined directory from the webserver’s document root (`/.well-known/acme-challenge`). However, if you have a web node cluster or are running the certification acquirement process from a machine that is not part of your web cluster, you may run into issues.

To solve this, we have a WordPress plugin, [WP ACME](https://github.com/dss-web/wp-acme) that will answer the ACME challenges presented by Let’s Encrypt.

WP ACME must get the challenges from a compatible ACME client. We’ve created both a [customized fork of acme-tiny](https://github.com/DekodeInteraktiv/acme-tiny-wp) and an [authentication plugin for WordPress for Certbot](https://github.com/dss-web/certbot-wordpress) (the official Let’s Encrypt client). Since Certbot did not suit us that well, we don’t maintain the authentication plugin any longer, but you are free to fork it. We strongly recommend you use the acme-tiny fork.

### Automated client initialization
To get a list of active domains from WordPress we created a WordPress plugin: [WP Autocert](https://github.com/dss-web/wp-autocert).

Then the [autocert-server](https://github.com/dss-web/autocert-server) script can be run in a cronjob and will pull the active domains from WordPress match them with the existing certificate and get a new one – with optional ACME challenge solving in WordPress.

autocert-server can optionally run a script after a new certificate is acquired, which lets you reload your web server or distribute the certificate to your nodes.

## Credits
The project is maintained by [Dekode](https://en.dekode.no/) and is sponsored by the [Norwegian Government Security and Service Organisation](https://dss.dep.no/english).

## Installation

### WordPress setup
Install WordPress as a multisite network.

Install [Mercator](https://github.com/humanmade/Mercator) or WordPress MU Domain Mapping (https://wordpress.org/plugins/wordpress-mu-domain-mapping/), and add your domain aliases.

Install [WP ACME](https://github.com/dss-web/wp-acme). If you install it as an mu-plugin, you only need the file `wp-acme.php`
Define a secret key that will be shared between WordPress and the ACME client. You can either add
`define( 'WP_ACME_SECRET_KEY', 'your-secret-key' );` to your `wp-config.php` or set a site option named `wp_acme_secret_key`, e.g. by doing `update_site_option( 'wp_acme_secret_key', 'your-secret-key' );`
There is no user interface or anything else to configure for this plugin.

Install [WP Autocert](https://github.com/dss-web/wp-autocert), go to Settings > WP Autocrat (`/wp-admin/network/settings.php?page=wp-autocert`), select the domains you want to get certificates for, and note the API Key. It will be used as a shared secret when communicating with the autocert-server script.

### Director node setup
On your server node that will function as your director node (it can even be your local machine), download [acme-tiny-wp](https://github.com/DekodeInteraktiv/acme-tiny-wp) and [autocert-server](https://github.com/dss-web/autocert-server).

Create a Let’s Encrypt account key (put it wherever it suites you, but make it readable by root only):
```
$ sudo mkdir -p /etc/ssl/acme
$ sudo chmod 0700 /etc/ssl/acme
$ sudo openssl genrsa 4096 > /etc/ssl/acme/accountkey.pem
$ sudo chmod 0400 /etc/ssl/acme/accountkey.pem  
$ sudo chown -R root:root /etc/ssl/acme
```

Copy the autocert-server example configuration and modify it with your info. Remember to make it readable by root only.
```
$ sudo cp example.ini my-config.ini  
$ sudo chmod 0600 my-config.php  
$ sudo chown root:root my-config.php
```

#### Explanation of autocert-server configuration directives

`[general]`

Directive | Use
--------- | ---
`post_hook` | A command to run whenever your certificates have changed. E.g. reload your webserver.

`[wordpress]`

Directive | Use
--------- | ---
`url` | The URL to your WordPress installation where WP Autocert is installed.
`shared_secret` | The shared secret from WP Autocert.

`[certificates]`

Directive | Use
--------- | ---
`ca_client` | The autocert-server CA client to use. For now, only `acme-tiny-wp` is supported.

`[acme-tiny-wp]`

Directive | Use
--------- | ---
`acme_tiny_wp_path` | The full path to the `acme_tiny_wp.py` script.
`openssl_conf` | The full path to your OpenSSL configurations file. Usually `/etc/ssl/openssl.cnf`
`wp_url` | The URL to your WordPress installation where WP ACME is installed.
`wp_secret` | The shared secret with WP ACME.
`server` | The ACME server to use. Use `https://acme-staging.api.letsencrypt.org/directory` for staging/testing and `https://acme-v01.api.letsencrypt.org/directory` for production.
`cert_key` | The full path to where the private certificate key is stored. If it doesn’t exists, it will be created for you.
`cert_file` | The full path to where the certificate will be stored.
`cert_csr` | The full path to where the CSR will be stored.  
`acme_account_key` | The full path to where the ACME account key is stored. If it doesn’t exists, it will be created for you.

## Usage

Run the autocert-server script:
`$ sudo ./autocert.php -c my-config.ini`

To have the process fully automated, put the script in a crontab. Run it every minute if you like.

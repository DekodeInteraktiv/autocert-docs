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
* [WP auth plugin for Certbot] (no longer maintained) – Use WordPress for ACME challenges with Certbot.

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
